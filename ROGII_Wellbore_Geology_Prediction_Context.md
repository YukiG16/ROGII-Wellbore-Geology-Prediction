# ROGII - Wellbore Geology Prediction --- Competition Context

> **Purpose:** This document consolidates the Kaggle **Overview** and
> **Data** information provided for the ROGII - Wellbore Geology
> Prediction competition into a clean Markdown reference that can be
> reused across ChatGPT sessions, other LLMs, notebooks, and project
> documentation.
>
> **Source scope:** Only the user-provided Kaggle Overview and Data
> pages are summarized here. No external assumptions or undocumented
> dataset details have been added.

------------------------------------------------------------------------

## 1. Competition Identity

-   **Competition:** ROGII - Wellbore Geology Prediction
-   **Host:** ROGII
-   **Type:** Featured Code Competition
-   **Primary objective:** Build a model that contributes to automating
    drilling operations in the oil and gas industry.
-   **Competition tags:** Multimodal, Geology, Evaluation, Mean Squared
    Error
-   **Official competition citation:** Igor Kuvaev, Rafael Aguilar, John
    Granmayeh, Ryan Holbrook, María Cruz, and Ashley Oldacre. *ROGII -
    Wellbore Geology Prediction*. Kaggle, 2026.

### One-line problem statement

Build machine-learning models to predict the geology encountered along a
horizontal wellbore, with the concrete prediction target being **TVT
(True Vertical Thickness)** in the evaluation zone of each horizontal
well.

------------------------------------------------------------------------

## 2. Problem Context

Drilling a horizontal well can be viewed as navigating underground
without a complete map. The wellbore passes through rock layers whose
geometry cannot be directly observed everywhere.

The competition describes several practical challenges:

-   Roughly **10,000 horizontal wells** are drilled worldwide each year.
-   Much of the drilling process still relies on manual expert
    interpretation.
-   Small deviations from the target geological zone can lead to
    significant resource waste.
-   Drilling into less favorable geology can reduce energy-recovery
    efficiency and may require corrective operations.
-   These corrective measures can increase the environmental footprint
    of a site.
-   Direct subsurface measurements are limited.
-   Wells, seismic surveys, and logging tools provide only partial
    information.
-   Geological layers can bend or break along faults, making the
    drill-bit position within a formation difficult to determine
    precisely.
-   Existing analytical tools may not fully capture the nuance of expert
    interpretation.

The competition therefore asks participants to develop machine-learning
models that can:

1.  Predict geology encountered along a horizontal wellbore.
2.  Identify favorable layers from drilling data.
3.  Help guide well placement more accurately during operations.
4.  Potentially reduce redundant drilling and resource waste.
5.  Improve operational safety by better predicting geological hazards.
6.  Support faster, more consistent, data-driven automation.

The competition's stated vision is that a clearer map beneath the
surface could make every meter of drilling count.

------------------------------------------------------------------------

## 3. Core Machine-Learning Task

### Input data

The competition provides:

-   Horizontal well trajectories.
-   Horizontal-well geological/log data.
-   Vertical reference logs called **Typewells**.
-   Well-path / geological cross-section visualizations for training
    wells.

### Prediction target

The target is:

**`TVT` --- True Vertical Thickness (ft)**

For the horizontal well data, TVT is described as the manually
interpreted geological position for each 1 ft of the lateral well.

The model must predict TVT for the **evaluation zone** where the target
is hidden.

### Important target-related field

The horizontal-well files also contain:

**`TVT_input` --- Input Target (ft)**

This is described as a copy of `TVT` provided as a feature. It contains
`NaN` values in the evaluation zone.

This field is therefore explicitly part of the supplied data
specification and should be inspected carefully when designing
validation and features.

------------------------------------------------------------------------

## 4. Dataset Organization

The dataset is organized into:

``` text
train/
test/
sample_submission.csv
```

Each well is identified by a unique **8-character hash**, for example:

``` text
015fe0d2
```

The competition page states that the dataset contains approximately
**2,327 files** and is approximately **1.33 GB** in size. The listed
file types are:

-   CSV
-   PNG
-   PPTX

------------------------------------------------------------------------

## 5. Training Data

Each training well has three associated files.

### 5.1 `{WELLNAME}__horizontal_well.csv`

This file contains trajectory, geological-surface, and log data.

