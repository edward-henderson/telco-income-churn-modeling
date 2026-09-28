# Telecom Income & Churn Modeling

Predicting customer churn and estimating customer income from telecom
account and service-usage data, with an emphasis on model-selection
tradeoffs, reproducibility, and the potential value of internal customer
data.

**[Live Tableau
Dashboard](https://public.tableau.com/app/profile/edward.henderson/viz/TelcoIncomeChurnModeling/Dashboard1)**

------------------------------------------------------------------------

## Business Problem

Telecommunications providers collect substantial customer information
through account history, service usage, tenure, and spending. This
project examines two related analytical questions:

1.  **Churn risk:** Can existing customer data identify customers with
    elevated churn risk?
2.  **Income estimation:** Does internal customer data contain enough
    predictive signal to support income-related segmentation and
    potentially reduce reliance on externally purchased demographic
    information for some analytical use cases?

The two questions are modeled separately using the same 1,000-customer
dataset.

## Dataset and Provenance

The analysis uses a course-provided telecom dataset supplied through
Colorado State University Global for a Statistics in Business Analytics
portfolio assignment.

The course materials describe the records as a random sample of
approximately half of an unnamed telecommunications company's customers.
Identifying information was removed for privacy, and the income field
was obtained by the telecommunications company from an outside vendor.
The original telecommunications company, outside income-data vendor,
collection dates, geography, and upstream data provider are not
identified in the available course materials.

-   **1,000 customer records**
-   **34 analytical variables**
-   **No missing values**
-   **Churn rate: 27.4%**
-   Demographic, account, service-usage, and spending variables
-   Targets: `churn` (binary) and `income` (continuous)

The original workbook is **not redistributed in this repository**
because redistribution rights have not been established.

## Analytical Approach

### Churn Classification

-   Logistic regression compared with random forest classification.
-   75/25 stratified train/test split with `random_state=42`.
-   Logistic-regression scaling fitted on training data only.
-   Accuracy, precision, recall, F1, ROC-AUC, and confusion matrices
    evaluated.
-   Majority-class baseline included because 72.6% of customers did not
    churn.

### Income Regression

The analysis compares linear regression on raw income, linear regression
using a log-transformed income target, and random forest regression.
`income` and the source dataset's precomputed `lninc` field are excluded
from the predictor matrix to prevent target leakage.

## Verified Results

### Churn Classification

  Model                        Accuracy   Precision      Recall          F1     ROC-AUC
  ------------------------- ----------- ----------- ----------- ----------- -----------
  Majority-class baseline         72.6%         ---        0.0%         ---       0.500
  Logistic Regression             73.2%       51.0%   **38.2%**       43.7%   **0.790**
  Random Forest               **75.6%**   **58.5%**       35.3%   **44.0%**       0.776

Random forest achieved higher accuracy and precision. Logistic
regression achieved higher recall and ROC-AUC and identified 26 churners
compared with 24 for random forest. Logistic regression is treated as
the preferred churn model when the decision objective emphasizes churn
detection, discrimination, interpretability, and more stable
generalization.

Random-forest feature importance identified tenure, long-distance
measures, and equipment-related measures among the important predictive
signals. These are predictive associations, not causal churn drivers.

### Income Regression

  -------------------------------------------------------------------------
  Model                            R²                MAE               RMSE
  ---------------- ------------------ ------------------ ------------------
  Linear                        0.442              52.34              97.77
  Regression ---                                         
  raw income                                             

  Linear                    **0.549**          **42.00**          **87.87**
  Regression ---                                         
  log target,                                            
  original-scale                                         
  evaluation                                             

  Random Forest                 0.335              49.42             106.68
  --- raw income                                         
  -------------------------------------------------------------------------

The log-target linear model produced the strongest held-out performance
of the tested implementations across R², MAE, and RMSE. Years employed
and education emerged as important income predictors, particularly in
the random-forest model.

These results indicate meaningful predictive signal. They do **not**
establish that internally estimated income can completely replace
externally sourced customer-level income data.

## Model Validation and Limitations

Core numerical results were independently reproduced from the preserved
source workbook. Important limitations include:

-   **Class imbalance:** churn prevalence is 27.4%; accuracy should not
    be interpreted alone.
-   **Modest churn recall:** logistic recall is 38.2% at the implemented
    threshold.
-   **Random-forest overfitting:** the churn random forest shows a
    material train/test performance gap.
-   **Single holdout split:** results rely on one 75/25 split rather
    than repeated cross-validation.
-   **No class rebalancing or threshold optimization:** these were not
    part of the validated implementation.
-   **High-income error behavior:** errors increase toward the upper
    tail, limiting precise individual estimates.
-   **Log retransformation:** direct exponentiation is used without a
    retransformation-bias correction.
-   **Predictive, not causal:** model importance and coefficients do not
    establish causal effects.
-   **Scope:** results describe this dataset and these implementations,
    not demonstrated production performance elsewhere.

## Repository Structure

```text
├── dashboard/
│   └── .gitkeep
├── data/
│   └── DATA_NOTE.md
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_churn_model.ipynb
│   └── 03_income_model.ipynb
├── src/
├── .gitignore
├── requirements.txt
└── README.md
```

The Tableau workbook is published separately through the live dashboard link above and is not distributed in this repository. The `src/` directory is currently reserved for future reusable code; the validated analysis is implemented in the three notebooks.

## Tech Stack

Python · pandas · NumPy · scikit-learn · matplotlib · seaborn · openpyxl
· Jupyter · Tableau

The project originated in CSU Global coursework and was subsequently
developed into a Python-based analytics case study.

## Data Access and Reproduction

The source workbook is not included in the public repository. Users who
independently have authorized access can reproduce the analysis by
placing an unchanged copy in the local data area.

Recommended convention:

``` text
data/
└── raw/
    └── TelcoExtraCSU-Global.xlsx
```

If the local filename differs, update the notebook data path
accordingly. Do not commit the source workbook to the public repository.

Install dependencies:

``` bash
pip install -r requirements.txt
```

Run notebooks in order:

``` text
1. notebooks/01_data_exploration.ipynb
2. notebooks/02_churn_model.ipynb
3. notebooks/03_income_model.ipynb
```

### Reproducibility Checkpoints

A successful run should reproduce approximately:

-   Dataset: **1,000 × 34**, with **0 missing values**
-   Churn rate: **27.4%**
-   Majority baseline: **72.6% accuracy**
-   Logistic: **73.2% accuracy, 38.2% recall, ROC-AUC 0.790**
-   Churn RF: **75.6% accuracy, 35.3% recall, ROC-AUC 0.776**
-   Log-target linear income: **R² 0.549, MAE 42.00, RMSE 87.87**
-   Income RF: **R² 0.335, MAE 49.42, RMSE 106.68**

## Appropriate Interpretation

This case study supports the conclusion that existing customer data
contains useful predictive signal for churn risk and income-related
patterns in this dataset. It also illustrates that increased model
complexity did not automatically produce better out-of-sample
performance.

It should **not** be interpreted as demonstrating causal churn drivers,
precise individual income estimation, complete replacement of purchased
demographic data, production-ready performance, or universal superiority
of one algorithm.

## Future Model Development

Potential future work includes cross-validation, class weighting or
other imbalance treatments, decision-threshold optimization, alternative
boosting models, retransformation-bias correction, and additional
segmentation analysis. These are future development opportunities, not
part of the validated implementation reported here.

------------------------------------------------------------------------

**Edward Henderson** \|
[LinkedIn](https://linkedin.com/in/EdwardOHenderson) \|
edward@edwardhenderson.net
