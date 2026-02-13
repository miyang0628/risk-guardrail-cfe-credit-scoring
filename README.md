# Risk Guardrails for Counterfactual Explanations in AI Credit Scoring

> **Counterfactual Explanation (CFE)-Based Risk Guardrail Design for Adverse Selection Prevention and Consumer Protection in AI Credit Scoring Models**

## Overview

This repository contains the full experimental code for the paper that proposes a **three-stage AI governance framework** to ensure accountability and financial stability in AI credit scoring systems. The framework prevents adverse selection and gaming risks by embedding risk guardrails directly into the counterfactual explanation generation process.

### Key Contributions

- **Extended Reliable Recourse Rate**: `Reliable RR = RR × (1 − VR) × (1 − Causal VR)` — a multi-layered metric that captures both immutability violations and causal inconsistencies
- **Three-stage guardrail architecture**: Immutability (Ω_imm) → Causality (Ω_cau) → Actionability (Ω_act) with hierarchical priority enforcement
- **Large-scale simulation**: 1,164 rejected borrowers × 4 paths = 4,656+ counterfactual paths analyzed
- **Algorithm robustness validation**: Random vs KD-Tree comparison confirming algorithm-independent guardrail protection

## Results Summary

### Guardrail Performance (Scenario Comparison)

| Scenario | RR (%) | VR (%) | Causal VR (%) | Reliable RR (%) | Avg Features Changed |
|:---:|:---:|:---:|:---:|:---:|:---:|
| **A** (No constraints) | 100.00 | 94.29 | 10.95 | 5.08 | 2.71 |
| **B** (Immutability only) | 40.46 | 0.00 | 30.43 | 28.15 | 3.63 |
| **C** (Full guardrails) | 11.51 | 0.00 | 51.99 | 5.53 | 2.01 |

> Without guardrails, **94.29%** of generated paths manipulate immutable variables. A single immutability constraint reduces this to **0.00%**, improving recourse quality by **5.5×**.

### Algorithm Robustness (Random vs KD-Tree)

| Scenario | Method | RR (%) | VR (%) | Reliable RR (%) | Avg Features Changed |
|:---:|:---:|:---:|:---:|:---:|:---:|
| A (No constraints) | Random | 100.00 | 93.87 | 6.13 | 2.71 |
| A (No constraints) | KD-Tree | 100.00 | **100.00** | **0.00** | **15.94** |
| C (Full guardrails) | Random | 11.48 | **0.00** | 11.48 | 2.03 |
| C (Full guardrails) | KD-Tree | **0.00** | — | **0.00** | — |

> KD-Tree fails entirely under full guardrails, confirming Random method's suitability for constrained environments.

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
    └── method_comparison.csv
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
```

## Environment

| Item | Specification |
|:---|:---|
| Python | 3.10 |
| XGBoost | 1.7.5 |
| DiCE | 0.11 |
| Optuna | 3.x |
| Random Seed | 42 |

## Citation

If you find this work useful, please cite:

```bibtex
@article{author2026risk_guardrail,
  title={Counterfactual Explanation (CFE)-Based Risk Guardrail Design
         for Adverse Selection Prevention and Consumer Protection
         in AI Credit Scoring Models},
  author={},
  year={2026}
}
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
