# Do you really need to buy customer income data? Probably not.

*How I used a telecom company's own usage data to predict both churn risk and household income — without a single third-party data purchase.*

## The problem

Telecom companies sit on a goldmine of behavioral data — tenure, spend, service mix, usage patterns — but often still pay for third-party demographic data to segment customers by income. At the same time, they're losing customers they never saw coming: in the dataset I worked with, 27.4% of customers had churned, and there was no system flagging who was at risk before they left.

I set out to answer two questions with the same dataset: **can internal usage data replace expensive external income data**, and **can it also predict who's about to leave**?

## The approach

I worked with 1,000 customer records and 34 variables spanning demographics, account tenure, and service usage. No missing data, which made this more about modeling judgment than data cleaning.

For churn, I compared logistic regression against a random forest classifier, evaluated on a held-out 25% test set. The churn rate meant accuracy alone would be misleading — a model that predicted "no churn" for everyone would already be right 72.6% of the time — so I leaned on precision, recall, and ROC-AUC instead.

For income, I noticed the target was heavily right-skewed (median $47K, but a handful of high earners pulling the mean to $77.5K). Rather than model raw income directly, I tested a log-transformed target — standard practice for skewed financial data, but easy to skip if you don't stop to look at the distribution first.

## What I found

**Churn:** logistic regression and random forest landed close on accuracy (73.2% vs 75.6%), but logistic regression won on ROC-AUC (0.79 vs 0.78) and recall (0.38 vs 0.35). In a retention context, missing a customer who's about to leave is usually more expensive than a wasted retention offer — so I recommended logistic regression despite its lower headline accuracy. Tenure, long-distance spend, and equipment spend were the strongest predictors: newer, lower-usage customers were the highest churn risk.

**Income:** the log-transform mattered more than the choice of algorithm. Linear regression on raw income explained 44% of the variance (R² = 0.44); the same model on log-transformed income, converted back to dollars, explained 55% — a meaningful jump from one modeling decision. Years employed and education level were the strongest predictors, which tracks with intuition but was worth confirming rather than assuming.

## Why it matters

Neither model is "solved" — an R² of 0.55 means there's still real error in income estimates, and a churn recall of 0.38 means plenty of at-risk customers still slip through. But that's not really the point. The finding that matters for the business is directional: **usage and account data carry real signal for both problems**, enough to justify building this as a standing internal tool rather than continuing to buy external income data or relying on gut-feel retention flags.

## What I'd do next

- Address the churn model's class imbalance directly (class weighting or SMOTE) to push recall higher — right now it's catching barely a third of actual churners.
- Test gradient boosting (XGBoost) against both baselines.
- Build the customer segmentation (`custcat`) into the dashboard so a retention team can filter by service tier, since churn rate varied from 15.7% to 37.3% across segments — that gap alone is worth a targeted campaign.

*Full code and an interactive dashboard breaking down churn risk and income estimates by customer segment: [GitHub repo] · [Tableau dashboard]*

---
*Edward Henderson | [LinkedIn](https://linkedin.com/in/EdwardOHenderson)*
