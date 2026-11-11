# Counterfactual Explanation-Based Risk Guardrails for Reliable Algorithmic Recourse in AI Credit Scoring

Replication code for the manuscript of the same title, currently under
double-blind review. This repository is released anonymously; it contains no
author, institution, or journal identifying information.

## Overview

Counterfactual explanation (CFE) techniques in AI credit scoring can generate
recourse pathways that are unreliable from a credit-risk perspective. This
repository implements and evaluates a three-stage guardrail framework —
feature governance, constraint-embedded CFE optimization, and operational
monitoring — that operationalizes three financial-risk constraints
(immutability, logical consistency, and action-cost bounds) within XGBoost
credit-scoring models.

Two complementary diagnostics are introduced beyond the conventional Recourse
Rate (RR):

- **Pathway Reliable Recourse Rate (PRRR)** — the *joint* share of successful
  recourse pathways that are simultaneously free of all three violation types
  (not the product of the three marginal pass rates).
- **Borrower Reliable Recourse Rate (BRRR)** — the share of rejected borrowers
  who receive at least one reliable pathway.

Primary experiments use the **FICO HELOC** dataset; cross-domain
transferability is validated on the **UCI German Credit** dataset.

## Repository structure

```
.
├── data/                      # datasets (see data/README.md)
│   └── german.data-numeric
├── notebooks/
│   ├── heloc_experiment.ipynb     # FICO HELOC experiment (single cell)
│   └── german_experiment.ipynb    # UCI German Credit transferability (single cell)
├── results/
│   ├── figures/               # generated figures (grayscale, 600 dpi, PNG + PDF)
│   └── tables/                # result tables (CSV)
├── requirements.txt
├── LICENSE
└── README.md
```

## Reproducing the results

Each notebook is a **single self-contained code cell** with one switch at the
top:

```python
RUN_EXPERIMENT = False
```

- **`RUN_EXPERIMENT = False` (default).** Rebuilds every figure and table from
  the committed result files in `results/tables/`. This runs in seconds and
  requires only `numpy`, `pandas`, `scipy`, `scikit-learn`, `matplotlib`, and
  `seaborn`. Use this to regenerate the figures exactly as reported.
- **`RUN_EXPERIMENT = True`.** Re-runs the full pipeline from scratch —
  Optuna hyperparameter search, XGBoost training, DiCE counterfactual
  generation across the three guardrail scenarios, sensitivity and subgroup
  analysis, and the five-seed bootstrap. This additionally requires
  `xgboost`, `dice-ml`, `optuna`, and `tqdm`, and takes substantially longer.

Setup and execution:

```bash
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute notebooks/heloc_experiment.ipynb
jupyter nbconvert --to notebook --execute notebooks/german_experiment.ipynb
```

All figures are written to `results/figures/` as 600-dpi PNG and PDF, in
grayscale and without captions (captions are supplied in the manuscript).

## Notes on reproducibility

- **Pinned libraries.** The counterfactual results depend on exact versions:
  `xgboost==1.7.6` and `dice-ml==0.11`. Newer `dice-ml` (>=0.12) changes the
  random-search behaviour and returns no counterfactuals, and `xgboost` 2.x
  removes `use_label_encoder` and alters the fitted model. `requirements.txt`
  pins these.
- **Fixed seeds.** `RANDOM_SEED = 42` governs the train/test split and model
  training; the bootstrap uses five fixed seeds `[42, 123, 456, 789, 2024]`.
  The Optuna sampler is explicitly seeded.
- **Bootstrap holds hyperparameters fixed.** The five-seed bootstrap varies
  only the train/test split and re-uses the saved primary-seed hyperparameters
  for every seed (it does *not* re-tune per seed). This isolates sensitivity to
  the split, matches the "assess sensitivity to train/test split variation"
  purpose stated in the paper, and makes the bootstrap fully reproducible from
  the committed hyperparameters without any further search.
- **Reliability metric.** Pathway Reliable RR is computed as the joint
  all-constraints-satisfied rate; confidence intervals use the Student-t
  distribution with four degrees of freedom (five seeds).
- **Tuned hyperparameters are committed.** Both `hyperparameters.csv` (HELOC)
  and `german_hyperparameters.csv` are stored in `results/tables/`. With
  `USE_SAVED_HYPERPARAMS = True` (default) the primary model loads these exact
  values instead of re-searching, so the primary model is reproduced exactly;
  set it to `False` to re-run the seeded search.
- **Verification.** Under the pinned stack, HELOC reproduces the committed
  model metrics to four decimals (AUC 0.799, ECE 0.021), and the German
  scenario and bootstrap tables reproduce exactly.
- Scenario C fixes the action-cost violation rate to zero by construction: the
  `permitted_range` parameter enforces the ±20% bound at the DiCE optimization
  stage, so no post-hoc measurement is performed for that scenario.

## Data

See `data/README.md`. Both the UCI German Credit data and the FICO HELOC
dataset are included for reproducibility. The FICO HELOC data remains subject
to FICO's data-usage license; users must comply with those terms, and
`data/README.md` explains how to re-obtain it from FICO if needed.

## License

Released under the MIT License (see `LICENSE`). The license applies to the
code in this repository, not to the third-party datasets, which remain subject
to their own terms.
