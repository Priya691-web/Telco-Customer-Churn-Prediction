# 📊 Telecom Customer Churn Prediction & Retention Analytics

> **End-to-End Machine Learning & Business Intelligence Portfolio Project**  
> *Role:* Junior Data Analyst | *Domain:* Telecommunications & Subscription Economics

---

## 🎯 1. Business Problem

### What is Customer Churn?
Customer churn occurs when existing subscribers cancel their service. In this dataset, `Churn = Yes` identifies accounts that terminated service in the preceding month.

### Why Does Churn Matter Financially?
1. **Acquisition vs. Retention Cost:** Acquiring a new telecom subscriber via marketing, sales commissions, and hardware setup costs **5x to 7x more** than retaining an existing subscriber.
2. **Recurring Revenue Loss:** Subscription models depend on steady monthly cash flow. When high-paying subscribers cancel, that revenue disappears immediately while fixed network infrastructure costs remain unchanged.

### The Analytical Objective
> **"Identify currently active customers who are at high risk of canceling, uncover root causes driving their dissatisfaction, and build an out-of-fold risk scoring system for the retention team to protect active monthly recurring revenue."**

---

## 📁 2. Dataset Overview

- **Source:** IBM Sample Telco Customer Churn Dataset (Kaggle)
- **File Name:** `WA_Fn-UseC_-Telco-Customer-Churn.csv`
- **Volume:** 7,043 subscriber records, 21 attributes
- **Target Variable:** `Churn` (`Yes` = 1,869 | `No` = 5,174)
- **Baseline Churn Rate:** **26.54%** (Class Imbalance Ratio ≈ 2.77 : 1)

### Feature Breakdown
| Category | Attributes |
| :--- | :--- |
| **Demographics** | `gender`, `SeniorCitizen`, `Partner`, `Dependents` |
| **Account Info** | `tenure`, `Contract`, `PaperlessBilling`, `PaymentMethod`, `MonthlyCharges`, `TotalCharges` |
| **Phone Services** | `PhoneService`, `MultipleLines` |
| **Internet & Add-Ons** | `InternetService`, `OnlineSecurity`, `OnlineBackup`, `DeviceProtection`, `TechSupport`, `StreamingTV`, `StreamingMovies` |
| **Target** | `Churn` |

---

## 🛠️ 3. Tools & Technologies

- **Programming:** Python 3.14 (`pandas`, `numpy`)
- **Data Visualization:** `matplotlib`, `seaborn`
- **Machine Learning & CV:** `scikit-learn` (`StratifiedKFold`, `Pipeline`, `StandardScaler`, `LogisticRegression`, `RandomForestClassifier`), `xgboost` (`XGBClassifier`)
- **Environment:** Jupyter Notebook (`telco_churn_prediction.ipynb`)
- **Business Intelligence:** Microsoft Power BI Desktop
- **Version Control:** Git & GitHub

---

## 🔬 4. Methodological Rigor & Analytical Decisions

### 1. Data Cleaning: The Tenure-Aware Imputation
`TotalCharges` contained 11 blank spaces (`' '`). Rather than dropping them, inspecting those records revealed that **all 11 had `tenure == 0`** (brand-new accounts who enrolled in the current billing cycle and had not yet been billed). We imputed `0.0`, preserving 100% of our customer records.

### 2. Feature Engineering
- `TenureGroup`: Binned customer tenure (`0-12m`, `13-24m`, `25-48m`, `49-72m`) to capture the exponential drop in churn after year one.
- `TotalServices`: Counter (0 to 6) summing active add-on features (`OnlineSecurity`, `TechSupport`, etc.). Subscribers with 0 add-ons churn at **43.99%**, while those with 4+ add-ons churn at **under 11%** (proving product stickiness).
- `MonthlyToTotalRatio`: Ratio of current monthly bill to cumulative historical charges to detect early-stage bill shock.

### 3. Out-of-Fold (OOF) Scoring & Leakage Prevention
Scoring customers with a model trained on those same customers produces in-sample overconfidence. We used **5-Fold Stratified Cross-Validation (`cross_val_predict`)** so that every single customer in the dataset received an out-of-fold probability from a model that **did not train on them**. Feature scaling was contained strictly inside each CV fold via `Pipeline([('scaler', StandardScaler()), ...])` to eliminate data leakage.

### 4. Ranking-Based Risk Tiers (Why Ranking Over Thresholds)
Class-weighted models intentionally shift raw probabilities upward to prioritize recall. Because class-weighted probabilities are uncalibrated for absolute percentages, they are ideal for **relative ranking**. We ranked all subscribers into three operational tiers based on OOF probability percentiles:
- **High Risk:** Top 20% (Percentile $\ge$ 80%)
- **Medium Risk:** Next 30% (Percentile 50% to 79%)
- **Low Risk:** Bottom 50% (Percentile < 50%)

