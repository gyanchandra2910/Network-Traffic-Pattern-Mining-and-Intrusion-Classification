# Network Traffic Pattern Mining and Intrusion Classification

UNSW-NB15 course project for **Analytics and Systems of Big Data**, IIITDM Kancheepuram.

**Team:** Gyan Chandra (CS23I1053) and ALURI VENKAT VAIBHAV (CS23I1040).

**Faculty:** B. Sivaselvan | **Deadline:** 26 October 2026.

The project covers data cleaning, traffic analysis, five frequent-pattern miners,
decision trees, mixed Naive Bayes and Random Forest.

- [Executed notebook](Network_Traffic_Project.ipynb)
- [PDF report](Project_Report.pdf)
- [Detailed README: setup, dataset, methods and results](README_DETAILED.md)

**Saved result:** validation-selected Random Forest achieved **87.12% test accuracy**,
**0.8649 macro F1** and **0.9814 ROC-AUC** under the documented cleaning policy.

Tested with **Python 3.13.12**:

```powershell
uv venv --python 3.13.12 .venv
uv pip install --python .venv\Scripts\python.exe -r requirements.txt
```

Open the notebook, select this environment and run all cells. Missing datasets
download automatically from the pinned mirror; raw data and `.venv` are excluded
from Git. See the detailed README for provenance and evaluation limitations.
