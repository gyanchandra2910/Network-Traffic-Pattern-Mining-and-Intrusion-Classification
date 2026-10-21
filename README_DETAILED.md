# Detailed project documentation

[Short overview](README.md) | [Executed notebook](Network_Traffic_Project.ipynb) | [PDF report](Project_Report.pdf)

## Project and team

**Network Traffic Pattern Mining and Intrusion Classification** studies network-flow
records through the KDD workflow: understand, clean, select, transform, mine/train,
evaluate and present. Pattern mining describes common traffic combinations; binary
classification predicts Normal or Attack.

| Team member | Roll number |
|---|---|
| Gyan Chandra | CS23I1053 |
| ALURI VENKAT VAIBHAV | CS23I1040 |

- **Course:** Analytics and Systems of Big Data
- **Institute:** IIITDM Kancheepuram
- **Faculty:** B. Sivaselvan
- **Submission deadline:** 26 October 2026
- **Scope:** concepts through classification, with Random Forest as the additional classification concept beyond the supplied slides.

## Repository contents

| File | Purpose |
|---|---|
| `Network_Traffic_Project.ipynb` | Main implementation, saved results, correctness checks and seven original figures |
| `Project_Report.pdf` | 12-page course report explaining the methods, results and limitations |
| `README.md` | Short repository overview |
| `README_DETAILED.md` | Setup, dataset provenance, experiment design and result details |
| `requirements.txt` | Twelve direct dependencies pinned to versions recorded by the execution |
| `.gitignore` | Excludes environments, downloaded datasets, caches, helpers and supplied reference materials |
| `.gitattributes` | Preserves notebook line endings and treats PDF reports as binary files |

The notebook contains 17 executed code cells with saved outputs and no error
outputs. The saved execution completed on **10 October 2026** in **189.3 seconds**.
These are recorded results from that execution; publishing the repository does not
represent a new training run.

## Setup and execution

The recorded environment is **Windows 11, CPython 3.13.12**, with 12 logical CPUs
and 15.2 GiB RAM. The random seed is **42**; Random Forest uses **two workers**.
Runtime measurements depend on hardware, library versions and host load.

Clone the repository, then create the environment. The following PowerShell
commands require Git and `uv` to be installed:

```powershell
git clone https://github.com/gyanchandra2910/Network-Traffic-Pattern-Mining-and-Intrusion-Classification.git
cd Network-Traffic-Pattern-Mining-and-Intrusion-Classification
uv venv --python 3.13.12 .venv
uv pip install --python .venv\Scripts\python.exe -r requirements.txt
```

Open `Network_Traffic_Project.ipynb` in a notebook-capable editor, select
`.venv\Scripts\python.exe` as the kernel, restart it and run all cells in order.
Run from the repository root. Network access is needed to install dependencies
and download the datasets for the first time.

A terminal alternative executes and saves the same notebook:

```powershell
@'
import nbformat
from nbclient import NotebookClient

path = 'Network_Traffic_Project.ipynb'
notebook = nbformat.read(path, as_version=4)
NotebookClient(
    notebook,
    timeout=1800,
    kernel_name='python3',
    resources={'metadata': {'path': '.'}},
).execute()
nbformat.write(notebook, path)
'@ | .\.venv\Scripts\python.exe -
```

`ipykernel` supplies the Python kernel used by this environment. A rerun replaces
the notebook's saved outputs and timings. No separate trained-model or result
exports are required.

## Dataset source and filename mapping

