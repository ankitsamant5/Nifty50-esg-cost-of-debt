# Nifty50-esg-cost-of-debt
Analysis of ESG risk and cost of debt for NIFTY50 firms using Python and Power BI
# NIFTY50 ESG Risk vs Cost of Debt 📉🌱

This project investigates whether **ESG risk** is associated with a company’s **cost of debt** for firms in the **NIFTY50 index (India)**.  

Using **NIFTY50 ESG risk scores** and detailed **NSE/BSE financial statement data**, I:

- Built a **clean firm-level dataset** with ESG + financials
- Estimated an **OLS regression** of cost of debt on ESG risk and financial controls
- Designed an **interactive Power BI dashboard** to explore the relationships

---

## 🎯 Research Question

> Do firms with **higher ESG risk** pay a **higher cost of debt**, after controlling for profitability, leverage, and size?

---

## 📂 Data

### 1. ESG Data (NIFTY50)

- Source: NIFTY50 ESG risk data (2024)
- Key columns used:
  - `nse` – NSE ticker
  - `company` – company name
  - `Sector`, `Industry`
  - `esg_risk_score` – overall ESG risk score (higher = worse ESG)
  - Optional fields: `esg_risk_exposure`, `esg_risk_management`, `esg_risk_level`, `controversy_score`

### 2. Financials (NSE/BSE)

- Source: “Detailed Financials Data of 4456 NSE and BSE companies” (zip)
- For each company, the zip contains:
  - `*_Basic_Info.csv` – includes NSE ticker (`NSE`)
  - `Yearly_Balance_Sheet.csv`
  - `Yearly_Profit_Loss.csv`

From these, for each NIFTY50 firm I extracted the **latest available year** and computed:

- `borrowings` (from balance sheet)
- `total_assets`
- `interest` (from profit & loss)
- `net_profit`

### 3. Final Modeling Dataset

The notebook creates a cleaned dataset, saved as e.g.:

- `nifty50_esg_cost_of_debt.csv`

with one row per firm and columns:

- `nse`, `company`, `Sector`, `Industry`
- `esg_risk_score`
- `cost_of_debt = |interest| / borrowings`
- `roa = net_profit / total_assets`
- `leverage = borrowings / total_assets`
- `size = log(total_assets)`

This CSV is what feeds both the regression and the Power BI dashboard.

---

## 🧮 Methodology

### 1. Variable Construction

- **Cost of debt**  
  \[
  \text{cost\_of\_debt} = \frac{|\text{interest expense}|}{\text{borrowings}}
  \]
  Interpreted as an approximate **interest rate on debt** (e.g., 0.08 ≈ 8%).

- **Profitability (ROA)**  
  \[
  \text{roa} = \frac{\text{net profit}}{\text{total assets}}
  \]

- **Leverage**  
  \[
  \text{leverage} = \frac{\text{borrowings}}{\text{total assets}}
  \]

- **Size**  
  \[
  \text{size} = \log(\text{total assets})
  \]

- **ESG Risk Bands (for visuals)**  
  Firms are bucketed into `"Low risk"`, `"Medium risk"`, `"High risk"` based on `esg_risk_score` thresholds or quartiles.

### 2. Regression Model

I estimated an OLS model:

\[
\text{cost\_of\_debt}_i = \beta_0
+ \beta_1 \,\text{esg\_risk\_score}_i
+ \beta_2 \,\text{roa}_i
+ \beta_3 \,\text{leverage}_i
+ \beta_4 \,\text{size}_i
+ \varepsilon_i
\]

- Dependent variable: `cost_of_debt`
- Regressors: `esg_risk_score`, `roa`, `leverage`, `size`
- Sample: 49 NIFTY50 firms (1 dropped due to missing cost_of_debt)

---

## 🔍 Key Results (High Level)

- **Model fit**:  
  - R² ≈ **0.26**, Adjusted R² ≈ **0.19**  
  - F-test p-value ≈ **0.01** → the regressors are **jointly significant**

- **ESG effect**:
  - Coefficient on `esg_risk_score` is **small and statistically insignificant** (p ≈ 0.49)
  - Directionally positive (higher ESG risk → slightly higher cost of debt), but **not robust enough** to claim a strong relationship in this sample.

- **Controls**:
  - `roa`, `leverage`, and `size` are not strongly significant individually, though ROA is marginal at the 10% level.
  - Sector-level differences in cost of debt appear more pronounced than differences across ESG bands.

- **Diagnostics**:
  - Residuals show **skewness and heavy tails**, driven by a few outliers with very high cost of debt.
  - Results should be interpreted with some caution due to outliers and small sample size.

---

## 📊 Power BI Dashboard

The Power BI report (e.g. `NIFTY50_ESG_CostOfDebt.pbix`) includes:

- **KPI cards**:
  - Average cost of debt (overall)
  - Average ESG risk score (overall)
  - Number of firms in sample

- **Table view**:
  - Firm-level view of `nse`, `company`, `Sector`, `esg_risk_score`, `cost_of_debt`, `roa`, `leverage`, `size`

- **Scatter plot**:
  - X-axis: `esg_risk_score`
  - Y-axis: `cost_of_debt`
  - One dot per company, with trendline
  - Optional color by `Sector` or `ESG_Risk_Band`

- **Cost of debt by ESG band**:
  - Column chart: average `cost_of_debt` across Low / Medium / High ESG risk bands

- **Cost of debt by sector**:
  - Column chart: average `cost_of_debt` by `Sector`

- **Slicers**:
  - `ESG_Risk_Band` filter
  - Optional `Sector` filter

This makes it easy to interactively explore how cost of debt varies by ESG profile and sector.

## 🔎 Additional Predictive Models

Beyond the baseline OLS regression, I also benchmarked several predictive models using the same feature set (`esg_risk_score`, `roa`, `leverage`, `size`) on an 80/20 train–test split:

- **Ridge Regression**
- **Lasso Regression**
- **Random Forest Regressor**
- **XGBoost Regressor**

Test-set performance (approximate):

- Ridge: R² ≈ -0.22, RMSE ≈ 0.060  
- Lasso: R² ≈ -0.25, RMSE ≈ 0.060  
- Random Forest: R² ≈ -3.94, RMSE ≈ 0.120  
- XGBoost: R² ≈ -4.90, RMSE ≈ 0.131  

A model with R² = 0 would perform as well as simply predicting the **mean** cost of debt on the test set, so the **negative R² values** indicate that none of these models reliably beat a simple baseline on this small sample. Ridge and Lasso are the most stable, while Random Forest and XGBoost clearly overfit.

These results reinforce the main conclusion of the project:  
> in this NIFTY50 snapshot, **cost of debt is difficult to predict from ESG risk and a small set of financial ratios alone**, and ESG does not emerge as a strong standalone driver of borrowing costs.


