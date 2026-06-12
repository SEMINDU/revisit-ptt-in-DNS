# Tests

This folder is reserved for validation checks for the Tranco DNS dependency workflow.

There is currently no formal automated test suite. Useful future checks would include:

- verifying that required parquet files exist before each notebook stage
- checking expected columns in generated dependency tables
- validating that TCB counts are non-negative and internally consistent
- confirming that figure-generation notebooks can read their required inputs
- testing helper functions in `code/helpers/census_helper.py`

Until automated tests are added, reproducibility should be checked by running the notebooks in the order described in [`../code/README.md`](../code/README.md) and confirming that the expected files in [`../data/README.md`](../data/README.md) and [`../outputs/README.md`](../outputs/README.md) are produced.
