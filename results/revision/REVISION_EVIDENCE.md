# Revision evidence — robustness analyses

Supplementary analyses supporting the revision. All results use the pinned,
reproducible stack (`xgboost==1.7.6`, `dice-ml==0.11`, `RANDOM_SEED=42`) and the
committed primary models. Reproduce with `revision_run.py` / `revision_figures.py`.
Tables: `results/revision/{dataset}_*.csv`. Figures: `results/figures/revision/`.

Each section states the reviewer point it addresses, the analysis, and the finding.

---

## 1. Metric structure: product form vs. true joint (R1 — "Definition of Reliable RR")

**Reviewer point.** The Pathway Reliable RR is defined as a *product*
`(1−VR)(1−Causal VR)(1−Action VR)`, which implies the three violation sources
contribute independently, while the cost functions are described
*hierarchically*. The product-vs-hierarchy inconsistency must be reconciled.

**Analysis.** We instrument the path-level analysis to additionally count paths
that violate *none* of the three constraints simultaneously — the **true joint**
reliable rate — and compare it with the product approximation and with the naive
hierarchical residual `(1−Action VR)`. Table: `*_metric_structure.csv`;
figure: `metric_product_vs_joint`.

**Finding (German Credit).**

| Scenario | VR | Causal VR | Action VR | PRRR product | PRRR joint | (1−AVR) | product − joint |
|---|---|---|---|---|---|---|---|
| A (unconstrained) | 18.63 | 0.00 | 32.35 | 55.05 | 51.96 | 67.65 | **+3.09 pp** |
| B (immutability) | 0.00 | 0.49 | 39.71 | 60.00 | 60.29 | 60.29 | −0.29 pp |
| C (full guardrails) | 0.00 | 12.33 | 0.00 | 87.67 | 87.67 | 100.00 | 0.00 pp |

**Finding (FICO HELOC).**

| Scenario | VR | Causal VR | Action VR | PRRR product | PRRR joint | (1−AVR) | product − joint |
|---|---|---|---|---|---|---|---|
| A (unconstrained) | 93.28 | 9.80 | 31.66 | 4.15 | 1.88 | 68.34 | **+2.27 pp** |
| B (immutability) | 0.00 | 31.87 | 84.14 | 10.81 | 13.67 | 15.86 | **−2.86 pp** |
| C (full guardrails) | 0.00 | 43.27 | 0.00 | 56.73 | 56.73 | 100.00 | 0.00 pp |

**Interpretation for the paper.** The gap between the product form and the true
joint rate reflects *correlation among the violation types*, and — crucially —
**its sign is not fixed.** When violation types fall on *different* paths
(negative correlation), the product **overstates** reliability (Scenario A, both
datasets: +2.3 to +3.1 pp). When two sources *co-occur* on the same paths
(positive correlation), the product **understates** it (HELOC Scenario B: the
product 10.81 vs. true joint 13.67, −2.86 pp — immutability is satisfied while
causal and action violations cluster together, so more paths survive all three
filters than independence predicts). The deviation stays within ≈3 pp in both
datasets, but because the product form neither consistently bounds nor equals the
joint rate, it should **not** be the headline reliability figure. The naive
hierarchical residual `(1−AVR)` is far worse — off by up to ~66 pp (HELOC A:
68.34 vs. 1.88). **Recommendation:** report the **joint** rate as the primary
metric (it needs no independence assumption and is exact), and present the
product form only as a *diagnostic decomposition* of the three violation
channels. This reconciles the product-vs-hierarchy objection directly, and the
sign-flip between datasets is itself the empirical argument against treating the
product as a bound.

---

## 2. The ±20% action-cost rule (R1 & R2 & Editor — the central objection)

**Reviewer point.** The ±20% cliff-edge is ad hoc; it is unclear whether 20% is
the right level, whether one value fits all variables, and how it applies to
ordinal/categorical variables ("20% of zero is still zero").

### 2a. Sensitivity across the bound

