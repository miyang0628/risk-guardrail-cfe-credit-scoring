# Risk Guardrails for Counterfactual Explanations in AI Credit Scoring

> **Counterfactual Explanation (CFE)-Based Risk Guardrail Design for Adverse Selection Prevention and Consumer Protection in AI Credit Scoring Models**

## Overview

This repository contains the full experimental code for the paper that proposes a **three-stage AI governance framework** to ensure accountability and financial stability in AI credit scoring systems. The framework prevents adverse selection and gaming risks by embedding risk guardrails directly into the counterfactual explanation generation process.

Primary experiments are conducted on the **FICO HELOC** dataset. Cross-dataset transferability is validated on the **UCI German Credit** dataset, confirming that the guardrail architecture transfers across lending domains without structural modification.

### Key Contributions

- **Extended Reliable Recourse Rate**: `Reliable RR = RR × (1 − VR) × (1 − Causal VR)` — a multi-layered metric that captures both immutability violations and causal inconsistencies
- **Three-stage guardrail architecture**: Immutability (Ω_imm) → Causality (Ω_cau) → Actionability (Ω_act) with hierarchical priority enforcement
- **Large-scale simulation**: 1,158 rejected borrowers × 4 paths = 4,632+ counterfactual paths analyzed (HELOC)
- **Cross-dataset transferability validation**: Full three-scenario replication on UCI German Credit (N = 1,000)
- **Algorithm robustness validation**: Random vs KD-Tree comparison confirming algorithm-independent guardrail protection across both datasets

---

## Results Summary

### Guardrail Performance — FICO HELOC (Primary)

| Scenario | RR (%) | VR (%) | Causal VR (%) | Reliable RR (%) | Avg Features Changed |
|:---:|:---:|:---:|:---:|:---:|:---:|
| **A** (No constraints) | 100.00 | 94.11 | 11.49 | 5.22 | 2.77 |
| **B** (Immutability only) | 36.87 | 0.00 | 28.22 | 26.47 | 3.27 |
| **C** (Full guardrails) | 11.05 | 0.00 | 46.92 | 5.87 | 1.99 |

> Without guardrails, **94.11%** of generated paths manipulate immutable variables. A single immutability constraint reduces this to **0.00%**, improving recourse quality by **5.1×**.

### Guardrail Performance — UCI German Credit (Transferability Validation)

| Scenario | RR (%) | VR (%) | Causal VR (%) | Reliable RR (%) | Avg Features Changed |
|:---:|:---:|:---:|:---:|:---:|:---:|
| **A** (No constraints) | 100.00 | 24.51 | 0.49 | 75.12 | 1.87 |
| **B** (Immutability only) | 100.00 | 0.00 | 0.00 | 100.00 | 1.80 |
| **C** (Full guardrails) | 72.55 | 0.00 | 12.16 | 63.73 | 2.03 |

> VR is eliminated completely under Ω_imm in both datasets. The directional pattern (Scenario B > Scenario C in Reliable RR) is preserved across lending jurisdictions.

### Algorithm Robustness (Random vs KD-Tree)

| Dataset | Scenario | Method | RR (%) | VR (%) | Reliable RR (%) | Avg Features Changed |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| HELOC | A | Random | 100.00 | 94.11 | 5.22 | 2.77 |
| HELOC | A | KD-Tree | 100.00 | **100.00** | **0.00** | **15.93** |
| HELOC | C | Random | 11.05 | **0.00** | 5.87 | 1.99 |
| HELOC | C | KD-Tree | **0.00** | — | **0.00** | — |
| German Credit | A | Random | 100.00 | 24.51 | 75.12 | 1.87 |
| German Credit | A | KD-Tree | 100.00 | **90.20** | **9.32** | **8.76** |
| German Credit | C | Random | 72.55 | **0.00** | 63.73 | 2.03 |
| German Credit | C | KD-Tree | **0.00** | — | **0.00** | — |

> KD-Tree fails entirely under full guardrails in **both datasets**, confirming algorithm-independent constraint enforcement.

### Sensitivity Analysis (Threshold Variation, HELOC, Scenario C)

| Threshold | RR (%) | VR (%) | Causal VR (%) | Reliable RR (%) | Avg Features Changed |
|:---:|:---:|:---:|:---:|:---:|:---:|
| ±10% | 7.25 | 0.00 | 30.93 | 5.01 | 2.03 |
| ±15% | 9.24 | 0.00 | 38.53 | 5.68 | 2.01 |
| ±20% | 11.05 | 0.00 | 46.92 | 5.87 | 1.99 |
| ±25% | 12.69 | 0.00 | 50.87 | 6.24 | 1.99 |
| ±30% | 14.34 | 0.00 | 52.53 | 6.80 | 2.05 |

### Subgroup Analysis

**FICO HELOC** — by ExternalRiskEstimate Quartile

| Quartile | N | Recourse Success | Reliable RR (%) | Avg Features Changed |
|:---:|:---:|:---:|:---:|:---:|
| Q1 (Low Score) | 293 | 2 | 0.34 | 1.62 |
| Q2 | 323 | 8 | 0.93 | 1.97 |
| Q3 | 288 | 44 | 7.10 | 2.02 |
| Q4 (High Score) | 255 | 74 | 17.04 | 1.98 |

**UCI German Credit** — by Loan Duration Quartile