---

## 📈 5. Key Exploratory Insights (Real Numbers)

| Dimension | Key Empirical Finding | Real Metric |
| :--- | :--- | :--- |
| **Overall Baseline** | Significant baseline attrition requiring proactive intervention | **26.54%** (1,869 / 7,043 churned) |
| **Contract Type** | Month-to-month contracts are the #1 driver of customer flight | **42.71% churn** vs. 11.27% (1-Yr) & 2.83% (2-Yr) |
| **Customer Tenure** | Attrition is heavily concentrated in the first year | Median churn tenure = **10.0 mos** vs. 38.0 mos retained |
| **Monthly Charges** | Churned customers pay substantially higher monthly bills | Median **\$79.65/mo** churned vs. \$64.43/mo retained |
| **Internet Service** | Fiber optic customers exhibit massive dissatisfaction | **41.89% churn** (1,297 / 3,096 fiber users) |
| **Payment Method** | Manual payment method gives recurring cancellation triggers | **45.29% churn** on Electronic check vs. ~15% on Auto-pay |
| **Service Stickiness** | Adding bundled services dramatically raises switching barriers | **43.99% churn** (0 add-ons) vs. **10.76%** (4+ add-ons) |

---

## 🤖 6. Honest Model Comparison (5-Fold Cross-Validation)

Rather than evaluating an arbitrary single train/test split, all three models were evaluated using **5-Fold Stratified Cross-Validation**. Performance is reported as **Mean ± Standard Deviation**:

| Model | ROC-AUC (Mean ± Std) | Recall (Mean ± Std) | Precision (Mean ± Std) | Accuracy (Mean ± Std) |
| :--- | :---: | :---: | :---: | :---: |
| **Logistic Regression (Balanced)** | **0.8458 ± 0.0120** | **80.20% ± 1.95%** | 51.32% ± 1.00% | 74.56% ± 0.81% |
| **Random Forest (Balanced)** | **0.8439 ± 0.0125** | **78.44% ± 1.96%** | 52.86% ± 1.46% | 75.71% ± 1.11% |
| **XGBoost Classifier** | **0.8462 ± 0.0113** | **79.83% ± 0.82%** | 51.85% ± 1.28% | 74.95% ± 1.01% |

### 🔍 Honest Assessment: No Single "Champion" Model
- The ROC-AUC gap across all three models is only **0.0023** (0.8439 to 0.8462), which is **far smaller than the standard deviation of ±0.012**.
- All three models achieve **~79%–80% Recall** and **~51%–53% Precision**.
- **Conclusion:** All three models perform comparably within 1 standard deviation. The tabular signals in this dataset (contracts, tenure, monthly charges) have strong linear components that Logistic Regression captures just as well as complex gradient boosted trees. In production, Logistic Regression offers ultra-low latency and total interpretability, while tree models validate that non-linear feature interactions do not substantially shift the boundary.

---

## 🎯 7. Precision & Business Cost-Benefit Analysis

### What Does ~51% Precision Mean?
Out of every 100 subscribers our model flags as high-risk, **51 are actual potential churners (True Positives)**, and **49 are loyal customers (False Positives)**.

