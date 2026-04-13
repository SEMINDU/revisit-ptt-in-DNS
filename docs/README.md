# Project Documentation

## 📖 Overview
This directory contains the supplementary documentation, rendered reports, and guides required to understand, run, and cite this research project. Following **FAIR principles**, this ensures the research is not just available, but **understandable**.

---

## 📂 Documentation Structure

Example of posible directories and files that should be available in the docs folder

| Component | Format | Description |
| :--- | :--- | :--- |
| **User Guide** | `.qmd` / `.md` | Step-by-step instructions for reproducing the analysis. |
| **API/Function Docs** | `.html` | Auto-generated documentation for functions in the `/R` folder. |
| **(Meta)Data Dictionary** | `.csv` / `.md` | Detailed definitions of all variables used in the analysis. |
| **Manuscript/Report** | `.pdf` / `.docx` | The final research paper or technical report. |
| **SMP DMP** | `.pdf` | The Software Data Management Plans (e.g., following NWO/ERC templates). |

---

## ⚖️ Agreements and concents

The table below is an example of how you should list whether agreements, consents, or similar files exist for the data and code in the repository.

| File | Exist | Reference | Description |
| :--- | :--- | :--- | :--- |
| Informed consent | `yes`/`no` | Repository with DOI | Participants-partners allowed sahring data/code |
| Ethical approval | `yes`/`no` | Shared upon request | Data/code reviewd |
| Legal agreements | `yes`/`no` | Private (UT-archived)  | Why is possible publishing |

*Note: Make sure that private and sensitive documentation files are included in your .gitignore file.*

## Research Data and Software Authorship and Contribution

It is recommended to use a structured taxonomy to recognize all forms of contribution during the generation of data and code used for research purposes. Authorship is granted to contributors who provide substantial intellectual input to create the data and/or code. Follow the [RSAC Task Force recommendations](https://www.researchsoft.org/tf-authorship-contribution/).

| Role | Description | Examples | Authorship Eligibility |
| :--- | :--- | :--- | :--- |
| **Conceptualization** | Framing the research goals and code/software requirements. | Defining core algorithms or user stories. | Yes |
| **Development** | Writing, testing, and debugging the primary source code. | Implementing features, UI/UX, or backend logic. | Yes |
| **Maintenance** | Long-term stewardship and issue resolution. | Code sustainability, Patching bugs, managing dependencies, and releases. | Yes |
| **Data Curation** | Processing and cleaning data specifically for software use. | Creating training sets or validation benchmarks. | Case-by-case |
| **Documentation** | Creating technical and user-facing manuals. | Writing API docs, tutorials, or the Wiki. | No |
| **Validation** | Verifying the software against research outputs. | Running stress tests or verifying mathematical proofs. | No |
| **Infrastructure** | Providing the underlying compute or CI/CD pipelines. | Setting up GitHub Actions or server environments. | No |


## 🚀 Viewing the Documentation
If this repository is configured with **GitHub Pages**, the rendered version of this documentation is available at:
👉 `https://<username>.github.io/<repo-name>/`

### To render locally (Quarto):
Ensure you have [Quarto](https://quarto.org) installed, then run:

```bash
quarto render docs/
```