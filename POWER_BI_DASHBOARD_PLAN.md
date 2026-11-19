# 📊 Power BI Dashboard Plan: Telco Customer Retention & Risk Intelligence

> **Dataset:** `telco_churn_predictions.csv` (7,043 rows, 18 columns)  
> **Key Improvement:** Explicit separation between **Revenue Already Lost** (past churners) and **Revenue at Risk** (active paying subscribers).

---

## 🎨 1. Dashboard Architecture & Visual Wireframe

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ 📞 TELCO CUSTOMER CHURN & REVENUE RISK INTELLIGENCE DASHBOARD                                          │
├───────────────────┬───────────────────┬───────────────────┬───────────────────┬────────────────────────┤
│ Total Customers   │ Revenue Lost (Mo) │ Active Customers  │ High-Risk Active  │ Active Rev at Risk(Mo) │
│     7,043         │    $139,131 / mo  │      5,174        │    469 Accounts   │     $36,330 / mo       │
│ ($456.1K MRR)     │ ($1.67M Annual)   │  ($317.0K MRR)    │  (9.1% of Active) │  ($436.0K Annual)      │
├───────────────────┴───────────────────┴───────────────────┴───────────────────┴────────────────────────┤
│ SLICERS / FILTERS: [Contract Type: All ▼] [Internet: All ▼] [Payment: All ▼] [Risk Tier: All ▼]        │
├─────────────────────────────────────┬─────────────────────────────────────┬────────────────────────────┤
│ VISUAL 1: Churn by Contract Type    │ VISUAL 2: Churn by Tenure Cohort    │ VISUAL 3: Churn by         │
│ (100% Stacked Bar Chart)            │ (Clustered Column Chart)            │ Payment Method             │
│ Month-to-Month: 42.7% Churn         │ 0-12m: 47.7% Churn                  │ Electronic Check: 45.3%    │
│ One-Year:       11.3% Churn         │ 13-24m: 28.7% Churn                 │ Mail Check:       19.1%    │
│ Two-Year:        2.8% Churn         │ 49-72m:  9.5% Churn                 │ Credit Card:      15.2%    │
├─────────────────────────────────────┴─────────────────────────────────────┴────────────────────────────┤
│ VISUAL 4: Product Stickiness (Churn Rate vs Number of Add-On Services: 0 to 6)                         │
│ 0 Services: 44.0% Churn  ──>  2 Services: 21.7% Churn  ──>  4+ Services: 10.8% Churn                  │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ VISUAL 5: Priority Retention Worklist (Active High-Risk Accounts: Churn = No & Risk Tier = High Risk) │
│ CustomerID │ Tenure │ Contract       │ Monthly Charges │ Churn Prob (OOF)  │ Risk Tier   │ Action  │
│ 7590-VHVEG │  1 mo  │ Month-to-month │     $29.85      │       88.9%       │ 🔴 High Risk │ Call    │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 📥 2. Data Ingestion & Setup in Power BI Desktop

1. Open **Power BI Desktop**.
2. Click **Get Data -> Text/CSV** $\rightarrow$ select `telco_churn_predictions.csv`.
3. In **Power Query Editor**, verify data types:
   - `customerID`, `risk_tier`, `predicted_churn_oof`, `Churn`, `Contract`, `InternetService`, `PaymentMethod`: **Text**
   - `tenure`, `TotalServices`: **Whole Number**
   - `churn_probability_oof`, `MonthlyCharges`, `TotalCharges`: **Decimal Number**
4. Click **Close & Apply**.

---

## 🧮 3. Corrected DAX Formulas to Create

Create a dedicated table called `_Measures` or add these directly into `telco_churn_predictions`:

### 1. Total Customer Base
```dax
Total Customers = COUNTROWS('telco_churn_predictions')
```

### 2. Total Portfolio Monthly Revenue (MRR)
```dax
Total Monthly Revenue = SUM('telco_churn_predictions'[MonthlyCharges])
```
*(Format as Currency with 0 decimals: `$456,117`)*

---

### 🔴 PAST CHURN MEASURES (Revenue Already Lost)

### 3. Customers Already Lost (Churn = Yes)
```dax
Customers Lost = 
CALCULATE(
    COUNTROWS('telco_churn_predictions'),
    'telco_churn_predictions'[Churn] = "Yes"
)
```
*(Result: 1,869 customers | 26.54% of base)*

### 4. Monthly Revenue Already Lost ($)
```dax
Monthly Revenue Already Lost = 
CALCULATE(
    SUM('telco_churn_predictions'[MonthlyCharges]),
    'telco_churn_predictions'[Churn] = "Yes"
)
```
*(Result: `$139,131 / mo`)*

### 5. Annualized Revenue Already Lost ($)
```dax
Annualized Revenue Already Lost = [Monthly Revenue Already Lost] * 12
```
*(Result: `$1,669,570 / yr`)*

---

### 🛡️ ACTIVE CUSTOMER MEASURES (Revenue Truly at Risk)

### 6. Active Customers Count (Churn = No)
```dax
Active Customers = 
CALCULATE(
    COUNTROWS('telco_churn_predictions'),
    'telco_churn_predictions'[Churn] = "No"
)
```
*(Result: 5,174 active customers | 73.46% of base)*

### 7. Active Customer Monthly Recurring Revenue ($)
```dax
Active Monthly Revenue = 
CALCULATE(
    SUM('telco_churn_predictions'[MonthlyCharges]),
    'telco_churn_predictions'[Churn] = "No"
)
```
*(Result: `$316,986 / mo`)*

### 8. High-Risk Active Customers Count
```dax
High Risk Active Customers = 
CALCULATE(
    COUNTROWS('telco_churn_predictions'),
    'telco_churn_predictions'[Churn] = "No",
    'telco_churn_predictions'[risk_tier] = "High Risk"
)
```
*(Result: 469 active customers)*