Fields specified by the competition:

  -----------------------------------------------------------------------
  Field                   Description             Availability
  ----------------------- ----------------------- -----------------------
  `WELLNAME`              Unique identifier for   Train
                          the well                

  `MD`                    Measured Depth (ft):    Train
                          total length of the     
                          wellbore from the       
                          surface                 

  `X`                     Easting (ft): spatial   Train
                          coordinate in the       
                          horizontal plane        

  `Y`                     Northing (ft): spatial  Train
                          coordinate in the       
                          horizontal plane        

  `Z`                     True Vertical Depth     Train
                          (ft): vertical distance 
                          below sea level         

  `ANCC`                  Predicted depth of a    Training only
                          geological formation    

  `ASTNU`                 Predicted depth of a    Training only
                          geological formation    

  `ASTNL`                 Predicted depth of a    Training only
                          geological formation    

  `EGFDU`                 Predicted depth of a    Training only
                          geological formation    

  `EGFDL`                 Predicted depth of a    Training only
                          geological formation    

  `BUDA`                  Predicted depth of a    Training only
                          geological formation    

  `TVT`                   True Vertical Thickness Training only / target
                          (ft); manually          
                          interpreted geological  
                          position                

  `GR`                    Gamma Ray (API); log    Train
                          measuring natural       
                          radioactivity of the    
                          rock                    

  `TVT_input`             Copy of `TVT`; `NaN` in Train
                          evaluation zone         
  -----------------------------------------------------------------------

> **Important:** The competition page explicitly says the
> geological-surface fields `ANCC`, `ASTNU`, `ASTNL`, `EGFDU`, `EGFDL`,
> and `BUDA` are training-only.

### 5.2 `{WELLNAME}__typewell.csv`

This is the vertical reference log used for geological correlation.

  -----------------------------------------------------------------------
  Field                               Description
  ----------------------------------- -----------------------------------
  `TVT`                               Vertical Depth Index (ft), serving
                                      as the primary depth reference for
                                      the vertical log and corresponding
                                      to the TVT geological position of
                                      the associated horizontal well

  `GR`                                Vertical Gamma Ray signature used
                                      for correlation

  `Geology`                           Categorical formation label,
                                      e.g. `EGFDL`, `BUDA`
  -----------------------------------------------------------------------

### 5.3 `{WELLNAME}.png`

A visualization of:

-   The well path.
-   The geological cross-section.

The competition is tagged **Multimodal**, so these images are part of
the published dataset specification. The provided Data page does not, by
itself, specify a required image-modeling approach.

------------------------------------------------------------------------

## 6. Test / Evaluation Data

The competition describes the test set as containing evaluation data for
approximately **200 wells**.

Each test well has two associated files:

### 6.1 `{WELLNAME}__horizontal_well.csv`

Contains:

-   Well trajectory.
-   Log data.

The `TVT` target is hidden by being replaced with `NaN` in the
evaluation zone.

### 6.2 `{WELLNAME}__typewell.csv`

Contains the vertical reference log for the test well.

### Important note about the visible test directory

The competition explicitly warns that the visible `test/` folder
contains only a few instances from the training set as **example data**
to help participants author submissions.

When a submission is rerun against the hidden test set, these example
files are replaced by the actual test data.

Therefore, the visible test directory should **not** automatically be
interpreted as the complete competition test distribution.

------------------------------------------------------------------------

## 7. Sample Submission

The submission file must contain:

``` csv
id,tvt
000d7d20_1442,0.0
000d7d20_1443,0.0
000d7d20_1444,0.0
000d7d20_1445,0.0
...
```

### Columns

  -----------------------------------------------------------------------
  Column                              Description
  ----------------------------------- -----------------------------------
  `id`                                Unique prediction-point identifier
                                      formatted as
                                      `{WELLNAME}_{row_index}`

  `tvt`                               Predicted True Vertical Thickness
                                      (ft)
  -----------------------------------------------------------------------

The competition Overview states that a prediction must be made for each
row in the test set.

The required submission filename is:

``` text
submission.csv
```

------------------------------------------------------------------------

## 8. Evaluation Metric

Submissions are evaluated using **Root Mean Squared Error (RMSE)**.

$$
\mathrm{RMSE}
=
\sqrt{
\frac{1}{n}
\sum_{i=1}^{n}
(y_i-\hat{y}_i)^2
}
$$

where:

-   $\hat{y}$ = predicted value
-   $y$ = original value
-   $n$ = number of rows in the test data

### Optimization direction

Because the metric is RMSE, **lower is better**.

The competition page itself specifies RMSE as the scoring metric; any
additional interpretation of model behavior should be validated
empirically rather than assumed.

------------------------------------------------------------------------

## 9. Submission / Notebook Constraints