| Quartile | N | Recourse Success | Reliable RR (%) | Avg Features Changed |
|:---:|:---:|:---:|:---:|:---:|
| Q1 (Short, ≤18mo) | 16 | 14 | 81.25 | 1.93 |
| Q2 (18–24mo) | 14 | 12 | 75.00 | 2.10 |
| Q3 (24–36mo) | 14 | 9 | 50.00 | 2.14 |
| Q4 (Long, >36mo) | 7 | 2 | 28.57 | 1.75 |

> A consistent monotonic recourse accessibility gradient is replicated across both datasets (HELOC: 50-fold disparity; German Credit: 2.8-fold disparity).

---

## Repository Structure

```
├── README.md
├── requirements.txt
├── risk_guardrail_experiment.ipynb          # Primary experiment (FICO HELOC)
├── german_credit_transferability.ipynb      # Transferability validation (UCI German Credit)
├── heloc_dataset_v1.csv                     # FICO HELOC dataset (not included, see below)
├── german.data-numeric                      # UCI German Credit dataset (not included, see below)
└── results/
    ├── model_performance.csv
    ├── hyperparameters.csv
    ├── scenario_comparison.csv
    ├── sensitivity_analysis.csv
    ├── subgroup_analysis.csv
    ├── method_comparison.csv
    ├── case_analysis_tables_8_9_10.csv
    ├── german_model_performance.csv
    ├── german_scenario_comparison.csv
    ├── german_sensitivity_analysis.csv
    ├── german_subgroup_analysis.csv
    ├── german_method_comparison.csv
    └── german_transferability_comparison.csv
```

---

## Getting Started

### Prerequisites

```bash
pip install -r requirements.txt
```

### Datasets

**Primary: FICO HELOC**
- **Source**: [FICO Explainable ML Challenge](https://community.fico.com/s/explainable-machine-learning-challenge)
- **Samples**: 10,459 | **Features**: 23 | **Target**: RiskPerformance (Binary)
- Place `heloc_dataset_v1.csv` in the root directory.

**Transferability Validation: UCI German Credit**
- **Source**: [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/144/statlog+german+credit+data)
- **Samples**: 1,000 | **Features**: 24 | **Target**: Risk (1 = Good, 2 = Bad)
- Place `german.data-numeric` in the root directory.

> ⚠️ Neither dataset is included in this repository. Please download each from the sources above.

### Run Experiments

**Primary experiment (FICO HELOC):**
```bash
jupyter notebook risk_guardrail_experiment.ipynb
```

**Transferability validation (UCI German Credit):**
```bash
jupyter notebook german_credit_transferability.ipynb
```

Or run as Python scripts:
```bash
jupyter nbconvert --to script risk_guardrail_experiment.ipynb
python risk_guardrail_experiment.py

jupyter nbconvert --to script german_credit_transferability.ipynb
python german_credit_transferability.py
```

---

## Experiment Pipeline

### Primary Experiment (`risk_guardrail_experiment.ipynb`)

```
Part 1: Data Loading & Preprocessing
    └── HELOC dataset, special code handling (-7, -8, -9 → 0)

Part 2: XGBoost Model Training
    └── Optuna Bayesian Optimization (100 trials, 5-fold CV)

Part 3: Guardrail Configuration
    ├── Immutable features (Ω_imm): 6 features fixed
    ├── Actionable features (Ω_act): 5 features with ±20% range
    └── Causal constraints (Ω_cau): 2 domain-specific rules

Part 4: Scenario Simulation
    ├── Scenario A: No constraints (Vanilla DiCE)
    ├── Scenario B: Immutability constraint only
    └── Scenario C: Full guardrails (Ω_imm + Ω_cau + Ω_act)

Part 5: Sensitivity Analysis
    └── Threshold variation: ±10%, ±15%, ±20%, ±25%, ±30%

Part 6: Subgroup Analysis
    └── ExternalRiskEstimate quartiles (Q1–Q4)

Part 7: Representative Case Extraction
    └── Vanilla vs Proposed side-by-side comparison

Part 8: Algorithm Robustness Validation
    └── Random vs KD-Tree under Scenario A & C

Part 9: Detailed Case Analysis
    └── Incidental pass vs Engineered pass (Tables 8, 9, 10)

Part 10A: Causal VR Paradox Analysis
    └── Absolute count decomposition (denominator compression effect)

Part 10B: Q1 vs Q4 Distribution Analysis
    └── Mann-Whitney U, headroom coverage, approval gap asymmetry
```

### Transferability Validation (`german_credit_transferability.ipynb`)

```
Part 1: Data Loading & Preprocessing
    └── german.data-numeric, column naming, target recode (1/2 → 1/0)

Part 2: XGBoost Model Training
    └── Optuna Bayesian Optimization (100 trials, 5-fold CV)

Part 3: Feature Classification & Guardrail Configuration
    ├── Immutable features (Ω_imm): 5 features (Age, Credit_History, etc.)
    ├── Actionable features (Ω_act): 5 features (Duration, Savings_Account, etc.)
    └── Causal constraints (Ω_cau): 2 domain-specific rules (European lending)

Part 4: Scenario Simulation (A / B / C)
Part 5: Sensitivity Analysis
Part 6: Subgroup Analysis (Loan Duration Quartiles)
Part 7: Representative Case Extraction
Part 8: Algorithm Robustness Validation (Random vs KD-Tree)
Part 9: Transferability Summary (HELOC vs German Credit comparison)
```

---

## Environment

| Item | Specification |
|:---|:---|
| Python | 3.10 |
| XGBoost | 1.7.5 |
| DiCE | 0.11 |
| Optuna | 3.x |
| scikit-learn | 1.x |
| scipy | 1.x |
| Random Seed | 42 |

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
