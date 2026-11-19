# Telco Customer Churn Prediction

I built this project to learn how a business can find customers who are likely to cancel, and what it can do about it. I used the IBM Telco Customer Churn dataset, did the analysis in Python, and scored customers by risk so a retention team knows who to contact first.

## The problem

A telecom company loses money every time a customer cancels, and winning a new customer usually costs more than keeping an existing one. So the questions I wanted to answer were:

- Who is leaving, and what do they have in common?
- Can we rank currently active customers by how likely they are to leave?
- What could the company do about it?

## Dataset

- IBM Telco Customer Churn dataset (Kaggle), file `WA_Fn-UseC_-Telco-Customer-Churn.csv`
- 7,043 customers, 21 columns
- Target column: `Churn` (1,869 left, 5,174 stayed), an overall churn rate of 26.54%
- Columns cover demographics, account details (contract, tenure, payment method, charges) and the services each customer has

## Tools

Python (pandas, numpy, matplotlib, seaborn, scikit-learn, XGBoost), Jupyter Notebook, Power BI, Git/GitHub

## What I did

**1. Cleaning.** `TotalCharges` had 11 blank values. All 11 were customers with tenure 0, meaning they had just joined and had not been billed yet, so I set their total to 0 instead of deleting them.

**2. Exploring the data.** I looked at churn across contract type, tenure, charges, internet service and payment method (results below).

**3. New features.** I added three columns:
- `TenureGroup`: tenure bucketed into 0-12, 13-24, 25-48 and 49-72 months
- `TotalServices`: how many add-on services a customer has (security, tech support, etc.)
- `MonthlyToTotalRatio`: current monthly bill compared with total charges so far

**4. Models.** I trained Logistic Regression, Random Forest and XGBoost. Only about a quarter of customers churn, so I used class weights to stop the models from ignoring the churners. I compared the models with 5-fold stratified cross-validation, with scaling done inside each fold so nothing leaks from the test data.

**5. Risk scoring.** Using cross-validated predictions (so no customer is scored by a model that trained on them), I ranked all customers by risk: top 20% High, next 30% Medium, bottom 50% Low.

## What I found

| What I looked at | Result |
| :--- | :--- |
| Overall churn | 26.54% (1,869 of 7,043) |
| Contract type | Month-to-month 42.71%, one-year 11.27%, two-year 2.83% |
| Tenure | Customers who left had a median tenure of 10 months, against 38 for those who stayed. First-year customers churned at 47.68% |
| Monthly charges | Median $79.65 for churned customers vs $64.43 for retained |
| Internet service | Fiber optic customers churned at 41.89% (1,297 of 3,096) |
| Payment method | Electronic check 45.29%, versus about 15% for automatic card or bank payments |
| Add-on services | 43.99% churn with no add-ons, 10.76% with four or more |

## Model comparison

| Model | ROC-AUC | Recall | Precision | Accuracy |
| :--- | :---: | :---: | :---: | :---: |
| Logistic Regression | 0.8458 ± 0.0120 | 80.20% ± 1.95% | 51.32% ± 1.00% | 74.56% ± 0.81% |
| Random Forest | 0.8439 ± 0.0125 | 78.44% ± 1.96% | 52.86% ± 1.46% | 75.71% ± 1.11% |
| XGBoost | 0.8462 ± 0.0113 | 79.83% ± 0.82% | 51.85% ± 1.28% | 74.95% ± 1.01% |

(Mean ± standard deviation across 5 folds.)

The three models perform almost the same. The AUC gap between best and worst is 0.0023, much smaller than the fold-to-fold variation, so I can't say one is clearly better. A simple Logistic Regression does just as well here and is easier to explain.

I focused on recall because missing a customer who is about to leave costs more than offering a discount to someone who would have stayed anyway. The price of that is precision of about 51%: roughly half of the customers the model flags would not actually have left.

## Risk tiers and revenue

Customers who already left can't be "at risk", so I split revenue into two parts:

- Revenue already lost (churned customers): $139,130.85 per month
- Revenue from active customers: $316,985.75 per month across 5,174 customers

| Risk tier | Active customers | Monthly revenue | Actual churn rate in tier |
| :--- | :---: | :---: | :---: |
| High (top 20%) | 469 | $36,329.65 | 66.71% |
| Medium (next 30%) | 1,406 | $101,001.00 | 33.46% |
| Low (bottom 50%) | 3,299 | $179,655.10 | 6.31% |

The actual churn rate falls from 66.71% in the High tier to 6.31% in the Low tier, so the ranking does separate risky customers from safe ones.

## Recommendations

1. **Contract offers for the 469 high-risk active customers.** Most churn is on month-to-month plans, so offer a discount for moving to a one-year contract.
2. **Check in with new customers.** Churn is highest in the first year, so reach out around days 30 and 60.
3. **Look at fiber optic service.** Fiber customers churn far more than DSL customers. Check service quality and consider bundling tech support or online security.
4. **Move people off electronic check.** Offer a small bill credit for switching to automatic payment.

## Limitations

- The dataset is a one-time snapshot. The model learns which customers look like past churners and ranks active customers by that similarity. It does not predict exactly who will leave.
- Class weights push the predicted probabilities up, so use them for ranking, not as exact percentages.
- I haven't tested how well the retention offers work. That would need a real campaign or A/B test.

## How to run

```bash
pip install pandas numpy matplotlib seaborn scikit-learn xgboost jupyter
jupyter notebook telco_churn_prediction.ipynb
```

Running all cells reproduces the charts and exports `telco_churn_predictions.csv`, which I use as the data source for the Power BI dashboard (see `POWER_BI_DASHBOARD_PLAN.md`).

## Files

- `telco_churn_prediction.ipynb`: full analysis
- `telco_churn_predictions.csv`: customers with risk scores and tiers
- `POWER_BI_DASHBOARD_PLAN.md`: dashboard layout and measures
- `images/`: charts
