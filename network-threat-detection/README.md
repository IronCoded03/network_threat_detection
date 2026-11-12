# Network Threat Detection with Machine Learning

> Detecting and classifying malicious network flows on the UNSW-NB15 dataset.
> Work in progress; see [ROADMAP.md](ROADMAP.md).

## Problem
_TODO: one paragraph on why ML-based detection complements signature-based IDS._

## Dataset
UNSW-NB15 (UNSW Canberra Cyber Range Lab). It contains flow features generated with Argus and Bro/Zeek, covering normal traffic and 9 attack categories. Download instructions are in [data/README.md](data/README.md). The data is not included in this repo.

_TODO: known limitations you found during EDA._

## Approach
_TODO: preprocessing, models tried, how you avoided leakage._

## Results
| Model | Precision | Recall | F1 | PR-AUC | FPR |
|---|---|---|---|---|---|
| Rule baseline | | | | | |
| Logistic regression | | | | | |
| Random Forest | | | | | |
| XGBoost | | | | | |
| Isolation Forest (unsupervised) | | | | | |

_TODO: per-category results, ablation, simulated zero-day._

## Reproduce
```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# place the dataset CSVs in data/, then run the notebooks in order
```

## Next steps
_TODO_

## Citation
N. Moustafa and J. Slay, "UNSW-NB15: a comprehensive data set for network intrusion detection systems (UNSW-NB15 network data set)," *Military Communications and Information Systems Conference (MilCIS)*, 2015.