Submissions must be made through **Kaggle Notebooks**.

For the Submit button to become active after a commit, the competition
specifies:

-   CPU Notebook runtime: **\<= 9 hours**
-   GPU Notebook runtime: **\<= 9 hours**
-   **Internet access disabled**
-   Freely and publicly available external data is allowed, including
    pre-trained models.
-   Submission file must be named `submission.csv`.

These constraints should be treated as hard engineering constraints when
designing the final pipeline.

------------------------------------------------------------------------

## 10. Competition Timeline

The Overview page contains the following dates:

  Event                                    Date
  ---------------------------------------- ----------------
  Start Date                               May 5, 2026
  Working Note Award deadline (optional)   July 6, 2026
  Entry Deadline                           July 29, 2026
  Team Merger Deadline                     July 29, 2026
  Final Submission Deadline                August 5, 2026

Deadlines are stated as **11:59 PM UTC** on the corresponding day unless
otherwise noted.

> The Overview page also displays a competition start date of May 6,
> 2026 in its timeline UI, while the detailed Timeline section states
> May 5, 2026. This document preserves both source values rather than
> silently reconciling them.

------------------------------------------------------------------------

## 11. Working Note Award

The Working Note Award is optional.

### Eligibility

Teams must be in the **Medal Zone on the public leaderboard** to be
eligible.

### Evaluation criteria

The organizers emphasize five areas.

#### 11.1 Breadth and Depth of Exploration

Meaningfully different approaches are valued, including successful and
unsuccessful approaches.

Approaches are considered distinct when they differ in meaningful ways
such as:

-   Feature set.
-   Modeling strategy.
-   Method for handling spurious correlations.
-   Estimation of incidence angle.
-   Other core methodological choices.

The following are treated as minor variations rather than distinct
approaches:

-   Hyperparameter tuning.
-   Window-size adjustments.
-   Minor variations of the same core idea.

For each meaningful approach, document:

1.  Underlying idea and motivation.
2.  Validation results.
3.  Conclusions and lessons learned.
4.  Why the approach succeeded or failed.

A smaller number of deeply analyzed approaches is preferred to a long
list of superficial experiments. Negative results are explicitly valued
when supported by thoughtful analysis.

#### 11.2 Insights About the Data and Wells

Important observations about the data and wells should be documented,
including:

-   Differences in behavior across wells.
-   Expected and unexpected findings.
-   Insights gained from the public dataset.
-   How the public data guided development.
-   Reasons for applying different methods to different wells, when
    applicable.
-   How the appropriate method for each well was determined.

#### 11.3 Physical Meaningfulness of the Solution

The final solution should be assessed not only by leaderboard
performance but also by whether it provides a physically meaningful
interpretation of the underlying data.

Participants are encouraged to discuss the boundary between:

-   Discovering genuine relationships in the data.
-   Optimizing specifically for the evaluation metric.

The final method should be considered in terms of:

-   Physical plausibility.
-   Robustness.
-   Predictive performance.

#### 11.4 Contribution of Individual Ideas

For each major idea, feature, model component, or methodological
decision:

-   Demonstrate its contribution to the final result.
-   Quantify its impact on validation performance where possible.
-   Show how improvements accumulated throughout development.

#### 11.5 Uncertainty Estimation

The organizers encourage solutions to estimate their own confidence.

A strong submission should discuss:

-   Where predictions are likely to be reliable.
-   Where the method may be uncertain.
-   Where the method may be prone to error.
-   How uncertainty is quantified.
-   How uncertainty is communicated.

------------------------------------------------------------------------

## 12. Prizes

### Main prizes

  Place        Prize
  ------- ----------
  1st       \$25,000
  2nd       \$13,000
  3rd        \$7,000
  4th        \$5,000

### Best Working Note Awards

Two awards are listed:

-   Award 1: \$2,500
-   Award 2: \$2,500

Total listed competition awards: **\$50,000**.

------------------------------------------------------------------------

## 13. Competition Participation Snapshot

The Overview page reports:

-   **16,341 Entrants**
-   **6,956 Participants**
-   **6,125 Teams**
-   **161,975 Submissions**

These values are included as competition-page metadata and may change
depending on when the page is viewed.

------------------------------------------------------------------------

## 14. Key Modeling Implications Explicitly Supported by the Data Specification

The following are direct consequences of the published fields and task
description, not additional assumptions about the hidden dataset.

### 14.1 This is a structured geological sequence / trajectory prediction problem

The horizontal-well data combine:

-   Measured depth (`MD`)
-   Spatial coordinates (`X`, `Y`)
-   Vertical depth (`Z`)
-   Geological surface predictions
-   Gamma Ray (`GR`)
-   Target-related information (`TVT`, `TVT_input`)

Therefore, the data are not simply an independent-row tabular regression
problem.

### 14.2 Well identity and within-well structure matter

Each well has its own:

-   Horizontal trajectory.
-   Typewell reference.
-   Geological context.

The dataset is explicitly organized around individual wells.

Any validation strategy should therefore be designed with the
**well-level structure** in mind rather than blindly treating every row
as independent.

> This is a modeling recommendation based on the dataset organization,
> not an additional competition rule.

### 14.3 The Typewell is a reference source

The Typewell contains:

-   `TVT`
-   `GR`
-   `Geology`

and is explicitly described as a **vertical reference log for geological
correlation**.

This makes the relationship between horizontal-well `GR` and Typewell
`GR` an important aspect to investigate.

The competition data page does not prescribe a specific correlation
algorithm, so any correlation method should be treated as a modeling
hypothesis to validate.

### 14.4 The evaluation zone creates a special prediction setting

`TVT_input` is copied from `TVT` but becomes `NaN` in the evaluation
zone.

This indicates that the prediction problem has an observable portion of
the well and a hidden-target portion.

The exact boundaries and patterns of the evaluation zone should be
established by inspecting the actual CSV files rather than assumed from
the competition description alone.

### 14.5 Training-only geological surfaces require care

`ANCC`, `ASTNU`, `ASTNL`, `EGFDU`, `EGFDL`, and `BUDA` are explicitly
described as training-only.

A final model must not rely on values that are genuinely unavailable for
the hidden evaluation data.

The actual hidden-test availability and exact preprocessing behavior
should be verified from the competition environment rather than inferred
beyond the published specification.

------------------------------------------------------------------------

## 15. Data Science Workflow Suggested by This Context

The competition description supports the following high-level workflow.
This section is a practical organization of the supplied information,
not an official competition recipe.

### Phase 1 --- Data audit

Inspect the actual Kaggle files to determine:

-   Number of wells.
-   Rows per well.
-   Columns and dtypes.
-   Missing-value patterns.
-   `MD` spacing.
-   Evaluation-zone boundaries.
-   Relationship between `TVT` and `TVT_input`.
-   Relationship between horizontal-well and Typewell records.
-   Distribution of geological surfaces.
-   `GR` behavior.
-   Whether all wells share the same schema.
-   Whether there are well-specific anomalies.

### Phase 2 --- Understand the geological relationship

Investigate:

-   Horizontal well trajectory.
-   Typewell `TVT` / `GR` relationship.
-   Horizontal-well `GR` versus Typewell `GR`.
-   Geological formation labels.
-   Relationship between geological surfaces and TVT.
-   Changes along measured depth.

### Phase 3 --- Establish leakage-safe validation

Because rows belong to wells and the hidden test set is organized by
wells, validation should explicitly investigate well-level
generalization.

At minimum, compare:

-   Row-wise random splitting.
-   Well-level splitting.

The competition description alone does not specify the official
train/validation split, so the actual validation policy should be
treated as a modeling decision.

### Phase 4 --- Build a simple baseline

A baseline should be interpretable and easy to debug.

Possible baseline families to investigate include:

-   Within-well interpolation / extrapolation where legitimately
    available.
-   Simple regression using trajectory and log features.
-   Tree-based regression.
-   Typewell correlation features.
-   Sequence/window features.

The competition source does not prescribe any of these models; they are
candidate approaches for experimentation.

### Phase 5 --- Add geological correlation

Investigate how the Typewell can be aligned with the horizontal well
using the supplied `GR`, `TVT`, and `Geology` information.

Potential feature concepts should be treated as hypotheses and validated
rather than assumed to be physically correct.

### Phase 6 --- Sequence-aware modeling

Because the target occurs along the lateral well and the data are
indexed along a trajectory, investigate models that can use local
context:

-   Rolling/window statistics.
-   Local gradients.
-   Neighboring measurements.
-   Sequence models.
-   Models using both local horizontal-well features and
    Typewell-derived context.

### Phase 7 --- Well-specific behavior

Analyze whether different wells exhibit substantially different:

-   GR patterns.
-   Geological relationships.
-   Trajectory geometry.
-   Target behavior.

If well-specific methods are used, document why and how the method is
selected.

### Phase 8 --- Uncertainty

Because the Working Note criteria explicitly value uncertainty
estimation, consider tracking:

-   Prediction intervals or distributions.
-   Ensemble dispersion.
-   Local residual behavior.
-   Well-specific confidence.
-   Regions of the trajectory where the model is unreliable.

### Phase 9 --- Final submission

The final notebook must:

1.  Run within the Kaggle runtime limit.
2.  Work with Internet disabled.
3.  Produce `submission.csv`.
4.  Contain `id` and `tvt`.
5.  Produce one prediction per required test row.

------------------------------------------------------------------------

## 16. Important Things NOT Specified by the Supplied Pages

The Overview and Data pages provided here do **not** fully specify
several implementation details.

Do not assume the following without inspecting the actual competition
files or other official competition materials:

-   Exact number of training wells.
-   Exact number of rows per well.
-   Exact evaluation-zone start/end indices for every well.
-   Exact missing-value distributions.
-   Exact relationship between row index and measured depth.
-   Exact sampling interval in every file.
-   Exact correlation algorithm between horizontal well and Typewell.
-   Exact geological meaning of every formation abbreviation beyond the
    labels shown.
-   Exact leaderboard split construction.
-   Exact hidden-test generation process beyond the statement that
    visible test examples are replaced during submission reruns.
-   Whether PNG information provides additional predictive value.
-   Whether any particular model architecture is expected or preferred.
-   Whether a particular physical equation should be imposed.
-   Whether the training-only geological-surface fields can be
    reconstructed from other available fields.

These should be investigated empirically or confirmed from additional
official competition documentation.

------------------------------------------------------------------------

## 17. LLM / Notebook Instructions

When this document is used as context for another LLM, follow these
principles:

1.  Treat the competition facts in this document as the **official
    source-derived context supplied by the user**.
2.  Do not invent missing dataset details.
3.  Distinguish clearly between:
    -   Official competition facts.
    -   Observations from actual CSV inspection.
    -   Modeling hypotheses.
    -   General machine-learning knowledge.
4.  When proposing features, explicitly check whether the required
    feature exists in the hidden test setting.
5.  Treat `TVT` as the target.
6.  Remember that `TVT_input` becomes `NaN` in the evaluation zone.
7.  Treat `ANCC`, `ASTNU`, `ASTNL`, `EGFDU`, `EGFDL`, and `BUDA` as
    training-only according to the supplied Data specification.
8.  Preserve well-level structure in validation experiments.
9.  Report RMSE for validation.
10. Track negative results, not only successful experiments.
11. Prefer a small number of meaningfully different, well-analyzed
    approaches over many superficial hyperparameter variations.
12. Consider physical plausibility and uncertainty, not only leaderboard
    score.
13. Do not claim that a modeling approach is officially endorsed by
    ROGII unless an official source explicitly says so.
14. If actual CSV files are available, inspect them directly before
    making assumptions about schema, missingness, sampling, alignment,
    or evaluation-zone structure.

------------------------------------------------------------------------

## 18. Compact Reference

``` text
Competition:
  ROGII - Wellbore Geology Prediction

Host:
  ROGII

Task:
  Predict TVT (True Vertical Thickness) in the evaluation zone
  of horizontal wells.

Core data:
  Horizontal well CSV
  Typewell CSV
  Training well PNG visualization

Horizontal well fields:
  WELLNAME
  MD
  X
  Y
  Z
  ANCC
  ASTNU
  ASTNL
  EGFDU
  EGFDL
  BUDA
  TVT
  GR
  TVT_input

Typewell fields:
  TVT
  GR
  Geology

Target:
  TVT

Important target-related feature:
  TVT_input
  -> copy of TVT
  -> NaN in evaluation zone

Evaluation:
  RMSE
  Lower is better

Submission:
  submission.csv
  columns: id, tvt

ID format:
  {WELLNAME}_{row_index}

Notebook constraints:
  CPU <= 9 hours
  GPU <= 9 hours
  Internet disabled
  Public external data / pretrained models allowed
  submission.csv required

Main prize:
  $25,000 / $13,000 / $7,000 / $5,000 for places 1-4

Working Note themes:
  1. Breadth & depth of exploration
  2. Data / well insights
  3. Physical meaningfulness
  4. Contribution of individual ideas
  5. Uncertainty estimation
```

------------------------------------------------------------------------

## 19. Source and Provenance

This context file was created from two user-provided copies of the
Kaggle competition pages:

-   **Overview** --- ROGII - Wellbore Geology Prediction
-   **Data** --- ROGII - Wellbore Geology Prediction

The source material was provided in the conversation on **August 20,
2026**.

No external web research was used to fill gaps in the source material.