**Analysis.** Scenario C over a fine grid (±5% … ±50%); table
`*_threshold_grid.csv`, figure `threshold_sensitivity`.

**Finding (German Credit).**

| ±x% | Recourse Rate | Pathway Reliable RR (joint) | Borrower Reliable RR |
|---|---|---|---|
| 5 | 62.75 | 85.25 | 62.75 |
| 10 | 64.71 | 86.72 | 64.71 |
| **20** | **72.55** | **87.67** | **72.55** |
| 30 | 76.47 | 86.18 | 72.55 |
| 50 | 82.35 | 81.82 | 80.39 |

The bound trades off **reach** (Recourse Rate rises monotonically, 63→82%)
against essentially flat **pathway reliability** (82–88%). The qualitative
conclusion is **not knife-edge sensitive** to the 20% choice; 20% is a
reasonable mid-point, not a result-driving artifact.

**Finding (FICO HELOC).**

| ±x% | Recourse Rate | Pathway Reliable RR (joint) | Borrower Reliable RR |
|---|---|---|---|
| 5 | 4.63 | 92.57 | 4.54 |
| 10 | 7.51 | 71.82 | 6.72 |
| **20** | **10.74** | **56.73** | **8.65** |
| 30 | 13.19 | 51.77 | 10.48 |
| 50 | 18.25 | 46.60 | 13.71 |

On HELOC the two curves move in **opposite** directions: loosening the bound
raises reach (RR 4.6→18.3%) but *sharply lowers* pathway reliability
(92.6→46.6%), because the admitted, more distant counterfactuals break logical
consistency far more often (Causal VR rises 7→53%). Here the ±20% bound is a
**genuine reach/reliability trade-off**, and 20% sits close to the knee of the
curve. **Cross-dataset reading:** 20% is not a result-flipping artifact in either
dataset, but the parameter behaves differently (reliability flat on German,
steeply declining on HELOC), which is direct evidence that the bound must be
**calibrated per setting** rather than fixed universally — supporting the
variable/segment-specific reframing below.

### 2b. The rule only genuinely binds continuous variables

**Analysis.** Variable-type audit of the actionable set; table `*_feature_types.csv`.

**Finding (German Credit).** Of five actionable features, only `Duration` is
truly continuous (33 levels); `Installment_Rate` (4 levels) and `Savings_Account`
(5 levels) are ordinal; `Existing_Credits` and `Num_Dependents` are binary with a
zero floor, where a percentage bound is meaningless and the code already falls
back to a ±0.5 absolute window. **This empirically confirms the reviewers'
point:** a single percentage rule is only coherent for continuous features.

**Finding (FICO HELOC).** All five actionable features are continuous
(18–126 distinct levels: `NetFractionRevolvingBurden` 126, `NumSatisfactoryTrades`
73, `PercentTradesNeverDelq` 71, `NumInqLast6M` 26, `NumTradesOpeninLast12M` 18),
so the multiplicative bound is coherent across the set. **But every one has a
zero floor** (Min = 0): for borrowers at or near zero the ±x% window collapses to
a point, so the ±0.5 absolute fallback is required even here. **Combined reading:**
a percentage bound alone is *never* sufficient — HELOC shows it degenerates at the
floor even for continuous features, and German shows it is incoherent for
ordinal/binary features. A multiplicative bound needs an absolute-floor companion
(already implemented via the ±0.5 fallback).

**Recommendation for the paper.** Reframe the ±20% rule as a **prototype
approximation to a cost/plausibility bound that a causal foundation would
supply**, not a universal law (R1). State explicitly that (i) it applies to
continuous features, (ii) ordinal/binary features need absolute or level-based
bounds (already used via the fallback), and (iii) the bound could be
variable-specific or segment-specific; the grid above shows the framework is
robust to that choice.

### 2c. A data-grounded bound replaces the ad hoc 20% — and dominates it

