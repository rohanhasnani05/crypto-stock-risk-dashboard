# Crypto vs Stock Market Diversification & Risk Insights Dashboard – SQL, Python, Power BI

This project explores how adding **cryptocurrencies** like Bitcoin and Ethereum affects a traditional portfolio of **stocks and gold**.

Using **Python** for data preparation, a **SQL-style star schema** for modeling, and **Power BI** for visualization, the project answers three key questions:

1. How do crypto, stocks, and gold compare on **return and volatility**?
2. Do crypto assets really provide **diversification** vs traditional markets?
3. What happens to a **60/40 portfolio** when we add different levels of crypto exposure?

---

## 1. Project Overview

### Goal

Provide a clear, data-driven view of:

- Individual asset performance (BTC, ETH, SPY, GOLD)
- Correlation between crypto and traditional assets
- Portfolio-level risk/return for different allocation scenarios

### Target Users

- Retail investors
- Financial advisors / relationship managers
- Product / business teams evaluating crypto offerings
- Hiring managers looking for Business Analyst / Data Analyst skills

---

## 2. Tech Stack

- **Language:** Python (Pandas, NumPy)
- **Modeling:** SQL-style star schema (dimension + fact tables, easily portable to any RDBMS)
- **Visualization:** Power BI Desktop
- **Data Source:** Historical daily prices from [Investing.com](https://www.investing.com/) (CSV downloads)
- **Documentation:** BRD, FRD, User Stories (Word)

---

## 3. Data & Model

### Assets

The project uses four assets:

- `BTC-USD` – Bitcoin (Crypto)
- `ETH-USD` – Ethereum (Crypto)
- `SPY` – S&P 500 ETF (Equity index)
- `GOLD` – Gold (Commodity)

### Logical Data Model (Star Schema)

**Dimensions**

- `dim_assets` – asset metadata (symbol, name, type, active flag)
- `dim_date` – calendar table (date, year, month, etc.)
- `dim_portfolio_scenarios` – scenario names and descriptions

**Facts**

- `fact_asset_prices` – daily close prices per asset
- `fact_asset_returns` – daily returns per asset
- `fact_correlations` – static pairwise correlations between assets
- `fact_rolling_correlations` – 90-day rolling correlations for selected pairs
- `fact_portfolio_allocations` – asset weights per scenario
- `fact_portfolio_daily_returns` – daily portfolio returns per scenario

> This structure is intentionally SQL-friendly and can be recreated in any relational database (e.g., SQL Server, PostgreSQL, MySQL) using the CSVs as source tables.

---

## 4. Folder Structure

Suggested structure (what this repo assumes):

```text
.
├─ data_raw/
│   ├─ BTC-USD_investing.csv
│   ├─ ETH-USD_investing.csv
│   ├─ SPY_investing.csv
│   └─ GOLD_investing.csv
├─ data_model/
│   ├─ dim_assets.csv
│   ├─ dim_portfolio_scenarios.csv
│   ├─ fact_portfolio_allocations.csv
│   ├─ fact_asset_prices.csv
│   ├─ fact_asset_returns.csv
│   ├─ fact_correlations.csv
│   ├─ fact_rolling_correlations.csv
│   └─ fact_portfolio_daily_returns.csv
├─ scripts/
│   ├─ build_prices_and_returns.py
│   ├─ build_correlations.py
│   └─ build_portfolio_returns.py
├─ docs/
│   ├─ BRD_Crypto_vs_Stock_Dashboard.docx
│   └─ FRD_Crypto_vs_Stock_Dashboard.docx
└─ powerbi/
    └─ Crypto_vs_Stock_Dashboard.pbix


```
---

## 5. How to Reproduce

### 5.1. Download Raw Price Data

1. Go to [Investing.com](https://www.investing.com/).
2. For each asset (BTC, ETH, SPY, GOLD):
   - Open the asset’s **Historical Data** page.
   - Choose **Daily** frequency.
   - Select a date range (e.g., from `2021-01-01` to today).
   - Click **Download Data**.
3. Rename and save into `data_raw/` as:

   - `BTC-USD_investing.csv`
   - `ETH-USD_investing.csv`
   - `SPY_investing.csv`
   - `GOLD_investing.csv`

### 5.2. Create Dimension & Allocation Files

Create the following CSVs in `data_model/`:

**`dim_assets.csv`**

| asset_symbol | asset_name  | asset_type | is_active |
|--------------|-------------|-----------|----------|
| BTC-USD      | Bitcoin     | Crypto    | 1        |
| ETH-USD      | Ethereum    | Crypto    | 1        |
| SPY          | S&P 500 ETF | Index     | 1        |
| GOLD         | Gold        | Commodity | 1        |

**`dim_portfolio_scenarios.csv`**

| scenario_id | scenario_name        | description                        | is_active |
|-------------|----------------------|------------------------------------|----------|
| 1           | 60/40 No Crypto      | Traditional SPY + Gold             | 1        |
| 2           | 55/35/10 BTC         | Add 10% BTC                        | 1        |
| 3           | 50/30/20 BTC+ETH     | 20% crypto split BTC & ETH         | 1        |

**`fact_portfolio_allocations.csv`**

| scenario_id | asset_symbol | weight_pct |
|-------------|--------------|-----------|
| 1           | SPY          | 0.60      |
| 1           | GOLD         | 0.40      |
| 2           | SPY          | 0.55      |
| 2           | GOLD         | 0.35      |
| 2           | BTC-USD      | 0.10      |
| 3           | SPY          | 0.50      |
| 3           | GOLD         | 0.30      |
| 3           | BTC-USD      | 0.10      |
| 3           | ETH-USD      | 0.10      |

### 5.3. Run Python Preprocessing

From the project root:

```bash
# 1) Build asset prices & returns
python scripts/build_prices_and_returns.py

# 2) Build static and rolling correlations
python scripts/build_correlations.py

# 3) Build portfolio daily returns
python scripts/build_portfolio_returns.py

```

These scripts will generate:

fact_asset_prices.csv

fact_asset_returns.csv

fact_correlations.csv

fact_rolling_correlations.csv

fact_portfolio_daily_returns.csv

inside data_model/.

---

## 6. Python Script Summary

### `build_prices_and_returns.py`

- Reads raw CSVs from `data_raw/`.
- Cleans the `Price` column (removes commas/symbols, converts to numeric).
- Standardizes to columns: `asset_symbol`, `price_date`, `close_price`.
- Computes **daily returns** per asset.
- Outputs:
  - `fact_asset_prices.csv`
  - `fact_asset_returns.csv`

### `build_correlations.py`

- Pivots `fact_asset_returns` into wide format (dates × assets).
- Computes a **static correlation matrix** of daily returns.
- Computes **90-day rolling correlations** for:
  - BTC vs SPY  
  - ETH vs SPY
- Outputs:
  - `fact_correlations.csv`
  - `fact_rolling_correlations.csv`

### `build_portfolio_returns.py`

- Joins asset returns with `fact_portfolio_allocations` on `asset_symbol`.
- Calculates **weighted daily return** per asset and scenario.
- Aggregates to **portfolio daily return** per scenario and date.
- Outputs:
  - `fact_portfolio_daily_returns.csv`

> All logic is written in Python but directly translates to SQL window functions and aggregations if you prefer to implement it in a database.

---

## 7. Power BI Dashboard

Open `powerbi/Crypto_vs_Stock_Dashboard.pbix` with **Power BI Desktop**.

### 7.1. Data Model (Relationships)

Key relationships:

- `dim_assets[asset_symbol]` → `fact_asset_prices[asset_symbol]`
- `dim_assets[asset_symbol]` → `fact_asset_returns[asset_symbol]`
- `dim_assets[asset_symbol]` → `fact_portfolio_allocations[asset_symbol]`
- `dim_portfolio_scenarios[scenario_id]` → `fact_portfolio_allocations[scenario_id]`
- `dim_portfolio_scenarios[scenario_id]` → `fact_portfolio_daily_returns[scenario_id]`
- `dim_date[Date]` → `fact_asset_prices[price_date]`
- `dim_date[Date]` → `fact_asset_returns[price_date]`
- `dim_date[Date]` → `fact_rolling_correlations[window_end_date]`
- `dim_date[Date]` → `fact_portfolio_daily_returns[price_date]`

### 7.2. Example DAX Measures

**Asset-level measures:**

```DAX
Avg Daily Return =
AVERAGE ( fact_asset_returns[daily_return] )

Annualized Return =
VAR DailyAvg = [Avg Daily Return]
RETURN
IF ( NOT ISBLANK ( DailyAvg ), POWER ( 1 + DailyAvg, 252 ) - 1 )

Daily Volatility =
STDEVX.P ( fact_asset_returns, fact_asset_returns[daily_return] )

Annualized Volatility =
VAR DailyStd = [Daily Volatility]
RETURN
IF ( NOT ISBLANK ( DailyStd ), DailyStd * SQRT ( 252 ) )

Portfolio Avg Daily Return =
AVERAGE ( fact_portfolio_daily_returns[portfolio_return] )

Portfolio Annualized Return =
VAR DailyAvg = [Portfolio Avg Daily Return]
RETURN
IF ( NOT ISBLANK ( DailyAvg ), POWER ( 1 + DailyAvg, 252 ) - 1 )

Portfolio Daily Volatility =
STDEVX.P (
    fact_portfolio_daily_returns,
    fact_portfolio_daily_returns[portfolio_return]
)

Portfolio Annualized Volatility =
VAR DailyStd = [Portfolio Daily Volatility]
RETURN
IF ( NOT ISBLANK ( DailyStd ), DailyStd * SQRT ( 252 ) )

```

---

## 8. Dashboard Pages

### Page 1 – Market Overview

Focus: **Individual asset performance**

- **Daily returns (small multiples)** for BTC, ETH, GOLD, SPY  
- **Price trend line chart**  
- **Summary table** with annualized return & volatility per asset  
- **Slicers** for date and asset symbol  

**Key insight:**  
Crypto shows significantly higher volatility and higher long-term returns compared to SPY and GOLD over the selected period.

<img width="1427" height="803" alt="image" src="https://github.com/user-attachments/assets/86fa3c2b-f67d-4cf7-8dcc-267bdd5d8df6" />

---

### Page 2 – Correlation & Diversification

Focus: **How assets move together**

- **Correlation heatmap** of daily returns between all assets (with conditional formatting)  
- **90-day rolling correlation** for BTC vs SPY and ETH vs SPY  
- **Insight text box** explaining:
  - Red = high positive correlation  
  - Near zero = better diversification  
  - Blue (if used) = negative correlation  

**Key insight:**  
BTC and ETH are highly correlated with each other, but have lower and time-varying correlation with SPY and GOLD, creating potential diversification benefits at certain times.

<img width="1422" height="801" alt="image" src="https://github.com/user-attachments/assets/20426fb7-1873-4111-b63c-55431b95975c" />

---

### Page 3 – Portfolio Scenarios

Focus: **Portfolio-level risk & return**

Scenarios:

1. **60/40 No Crypto** – 60% SPY, 40% GOLD  
2. **55/35/10 BTC** – 55% SPY, 35% GOLD, 10% BTC  
3. **50/30/20 BTC+ETH** – 50% SPY, 30% GOLD, 10% BTC, 10% ETH  

Visuals:

- **Scenario allocations matrix** (which asset gets what %)  
- **Scenario metrics table** with portfolio annualized return & volatility  
- **Risk vs Return scatter plot** (each point = one scenario)  

**Key insight:**  
Adding crypto increases both risk and return. The scatter plot helps stakeholders visually assess whether the additional volatility is justified by the higher expected return.

<img width="1423" height="802" alt="image" src="https://github.com/user-attachments/assets/23adda0b-fdd6-4985-bb86-0aabba1adaf1" />

---

## 9. Business Analysis Deliverables

Alongside the technical implementation, the project includes:

- **BRD (Business Requirements Document)**  
  - Problem statement, scope, personas, high-level requirements.
- **FRD (Functional Requirements Document)**  
  - Data model, transformations, metrics, and report design.
- **User Stories & Acceptance Criteria**  
  - Written from perspectives of advisors, investors, and product managers.

This makes the project explainable both as:

- A **Business Analyst** case (requirements → design → solution), and  
- A **Data/Analytics** case (data pipeline → model → dashboard).

---

## 10. Possible Extensions

- Add more asset classes (bonds, other indices, altcoins).  
- Implement the full model in a relational database and expose it via SQL views.  
- Introduce a non-zero risk-free rate and compute Sharpe ratios and Sortino ratios.  
- Add parameterized “What-if” analysis for custom portfolio weights in Power BI.  
- Connect to an API (e.g., CoinGecko, Alpha Vantage) for scheduled refresh.

---

## 11. Disclaimer

This project is for **educational and demonstration purposes only**.  
It does **not** constitute financial advice or an investment recommendation.
