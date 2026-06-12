# Code

This folder contains the notebooks and helper module used to reproduce the Tranco DNS dependency analysis. The notebooks query an IYP/Neo4j graph, generate intermediate parquet files in `data/`, and produce figures in `outputs/figures/`.

Do not run the final analysis first. It depends on data products created by the modeling and infrastructure notebooks.

## Environment

Required Python packages include:

- `pandas`
- `numpy`
- `matplotlib`
- `IPython`
- `requests`
- `beautifulsoup4`
- `neo4j`
- `pyarrow`
- `pytricia`

The dependency notebooks also require:

- a local Neo4j server at `neo4j://localhost:7687`
- the IYP/Neo4j graph used for the study, preferably the February 15, 2026 snapshot
- Neo4j APOC procedures, especially `apoc.path.expandConfig`
- notebook credentials matching the local database, or the credential cells updated before running

The notebooks use `Path.cwd().parent` for paths. Start Jupyter with `code/` as the working directory, or update the path cells.

## Notebooks

### 1. `ptt-iyp-modeling_dependency_tranco_list.ipynb`

Builds the core Tranco DNS dependency dataset. It loads Tranco domains from the IYP graph, fetches TLD and public suffix metadata, classifies domains by TLD group and parent-domain status, and queries Neo4j for authoritative nameserver dependency paths.

Main outputs:

- `data/names/tranco_categorized_names.parquet`
- `data/result/dns_dependency/ccTLD_domains_dependency_no_parent.parquet`
- `data/result/dns_dependency/gTLD_domains_dependency_no_parent.parquet`
- `data/result/dns_dependency/gen_resTLD_domains_dependency_no_parent.parquet`
- `data/result/dns_dependency/sponTLD_domains_dependency_no_parent.parquet`
- `data/result/dns_dependency/infrTLD_domains_dependency_no_parent.parquet`
- `data/result/dns_dependency/special_use_domains_dependency_no_parent.parquet`
- `data/result/dns_dependency/domain_with_parent_rel.parquet`

### 2. `ptt-iyp-analysis (tranco)_infrastructure_NS_service_provider.ipynb`

Summarizes nameserver infrastructure providers from the dependency outputs. It loads domain-to-nameserver data, counts frequent nameserver infrastructure, groups related nameserver labels, and prepares the provider table used by the final analysis.

Main output:

- default notebook write path: `data/result/dns_dependency/tranco_NS_Infrastructure.parquet`


The final analysis notebook reads the file from `data/result/NS_infrastructure/tranco_NS_Infrastructure.parquet`. After running this notebook, make sure the generated file is available at that path, or update the final analysis path cell locally.

### 3. `ptt-iyp-analysis_(tranco)_infrastructure_ip_prefix_AS.ipynb`

Maps nameserver dependencies to IP addresses, BGP prefixes, and autonomous systems. It reads the DNS dependency parquet files and queries Neo4j from authoritative nameservers through IP, prefix, and AS relationships.

Main outputs:

- `data/result/dns_dependency/IP_Prefix_AS_dependency_sample.parquet`
- `data/result/IP_Prefix_AS_dependency/chunk_*.parquet`

This is a long-running notebook. It writes chunked parquet files so the result can be stored and reused without repeating the full graph query.

### 4. `ptt-iyp-analysis_(tranco)_infrastructure_IP_anycast.ipynb`

Classifies dependent IP addresses by anycast coverage. It reads the IP/prefix/AS dependency data, downloads or reuses Anycast Census/LACeS snapshots, builds IPv4 and IPv6 prefix tries with `pytricia`, and saves a per-domain anycast summary.

Main outputs:

- `data/anycast_census/laces_ipv4_*.parquet`
- `data/anycast_census/laces_ipv6_*.parquet`
- `data/anycast_census/laces_ipv4_anycast_high_*.parquet`
- `data/anycast_census/laces_ipv6_anycast_high_*.parquet`
- `data/result/anycast_results/domain_anycast_summary.parquet`

### 5. `ptt-iyp-analysis_(tranco).ipynb`

Runs the final scientific analysis and figure generation. It combines dependency outputs, TLD metadata, nameserver infrastructure providers, IP/prefix/AS dependency data, and anycast summaries.

The analysis covers:

- resolved and unresolved Tranco domains by TLD group
- TCB distributions for all resolved domains
- ccTLD versus gTLD-style comparisons
- old-TLD comparisons using TLDs available around the original transitive-trust study period
- average and shrinkage-adjusted TCB by TLD
- relationship between Tranco rank and TCB
- top-ranked versus lower-ranked domain comparisons
- dependency on top nameserver infrastructure providers
- IP, prefix, AS, and anycast dependency summaries

Main outputs:

- `data/names/tranco_unresolved_domains_by_type.parquet`
- `data/names/ccTLD_registration_date_may_10_2026.parquet`
- `data/result/temp/resolved_summary.parquet`
- figures under `outputs/figures/tranco_list/`

## Helper Module

`helpers/census_helper.py` downloads and filters Anycast Census/LACeS snapshots. It can be imported from notebooks or run as a command-line script to store either full filtered census tables or prefix-only CSV files.

## Suggested Run Order

| Order | File | Role |
| --- | --- | --- |
| 1 | `ptt-iyp-modeling_dependency_tranco_list.ipynb` | Generate domain classification and DNS dependency data. |
| 2 | `ptt-iyp-analysis (tranco)_infrastructure_NS_service_provider.ipynb` | Generate nameserver infrastructure provider data. |
| 3 | `ptt-iyp-analysis_(tranco)_infrastructure_ip_prefix_AS.ipynb` | Generate IP, prefix, and AS dependency data. |
| 4 | `ptt-iyp-analysis_(tranco)_infrastructure_IP_anycast.ipynb` | Generate anycast coverage summaries. |
| 5 | `ptt-iyp-analysis_(tranco).ipynb` | Run the final analysis and create figures. |
