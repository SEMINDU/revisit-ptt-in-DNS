# Research Software & Analysis Code

## Software Architecture
This repository contains the code used to reconstruct and analyse DNS dependency chains for domains in the Cisco Umbrella Top 1 Million dataset.

The current implementation is notebook-based and split into two main stages:

- `ppt-iyp-cisco-top1m-fetch.ipynb`: data acquisition, dependency reconstruction, and summary-file generation.
- `ppt-iyp-cisco-top1m-analy.ipynb`: loading processed outputs, statistical summarisation, and figure generation.

The workflow relies on intermediate CSV files stored under `data/`, which are then reused for downstream analysis and visualisation.

## Directory Map
Suggested repository structure:

- `code/` or `notebooks/`: source notebooks and, later, extracted reusable functions/scripts.
- `data/names/`: domain and TLD classification files such as `categorized_names.csv`.
- `data/result/tmp/`: temporary chunked outputs generated during large-scale dependency fetching.
- `data/result/without_allias/`: processed summary tables used for analysis.
- `data/result/without_allias/figures/`: generated plots and figures.
- `outputs/`: optional directory for manuscript-ready tables and figures.
- `docs/`: project documentation and repository metadata.

## Execution Workflow
To reproduce the analysis from scratch, run the workflow in the following order and do not proceed to the next step until the expected outputs from the current step have been fully generated.

1. `ppt-iyp-cisco-top1m-fetch.ipynb`  
   This notebook performs data acquisition and DNS dependency reconstruction for the Cisco Umbrella Top 1 Million dataset. Because this stage can generate a large volume of intermediate results, partial outputs are written to `data/result/tmp/` during execution. These temporary files are not included in the repository and must be produced locally before any downstream analysis can be run.

   Expected outputs from this step include:
   - chunked intermediate files in `data/result/tmp/`
   - processed summary tables in `data/result/`
   - unresolved or exception files, where applicable

   This step must complete successfully before continuing.

2. `ppt-iyp-cisco-top1m-analy.ipynb`  
   This notebook reads the processed outputs created by the fetch stage. It assumes that all required intermediate and final files from Step 1 already exist. It then performs statistical summarisation, comparative analysis, and figure generation.

   Expected outputs from this step include:
   - analytical summary tables
   - plots and figures saved under `data/result/without_allias/figures/`

## Reproducibility Note
The repository does not store large intermediate files generated during the fetch stage. Users must run the full data-construction workflow locally to create these files before executing the analysis notebook. The analysis step is therefore not standalone and depends on the successful completion of the preceding data-generation step.

- run the notebooks in the documented order,
- preserve the expected `data/` directory structure,
- ensure the Neo4j database is available before starting the fetch notebook,
- record the Python version and package versions in an environment file such as `requirements.txt` or `environment.yml`.

## Coding and Open Science Practices
This repository is being developed with reusability and transparency in mind.

Planned or recommended improvements include:

- refactoring notebook functions into standalone Python modules,
- separating pure functions from execution code,
- adding docstrings to reusable functions,
- including automated tests for critical processing steps,
- documenting all input datasets, intermediate files, and outputs.

## Metadata and Citation
Repository-level metadata should be provided through files such as:

- `CITATION.cff`
- `codemeta.json`
- `LICENSE`
- `README.md`

If archived on Zenodo or a similar service, the repository DOI should be added to `CITATION.cff`.

## Maintainer
Maintained by: [Your Name / ORCID]

This repository is being prepared in line with FAIR and research software good-practice principles.