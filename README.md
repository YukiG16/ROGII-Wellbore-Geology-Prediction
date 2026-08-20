# ROGII - Wellbore Geology Prediction

Machine learning project for the **ROGII - Wellbore Geology Prediction** Kaggle competition.

> Build a model that contributes to automating drilling operations in the oil and gas industry.

The competition focuses on predicting **TVT (True Vertical Thickness)** along the evaluation zone of horizontal wells using horizontal-well trajectory/log data and corresponding vertical reference logs (**Typewells**).

---

## Overview

Horizontal drilling requires navigating through geological formations that cannot be directly observed everywhere. The competition provides data from horizontal wells together with vertical reference logs and asks participants to predict the geological position represented by `TVT`.

The key challenge is not simply tabular regression. The data contain:

- Well trajectories
- Measured depth and spatial coordinates
- Gamma Ray logs
- Geological surface information
- Vertical reference logs
- Within-well sequential structure
- A partially observed `TVT` signal through `TVT_input`

This repository is intended to document the **data exploration, validation strategy, modeling experiments, and final submission pipeline** in a reproducible manner.

---

## Competition Information

| Item | Description |
|---|---|
| Competition | ROGII - Wellbore Geology Prediction |
| Host | ROGII |
| Type | Featured Code Competition |
| Task | Predict `TVT` in the evaluation zone |
| Target | `TVT` — True Vertical Thickness (ft) |
| Metric | RMSE |
| Optimization | Lower is better |
| Submission | `submission.csv` |
| Notebook runtime | CPU/GPU ≤ 9 hours |
| Internet | Disabled during submission |
| Data | Horizontal wells + Typewells + training visualizations |

The competition is described as a **multimodal geology prediction** problem.

For the detailed source-derived competition specification, see:

**[`ROGII_Wellbore_Geology_Prediction_Context.md`](./ROGII_Wellbore_Geology_Prediction_Context.md)**

---

## Problem Definition

### Target

The prediction target is:

```text
TVT — True Vertical Thickness (ft)
```

For horizontal-well data, `TVT` represents the manually interpreted geological position for each point along the lateral well.

The model must predict `TVT` in the **evaluation zone**, where the target is hidden.

### `TVT_input`

The horizontal-well data contain:

```text
TVT_input
```

This is described by the competition as a copy of `TVT`, with values becoming `NaN` in the evaluation zone.

This makes the prediction setting particularly interesting: part of the well contains target-related information while the evaluation zone must be inferred.

---

## Dataset

The competition data are organized approximately as:

```text
train/
test/
sample_submission.csv
```

Each well is identified by a unique 8-character hash such as:

```text
015fe0d2
```

### Horizontal well

Each training well has a file:

```text
{WELLNAME}__horizontal_well.csv
```

Relevant fields include:

| Field | Description |
|---|---|
| `WELLNAME` | Unique well identifier |
| `MD` | Measured Depth (ft) |
| `X` | Easting (ft) |
| `Y` | Northing (ft) |
| `Z` | True Vertical Depth (ft) |
| `ANCC` | Predicted geological formation depth |
| `ASTNU` | Predicted geological formation depth |
| `ASTNL` | Predicted geological formation depth |
| `EGFDU` | Predicted geological formation depth |
| `EGFDL` | Predicted geological formation depth |
| `BUDA` | Predicted geological formation depth |
| `TVT` | Target: True Vertical Thickness (ft) |
| `GR` | Gamma Ray (API) |
| `TVT_input` | Copy of `TVT`; `NaN` in evaluation zone |

> **Important:** `ANCC`, `ASTNU`, `ASTNL`, `EGFDU`, `EGFDL`, and `BUDA` are described as **training-only** fields in the competition Data specification. They must therefore be handled carefully when designing a model intended for the hidden test data.

### Typewell

Each well also has:

```text
{WELLNAME}__typewell.csv
```

with:

| Field | Description |
|---|---|
| `TVT` | Vertical depth index / primary depth reference |
| `GR` | Vertical Gamma Ray signature |
| `Geology` | Geological formation label |

The Typewell is explicitly described as a **vertical reference log for geological correlation**.

### Training visualization

Training wells also contain:

```text
{WELLNAME}.png
```

These visualize:

- Well path
- Geological cross-section

The competition is tagged as multimodal, although the supplied Data specification does not prescribe a particular image-modeling method.

---

## Test Data

The competition describes the test set as containing approximately 200 wells.

Test wells provide:

```text
{WELLNAME}__horizontal_well.csv
{WELLNAME}__typewell.csv
```

The `TVT` target is hidden in the evaluation zone.

### Important Kaggle behavior

The visible `test/` directory contains only a small number of example instances from the training data for notebook development.

When a submission is rerun on the hidden test set, these example files are replaced with the actual evaluation data.

Therefore, the visible test directory should **not** be assumed to represent the complete hidden test distribution.

---

## Evaluation

Submissions are evaluated using **Root Mean Squared Error (RMSE)**:

$$
RMSE =
\sqrt{
\frac{1}{n}
\sum_{i=1}^{n}
(y_i-\hat{y}_i)^2
}
$$

where:

- $y$ = true TVT
- $\hat{y}$ = predicted TVT
- $n$ = number of prediction rows

**Lower RMSE is better.**

Local validation should therefore use RMSE as the primary model-selection metric.

---

## Submission Format

The final submission must be:

```text
submission.csv
```

with:

```csv
id,tvt
015fe0d2_1654,0.0
015fe0d2_1655,0.0
...
```

### Columns

- `id`: `{WELLNAME}_{row_index}`
- `tvt`: predicted `TVT`

A prediction is required for each prediction point in the test data.

---

## Modeling Strategy

This project treats the problem as a **structured, well-level, sequential geological prediction problem**, rather than ordinary independent-row regression.

The development process is intended to proceed incrementally.

### 1. Data Audit

First establish what the actual Kaggle files contain.

Key checks:

- Number of wells
- Rows per well
- Column schema and dtypes
- Missing values
- `MD` spacing
- Evaluation-zone boundaries
- `TVT` / `TVT_input` relationship
- Horizontal-well / Typewell relationships
- `GR` distributions
- Geological surface distributions
- Well-specific anomalies

### 2. Geological Exploration

Investigate:

- Horizontal well trajectory
- `MD`, `X`, `Y`, `Z` relationships
- Horizontal-well `GR`
- Typewell `GR`
- Typewell `Geology`
- Geological surface fields
- `TVT` behavior
- Relationship between horizontal-well and Typewell signals

### 3. Leakage-Safe Validation

The data are organized around individual wells.

Therefore, validation should explicitly test **well-level generalization** rather than relying only on random row splits.

Candidate validation schemes should be compared empirically.

### 4. Baseline

Start with a simple and interpretable baseline before introducing complex architectures.

Candidate baseline families include:

- Simple regression
- Tree-based regression
- Within-well interpolation/extrapolation where valid
- Typewell-derived features
- Local sequence/window features

### 5. Geological Correlation

Investigate how the Typewell can be aligned with the horizontal well using:

- `GR`
- Typewell `TVT`
- `Geology`
- Horizontal-well trajectory information

Correlation methods should be treated as **hypotheses to validate**, not assumed to be physically correct.

### 6. Sequence-Aware Features

Potential directions include:

- Rolling statistics
- Local gradients
- Neighboring observations
- Window-based features
- Sequential models
- Local geological context

### 7. Well-Specific Behavior

Investigate whether wells differ substantially in:

- Gamma Ray patterns
- Geological relationships
- Trajectory geometry
- Target behavior

If well-specific strategies are introduced, the method-selection rule should be documented and validated.

### 8. Uncertainty

The competition's Working Note criteria explicitly value uncertainty estimation.

Potential approaches include:

- Prediction intervals
- Ensemble dispersion
- Local residual analysis
- Well-specific confidence
- Identification of unreliable trajectory regions

---

## Repository Philosophy

The main goals of this repository are:

1. **Reproducibility**
2. **Leakage-resistant validation**
3. **Geological interpretability**
4. **Experiment tracking**
5. **Clear separation between facts and hypotheses**
6. **Efficient Kaggle execution**
7. **Documented negative results**

A high leaderboard score is not the only objective.

Where possible, experiments should explain:

- What was changed
- Why it was changed
- How validation changed
- Whether the improvement is robust
- Whether the behavior is physically meaningful
- What was learned when an approach failed

---

## Recommended Experiment Structure

Each major experiment should record:

```text
Experiment ID
-------------
Hypothesis:
Data/features:
Validation scheme:
Model:
Key parameters:
Validation RMSE:
Public LB:
Private LB:
Result:
Interpretation:
Next step:
```

Minor hyperparameter variations do not need to be treated as completely separate research directions.

Meaningfully different approaches should receive deeper analysis.

---