### Why is 51% Precision Acceptable in Churn Retention?
In subscription businesses, the financial penalty of a **False Negative** (losing a customer who pays ~$74/month = ~$890/year) is severe. In contrast, the penalty of a **False Positive** (offering a loyalty discount or perk to a customer who wasn't going to leave) is minor.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ 💰 SIMPLE COST-BENEFIT EXAMPLE (Clearly Labeled Assumptions)                           │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ ASSUMPTION 1: The retention team targets 100 model-flagged subscribers.                │
│ ASSUMPTION 2: At 51% precision, 51 are actual churners and 49 are loyal (false alerts).│
│ ASSUMPTION 3: The company offers a $20 retention incentive per customer.               │
│               Total Campaign Cost = 100 × $20 = $2,000.                                │
│ ASSUMPTION 4: The retention offer successfully convinces 40% of real churners to stay  │
│               (40% of 51 ≈ 20 customers saved).                                        │
│ ASSUMPTION 5: Average annual revenue of a retained subscriber = $74.44 × 12 = $893.28. │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ 💵 FINANCIAL OUTCOME:                                                                  │
│ - Annual Revenue Saved: 20 saved customers × $893.28 = $17,865.60                      │
│ - Total Campaign Cost: $2,000.00                                                       │
│ - NET FINANCIAL RETURN: $17,865.60 - $2,000.00 = +$15,865.60                           │
│ - CAMPAIGN ROI: +793% (Nearly 8x Return on Retention Spend)                            │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 💰 8. Revenue Analysis: Revenue at Risk vs. Revenue Already Lost

A critical distinction: **Customers who already churned cannot have revenue "at risk" — their revenue is already lost.** Revenue at risk applies strictly to active customers who are currently paying.

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────┐
│ 📊 OVERALL PORTFOLIO (7,043 Customers): $456,116.60 / month ($5,473,399.20 / year)              │
├─────────────────────────────────────────────────────────────────────────────────────────────────┤
│ 🔴 REVENUE ALREADY LOST (Churned Customers: Churn = Yes)                                        │
│    - 1,869 customers (26.54% of base) have already terminated service                          │
│    - Monthly Revenue Already Lost:    $139,130.85 / month                                       │
│    - Annualized Revenue Already Lost: $1,669,570.20 / year ($1.67M / yr)                        │
├─────────────────────────────────────────────────────────────────────────────────────────────────┤
│ 🛡️ REVENUE AT RISK (Active Customers Only: Churn = No)                                          │
│    - 5,174 active, paying subscribers (73.46% of base)                                          │
│    - Total Active Monthly Revenue:    $316,985.75 / month ($3,803,829.00 / year)                │
└─────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Active Customer Breakdown by Risk Tier
| Risk Tier | Active Customer Count | % of Active Base | Monthly Revenue at Risk | % of Active Revenue | Annualized Revenue | Actual Tier Churn Rate (Validation) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| 🔴 **High Risk (Top 20%)** | **469** | 9.1% | **\$36,329.65 / mo** | 11.5% | **\$435,955.80 / yr** | **66.71%** (940 / 1,409 churned) |
| 🟡 **Medium Risk (Next 30%)**| **1,406** | 27.2% | **\$101,001.00 / mo** | 31.9% | **\$1,212,012.00 / yr**| **33.46%** (707 / 2,113 churned) |
| 🟢 **Low Risk (Bottom 50%)** | **3,299** | 63.8% | **\$179,655.10 / mo** | 56.7% | **\$2,155,861.20 / yr**| **6.31%** (222 / 3,521 churned) |
| **Active Total** | **5,174** | 100.0% | **\$316,985.75 / mo** | 100.0% | **\$3,803,829.00 / yr**| — |

> **Monotonic Validation:** The actual churn rate drops monotonically from **66.71% in High Risk** down to **6.31% in Low Risk** (a **10x difference**), proving that our out-of-fold ranking effectively separates customer risk.

---

## 💡 9. Strategic Business Recommendations

1. **Targeted Outreach to the 469 High-Risk Active Subscribers:**
   - *Target:* The 469 active customers sitting in the High Risk tier representing **\$36,329.65 in monthly revenue (\$435.9K/year)**.
   - *Action:* Offer a \$10/month 1-year contract incentive to month-to-month accounts. Converting 25% of this group saves **~\$9,000/month (~$108,000/year)**.
2. **First-Year Onboarding (`tenure <= 12m`):**
   - *Target:* New subscribers entering billing cycles 1 through 4 where churn peaks at 47.7%.
   - *Action:* Deploy automated check-ins at Days 30 and 60, dedicated onboarding tutorials, and transparent bill explanations to mitigate early bill shock.
3. **Fiber Optic Service Quality & Value Audit:**
   - *Target:* 3,096 fiber optic subscribers paying >\$70/month who churn at 41.89%.
   - *Action:* Audit regional line latency and bundle complimentary `OnlineSecurity` and `TechSupport` to raise switching barriers.
4. **Auto-Pay Migration Incentive:**
   - *Target:* 2,365 customers using electronic checks (45.29% churn rate).
   - *Action:* Offer a one-time \$25 bill credit for switching to automated bank debit or credit card payments, moving customers toward the ~15% auto-pay churn baseline.

---

## 🚀 10. How to Run This Project

### Prerequisites
```bash
pip install pandas numpy matplotlib seaborn scikit-learn xgboost jupyter nbformat
```

### Running the Jupyter Notebook
```bash
jupyter notebook telco_churn_prediction.ipynb
```
Running all cells executes the 5-fold cross-validation pipeline, generates the out-of-fold risk tiers, displays all charts, and exports `telco_churn_predictions.csv`.

### Power BI Dashboard
- Import [`telco_churn_predictions.csv`](./telco_churn_predictions.csv) into Power BI Desktop.
- Follow [`POWER_BI_DASHBOARD_PLAN.md`](./POWER_BI_DASHBOARD_PLAN.md) to set up KPI cards, DAX measures, and interactive risk tier tables.