### 9. Active Monthly Revenue at Risk ($)
```dax
Active Monthly Revenue at Risk = 
CALCULATE(
    SUM('telco_churn_predictions'[MonthlyCharges]),
    'telco_churn_predictions'[Churn] = "No",
    'telco_churn_predictions'[risk_tier] = "High Risk"
)
```
*(Result: `$36,330 / mo`)*

### 10. Active Annualized Revenue at Risk ($)
```dax
Active Annualized Revenue at Risk = [Active Monthly Revenue at Risk] * 12
```
*(Result: `$435,956 / yr`)*

### 11. High-Risk Active Revenue Share (%)
```dax
High Risk Active Revenue % = 
DIVIDE([Active Monthly Revenue at Risk], [Active Monthly Revenue], 0)
```
*(Result: `11.5%`)*

---

## 📊 4. Detailed Visual Specifications

### 🏆 Top KPI Ribbon (5 Cards)
- **Card 1: Total Portfolio Customers:** `[Total Customers]` $\rightarrow$ **7,043** (Subtitle: `Total MRR: $456.1K`)
- **Card 2: Monthly Revenue Already Lost:** `[Monthly Revenue Already Lost]` $\rightarrow$ **$139.1K / mo** (Callout color: `#D9534F` Red; Subtitle: `1,869 Churned ($1.67M/yr)`)
- **Card 3: Active Customer Base:** `[Active Customers]` $\rightarrow$ **5,174** (Callout color: `#2B5C8F` Navy; Subtitle: `Active MRR: $317.0K`)
- **Card 4: High-Risk Active Accounts:** `[High Risk Active Customers]` $\rightarrow$ **469** (Subtitle: `9.1% of Active Accounts`)
- **Card 5: Active Revenue at Risk:** `[Active Monthly Revenue at Risk]` $\rightarrow$ **$36.3K / mo** (Callout color: `#D9534F` Red; Subtitle: `$436.0K Annualized at Risk`)

---

### 🎛️ Slicers / Interactive Filters
1. `risk_tier` (`High Risk`, `Medium Risk`, `Low Risk`)
2. `Contract` (`Month-to-month`, `One year`, `Two year`)
3. `InternetService` (`Fiber optic`, `DSL`, `No`)
4. `PaymentMethod` (`Electronic check`, `Mailed check`, `Bank transfer`, `Credit card`)

---

### 📉 Visual 1: Churn Rate by Contract Type (100% Stacked Bar)
- **Y-Axis:** `Contract`
- **X-Axis:** `[Total Customers]`
- **Legend:** `Churn` (`No` = `#2B5C8F` Navy, `Yes` = `#D9534F` Coral Red)
- **Takeaway:** Month-to-month has a **42.7% churn rate**, 15x higher than two-year contracts (**2.8%**).

---

### ⏳ Visual 2: Churn Distribution by Tenure Cohort (Clustered Column)
- **X-Axis:** `TenureGroup` (`0-12m`, `13-24m`, `25-48m`, `49-72m`)
- **Y-Axis:** `[Total Customers]`
- **Legend:** `Churn` (`No` vs. `Yes`)
- **Takeaway:** First-year subscribers suffer a **47.7% churn rate**.

---

### 💳 Visual 3: Churn Rate by Payment Method (Clustered Bar)
- **Y-Axis:** `PaymentMethod`
- **X-Axis:** Actual Churn Rate %
- **Takeaway:** Electronic check payers churn at **45.3%**, nearly triple auto credit card (**15.2%**).

---

### 🧩 Visual 4: Product Stickiness (Line and Clustered Column)
- **X-Axis:** `TotalServices` (0 to 6)
- **Column Y-Axis:** `[Total Customers]`
- **Line Y-Axis:** Actual Churn Rate %
- **Takeaway:** Churn drops smoothly from **44.0% at 0 services** down to **10.8% at 4 services**.

---

### 📋 Visual 5: Priority Action Table (Retention Worklist)
- **Visual Type:** `Table`
- **Filters on Visual:** `Churn` = `No` (Active Customers Only)
- **Columns:**
  1. `customerID`
  2. `risk_tier`
  3. `churn_probability_oof` (Format as percentage: `88.9%`)
  4. `MonthlyCharges` (Format as currency: `$29.85`)
  5. `tenure`
  6. `Contract`
  7. `InternetService`
  8. `PaymentMethod`
- **Sorting:** Descending by `churn_probability_oof` or `MonthlyCharges`.
- **Conditional Formatting:**
  - `risk_tier`: High Risk = Light Red background, Medium = Light Yellow, Low = Light Green.
  - `churn_probability_oof`: Red data bar showing risk magnitude.

---

## 🎯 5. How to Walk Through This Dashboard in an Interview

1. **Highlight the Business Discipline:**
   *"The first thing you'll notice in my dashboard is that I separated Revenue Already Lost from Revenue at Risk. A common mistake is reporting revenue of already-churned customers as 'at risk'. Here, executives can see that $139.1K/month was already lost from past churn, but we have exactly 5,174 active paying subscribers generating $317K/month."*

2. **Point to the Actionable Target Cohort:**
   *"Out of our active subscribers, our out-of-fold model isolates exactly 469 High-Risk accounts. This represents $36,330 in monthly revenue ($436K annualized). That is our targeted retention priority."*

3. **Demonstrate Operational Handoff:**
   *"If I click 'High Risk' in the slicer, the bottom worklist instantly filters to those 469 active customers. A retention specialist can immediately see that Customer 7590-VHVEG has an 88.9% churn risk, is in month 1 of a month-to-month plan paying $29.85, and can immediately trigger our $10/month 1-year contract conversion offer."*
