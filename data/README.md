# Data Description

## Overview

This directory contains the input, intermediate, and generated data used by the Tranco DNS dependency analysis workflow. Most files are created by running the notebooks in `code/`, and large generated parquet files are expected to be reproduced locally rather than committed.

For a full description of rendered reports and final presentation outputs, see `outputs/reports/README.md`.

## Directory Structure

### `data/names/`

Reference and derived name metadata used during preprocessing and analysis.

Typical files include:

- `tranco_categorized_names.parquet`  
  Categorized Tranco domains with rank, TLD, TLD type, and parent-status metadata.

- `ccTLD_registration_date_may_10_2026.parquet`  
  Country-code TLD registration dates used to identify older ccTLDs for historical comparison.

### `data/result/`

Main result directory for generated parquet data. The current Tranco workflow uses the subfolders below.

### `data/result/dns_dependency/`

Dependency outputs generated from the Tranco notebooks.

Typical files include:

- `ccTLD_domains_dependency_no_parent.parquet`
- `gTLD_domains_dependency_no_parent.parquet`
- `gen_resTLD_domains_dependency_no_parent.parquet`
- `sponTLD_domains_dependency_no_parent.parquet`
- `infrTLD_domains_dependency_no_parent.parquet`
- `special_use_domains_dependency_no_parent.parquet`
- `domain_with_parent_rel.parquet`
- `unresolved_domains_by_type.parquet`
- `IP_Prefix_AS_dependency_sample.parquet`
- `IP_Prefix_AS_dependency/chunk_*.parquet`

The `IP_Prefix_AS_dependency/` directory contains chunked outputs from the long-running IP, prefix, and AS dependency mapping notebook.

### `data/result/NS_infrastructure/`

Nameserver infrastructure summaries used by the final analysis notebook.

Typical file:

- `tranco_NS_Infrastructure.parquet`

This file identifies common nameserver service-provider infrastructure and is used to compare domain dependency on the top nameserver providers.

## Reproducibility Notes

- Run the notebooks in the order documented in `code/README.md`.
- Use the same IYP/Neo4j dump described in `code/README.md` for reproducible dependency paths.
- Generated parquet files can be large and may need to be regenerated locally.
- If an analysis notebook reports a missing parquet file, rerun the notebook that produces that file before continuing.