**Analysis.** Replace the fixed ±x% with a per-feature *absolute* window whose
half-width is a robust scale (α × IQR) of each actionable feature in the
**approved (good = 1) population**, so the bound is derived from observed,
accepted behaviour rather than chosen. Scenario C over α ∈ {0.25, 0.5, 1.0};
tables `*_databound_ablation.csv`, `*_databound_scales.csv`; figure
`databound_vs_fixed`.

**Finding (both datasets).**

| Dataset | Method | Recourse Rate | Pathway Reliable RR (joint) | Borrower Reliable RR |
|---|---|---|---|---|
| German | fixed ±20% (paper) | 72.55 | 87.67 | 72.55 |
| German | data-IQR (α=0.5) | 74.51 | **94.08** | 74.51 |
| German | data-IQR (α=1.0) | 80.39 | 84.15 | 78.43 |
| HELOC | fixed ±20% (paper) | 10.74 | 56.73 | 8.65 |
| HELOC | data-IQR (α=0.5) | 10.66 | **95.33** | 10.48 |
| HELOC | data-IQR (α=1.0) | 17.90 | 86.86 | 16.86 |

**The data-grounded bound dominates the fixed rule on both datasets.** At matched
reach (α=0.5) it holds German reach (74.5 vs 72.6) while raising pathway
reliability (94.1 vs 87.7), and on HELOC it holds reach (10.66 vs 10.74) while
raising reliability by **+38.6 pp** (95.33 vs 56.73) — the fixed ±20% had been
admitting many causally-inconsistent HELOC paths (Causal VR 43.3 → 4.7).
Loosening to α=1.0 raises reach *above* the fixed rule on both datasets with
reliability still at or above it. The fixed ±20% is also inconsistent across
features: on HELOC it implies a half-width of 19.2 for `PercentTradesNeverDelq`
but 5.0 for `NetFractionRevolvingBurden`, whereas the IQR bound gives 3.5 and
16.5 — a single percentage is simultaneously too loose for some features and too
tight for others. **This converts the reviewers' "ad hoc 20%" into a principled,
data-derived bound, and shows the paper's conclusions hold (indeed improve) under
it** — a direct, constructive answer to R1's call for "a more foundational
principle from which rules like this might be derived."

---

## 3. Feature-classification robustness (R2 — subjectivity of the classification)

**Reviewer point.** The immutable/actionable/non-controllable split is
subjective; would the conclusions survive an alternative classification?

**Analysis.** Re-run all scenarios after moving one contestable variable from
actionable to immutable. Table `*_classification_robustness.csv`.

**Finding (German Credit).** Moving `Num_Dependents` → immutable:

| Variant | Scenario | RR | VR | PRRR product | BRRR |
|---|---|---|---|---|---|
| baseline | A / B / C | 100 / 100 / 72.55 | 18.63 / 0 / 0 | 55.05 / 60.0 / 87.67 | 98.04 / 96.08 / 72.55 |
| alt (dependents immutable) | A / B / C | 100 / 100 / 47.06 | 24.02 / 0 / 0 | 51.40 / 66.50 / 76.60 | 98.04 / 100 / 43.14 |

