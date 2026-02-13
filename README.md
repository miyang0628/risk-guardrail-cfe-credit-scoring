# Risk Guardrails for Counterfactual Explanations in AI Credit Scoring

> **Counterfactual Explanation (CFE)-Based Risk Guardrail Design for Adverse Selection Prevention and Consumer Protection in AI Credit Scoring Models**

## Overview

This repository contains the full experimental code for the paper that proposes a **three-stage AI governance framework** to ensure accountability and financial stability in AI credit scoring systems. The framework prevents adverse selection and gaming risks by embedding risk guardrails directly into the counterfactual explanation generation process.

### Key Contributions

- **Extended Reliable Recourse Rate**: `Reliable RR = RR × (1 − VR) × (1 − Causal VR)` — a multi-layered metric that captures both immutability violations and causal inconsistencies
- **Three-stage guardrail architecture**: Immutability (Ω_imm) → Causality (Ω_cau) → Actionability (Ω_act) with hierarchical priority enforcement
- **Large-scale simulation**: 1,158 rejected borrowers × 4 paths = 4,632+ counterfactual paths analyzed
- **Algorithm robustness validation**: Random vs KD-Tree comparison confirming algorithm-independent guardrail protection

## Results Summary

### Guardrail Performance (Scenario Comparison)

| Scenario | RR (%) | VR (%) | Causal VR (%) | Reliable RR (%) | Avg Features Changed |
|:---:|:---:|:---:|:---:|:---:|:---:|
| **A** (No constraints) | 100.00 | 94.11 | 11.49 | 5.22 | 2.77 |
| **B** (Immutability only) | 36.87 | 0.00 | 28.22 | 26.47 | 3.27 |
| **C** (Full guardrails) | 11.05 | 0.00 | 46.92 | 5.87 | 1.99 |

> Without guardrails, **94.11%** of generated paths manipulate immutable variables. A single immutability constraint reduces this to **0.00%**, improving recourse quality by **5.1×**.

### Algorithm Robustness (Random vs KD-Tree)

| Scenario | Method | RR (%) | VR (%) | Causal VR (%) | Reliable RR (%) | Avg Features Changed |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| A (No constraints) | Random | 100.00 | 94.11 | 11.49 | 5.22 | 2.77 |
| A (No constraints) | KD-Tree | 100.00 | **100.00** | 18.57 | **0.00** | **15.93** |
| C (Full guardrails) | Random | 11.05 | **0.00** | 46.92 | 5.87 | 1.99 |
| C (Full guardrails) | KD-Tree | **0.00** | — | — | **0.00** | — |

> KD-Tree fails entirely under full guardrails, confirming Random method's suitability for constrained environments.

### Sensitivity Analysis (Threshold Variation under Full Guardrails)

| Threshold | RR (%) | VR (%) | Causal VR (%) | Reliable RR (%) | Avg Features Changed |
|:---:|:---:|:---:|:---:|:---:|:---:|
| ±10% | 7.25 | 0.00 | 30.93 | 5.01 | 2.03 |
| ±15% | 9.24 | 0.00 | 38.53 | 5.68 | 2.01 |
| ±20% | 11.05 | 0.00 | 46.92 | 5.87 | 1.99 |
| ±25% | 12.69 | 0.00 | 50.87 | 6.24 | 1.99 |
| ±30% | 14.34 | 0.00 | 52.53 | 6.80 | 2.05 |

### Subgroup Analysis (by ExternalRiskEstimate Quartile)

| Quartile | N | Recourse Success | Reliable RR (%) | Avg Features Changed |
|:---:|:---:|:---:|:---:|:---:|
| Q1 (Low Score) | 293 | 2 | 0.34 | 1.62 |
| Q2 | 323 | 8 | 0.93 | 1.97 |
| Q3 | 288 | 44 | 7.10 | 2.02 |
| Q4 (High Score) | 255 | 74 | 17.04 | 1.98 |

> Recourse accessibility gap between Q1 (0.34%) and Q4 (17.04%) highlights the need for fairness-aware guardrail design.

## Repository Structure

```
├── README.md
├── requirements.txt
├── risk_guardrail_experiment.ipynb    # Full experiment notebook
├── heloc_dataset_v1.csv               # FICO HELOC dataset (not included, see below)
└── results/
    ├── model_performance.csv
    ├── hyperparameters.csv
    ├── scenario_comparison.csv
    ├── sensitivity_analysis.csv
    ├── subgroup_analysis.csv
    ├── method_comparison.csv
    └── case_analysis_tables_8_9_10.csv
```

## Getting Started

### Prerequisites

```bash
pip install -r requirements.txt
```

### Dataset

This study uses the **FICO HELOC** (Home Equity Line of Credit) dataset:
- **Source**: [FICO Explainable ML Challenge](https://community.fico.com/s/explainable-machine-learning-challenge)
- **Samples**: 10,459 | **Features**: 23 | **Target**: RiskPerformance (Binary)

> ⚠️ The dataset is not included in this repository. Please download it from the FICO website and place `heloc_dataset_v1.csv` in the root directory.

### Run Experiments

```bash
jupyter notebook risk_guardrail_experiment.ipynb
```

Or run as a Python script:

```bash
jupyter nbconvert --to script risk_guardrail_experiment.ipynb
python risk_guardrail_experiment.py
```

## Experiment Pipeline

```
Part 1: Data Loading & Preprocessing
    └── HELOC dataset, special code handling (-7, -8, -9 → 0)

Part 2: XGBoost Model Training
    └── Optuna Bayesian Optimization (100 trials, 5-fold CV)

Part 3: Guardrail Configuration
    ├── Immutable features (Ω_imm): 5 features fixed
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
```

## Environment

| Item | Specification |
|:---|:---|
| Python | 3.10 |
| XGBoost | 1.7.5 |
| DiCE | 0.11 |
| Optuna | 3.x |
| Random Seed | 42 |
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
