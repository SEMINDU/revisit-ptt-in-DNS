# Tranco DNS Dependency Analysis Code

This folder contains the notebooks used to model and analyse DNS dependency for the Tranco domain list. The workflow reconstructs nameserver dependency paths from the IYP/Neo4j graph, summarizes the transitive closure base (TCB) for each domain, identifies dominant nameserver infrastructure, maps nameserver dependencies to IP prefixes and autonomous systems, and then runs the final analysis.

Run the notebooks in the order documented below. Later notebooks depend on parquet outputs created by earlier notebooks.

## Reproducibility Requirements

Before running the notebooks, make sure the Python environment and Neo4j database are ready.

Required Python libraries include:

- `pandas`
- `numpy`
- `matplotlib`
- `IPython`
- `requests`
- `beautifulsoup4`
- `neo4j`
- `pyarrow` or another parquet engine supported by pandas

The notebooks assume a local Neo4j database is running at:

```text
neo4j://localhost:7687
```

with credentials:

```text
username: neo4j
password: password
```

For reproducibility, use the same IYP/Neo4j instance dump used for the experiment: February 15, 2026. The dependency queries rely on labels and relationships from that graph, including OpenINTEL, IYP, and BGPKIT-derived relationships. The Neo4j APOC procedures must also be available because the notebooks use `apoc.path.expandConfig`.

## Expected Directory Structure

The notebooks read and write files relative to the repository root:

```text
code/
data/
  names/
  result/
    dns_dependency/
    NS_infrastructure/
    IP_Prefix_AS_dependency/
outputs/
  figures/
    tranco_list/
```

Most generated data is stored as parquet files. Large intermediate result files are not expected to be committed to the repository and should be regenerated locally.

The notebooks use `Path.cwd().parent` to locate `data/` and `outputs/`. For the paths to resolve correctly, start the notebooks with `code/` as the working directory, or update the path cells before running them.

## Execution Order

### 1. `ptt-iyp-modeling_dependency_tranco_list.ipynb`

Run this notebook first.

This notebook builds the core dependency dataset for the Tranco domains. It connects to Neo4j, checks the Tranco domain nodes, classifies domains by TLD type, identifies parent-domain status, and creates the domain groups used by the rest of the workflow.

The notebook fetches TLD metadata from IANA and the public suffix list, categorizes domains, and saves the categorized domain table. It then runs Cypher dependency queries over the Neo4j graph to find authoritative nameserver dependency paths for each domain group.

Main outputs:

- `data/names/tranco_categorized_names.parquet`
- `data/result/tranco_list/ccTLD_domains_dependency_no_parent.parquet`
- `data/result/tranco_list/gTLD_domains_dependency_no_parent.parquet`
- `data/result/tranco_list/gen_resTLD_domains_dependency_no_parent.parquet`
- `data/result/tranco_list/sponTLD_domains_dependency_no_parent.parquet`
- `data/result/tranco_list/infrTLD_domains_dependency_no_parent.parquet`
- `data/result/tranco_list/special_use_domains_dependency_no_parent.parquet`
- `data/result/tranco_list/domain_with_parent_rel.parquet`

This step can take a long time because it queries the graph for many domains. Do not run the infrastructure or analysis notebooks until these outputs exist.

### 2. `ppt-iyp-analysis (tranco)_infrastructure_NS_service_provider.ipynb`

Run this notebook after the modeling dependency notebook has completed.

This notebook studies nameserver service-provider infrastructure in the resolved dependency data. It loads the dependency parquet files created in Step 1, combines the domain groups, counts nameserver frequency, groups nameservers into provider-style labels, normalizes AWS DNS infrastructure naming, and identifies the most common nameserver infrastructure providers.

The result from this notebook is later used by the final analysis notebook to measure how much domains depend on the top 5 and top 10 nameserver infrastructure providers.

Main output:

- `data/result/tranco_list/tranco_NS_Infrastructure.parquet`

Note: the final analysis notebook reads the nameserver infrastructure file from `data/result/NS_infrastructure/tranco_NS_infrastructure.parquet`, while this notebook writes `tranco_NS_Infrastructure.parquet`. Before running the final analysis, make sure the generated file is available at the path and filename used by the analysis notebook.