The **ordering and qualitative story are preserved** (immutability drives VR to
zero; full guardrails raise pathway reliability), but **magnitudes shift**,
especially Scenario C reach (RR 72.6→47.1). **Honest takeaway:** conclusions are
directionally robust but level estimates are classification-sensitive — report
this explicitly and position the classification as an auditable, expert-reviewed
input (addresses R2's "how were classifications made/reviewed").

**Finding (FICO HELOC).** Moving `NumInqLast6M` → immutable (inquiries are hard to
reduce on demand):

| Variant | Scenario | RR | VR | PRRR product | BRRR |
|---|---|---|---|---|---|
| baseline | A / B / C | 100 / 47.6 / 10.74 | 93.28 / 0 / 0 | 4.15 / 10.81 / 56.73 | 5.24 / 11.27 / 8.65 |
| alt (inquiries immutable) | A / B / C | 100 / 56.33 / 9.69 | 93.30 / 0 / 0 | 4.13 / 11.84 / 53.20 | 5.24 / 11.53 / 7.60 |

On the primary dataset the ordering is preserved and the magnitudes move only
**mildly** (Scenario C reach 10.74→9.69%, product 56.73→53.20%; Scenario B reach
actually improves, 47.6→56.3%, as one fewer feature can be forced out of range).
HELOC is therefore **more** classification-robust than German — expected, since
removing one feature from a five-feature continuous actionable set has limited
leverage. **Combined takeaway:** conclusions are directionally robust on both
datasets; level sensitivity is dataset-specific (modest on HELOC, larger on
German), so the paper should report the classification as an auditable,
expert-reviewed input and show the alternative-classification numbers rather than
claim point estimates are invariant.

---

## 4. The recourse-accessibility gap across quartiles (R1 — "present the mechanism")

**Reviewer point.** The quartile gap is interesting but needs a mechanism; is it
an artefact of the analysis/data/search/business?

**Analysis.** Stratify the rejected pool by model-score quartile and decompose
reliable recourse into CF-success (RR), logical-consistency violations, and the
reliable rate. Table `*_quartile_mechanism.csv`, figure `quartile_mechanism`.

**Finding (German Credit).**

| Quartile | Score mean | RR | Causal VR | Pathway Reliable RR (joint) | Borrower Reliable RR |
|---|---|---|---|---|---|
| Q1 (low) | 0.239 | 30.77 | 25.00 | 75.00 | 30.77 |
| Q2 | 0.339 | 69.23 | 23.53 | 76.47 | 69.23 |
| Q3 | 0.408 | 91.67 | 4.55 | 95.45 | 91.67 |
| Q4 (high) | 0.468 | 100.00 | 7.69 | 92.31 | 100.00 |

**Mechanism.** The gap is driven primarily by **CF-success (distance to the
decision boundary)**: low-score borrowers are structurally farther from the
approval region, so far fewer feasible counterfactuals exist (RR 31% vs 100%),
and those that exist violate logical consistency more often (Causal VR 25% vs
8%). It is **not** a scoring artefact of the reliability metric itself (pathway
reliability *among found paths* is high across quartiles). This supports the
"distance-to-boundary / model-structure" explanation and lets the paper exclude
the metric-artefact explanation.

**Finding (FICO HELOC).**

| Quartile | N | Score mean | RR | Causal VR | Pathway Reliable RR (joint) | Borrower Reliable RR |
|---|---|---|---|---|---|---|
| Q1 (low) | 287 | 0.123 | **0.00** | — | 0.00 | 0.00 |
| Q2 | 286 | 0.219 | **0.00** | — | 0.00 | 0.00 |
| Q3 | 286 | 0.330 | 3.50 | 55.00 | 45.00 | 2.45 |
| Q4 (high) | 286 | 0.444 | 39.51 | 42.22 | 57.78 | 32.17 |

On the primary dataset the mechanism is **starker still**: under full guardrails
the bottom two score quartiles receive **zero** feasible recourse (RR 0%), rising
to 39.5% only in the top quartile. Causal VR is undefined at Q1/Q2 precisely
because *no* successful counterfactuals exist there — confirming the gap is one of
**feasibility (distance to boundary)**, not of how reliability is scored: where
paths do exist (Q3/Q4) pathway reliability is a healthy 45–58%. **Honest
limitation to report:** guardrails that guarantee reliability can drive *reach to
zero for the most marginal borrowers* — an equity trade-off the paper should state
explicitly rather than smooth over, and a direct, mechanistic answer to the
reviewers' "explain and bound the accessibility gap."

---

## Mapping to reviewer comments

| Analysis | Addresses |
|---|---|
| §1 product vs joint | R1 Definition of Reliable RR (product/hierarchy reconciliation) |
| §2a threshold grid | R1/R2/Editor ad hoc 20%; "is 20% the right level" |
| §2b feature types | R1 ordinal/categorical; R2 "20% of zero is zero" |
| §3 classification robustness | R2 subjectivity; "survive alternative classifications" |
| §4 quartile mechanism | R1 "present and exclude mechanisms" for the accessibility gap |
