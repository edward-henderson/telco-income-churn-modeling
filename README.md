# Telecom Income & Churn Modeling

Predicting customer churn and estimating household income from service-usage data for a telecommunications provider — reducing reliance on costly third-party demographic data while identifying customers at risk of leaving.

**[Live Dashboard →](#)** *(Tableau Public link goes here)*

---

## Business Problem

Telecom providers face two related challenges:

1. **Churn risk** — identifying which customers are likely to cancel service, so retention offers can be targeted before they leave rather than after.
2. **Income estimation** — third-party demographic and income data is expensive to license. If a company's own service-usage and account data (tenure, spend, equipment, service mix) can estimate income reasonably well, it reduces dependence on external data providers for segmentation and marketing decisions.

This project tackles both using the same 1,000-customer dataset, treating them as two separate modeling problems that share a feature set.

## Dataset

- **1,000 customers**, **34 variables**, no missing values
- Demographics: age, marital status, address tenure, education, employment, retirement status, gender, household size
- Account/usage: service tenure, region, customer category, and monthly spend/usage across long-distance, toll-free, equipment, card, and wireless services
- Service flags: multi-line, voicemail, internet, caller ID, call waiting, call forwarding, conferencing, e-billing
- Targets: `income` (continuous) and `churn` (binary)

*Note: this is course-provided sample data used for a Colorado State University Global capstone project; the raw file is not redistributed here — see [Data Access](#data-access) below.*

**Churn rate:** 27.4% (a class imbalance that matters for model evaluation — see [Approach](#approach))

## Approach

### Churn (classification)
- Baseline: always predicting "no churn" gets 72.6% accuracy — so accuracy alone is a misleading metric here. Precision, recall, and ROC-AUC matter more.
- Compared logistic regression against a random forest classifier.
- Evaluated with an 75/25 stratified train/test split to preserve the churn class ratio.

### Income (regression)
- Compared linear regression against a random forest regressor.
- Income is heavily right-skewed (median $47K, mean $77.5K, max $1,668K), so I also tested a **log-transformed target**, which is standard practice for skewed financial variables.
- Excluded `lninc` (a pre-computed log-income field) from the feature set to avoid target leakage.

## Results

### Churn Classification

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|
| Baseline (majority class) | 72.6% | — | — | — | — |
| Logistic Regression | 73.2% | 0.51 | 0.38 | 0.44 | **0.79** |
| Random Forest | **75.6%** | **0.59** | 0.35 | 0.44 | 0.78 |

Logistic regression edges out random forest on ROC-AUC and recall, meaning it's slightly better at *catching* customers who are actually going to churn — arguably the more important error to avoid in a retention context, since missing an at-risk customer costs more than a wasted retention offer.

**Top predictors of churn:** tenure, long-distance monthly spend, equipment monthly spend, and employment length — newer, lower-usage, less-established customers churn more.

### Income Regression

| Model | R² | MAE | RMSE |
|---|---|---|---|
| Linear Regression (raw income) | 0.44 | $52.3K | $97.8K |
| Linear Regression (log income) | **0.55** | **$42.1K** | — |
| Random Forest (raw income) | 0.33 | $49.6K | $107.2K |

Log-transforming the target improved R² by ~25% and cut average error by $10K — the single biggest lever in this half of the project, and a good example of why understanding your target's distribution matters before picking a model.

**Top predictors of income:** years employed, education level, card-service monthly spend, and address tenure.

## Repo Structure

```
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_churn_model.ipynb
│   └── 03_income_model.ipynb
├── src/
│   └── modeling.py
├── dashboard/
│   └── telco_dashboard.twbx
├── requirements.txt
└── README.md
```

## Tech Stack

Python (pandas, scikit-learn) · Tableau · originally prototyped in SAS Studio as part of a CSU Global capstone

## Data Access

The dataset is provided through a Colorado State University Global course and is not included in this repo. A data dictionary and sample rows are available on request — reach out via [LinkedIn](https://linkedin.com/in/EdwardOHenderson).

## Next Steps

- Tune random forest hyperparameters and test gradient boosting (XGBoost) for both tasks
- Address class imbalance in the churn model (SMOTE or class weighting) to improve recall
- Build a customer segmentation layer (`custcat`) into the dashboard for marketing use cases

---

**Edward Henderson** | [LinkedIn](https://linkedin.com/in/EdwardOHenderson) | edward@edwardhenderson.net
