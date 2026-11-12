# Roadmap: ML Network Threat Detection (UNSW-NB15)

The goal is a model that flags malicious network flows and identifies the attack category, with an honest evaluation you can defend in an interview. Each phase ends with a commit and a short write-up in `README.md`.

---

## Before you start (do these first)

1. **Create the repo in GitHub Desktop.** Choose File → New repository, set the local path, and *don't* initialize a README because this skeleton already has one. Copy the contents of this folder into the repo, check that `.gitignore` is included, and make your first commit.
2. **Make a virtual environment and keep it out of the repo.** `.gitignore` already ignores `.venv/`.
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```
3. **Download the data, but never commit it.** Put `UNSW_NB15_training-set.csv`, `UNSW_NB15_testing-set.csv` and `UNSW-NB15_features.csv` in `data/`. The whole `data/` folder is git-ignored except its README. Before every push, check that GitHub Desktop's "Changes" list shows no `.csv` or `.joblib` files.
4. **Use only public data.** Nothing from work, SCRAM, or any cleared environment goes in this repo, including screenshots, hostnames and IP ranges.

---

## Phase 1: Explore the data (notebook `01_eda.ipynb`)

**Do:**
- Load both CSVs and confirm the record counts (175,341 train / 82,332 test).
- Look at the class balance of `label` (normal vs attack) and of `attack_cat`, which has 9 categories. Expect Worms and Analysis to be tiny.
- Separate the categorical columns (`proto`, `service`, `state`) from the numeric ones.
- Check for duplicate rows and for rows that appear in both train and test.
- Plot feature distributions for normal vs attack traffic.

**Questions to answer in your write-up:**
- Which columns are targets or IDs and must be dropped from the features (`id`, `label`, `attack_cat`)?
- Do any single features separate the classes almost perfectly? Look at the TTL-related features (`sttl`, `dttl`, `ct_state_ttl`). Decide whether that's real attack behavior or an artifact of how the testbed was built. This makes a strong interview talking point.

**References:** *Hands-On Machine Learning* (Géron), ch. 2; pandas "10 minutes to pandas"; `UNSW-NB15_features.csv` for feature meanings.

---

## Phase 2: Binary detection baseline (`02_binary.ipynb`)

**Do:**
- Build a scikit-learn `Pipeline` with `ColumnTransformer`: one-hot encode the categoricals and scale the numerics for linear models.
- Baseline 1 is a simple hand-written rule from your EDA. It's the thing your model has to beat.
- Baseline 2 is logistic regression.
- The main model is a Random Forest, then XGBoost or LightGBM.

**Evaluate with:** precision, recall, F1, PR-AUC, and the confusion matrix. Accuracy alone is misleading on imbalanced data. Report the **false positive rate** explicitly, because that's what a SOC analyst cares about.

**Rules:**
- Tune with cross-validation on the training set only.
- Touch the test set once, at the end. Leakage from the test set is the most common mistake in IDS projects.

**References:** *Hands-On ML* ch. 3 (classification metrics) and ch. 7 (ensembles); scikit-learn user guide sections "Pipelines and composite estimators" and "Model evaluation".

---

## Phase 3: Attack category classification (`03_multiclass.ipynb`)

**Do:**
- Predict `attack_cat` across the 9 attack categories plus Normal.
- Handle the imbalance with class weights first. Try resampling (imbalanced-learn) only if needed, and only on the training folds.
- Report per-class precision and recall, plus macro-F1 rather than only weighted F1.

**Write-up:** Map each category to the closest MITRE ATT&CK tactic, for example Reconnaissance → TA0043. This links the ML work to your security background.

**References:** imbalanced-learn docs; MITRE ATT&CK Enterprise tactics.

---

## Phase 4: Explainability and robustness (`04_explain.ipynb`)

**Do:**
- Use SHAP on the best tree model and identify the top features overall and per attack category.
- **Ablation:** retrain without the TTL features and compare scores. If performance collapses, the model was learning the testbed, not the attacks.
- **Simulated zero-day:** remove one attack category (e.g. Backdoors) from training entirely, then measure how much of it the model still flags as malicious.

**References:** SHAP docs ("An introduction to explainable AI with Shapley values"); *Hands-On ML* ch. 7 (feature importance).

---

## Phase 5: Unsupervised anomaly detection (`05_anomaly.ipynb`)

**Do:**
- Train an Isolation Forest on **normal traffic only**, then score the test set.
- Compare it with the supervised model, both overall and on the held-out "zero-day" category from Phase 4.

**Question to answer:** When would you deploy supervised detection, unsupervised detection, or both? A short, honest answer here is worth more than a high accuracy number.

**References:** *Hands-On ML* ch. 9; scikit-learn "Novelty and Outlier Detection".

---

## Phase 6: Package it (`src/`)

**Do:**
- Move the final pipeline from notebooks into `src/train.py` and `src/predict.py`. Keep it minimal.
- Save the trained pipeline with `joblib` to `models/`, which is git-ignored.
- `predict.py` should take a CSV and output a label and score per row.
- Optional: a `Dockerfile`, then a small FastAPI endpoint. This sets up the later EKS and SageMaker steps of your roadmap.

---

## Phase 7: Finish the README

A hiring manager should get the whole story in two minutes:
- The problem and why it matters
- The dataset and its known limitations
- A results table: baseline vs models, with FPR
- The ablation and zero-day findings
- How to reproduce it (setup, download, run)
- Next steps

Pin the repo on your GitHub profile, and link it on your resume with one line of results, e.g. "XGBoost IDS on UNSW-NB15: X% recall at Y% FPR; showed TTL-feature leakage via ablation."

---

## Interview prep: be ready to answer

- Why PR-AUC over ROC-AUC here?
- How did you prevent leakage between train and test?
- What did SHAP show, and did it make security sense?
- Would this model work on your network? (Honest answer: not without retraining. Dataset shift is the core problem in ML for IDS.)
- What would you change to run this on live Zeek logs?
