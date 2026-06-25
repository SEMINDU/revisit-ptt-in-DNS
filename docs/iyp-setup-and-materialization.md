# IYP Setup and Relationship Materialization

This document records the local Internet Yellow Pages (IYP) setup and the graph materialization steps required to reproduce this study.

These steps are part of the required reproduction workflow. Perform them **immediately after** the local IYP instance has been installed, configured, loaded with the required database dump, and verified as reachable through Neo4j. Do this before running any notebooks in `code/`.

For the base IYP installation, first follow the official tutorial:

- [IYP tutorial](https://tutorial.iyp.ihr.live/)
- [Hosting a local IYP instance](https://tutorial.iyp.ihr.live/content/local-instance.html)

## Local Workstation

The local instance was installed on an HP ZBook X G1i 16 workstation with 1 TB of storage and 32 GB of memory, running Ubuntu 24.04.4 LTS.

## Neo4j Memory Configuration

The Neo4j memory configuration was tuned to support long-running graph traversal queries. The IYP container was allowed to use up to 16 GB of memory. Within Neo4j, 5 GB was allocated to the JVM heap and 7 GB to the page cache.

The heap memory supports query execution. The page cache stores frequently accessed graph data and indexes in memory. These settings are important for the dependency traversals in this study because traversal queries can keep substantial intermediate state while exploring DNS delegation paths. With smaller heap and page-cache values, out-of-memory errors are common.

The JVM option `-XX:+ExitOnOutOfMemoryError` was also enabled so that Neo4j exits cleanly if an out-of-memory error occurs.

## APOC Plugin

The Awesome Procedures on Cypher (APOC) plugin must be enabled before running the study notebooks. The dependency modeling uses `apoc.path.expandConfig` to reconstruct DNS dependency paths across multiple entity and relationship types.

Standard Cypher variable-length patterns can express simple recursive paths, but they provide limited control over traversal direction, allowed node labels, uniqueness, and stopping conditions. `apoc.path.expandConfig` provides the controls needed in this analysis, including relationship filters, label restrictions, breadth-first expansion, and path uniqueness.

Example Neo4j/IYP container configuration:

```bash
NEO4J_PLUGINS=["apoc"]
NEO4J_dbms_security_procedures_unrestricted=apoc.*
NEO4J_dbms_security_procedures_allowlist=apoc.*
```

## Materialized Relationships and Label

After the base IYP local instance is running, the graph used for this study was extended with five source-specific materialized relationships and one additional label.

Materialized relationships:

- `MANAGED_BY_SOURCE_OPENINTEL`
- `PART_OF_SOURCE_OPENINTEL`
- `PART_OF_SOURCE_IYP`
- `RESOLVES_TO_SOURCE_OPENINTEL`
- `ORIGINATE_SOURCE_BGPKIT`

Additional label:

- `HAS_GLUE_SOURCE_OPENINTEL`

The definitions, construction procedure, and scientific motivation for these additions are described in the associated manuscript:

### `MANAGED_BY_SOURCE_OPENINTEL`

`MANAGED_BY_SOURCE_OPENINTEL` is derived from the original `MANAGED_BY` relationship. It maps a `DomainName` node to its corresponding `AuthoritativeNameServer` nodes, using only data from OpenINTEL DNSGraph.

### `PART_OF_SOURCE_OPENINTEL`

`PART_OF_SOURCE_OPENINTEL` is derived from the original `PART_OF` relationship. It maps an `AuthoritativeNameServer` node to its parent `DomainName` node, using only data from OpenINTEL DNSGraph.

### `PART_OF_SOURCE_IYP`

`PART_OF_SOURCE_IYP` is derived from the original `PART_OF` relationship. It maps an `IP` node to the `BGPPrefix` node that contains it, using IYP ip2prefix data.

### `RESOLVES_TO_SOURCE_OPENINTEL`

`RESOLVES_TO_SOURCE_OPENINTEL` is derived from the original `RESOLVES_TO` relationship. It maps an `AuthoritativeNameServer` node to its corresponding `IP` nodes, using only data from OpenINTEL DNSGraph.

### `ORIGINATE_SOURCE_BGPKIT`

`ORIGINATE_SOURCE_BGPKIT` is derived from the original `ORIGINATE` relationship. It maps a `BGPPrefix` node to the `AS` node that originates it, using BGPKIT pfx2asn data.

### `HAS_GLUE_SOURCE_OPENINTEL`

`HAS_GLUE_SOURCE_OPENINTEL` is added to authoritative nameservers that have glue records in OpenINTEL DNSGraph data. These nameservers act as termination points in the exploration of a domain's DNS delegation graph.

## Example Materialization Queries

The example below materializes the OpenINTEL-specific `RESOLVES_TO` relationship. The same pattern can be adapted for the other source-specific relationships by changing the source relationship type, reference filter, node labels, and target relationship type.

```cypher
:auto
MATCH (a:AuthoritativeNameServer)-[r:RESOLVES_TO]->(b:IP)
WHERE r.reference_name = "openintel.dnsgraph_nl"

CALL (a, b, r) {
  MERGE (a)-[nRel:RESOLVES_TO_SOURCE_OPENINTEL]->(b)
  SET nRel += properties(r)
} IN TRANSACTIONS OF 1000 ROWS
```

The example below adds the `HAS_GLUE_SOURCE_OPENINTEL` label to authoritative nameservers with OpenINTEL glue records.

```cypher
:auto
MATCH (ns:AuthoritativeNameServer)-[r:RESOLVES_TO]->(ip)
WHERE r.reference_name = "openintel.dnsgraph_nl"
  AND r.glue = true

CALL (ns) {
  SET ns:HAS_GLUE_SOURCE_OPENINTEL
} IN TRANSACTIONS OF 1000 ROWS;
```

Run these materialization steps before executing the notebooks in `code/`, because the notebook traversal queries expect the materialized relationship types and glue label to exist in the local IYP graph.

## Example Traversal Queries

After the local IYP instance has been installed, APOC is enabled, and the study-specific relationships and label have been materialized, the following queries can be run in the Neo4j Browser or Cypher shell to verify that the graph supports the dependency traversals used in the notebooks.

### Domain Dependency Paths

This query reconstructs DNS delegation dependency paths for `power-electronics.com`. It starts from the domain node, follows OpenINTEL-derived management and parent-domain relationships, and stops at authoritative nameservers marked with `HAS_GLUE_SOURCE_OPENINTEL`.

```cypher
// TCB_nodes
MATCH (start:DomainName {name: "power-electronics.com"})

CALL (start) {
  CALL apoc.path.expandConfig(start, {
    relationshipFilter: "MANAGED_BY_SOURCE_OPENINTEL>|PART_OF_SOURCE_OPENINTEL>",
    minLevel: 1,
    labelFilter: "+DomainName|>HostName|>AuthoritativeNameServer|/HAS_GLUE_SOURCE_OPENINTEL",
    uniqueness: "NODE_PATH",
    bfs: true
  })
  YIELD path

  WITH path, nodes(path)[-1] AS leafnode
  WHERE leafnode:HAS_GLUE_SOURCE_OPENINTEL
  RETURN collect(DISTINCT path) AS valid_paths
}

RETURN valid_paths
```

### Nameserver to Prefix and AS Dependency Paths

This query maps a list of authoritative nameservers to IP addresses, BGP prefixes, and originating AS nodes. It uses the materialized OpenINTEL, IYP, and BGPKIT relationships.

```cypher
WITH [
  "ns2.serviciodns.es",
  "ns1.serviciodns.es",
  "ns3.serviciodns.es",
  "ns-208-c.gandi.net",
  "ns-236-a.gandi.net",
  "ns-212-b.gandi.net",
  "dns0.gandi.net",
  "dns6.gandi.net",
  "dns4.gandi.net",
  "e.gandi-ns.fr",
  "dns1.gandi.net",
  "dns3.gandi.net",
  "dns2.gandi.net"
] AS ns_list

UNWIND ns_list AS ns_name
OPTIONAL MATCH (ns:AuthoritativeNameServer {name: ns_name})

CALL (ns) {
  CALL apoc.path.expandConfig(ns, {
    relationshipFilter: "RESOLVES_TO_SOURCE_OPENINTEL>|PART_OF_SOURCE_IYP>|<ORIGINATE_SOURCE_BGPKIT",
    minLevel: 1,
    labelFilter: "+AuthoritativeNameServer|+BGPPrefix|+AS|+IP",
    uniqueness: "NODE_PATH",
    bfs: true
  })
  YIELD path

  RETURN collect(path) AS ns_prefix_AS_path
}

RETURN ns_prefix_AS_path
```

If these queries return paths, the local graph has the key materialized relationships and APOC traversal support required by the analysis notebooks.
