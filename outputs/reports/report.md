# High-Level Study Report

This short report summarizes the purpose and reproducibility structure of the Tranco DNS dependency study. The full scientific interpretation should be taken from the associated paper or manuscript.

## Study Goal

The study revisits transitive trust in DNS by measuring the dependency footprint of Tranco-ranked domains. It reconstructs authoritative nameserver dependency paths from the IYP/Neo4j graph and uses these paths to compute the transitive closure base (TCB) for domains and TLD groups.

## Analysis Scope

The workflow examines:

- TCB distributions for resolved Tranco domains
- differences between ccTLD and gTLD-style groups
- old-TLD comparisons inspired by earlier DNS transitive-trust work
- nameserver infrastructure provider concentration
- IP, BGP prefix, AS, and anycast infrastructure dependencies
- possible relationships between Tranco rank and dependency size

## Reproducibility Trail

The executable workflow is in `code/`. Generated parquet data belongs in `data/`. Figures are written to `outputs/figures/`, mainly under `outputs/figures/tranco_list/`.

For exact reproduction, run the notebooks in the order described in `code/README.md` and use the same IYP/Neo4j graph snapshot described there.
