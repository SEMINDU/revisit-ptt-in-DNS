# Software Verification & Testing

## ✅ Overview
This directory contains the automated test suite for this research project. Following **FAIR4RS** guidelines, these tests ensure the code is **reliable**, **reproducible**, and **robust** against changes in dependencies or data.

---

## 🛠️ Testing Framework
This project uses the [**testthat**](https://testthat.r-lib.org) R package. 
*   **Unit Tests:** Verify individual functions in the `/R` directory.
*   **Integration Tests:** Verify that the data pipeline from `raw` to `processed` works as expected.

---

## 🚀 How to Run Tests
To verify the integrity of the analysis, run the following command in the R console:

```r
# Run all tests in the project
testthat::test_local()