### 3. `ptt-iyp-analysis_(tranco)_infrastructure_ip_prefix_AS.ipynb`

Run this notebook after the modeling dependency notebook has completed.

This notebook maps nameserver dependencies to lower-level infrastructure: IP addresses, BGP prefixes, and autonomous systems. It loads the domain-to-nameserver dependency outputs from Step 1, connects to Neo4j, and runs a Cypher query that expands from authoritative nameservers through IP, prefix, and AS relationships.

This code is expensive to run. The full run can take more than 1200 minutes, so it writes chunked parquet outputs to make the result easier to store and resume.

Main outputs:

- `data/result/tranco_list/IP_Prefix_AS_dependency_sample.parquet`
- `data/result/tranco_list/IP_Prefix_AS_dependency/chunk_*.parquet`

The sample output is useful for checking that the query behaves correctly before running the full dataset.

### 4. `ppt-iyp-analysis_(tranco).ipynb`

Run this notebook at the end.

This is the main analysis notebook. It loads the dependency results from Step 1, the nameserver infrastructure results from Step 2, and supporting TLD metadata. It combines the resolved domain dependency data, checks missing and unresolved domains, saves a reusable resolved summary table, and produces the statistical and visual analysis.

The analysis includes:

- TCB distribution for resolved Tranco domains
- TCB comparison between ccTLD and gTLD-style groups
- old-TLD analysis based on TLDs available around the original 2004 study period
- average TCB by TLD group
- group-size effects on average TCB
- shrinkage-adjusted average TCB for small TLD groups
- relationship between Tranco rank and TCB
- top-ranked versus least-ranked domain comparisons
- dependency on top 5 and top 10 nameserver infrastructure providers
- examples of domains with and without parent-domain dependency effects

Main outputs:

- `data/result/tranco_list/unresolved_domains_by_type.parquet`
- `data/names/ccTLD_registration_date_may_10_2026.parquet`
- figures under `outputs/figures/tranco_list/`

For more detailed information about generated outputs and reports, refer to `outputs/reports/README.md`.

## Notebook Summary

| Notebook | Purpose | Run order |
| --- | --- | --- |
| `ptt-iyp-modeling_dependency_tranco_list.ipynb` | Builds categorized Tranco domain lists and queries Neo4j for domain-to-nameserver dependency paths. | 1 |
| `ppt-iyp-analysis (tranco)_infrastructure_NS_service_provider.ipynb` | Counts and groups nameserver infrastructure providers from resolved dependency data. | 2 |
| `ptt-iyp-analysis_(tranco)_infrastructure_ip_prefix_AS.ipynb` | Maps nameserver dependencies to IP addresses, BGP prefixes, and ASNs. | 3 |
| `ppt-iyp-analysis (tranco).ipynb` | Runs the final statistical analysis and generates figures. | 4 |

## Re-running the Workflow

To reproduce the results from scratch:

1. Install the required Python libraries.
2. Install Neo4j and APOC.
3. Load the IYP/Neo4j dump from February 15, 2026.
4. Start Neo4j and confirm the notebook credentials match the local database.
5. Run `ptt-iyp-modeling_dependency_tranco_list.ipynb` completely.
6. Run `ppt-iyp-analysis (tranco)_infrastructure_NS_service_provider.ipynb`.
7. Run `ptt-iyp-analysis_(tranco)_infrastructure_ip_prefix_AS.ipynb` if IP, prefix, and AS dependency outputs are needed.
8. Run `ppt-iyp-analysis (tranco).ipynb` last.

If a notebook fails because an input parquet file is missing, go back to the notebook that generates that file and complete that step first.

## Notes

- The notebooks are currently research notebooks rather than standalone Python scripts.
- Several outputs are large and are expected to be generated locally.
- The Neo4j graph version matters for reproducibility because dependency paths can change when the underlying IYP/OpenINTEL/BGPKIT data changes.
- The IP/prefix/AS mapping notebook is the longest-running step and should be planned as a long batch run.
