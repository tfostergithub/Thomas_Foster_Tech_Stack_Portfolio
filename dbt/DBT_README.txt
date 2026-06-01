# Thomas E. Foster — DBT Portfolio
## Healthcare Claims Analytics Pipeline

> **Senior Data Analyst · DBT · Snowflake · Statistical Analytics**
> Pea Ridge, AR  |  (540) 570-6094  |  thomasfoster79@yahoo.com
>
> *Production-grade DBT pipeline implementing 9 statistical models natively in SQL — no Python required.*

---

## About This Portfolio

This repository showcases a complete **DBT analytics pipeline** built on Snowflake for healthcare claims data. The pipeline implements nine statistical and financial models entirely in SQL — including regression, time series, volatility, and risk models — and delivers a single wide mart table consumed directly by Tableau and Power BI dashboards.

Every model in this portfolio was built to answer a real financial or actuarial question without leaving the Snowflake/DBT stack.

---

## Pipeline Architecture

```
RAW SOURCE (healthcare_db.healthcare_schema.claims)
        |
        v
[ CONFIGURATION ]
  dbt_project.yml         — Project blueprint, schema routing, materialization defaults
  sources.yml             — Source declaration, column docs, lineage tracking

        |
        v
[ STAGING LAYER ]
  stg_healthcare_claims   — Type-safe entry point (::date, ::float, ::int casts)
  test_source_validation  — Development connectivity check

        |
        v
[ INTERMEDIATE LAYER ]
  int_moving_average              — 7-period & 30-period rolling averages
  int_simple_linear_regression    — OLS regression (time as predictor)
  int_multiple_linear_regression  — 2-predictor OLS via Cramer's Rule
  int_exponential_smoothing       — Weighted smoothing (alpha=0.3, 10 lags)
  int_arima                       — ARIMA differencing + AR lag features
  int_garch                       — GARCH(1,1) conditional volatility
  int_value_at_risk               — Parametric VaR at 90% / 95% / 99%
  int_sharpe_ratio                — Risk-adjusted return, annualized
  int_straight_line_forecast      — 12-period compound growth (Jinja2 loop)

        |
        v
[ MART LAYER ]
  fct_claims_analytics    — Wide table: all models joined for Tableau/Power BI
```

---

## Skills & Technologies