Dataset context comes from the [official UNSW-NB15 documentation](https://research.unsw.edu.au/projects/unsw-nb15-dataset).
The actual CSV downloads use the [Mireu-Lab Hugging Face mirror](https://huggingface.co/datasets/Mireu-Lab/UNSW-NB15/tree/3eb02c4a0a29866b7abcb5ef77c45cf4fcc8f6b0),
pinned to revision `3eb02c4a0a29866b7abcb5ef77c45cf4fcc8f6b0`.

| Original mirror filename | Local destination | Role | Raw rows |
|---|---|---|---:|
| `test.csv` | `data/UNSW_NB15_training-set.csv` | Training | 175,341 |
| `train.csv` | `data/UNSW_NB15_testing-set.csv` | Testing | 82,332 |

The mirror filenames are reversed relative to the expected training/testing roles.
The notebook uses an explicit mapping and already used the correct roles before
the local filenames were corrected. Roles are not inferred from file size.

Expected SHA-256 values:

```text
UNSW_NB15_training-set.csv
bec7dd5ec88dc2a0ccc7a07879d338395ed7421750f675fd0339e07dfe0648fa

UNSW_NB15_testing-set.csv
734fe6642edf758f7c94d7d9149426b49d202fe8e7bf0bef47392489c3c0a559
```

Starting with an empty `data/` directory is supported: missing mirror files are
downloaded directly into their canonical local destinations. The downloader uses
bounded retries and validates CSV structure, schema, row count and hash. Matching
cached files are reused; conflicting files fail validation and are not overwritten.

An independent source review recorded in the notebook found that every row and all
44 common columns, excluding `id`, exactly matched the standard configuration of
[lacg030175/UNSW-NB15](https://huggingface.co/datasets/lacg030175/UNSW-NB15#configurations).
This establishes agreement between the two mirrors. Direct identity with the
original UNSW-hosted files remains unverified.

### Schema and target

The files contain 45 columns: 39 numeric predictors, three categorical predictors
(`proto`, `service`, `state`), and `id`, `attack_cat`, `label`.

- `label` is the binary target: **0 = Normal; 1 = Attack**.
- `id` and `attack_cat` are excluded from classifier predictors.
- The complete 45-column schema and class distributions are recorded in the notebook.
- Raw training contains 56,000 Normal and 119,341 Attack rows; test contains 37,000 Normal and 45,332 Attack rows.

## Cleaning and evaluation protocol

The fixed policy strips categorical whitespace, marks missing/invalid values,
checks numeric domains and retains valid extreme traffic. The unspecified-service
token `-` remains a category. No raw CSV contents are modified.

Predictor identity uses all 42 traffic predictors, excluding the identifier and
both label fields. Training identities with conflicting labels are removed;
remaining training predictor rows are deduplicated; training identities occurring
in test are excluded. Test keeps its valid rows and multiplicities.

| Audit or partition | Rows / count |
|---|---:|
| Raw training | 175,341 |
| Extra training predictor copies | 74,301 |
| Ambiguous training rows | 940 |
| Training rows overlapping test before other exclusions | 10,279 |
| Training rows removed by the union of policies | 75,734 |
| Eligible training | 99,607 |
| Training-fit partition | 79,685 |
| Validation partition | 19,922 |
| Retained held-out test | 82,332 |
| Unique test predictor identities | 53,946 |

Exclusion counts overlap and must not be added. The stratified training/validation
split uses 80/20 proportions and seed 42.

**This is a documented custom cleaning protocol.** The overlap audit consults test
predictors when excluding training identities. Test labels do not enter model
fitting, hyperparameter selection or threshold tuning. Learned preprocessing is
fitted on the training-fit partition during validation, then refitted on all
eligible training rows for final evaluation. The test partition is therefore not
described as completely untouched throughout data preparation.

Cleaning removes about 43.2% of raw training rows and changes their weighting and
class distribution. Published results require comparable preprocessing before a
direct performance comparison is meaningful.

The separate unique-test sensitivity check keeps the **first source occurrence**
per predictor identity, with no retuning. A source review found six test identities
(14 rows) with conflicting labels; the first-occurrence convention also applies
to these identities. This affects interpretation of the secondary check. The
primary evaluation retains every valid test row with its original label.

## Analysis and transformations

The notebook includes descriptive statistics, class/protocol distributions,
source-byte histograms, Spearman correlations, a fit-only byte boxplot and a
seeded scatter sample of at most 3,000 fit rows.

A training-fitted selector removes constant and near-redundant numeric attributes
using absolute Spearman correlation at least 0.995 on at most 20,000 sampled rows.
It removes `ct_ftp_cmd` and `ct_flw_http_mthd`, reducing 42 raw predictors to 40.

Actual primary transformations are:

- **Trees and forest:** median numeric imputation, categorical one-hot encoding with minimum frequency 20 and unseen-category handling, and float32 sparse input.
- **Mixed Naive Bayes:** median numeric imputation, `log1p`, z-score scaling, and categorical encoding with a reserved unseen category.
- **Mining:** training-derived numeric quantile intervals and categorical `feature=value` items.

Min-max, z-score and strict decimal-scaling comparisons on raw fit `sbytes` are
separate learning demonstrations. So are five-/ten-bin mean, median and boundary
smoothing experiments on 5,000 fit values. IQR and raw/log-skewness calculations
are diagnostics. Smoothing and IQR flags do not modify the classifier or primary
mining inputs; valid extreme values are retained.

## Frequent itemsets and association rules

The main sample contains **5,000 eligible training flows**, selected with seed 42.
Each transaction uses eight attributes: `proto`, `service`, `state`, `dur`,
`sbytes`, `dbytes`, `ct_srv_src`, `sttl`. There are 79 distinct tokens. Numeric
quartiles are learned from training; repeated cutpoints are collapsed. Primary
transactions contain no `id`, `label` or `attack_cat`.

| Miner | Mechanism |
|---|---|
| Apriori | mlxtend level-wise candidate generation and subset pruning |
| FP-Growth | mlxtend FP-tree and conditional pattern growth |
| Hash Apriori | Educational pair-bucket pruning followed by exact counting and Apriori joins; not full DHP |
| Dynamic Itemset Counting | Educational circular stations, intermediate candidate insertion and four candidate states |
| Vertical TID | Educational ECLAT-style prefix recursion and transaction-ID intersections |

All methods use the same transactions, support threshold and maximum itemset
length **3**. The primary support grid is **0.20, 0.30, 0.40**; confidence values are
**0.70, 0.80, 0.90**, as requested in the project brief.

| Minimum support | Minimum count | Frequent itemsets, identical across all five miners |
|---:|---:|---:|
| 0.20 | 1,000 | 66 |
| 0.30 | 1,500 | 18 |
| 0.40 | 2,000 | 13 |

For a rule `A -> B`, support is `count(A union B) / N`, confidence is
`count(A union B) / count(A)`, and lift is `confidence / support(B)`.
Closed/maximal flags apply only among searched sets of length at most three.

A separate **0.05-support / 0.80-confidence** traffic experiment produces **597
frequent sets** and **412 positive-lift, single-consequent rules**. A separate
descriptive attack-consequent analysis adds the binary label only for that analysis.
For example, `dbytes=B4; sttl=B2 -> label=attack` has support 0.0538, confidence
0.9926, lift 2.0845 and count 269. These are training associations, not validated
attack-family signatures or causal explanations.

Correctness checks include 575 brute-force comparisons, including a bounded
real-data projection, and 15 full-feature real-data comparisons. All primary
experiments agree on exact integer counts across miners.

The runtime experiment uses three repetitions at 500, 1,500 and 3,000 transactions.
At 3,000 transactions and support 0.20, saved median times are:

| Miner | Median seconds |
|---|---:|
| Apriori | 0.0070 |
| FP-Growth | 0.0166 |
| Hash Apriori | 0.1038 |
| DIC | 0.1053 |
| Vertical TID | 0.0070 |

The notebook reports minimum and maximum times as well. Apriori has the smallest
unrounded median at 3,000 rows; Vertical TID rounds to the same displayed value.
Representation construction is included; file I/O, discretization, rule generation
and correctness comparisons are excluded. Implementations differ in optimization,
and these local measurements do not establish distributed scalability.

## Classification and saved results

The comparison includes a most-frequent-class baseline, binary CART-style Gini and
entropy trees, a mixed Gaussian/categorical Naive Bayes implementation, and a
64-tree Random Forest. The entropy tree is not ID3 or C4.5. Mixed NB uses additive
categorical smoothing and a positive Gaussian variance adjustment.

Four settings per learned model and one baseline give 17 validation candidates.
Each family's setting is selected using validation macro F1, then validation
ROC-AUC, then fixed grid order. The overall model is fixed before test evaluation.

| Model | Selected setting | Validation macro F1 |
|---|---|---:|
| Majority baseline | Most frequent class | 0.3402 |
| Gini tree | Depth 20; minimum leaf 20 | 0.9192 |
| Entropy tree | Depth 20; minimum leaf 5 | 0.9212 |
| Mixed Naive Bayes | Alpha 0.5; variance smoothing 0.01 | 0.8823 |
| Random Forest | Unlimited depth; minimum leaf 2; 64 trees | 0.9319 |

Final models are refitted on all 99,607 eligible training rows and evaluated on
82,332 retained test rows. **Attack is the positive class.** Undefined precision/F1
is zero; macro F1 averages both classes equally; ROC-AUC uses attack scores.

| Model | Accuracy | Attack precision | Attack recall | Attack F1 | Specificity | Macro F1 | ROC-AUC |
|---|---:|---:|---:|---:|---:|---:|---:|
| Majority baseline | 0.4494 | 0.0000 | 0.0000 | 0.0000 | 1.0000 | 0.3101 | 0.5000 |
| Gini tree | 0.8737 | 0.8249 | 0.9782 | 0.8951 | 0.7457 | 0.8683 | 0.9584 |
| Entropy tree | 0.8778 | 0.8379 | 0.9647 | 0.8968 | 0.7713 | 0.8735 | 0.9357 |
| Mixed Naive Bayes | 0.7793 | 0.7173 | 0.9889 | 0.8315 | 0.5225 | 0.7559 | 0.8652 |
| Random Forest | 0.8712 | 0.8170 | 0.9873 | 0.8941 | 0.7290 | 0.8649 | 0.9814 |

| Model | Fit seconds | Label prediction seconds | Probability seconds |
|---|---:|---:|---:|
| Majority baseline | 0.0024 | 0.0008 | 0.0005 |
| Gini tree | 6.8458 | 0.2248 | 0.2192 |
| Entropy tree | 7.0935 | 0.2078 | 0.2000 |
| Mixed Naive Bayes | 0.7935 | 0.1789 | 0.1983 |
| Random Forest | 54.6030 | 0.7441 | 0.6923 |

**Random Forest remains the validation-selected model.** The entropy tree has the
highest test accuracy (87.78%) and macro F1 (0.8735); the forest has the highest
test ROC-AUC (0.9814). Test results are reported without changing the selection.

The forest flags **10,027 normal flows** and misses **575 attacks**. On the separate
53,946-identity sensitivity check, its accuracy is 0.8229 and macro F1 is 0.8208.
This demonstrates that repeated test flows affect metric weights. Confusion
matrices, ROC curves and grouped forest feature importance are embedded in the
notebook and report.

## Scope, limitations and attribution

The data represents a controlled cyber range. Binary classification does not
establish recognition of each attack family or real-time operation. Model search
uses one seeded validation split and a small grid. Probabilities are uncalibrated;
default decisions do not encode investigation costs. Mining is bounded to eight
attributes, 5,000 training rows and length three; high support can hide rare
patterns. The study makes no deployment or distributed-performance claim.

Implemented concepts, separate demonstrations and explained-only slide algorithms
are distinguished in the notebook's coverage table and the report. The supplied
course slides provide course context. Essential implementation documentation is
linked for [scikit-learn trees](https://scikit-learn.org/stable/modules/tree.html),
[Naive Bayes](https://scikit-learn.org/stable/modules/naive_bayes.html),
[forests](https://scikit-learn.org/stable/modules/ensemble.html#forest), and mlxtend
[Apriori](https://rasbt.github.io/mlxtend/user_guide/frequent_patterns/apriori/) /
[FP-Growth](https://rasbt.github.io/mlxtend/user_guide/frequent_patterns/fpgrowth/).

The official UNSW documentation's dataset attribution and its five requested
publications are retained in the notebook and report. Consult the
[official dataset source](https://research.unsw.edu.au/projects/unsw-nb15-dataset)
for dataset use terms; source data and user-provided course materials are not
redistributed in this repository.