## Important Constraints

The Kaggle submission environment specifies:

- CPU Notebook runtime: **≤ 9 hours**
- GPU Notebook runtime: **≤ 9 hours**
- Internet access: **disabled**
- Publicly available external data and pretrained models: allowed
- Submission filename: `submission.csv`

These constraints should influence the final pipeline design from the beginning.

---

## What We Do Not Assume

The published Overview/Data material does **not** fully specify:

- Exact number of training wells
- Exact rows per well
- Exact evaluation-zone boundaries
- Exact missing-value patterns
- Exact sampling interval
- Exact relationship between row index and `MD`
- Exact horizontal-well / Typewell alignment method
- Exact leaderboard split construction
- Whether PNGs improve predictive performance
- Whether a particular model architecture is preferred
- Whether a specific physical equation should be imposed
- Whether training-only geological surfaces can be reconstructed for test data

These questions should be answered by inspecting the actual Kaggle data or additional official competition documentation.

**Do not invent missing dataset details.**

---

## Project Status

This repository is intended to evolve through the following stages:

- [ ] Dataset inspection
- [ ] EDA
- [ ] Evaluation-zone analysis
- [ ] Well-level validation
- [ ] Simple baseline
- [ ] Typewell correlation baseline
- [ ] Sequence-aware features
- [ ] Advanced models
- [ ] Uncertainty estimation
- [ ] Ensemble / blending
- [ ] Final Kaggle submission
- [ ] Experiment and error analysis documentation

---

## Repository Structure

A suggested structure is:

```text
.
├── README.md
├── ROGII_Wellbore_Geology_Prediction_Context.md
│
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_validation.ipynb
│   ├── 03_baseline.ipynb
│   ├── 04_typewell_correlation.ipynb
│   ├── 05_sequence_model.ipynb
│   └── 06_final_submission.ipynb
│
├── src/
│   ├── data.py
│   ├── validation.py
│   ├── features.py
│   ├── correlation.py
│   ├── models.py
│   └── inference.py
│
├── experiments/
│   ├── README.md
│   └── results.csv
│
├── reports/
│   ├── eda/
│   ├── validation/
│   └── error_analysis/
│
└── submissions/
    └── submission.csv
```

The actual repository structure may change as the project develops.

---

## Source of Competition Context

The competition-specific information in this repository is based on the supplied Kaggle **Overview** and **Data** pages.

For the detailed source-derived specification, see:

[`ROGII_Wellbore_Geology_Prediction_Context.md`](./ROGII_Wellbore_Geology_Prediction_Context.md)

The Context file deliberately distinguishes between:

- Official competition information
- Observations that should be made from actual CSV data
- Modeling hypotheses
- General machine-learning considerations

This distinction is important when using the repository as context for another LLM.

---

## Working With LLMs

When using this repository with ChatGPT or another LLM:

> Treat `ROGII_Wellbore_Geology_Prediction_Context.md` as the source-derived competition specification.

The LLM should:

1. Not invent missing dataset details.
2. Inspect actual CSV files before making assumptions about schema or sampling.
3. Treat `TVT` as the prediction target.
4. Treat `TVT_input` as target-related information that becomes `NaN` in the evaluation zone.
5. Treat the geological surface fields as training-only according to the supplied Data specification.
6. Preserve well-level structure during validation.
7. Use RMSE for validation.
8. Distinguish source facts from modeling hypotheses.
9. Track unsuccessful experiments and lessons learned.
10. Consider physical plausibility and uncertainty, not only leaderboard performance.

---

## Competition Timeline

According to the supplied competition Overview:

| Event | Date |
|---|---|
| Start Date | May 5, 2026 |
| Working Note Award deadline | July 6, 2026 |
| Entry Deadline | July 29, 2026 |
| Team Merger Deadline | July 29, 2026 |
| Final Submission Deadline | August 5, 2026 |

Deadlines were specified as 11:59 PM UTC unless otherwise noted.

---

## License / Data Usage

This repository does not redistribute the competition dataset.

Users should obtain the competition data directly through Kaggle and comply with the competition's rules and data usage terms.

---

## Acknowledgement

Competition:

**ROGII - Wellbore Geology Prediction**

Host:

**ROGII**

Official competition citation listed in the supplied material:

> Igor Kuvaev, Rafael Aguilar, John Granmayeh, Ryan Holbrook, María Cruz, and Ashley Oldacre. *ROGII - Wellbore Geology Prediction*. Kaggle, 2026.
