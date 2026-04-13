# Research Reports & Analysis Narratives

## 📑 Overview
This directory contains the final rendered outputs of the analysis pipeline. Following **FAIR4RS** and **Open Science** guidelines, these reports combine prose, code, and results to provide a complete "audit trail" of the scientific process.

---

## ⚡ Reproducibility Rule
Every document in this folder is **computationally reproducible**. 
*   **Source:** Scripts in `/code` or `.qmd`/`.Rmd` files in the root or `/docs`.
*   **Constraint:** These reports must **never** be manually edited after rendering. Any corrections to the text or data must be made in the source files to preserve the "Provenance" of the research findings.

---

## 📂 Report Inventory

| Report Name | Format | Description |
| :--- | :--- | :--- |
| `01_exploratory_analysis.html` | HTML | Initial data distribution and quality checks. |
| `02_statistical_results.pdf` | PDF | Final hypothesis testing and model summaries. |
| `03_summary_for_stakeholders.docx` | Word | Non-technical summary for policy/industry partners. |

---

## 🛠️ Rendering Instructions
To regenerate these reports using **Quarto** (the modern standard for R/Python research), run the following from the project root:

```bash
# Render a specific report
quarto render code/analysis.qmd --output-dir outputs/reports/

# Render all project documentation
quarto render