![DBT](https://img.shields.io/badge/DBT-FF694B?style=for-the-badge&logo=dbt&logoColor=white)
![Snowflake](https://img.shields.io/badge/Snowflake-29B5E8?style=for-the-badge&logo=snowflake&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white)
![Tableau](https://img.shields.io/badge/Tableau-E97627?style=for-the-badge&logo=tableau&logoColor=white)

| Category | Skills |
|---|---|
| **DBT Features** | CTEs, Jinja2 templating, source() macro, ref() macro, materialization config, YAML schema docs |
| **SQL Techniques** | Window Functions, LAG(), AVG() OVER, ROW_NUMBER(), STDDEV_SAMP(), NULLIF(), POWER() |
| **Statistical Models** | OLS Regression, Multiple Regression, ARIMA, GARCH, Exponential Smoothing, Moving Average |
| **Risk Analytics** | Value at Risk (VaR), Sharpe Ratio, Conditional Volatility |
| **Forecasting** | Compound Growth, Straight-Line Forecast, Jinja2 loop-generated UNION ALL spine |
| **Platform** | Snowflake SQL, DBT Core, YAML configuration, Source freshness |

---

## Portfolio Documents

---

### ⚙️ Configuration Layer

---

#### 1. DBT Project Configuration — `dbt_project.yml`
📄 [`01_DBT_Project_Config_YAML.pdf`](./01_DBT_Project_Config_YAML.pdf)

**What It Does**
The master configuration file for the entire DBT project. Defines project name, version, folder structure, and materialization defaults — routing `healthcare` schema models to tables and `demo` schema to views. Every downstream model inherits its materialization from this file unless explicitly overridden.

**Key Config**
```yaml
models:
  healthcare_data_project:
    healthcare:
      +schema: healthcare
      +materialized: table   # Optimized for Tableau/Power BI consumption
    demo:
      +schema: demo
      +materialized: view    # Lightweight for development/sandbox
```

`YAML` `dbt_project.yml` `Schema Routing` `Materialization` `Project Setup`

---

#### 2. Healthcare Source Definition — `sources.yml`
📄 [`02_DBT_Healthcare_Source_YAML.pdf`](./02_DBT_Healthcare_Source_YAML.pdf)

**What It Does**
Declares and documents the external healthcare claims source table (`healthcare_db.healthcare_schema.claims`, ~50k rows). Enables the `{{ source('healthcare', 'claims') }}` Jinja macro used in every downstream model, automatic DAG lineage tracking, and source freshness checks. Documents all 14 columns.

**Key Config**
```yaml
sources:
  - name: healthcare
    database: healthcare_db
    schema: healthcare_schema
    tables:
      - name: claims
        description: "Healthcare claims time series - 50k rows"
```

`YAML` `sources.yml` `Data Lineage` `Column Documentation` `Source Testing`

---

### 🏗️ Staging Layer

---

#### 3. Staging — Healthcare Claims Materialized View
📄 [`03_DBT_Staging_Healthcare_Claims.pdf`](./03_DBT_Staging_Healthcare_Claims.pdf)

**What It Does**
The single type-safe entry point for the entire pipeline. Applies explicit Snowflake type casts to all 14 columns — enforcing `::date`, `::float`, and `::int` types at the boundary so no downstream model has to worry about type coercion or string arithmetic.

**Key SQL**
```sql
{{ config(materialized='view') }}
select
    period_date::date     as period_date,
    metric_value::float   as metric_value,
    is_fraud_flag::int    as is_fraud_flag,
    ...
from {{ source('healthcare', 'claims') }}
```

`Staging` `Type Casting` `Materialized View` `Data Quality` `Snowflake`

---

#### 4. Source Validation — Test Query
📄 [`04_DBT_Test_Query.pdf`](./04_DBT_Test_Query.pdf)

**What It Does**
A development-time validation query that confirms the Snowflake source connection is live and all 14 columns are accessible before running a full `dbt build`. Used to catch schema migrations and connectivity failures early.

`Testing` `Source Validation` `Development` `Data Quality`

---

### 📐 Intermediate Statistical Models

---

#### 5. Moving Average — 7-Period & 30-Period Rolling Windows
📄 [`05_DBT_Moving_Average.pdf`](./05_DBT_Moving_Average.pdf)

**What It Does**
Computes 7-period and 30-period rolling averages of the claims metric for each entity using `AVG() OVER (ROWS BETWEEN N PRECEDING AND CURRENT ROW)`. Includes `window_row_count` to flag warm-up periods. Serves as the primary spine of the mart layer.

**Key SQL**
```sql
avg(metric_value) over (
    partition by entity_id
    order by period_date
    rows between 6 preceding and current row
) as moving_avg_7
```

`Moving Average` `Window Functions` `ROWS BETWEEN` `Time Series` `Mart Spine`

---

#### 6. Simple Linear Regression — Pure SQL OLS
📄 [`06_DBT_Simple_Linear_Regression.pdf`](./06_DBT_Simple_Linear_Regression.pdf)

**What It Does**
Implements Ordinary Least Squares regression entirely in SQL using row number as the time predictor (X) and claim metric as Y. Derives `beta_0` (intercept) and `beta_1` (slope) analytically, outputs fitted values and residuals for trend analysis and anomaly detection.

**Key SQL**
```sql
-- Closed-form OLS beta_1
(sum(x * y) - count(*) * avg(x) * avg(y))
    / nullif(sum(x*x) - count(*) * avg(x) * avg(x), 0) as beta_1
```

`Linear Regression` `OLS` `Analytic Formula` `Fitted Values` `Residuals`

---

#### 7. Multiple Linear Regression — 2-Predictor OLS via Cramer's Rule
📄 [`07_DBT_Multi_Linear_Regression.pdf`](./07_DBT_Multi_Linear_Regression.pdf)

**What It Does**
Extends OLS to two predictors (x1, x2) by solving the 2x2 normal equation system using Cramer's Rule in pure SQL. Mean-centers all variables for numerical stability before computing both slope coefficients and the intercept. No Python or external ML required.

**Key SQL**
```sql
-- Cramer's Rule for 2x2 system
(sx1y * sxx2 - sx2y * sx1x2)
    / nullif((sxx1 * sxx2 - sx1x2 * sx1x2), 0) as beta_1
```

`Multiple Regression` `Cramer's Rule` `Mean-Centering` `OLS` `Pure SQL`

---

#### 8. Exponential Smoothing — Weighted Decay (alpha=0.3)
📄 [`08_DBT_Exponential_Smoothing.pdf`](./08_DBT_Exponential_Smoothing.pdf)

**What It Does**
Applies weighted exponential smoothing with `alpha=0.3` across up to 10 historical lags. Each lag is weighted by `alpha * (1-alpha)^k` — so recent observations carry more weight than older ones. Provides a noise-filtered view of the claims metric that reacts faster than a simple moving average.

**Key SQL**
```sql
  0.3 * metric_value
+ 0.3 * 0.7    * lag(metric_value, 1) over (partition by entity_id order by period_date)
+ 0.3 * 0.7*0.7 * lag(metric_value, 2) over (partition by entity_id order by period_date)
+ 0.3 * power(0.7, 3) * lag(metric_value, 3) over (...)
-- ... extended to 10 lags
```

`Exponential Smoothing` `LAG()` `Weighted Decay` `Time Series` `alpha=0.3`

---

#### 9. ARIMA — Feature Engineering (Differencing + AR Lags)
📄 [`09_DBT_ARIMA.pdf`](./09_DBT_ARIMA.pdf)

**What It Does**
Prepares all feature inputs for an ARIMA(p,d,q) model in SQL. Applies first and second-order differencing to remove trend and non-stationarity, then generates three autoregressive lag features (lags 1, 2, 3 of the differenced series) as ARIMA predictors.

**Key SQL**
```sql
-- First differencing: removes linear trend
metric_value - lag(metric_value, 1) over (partition by entity_id order by period_date) as diff_1

-- AR(3) lag features on differenced series
lag(diff_1, 1) over (partition by entity_id order by period_date) as ar_lag_1
```

`ARIMA` `Differencing` `AR Lags` `Stationarity` `Time Series` `Feature Engineering`

---

#### 10. GARCH(1,1) — Conditional Volatility Model
📄 [`10_DBT_GARCH.pdf`](./10_DBT_GARCH.pdf)

**What It Does**
Approximates a GARCH(1,1) model to estimate time-varying volatility in healthcare financial returns. Seeds unconditional variance with `omega / (1 - alpha - beta)`, then applies the GARCH recursion `sigma^2_t = omega + alpha*eps^2_{t-1} + beta*sigma^2_{t-1}` via `LAG()`. Parameters configurable via Jinja2 `{% set %}` variables.

**Key SQL**
```sql
{% set omega = 0.000001 %}
{% set alpha = 0.09 %}
{% set beta  = 0.90 %}

{{ omega }}
  + {{ alpha }} * lag(eps_sq,     1) over (partition by entity_id order by period_date)
  + {{ beta  }} * lag(sigma_sq_0, 1) over (partition by entity_id order by period_date)
  as sigma_sq_approx
```

`GARCH` `Conditional Volatility` `Jinja2 Variables` `Risk Analytics` `LAG()`

---

#### 11. Value at Risk — Parametric VaR (90% / 95% / 99%)
📄 [`11_DBT_Value_at_Risk.pdf`](./11_DBT_Value_at_Risk.pdf)

**What It Does**
Computes parametric VaR at three confidence levels for each entity/payer/state/specialty segment. Uses the closed-form formula `VaR = mu - z * sigma` with `STDDEV_SAMP` for unbiased standard deviation. Delivers risk thresholds that finance leadership can use directly in capital allocation decisions.

**Key SQL**
```sql
mu - 1.2816 * sigma as var_90_pct,   -- 10% tail
mu - 1.6449 * sigma as var_95_pct,   -- 5% tail  (industry standard)
mu - 2.3263 * sigma as var_99_pct    -- 1% tail  (stress scenario)
```

`Value at Risk` `VaR` `Parametric` `STDDEV_SAMP` `Risk Management` `Finance`

---

#### 12. Sharpe Ratio — Risk-Adjusted Return, Annualized
📄 [`12_DBT_Sharpe_Ratio.pdf`](./12_DBT_Sharpe_Ratio.pdf)

**What It Does**
Computes periodic and annualized Sharpe Ratios for each entity/payer/state/specialty. Subtracts the monthly risk-free rate (0.4167% ≈ 5% annual / 12) from returns to compute excess return, then divides by standard deviation. Annualizes by multiplying by `sqrt(12)`.

**Key SQL**
```sql
mean_excess_return / nullif(sigma_p, 0)              as sharpe_periodic,
(mean_excess_return / nullif(sigma_p, 0)) * sqrt(12) as sharpe_annualised
```

`Sharpe Ratio` `Risk-Adjusted Return` `STDDEV_SAMP` `sqrt(12)` `Annualization` `Finance`

---

#### 13. Straight Line Forecast — 12-Period Compound Growth (Jinja2)
📄 [`13_DBT_Straight_Line_Forecast.pdf`](./13_DBT_Straight_Line_Forecast.pdf)

**What It Does**
Generates a 12-period forward compound-growth forecast per entity using a Jinja2 `{% for %}` loop that compiles to 12 UNION ALL SELECT blocks at dbt run time. The growth rate is set via `{% set r = 0.05 %}` at the top of the file — change one line to retune all 12 periods instantly. Auto-anchors to each entity's most recent actual value.

**Key DBT / SQL**
```sql
{% set r = 0.05 %}
{% for i in ns.period_list %}
select
    entity_id, base_date, base_value,
    {{ i }} as periods_ahead,
    base_value * power(1 + {{ r }}, {{ i }}) as forecast_value
from latest_actuals
{% if not loop.last %} union all {% endif %}
{% endfor %}
```

`Forecasting` `Jinja2 Loop` `Compound Growth` `UNION ALL Spine` `POWER()` `FP&A`

---

### 🏪 Mart Layer

---

#### 14. Healthcare Claims Analytics Mart — Combined Statistical Output
📄 [`14_DBT_Mart_Combined.pdf`](./14_DBT_Mart_Combined.pdf)

**What It Does**
The final wide mart table (`fct_claims_analytics`) that joins all active intermediate models into one flat row per entity/date — ready for Tableau or Power BI to connect to directly. Moving averages, SLR fitted values and residuals, VaR at all three confidence levels, and Sharpe ratio are all active. ARIMA, GARCH, MLR, and Exponential Smoothing joins are present but commented out, ready to activate once those models complete validation.

**Key SQL**
```sql
{{ config(materialized='table') }}

select
    ma.entity_id, ma.payer, ma.state, ma.specialty, ma.period_date,
    ma.metric_value, ma.moving_avg_7, ma.moving_avg_30,
    lr.fitted_value as slr_fitted_value,
    lr.residual     as slr_residual,
    var.var_90_pct, var.var_95_pct, var.var_99_pct,
    sr.sharpe_periodic, sr.sharpe_annualised

from {{ ref('int_moving_average') }} ma
left join {{ ref('int_simple_linear_regression') }} lr  on ma.entity_id = lr.entity_id and ma.period_date = lr.period_date
left join {{ ref('int_value_at_risk') }}            var on ma.entity_id = var.entity_id
left join {{ ref('int_sharpe_ratio') }}             sr  on ma.entity_id = sr.entity_id
```

`Mart` `Wide Table` `ref() macro` `Tableau` `Power BI` `LEFT JOIN` `Final Output`

---

## Repository Structure

```
dbt-portfolio/
├── README.md
│
├── 01_DBT_Project_Config_YAML.pdf
├── 02_DBT_Healthcare_Source_YAML.pdf
│
├── 03_DBT_Staging_Healthcare_Claims.pdf
├── 04_DBT_Test_Query.pdf
│
├── 05_DBT_Moving_Average.pdf
├── 06_DBT_Simple_Linear_Regression.pdf
├── 07_DBT_Multi_Linear_Regression.pdf
├── 08_DBT_Exponential_Smoothing.pdf
├── 09_DBT_ARIMA.pdf
├── 10_DBT_GARCH.pdf
├── 11_DBT_Value_at_Risk.pdf
├── 12_DBT_Sharpe_Ratio.pdf
├── 13_DBT_Straight_Line_Forecast.pdf
│
└── 14_DBT_Mart_Combined.pdf
```

---

## What Makes This Portfolio Stand Out

| Capability | Detail |
|---|---|
| **Pure SQL Statistics** | OLS, Multiple Regression, GARCH, VaR, Sharpe — all implemented natively in SQL without Python |
| **Jinja2 Templating** | GARCH parameters as `{% set %}` vars; forecast spine generated by `{% for %}` loop at compile time |
| **DBT Best Practices** | staging → intermediate → mart layering; `source()` and `ref()` macros throughout; YAML documentation |
| **Snowflake Optimized** | `::` type casting, `STDDEV_SAMP`, `POWER()`, `SQRT()`, window frames — all Snowflake-native |
| **BI-Ready Output** | Mart table designed as a single flat join for direct Tableau / Power BI connection |
| **Extensible Design** | Commented-out mart joins (ARIMA, GARCH, MLR, ES) ready to activate post-validation |

---

## Contact

**Thomas E. Foster**
📧 thomasfoster79@yahoo.com
📞 (540) 570-6094
📍 Pea Ridge, AR 72751
🎓 PhD Candidate — Data Science (AI Engineering & Healthcare Claims Analytics)

> *All models are original work. Built on Snowflake using DBT Core. Healthcare data is synthetic (50k rows). No patient-identifiable information is present.*
