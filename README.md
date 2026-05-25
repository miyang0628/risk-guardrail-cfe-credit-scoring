# Risk Guardrails for Counterfactual Explanations in AI Credit Scoring

## Overview

This repository contains the full experimental code for the paper:

> **"Designing Counterfactual Explanation-Based Risk Guardrails for Adverse Selection Prevention and Consumer Protection in AI Credit Scoring Models"**
> *Submitted for double-blind peer review. Author information withheld.*

This study designs and empirically validates a three-stage credit risk guardrail framework—comprising feature governance, constraint-embedded CFE optimization, and operational monitoring—to reduce adverse selection risk in AI-driven credit markets. The framework operationalizes three financial risk constraints (immutability, logical consistency, and action cost bounds) within XGBoost-based credit scoring models.

Primary experiments are conducted on the **FICO HELOC** dataset. Cross-dataset consistency of core guardrail properties is validated on the **UCI German Credit** dataset.

---

## Key Contributions

- **Two-level metric framework**: Pathway Reliable Recourse Rate and Borrower Reliable Recourse Rate — resolving the denominator inconsistency of prior single-composite formulations
- **Three-stage guardrail architecture**: Immutability (Ω_imm) → Logical Consistency (Ω_cau) → Action Cost (Ω_act) with hierarchical priority enforcement (Hard > Functional > Soft)
- **Bootstrap robustness**: Full pipeline replicated across 5 random seeds ({42, 123, 456, 789, 2024}) with 95% confidence intervals
- **Cross-dataset validation**: Core guardrail properties confirmed on UCI German Credit without architectural modification

---

## Main Results

### FICO HELOC — Primary Results (seed = 42)

| Scenario | RR (%) | VR (%) | Causal VR (%) | Action VR (%) | Pathway Reliable RR (%) | Borrower Reliable RR (%) |
|---|---|---|---|---|---|---|
| A (No constraints) | 100.00 | 88.17 | 9.72 | 32.37 | 7.22 | 4.30 |
| B (Immutability only) | 60.84 | 0.00 | 29.66 | 89.36 | 7.48 | 10.27 |
| C (Full guardrails) | 10.71 | 0.00 | 42.71 | 0.00* | 57.29 | 8.87 |

\* Action VR = 0% by construction under Scenario C: the `permitted_range` parameter enforces the ±20% bound at the DiCE optimization stage.

### Bootstrap Robustness — FICO HELOC (5 seeds, mean ± 95% CI)

| Metric | Scenario A | Scenario B | Scenario C |
|---|---|---|---|
| RR (%) | 100.00 [100.00, 100.00] | 47.58 [32.26, 62.91] | 10.56 [7.06, 14.06] |
| VR (%) | 93.53 [89.63, 97.42] | 0.00 [0.00, 0.00] | 0.00 [0.00, 0.00] |
| Causal VR (%) | 9.59 [7.46, 11.72] | 23.66 [15.56, 31.77] | 41.82 [38.94, 44.71] |
| Action VR (%) | 29.48 [22.79, 36.18] | 81.65 [74.42, 88.87] | 0.00 [0.00, 0.00]* |
| Pathway Reliable RR (%) | 4.09 [1.79, 6.38] | 13.98 [8.64, 19.33] | **58.18 [55.29, 61.06]** |
| Borrower Reliable RR (%) | 4.74 [3.56, 5.92] | **11.01 [8.77, 13.25]** | 8.88 [5.86, 11.89] |

### UCI German Credit — Transferability Validation (seed = 42)

| Scenario | RR (%) | VR (%) | Causal VR (%) | Action VR (%) | Pathway Reliable RR (%) |
|---|---|---|---|---|---|
| A (No constraints) | 100.00 | 21.61 | 0.00 | 35.17 | 50.82 |
| B (Immutability only) | 100.00 | 0.00 | 0.85 | 43.64 | 55.88 |
| C (Full guardrails) | 88.14 | 0.00 | 8.21 | 0.00* | 91.79 |

**Core guardrail properties consistent across both datasets:**
1. VR eliminated completely by Ω_imm (HELOC: 88.17%→0%; German: 21.61%→0%)
2. Pathway Reliable RR highest under Scenario C in both datasets

### Algorithm Sensitivity (FICO HELOC, seed = 42)

| Scenario | Method | RR (%) | VR (%) | Pathway Reliable RR (%) |
|---|---|---|---|---|
| A | Random | 100.00 | 88.17 | 7.22 |
| A | KD-Tree | 100.00 | 100.00 | 0.00 |
| C | Random | 10.71 | 0.00 | 57.29 |
| C | KD-Tree | 0.00 | — | 0.00 |

KD-Tree fails entirely under Scenario C in both datasets, establishing Random search as the practically superior strategy in constrained financial environments.

### Subgroup Analysis — FICO HELOC (Scenario C, seed = 42)

| Quartile | N | Pathway Reliable RR (%) |
|---|---|---|
| Q1 (Lowest Score) | 293 | 0.34 |
| Q2 | 323 | 0.93 |
| Q3 | 288 | 7.10 |
| Q4 (Highest Score) | 255 | 17.04 |

50-fold disparity (Q1: 0.34% vs. Q4: 17.04%) confirms that uniform ±20% thresholds encode structurally differential recourse access.

---

## Repository Structure

```
├── README.md
├── requirements.txt
├── heloc_bootstrap_v3.py               # Primary experiment (FICO HELOC) + Bootstrap
├── german_bootstrap_v3.py              # Transferability validation (UCI German Credit) + Bootstrap
├── heloc_dataset_v1.csv                # FICO HELOC dataset (not included, see below)
├── german.data-numeric                 # UCI German Credit dataset (not included, see below)
└── results/
    ├── scenario_comparison.csv
    ├── bootstrap_raw.csv
    ├── bootstrap_summary.csv
    ├── bootstrap_calibration.csv
    ├── sensitivity_analysis.csv
    ├── subgroup_analysis.csv
    ├── method_comparison.csv
    ├── model_performance.csv
    ├── hyperparameters.csv
    ├── german_scenario_comparison.csv
    ├── german_bootstrap_raw.csv
    ├── german_bootstrap_summary.csv
    ├── german_bootstrap_calibration.csv
    ├── german_sensitivity_analysis.csv
    ├── german_subgroup_analysis.csv
    ├── german_method_comparison.csv
    ├── german_model_performance.csv
    └── german_transferability_comparison.csv
```

---

## Datasets

### Primary: FICO HELOC
- **Source**: [FICO Explainable ML Challenge](https://community.fico.com/s/explainable-machine-learning-challenge)
- **Samples**: 10,459 | **Features**: 23 | **Target**: RiskPerformance (Binary)
- Place `heloc_dataset_v1.csv` in the root directory.

### Transferability Validation: UCI German Credit
- **Source**: [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/144/statlog+german+credit+data)
- **Samples**: 1,000 | **Features**: 24 | **Target**: Risk (1 = Good, 2 = Bad)
- Place `german.data-numeric` in the root directory.

> Neither dataset is included in this repository. Please download each from the sources above.

---

## Setup & Requirements

```bash
pip install -r requirements.txt
```

**Key dependencies:**

| Package | Version |
|---|---|
| Python | 3.10 |
| XGBoost | 1.7.5 |
| DiCE-ML | 0.11 |
| Optuna | 3.x |
| scikit-learn | 1.x |
| scipy | 1.x |
| matplotlib | 3.x |

---

## Running Experiments

### Primary Experiment (FICO HELOC)

```bash
python heloc_bootstrap_v3.py
```

Executes in two parts:
- **Part A** (100-sample validation): Verifies Action VR = 0% under Scenario C before full run
- **Part B** (full experiment): All scenarios, bootstrap robustness, sensitivity analysis, subgroup analysis, algorithm comparison

### Transferability Validation (UCI German Credit)

```bash
python german_bootstrap_v3.py
```

Same two-part structure as HELOC experiment.

---

## Experiment Pipeline

### PART A — Validation (100 samples)
Checks that Scenario C Action VR = 0% by construction before committing to full run.

### PART B — Full Experiment
```
B-1: Primary Scenario Simulation (A / B / C, seed 42, full test set)
B-2: Sensitivity Analysis (threshold ±10% ~ ±30%, Scenario C)
B-3: Subgroup Analysis (ExternalRiskEstimate quartiles Q1–Q4)
B-4: Algorithm Robustness Validation (Random vs KD-Tree)
B-5: Representative Case Extraction (incidental pass vs engineered pass)
B-6: Bootstrap Robustness (5 seeds × 3 scenarios)
B-7: Bootstrap Summary (Mean ± 95% CI, calibration metrics)
```

---

## Variable Classification (FICO HELOC)

| Classification | Variables | Constraint |
|---|---|---|
| **Immutable** | ExternalRiskEstimate, MSinceOldestTradeOpen, AverageMInFile, MSinceMostRecentDelq, MSinceMostRecentInqexcl7days, NumBank2NatlTradesWHighUtilization | Fixed: x'_i = x_i (infinite penalty) |
| **Actionable** | NetFractionRevolvingBurden, NumSatisfactoryTrades, NumTradesOpeninLast12M, PercentTradesNeverDelq, NumInqLast6M | ±20% limit, domain clipping [0,100%] |
| **Logical** | PercentTradesNeverDelq ↔ NumTrades90Ever2DerogPubRec, NetFractionRevolvingBurden → NumSatisfactoryTrades | Domain-expert if-then rules (soft penalty) |

---

## Guardrail Constraint Design

### Ω_imm: Immutability (Hard Constraint)
Infinite penalty on any modification of immutable variables. Guarantees VR = 0%.

### Ω_cau: Logical Consistency (Functional Constraint)
Two domain-expert if-then rules:
- **Rule 1**: `x'_Derog ≤ NumTotalTrades × (1 − x'_NeverDelq / 100)`
- **Rule 2**: `if x'_Revolving < x_Revolving → x'_Satisfactory ≥ x_Satisfactory × 0.9`

Implemented as soft penalty (not hard elimination).

### Ω_act: Action Cost (Soft Constraint)
Exponential penalty for changes exceeding ±20%. Under Scenario C, `permitted_range` enforces the bound at generation stage → Action VR = 0% by construction.

---

## Reproducibility

| Item | Configuration |
|---|---|
| Primary seed | 42 |
| Bootstrap seeds | {42, 123, 456, 789, 2024} |
| Train/test split | 8:2 (stratified) |
| Optuna trials | 100 (primary), 30 (bootstrap seeds 2–5) |
| CV folds | 5-fold stratified |
| CFs per sample | 4 |
| Actionability threshold | ±20% |

---

## Output Files

| File | Description |
|---|---|
| `scenario_comparison.csv` | Primary scenario A/B/C results (seed 42) |
| `bootstrap_raw.csv` | Seed-level results (5 seeds × 3 scenarios) |
| `bootstrap_summary.csv` | Mean ± 95% CI across seeds |
| `bootstrap_calibration.csv` | Brier Score, ECE per seed |
| `sensitivity_analysis.csv` | Threshold ±10%~±30% sensitivity |
| `subgroup_analysis.csv` | Credit score quartile subgroup results |
| `method_comparison.csv` | Random vs KD-Tree algorithm comparison |
| `model_performance.csv` | XGBoost performance metrics |
| `german_*` | Corresponding files for German Credit dataset |

---

## License

MIT License

---

*This repository is prepared for double-blind peer review.
Author information will be disclosed upon acceptance.*
