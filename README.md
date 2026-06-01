# Thomas E. Foster — Data Science & Analytics Portfolio

**Data Scientist | Data Analyst | AI Engineer | Predictive Analytics & Business Insights**

[![Oracle SQL](https://img.shields.io/badge/Oracle%20SQL-PL%2FSQL-F80000?style=flat-square&logo=oracle&logoColor=white)](https://www.oracle.com/)
[![Python](https://img.shields.io/badge/Python-Pandas%20%7C%20NumPy%20%7C%20Scikit--learn-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Snowflake](https://img.shields.io/badge/Snowflake-Cloud%20DW-29B5E8?style=flat-square&logo=snowflake&logoColor=white)](https://www.snowflake.com/)
[![dbt](https://img.shields.io/badge/dbt-Analytics%20Engineering-FF694B?style=flat-square&logo=dbt&logoColor=white)](https://www.getdbt.com/)
[![Power BI](https://img.shields.io/badge/Power%20BI-Business%20Intelligence-F2C811?style=flat-square&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![Tableau](https://img.shields.io/badge/Tableau-Data%20Visualization-E97627?style=flat-square&logo=tableau&logoColor=white)](https://www.tableau.com/)
[![Azure](https://img.shields.io/badge/Azure-Data%20Factory%20%7C%20SQL-0078D4?style=flat-square&logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/)

---

📧 thomasfoster79@yahoo.com &nbsp;|&nbsp; 📞 (540) 570-6094 &nbsp;|&nbsp; 📍 Pea Ridge, AR 72751 &nbsp;|&nbsp; 🔗 [github.com/thomasfoster](https://github.com/thomasfoster)

---

## Professional Summary

Senior Data Analyst and Data Scientist with **10+ years of experience** delivering enterprise-grade analytics across logistics, manufacturing, and healthcare finance. Expert in Python, SQL (T-SQL, PL/SQL), Snowflake, dbt, Power BI, Tableau, and Azure. Currently pursuing a **PhD in Data Science** with a focus on AI Engineering and Healthcare Claims Analytics — applying advanced statistical modeling, machine learning, and cloud data engineering to solve real-world business problems at scale.

---

## Experience

### Simmons Foods — Senior Data Analyst *(May 2022 – Feb 2026)*
> Rogers, AR | Snowflake · dbt · Python · SQLConnect · Tableau · Oracle Fusion Cloud · JDE

- Built enterprise-grade data models and automated ETL pipelines using **Snowflake**, **dbt**, and **Python**, replacing manual reporting with scalable analytics infrastructure
- Designed **Tableau dashboards** for Work Order KPIs, production performance variance (PPV), and freight cost allocation — serving clients including Walmart, Blue Buffalo, PetCo, and Chick-fil-A
- Developed PPV and variance reporting across 24-month rolling forecast windows using advanced SQL (CONNECT BY, LAG, CROSS JOIN, GROUPING SETS)
- Built lot expiration aging buckets and inventory analytics with Oracle PL/SQL for enterprise supply chain visibility
- **Two-time Employee of the Quarter** award recipient

### JB Hunt Transport — Data Scientist III / GIS Analyst *(April 2010 – April 2022)*
> Lowell, AR | Python · ArcGIS · Azure SQL · Machine Learning · REST APIs · Power BI

- Built **ML models** for payment prediction and transshipment optimization, processing millions of freight events weekly
- Engineered **Predictive Delivery ETA** models achieving **±30 minute accuracy** across long-haul lanes
- Developed GIS spatial analytics tools with real-time weather overlay integration using **ArcGIS** and REST APIs
- Built asset visibility dashboards used by operations teams for fleet and intermodal tracking

---

## Skills & Technologies

| Domain | Technologies |
|---|---|
| **Programming** | Python (Pandas, NumPy, Matplotlib, Scikit-learn), PL/SQL, T-SQL |
| **Databases & ETL** | Snowflake, Oracle Fusion Cloud, Azure SQL, SQLConnect, Azure Data Factory, JDE, OTBI |
| **Visualization & BI** | Tableau, Power BI, Excel (Advanced), OTBI Reports |
| **Cloud & Tools** | Azure (Data Factory, OpenAI Service, SQL), Snowflake Cloud DW, dbt Core |
| **Analytics & Modeling** | Regression (OLS, MLR), ARIMA, GARCH, Value at Risk, Sharpe Ratio, Benford's Law, Exponential Smoothing, Moving Averages |
| **SQL Techniques** | CTEs, Window Functions (ROW_NUMBER, DENSE_RANK, LAG, LEAD), GROUPING SETS, CONNECT BY, LISTAGG, REGEXP_SUBSTR, LOD Expressions |
| **GIS & Integrations** | ArcGIS, Mapbox, Haversine Great-Circle Distance, REST APIs, Geospatial Tableau |

---

## Portfolio Sections

| Section | PDFs | Focus |
|---|---|---|
| [📂 SQL](#-sql-portfolio--oracle-plsql--advanced-query-engineering) | 5 | Oracle PL/SQL · CTEs · Window Functions · GROUPING SETS |
| [📂 dbt](#-dbt-portfolio--analytics-engineering--statistical-modeling) | 14 | dbt Core · Healthcare Claims · Snowflake · Statistical Models |
| [📂 Snowflake](#-snowflake-portfolio--cloud-data-warehouse-infrastructure) | 1 | Cloud DDL · Stage · COPY INTO · Data Pipeline |
| [📂 Tableau](#-tableau-portfolio--data-visualization--business-intelligence) | 6 | LOD · Geospatial · Forecasting · Fraud Detection · Animation |

**Total: 26 portfolio PDFs**

---

## 📂 SQL Portfolio — Oracle PL/SQL · Advanced Query Engineering

> Enterprise SQL written for Oracle Fusion Cloud and JD Edwards (JDE) environments. Code is masked per organizational data governance policies — org codes, GL accounts, and item numbers replaced with descriptive placeholders.

---

### 01 · PPV Analysis — Production Performance Variance

**File:** `sql/01_PPV_Analysis.pdf`

**Business Problem:** Manufacturing operations needed a single, auditable query to surface production performance variance (PPV) across work orders, comparing actual quantities to standard — ranked by plant, item class, and deviation magnitude.

**Technical Highlights:**
- 8 chained CTEs building from raw WO transactions to final ranked output
- `ROW_NUMBER()` for dedup; `DENSE_RANK() KEEP (DENSE_RANK FIRST ORDER BY ...)` for within-group ranking
- Package yield efficiency calculation: `(actual_output / standard_output) * 100`
- Filters for completed WOs only; excludes rework and scrap sub-types

```sql
-- Simplified structure (masked)
WITH base_wo AS (
    SELECT wo_id, plant_code, item_class,
           actual_qty, standard_qty,
           (actual_qty - standard_qty) AS variance_qty
    FROM   [WO_TRANSACTIONS_TABLE]
    WHERE  wo_status = 'COMPLETE'
),
ranked AS (
    SELECT *,
           DENSE_RANK() OVER (PARTITION BY plant_code, item_class
                              ORDER BY ABS(variance_qty) DESC) AS rank_by_variance
    FROM   base_wo
)
SELECT * FROM ranked WHERE rank_by_variance <= 10;
```

**Skills:** `CTEs` · `ROW_NUMBER` · `DENSE_RANK KEEP` · `Window Functions` · `Oracle PL/SQL`

---

### 02 · Labor Shift Analytics — Shift Detail Summary

**File:** `sql/02_Labor_Shift_Analytics.pdf`

**Business Problem:** HR and operations needed a consolidated shift summary report showing labor hours, employee counts, and department rollups with timezone-adjusted clock times — derived from raw badge-swipe and time-card transactional data.

**Technical Highlights:**
- `GROUPING SETS` for simultaneous shift-level and department-level rollups in one pass
- `REGEXP_SUBSTR` to parse department codes from composite cost-center strings
- `LISTAGG` to concatenate employee badge IDs per shift for audit trail
- Oracle `FROM_TZ` / `AT TIME ZONE` for CST→UTC conversion on punch timestamps

```sql
-- Timezone conversion + GROUPING SETS (masked)
SELECT
    REGEXP_SUBSTR(cost_center, '[^-]+', 1, 2)  AS dept_code,
    shift_date,
    SUM(hours_worked)                           AS total_hours,
    COUNT(DISTINCT employee_id)                 AS headcount,
    LISTAGG(employee_id, ', ')
        WITHIN GROUP (ORDER BY employee_id)     AS badge_list,
    GROUPING(shift_date)                        AS is_subtotal
FROM   [TIMECARD_TABLE]
GROUP BY GROUPING SETS (
    (dept_code, shift_date),
    (dept_code)
);
```

**Skills:** `GROUPING SETS` · `REGEXP_SUBSTR` · `LISTAGG` · `Timezone Conversion` · `Oracle PL/SQL`

---

### 03 · Finance Freight Costs — Weighted Allocation & PPV

**File:** `sql/03_Finance_Freight_Costs.pdf`

**Business Problem:** Finance needed freight costs allocated to individual work orders using weighted distribution logic — splitting domestic vs. foreign freight across GL segments and validating against standard cost PPV thresholds.

**Technical Highlights:**
- Weighted allocation: each WO's share = `(WO_qty / total_qty_in_batch) * total_freight_cost`
- GL segment filtering via `SUBSTR` on account string (segments 1–5)
- Domestic/Foreign split using carrier country-of-origin join
- Final PPV calc: `allocated_freight - standard_freight_rate * actual_qty`

```sql
-- Weighted freight allocation (masked)
WITH freight_base AS (
    SELECT batch_id,
           SUM(freight_amount) AS total_batch_freight
    FROM   [FREIGHT_LINES_TABLE]
    WHERE  SUBSTR(gl_account, 4, 6) = '[FREIGHT_ACCOUNT_CODE]'
    GROUP BY batch_id
),
wo_weights AS (
    SELECT wo_id, batch_id,
           actual_qty / SUM(actual_qty) OVER (PARTITION BY batch_id) AS weight
    FROM   [WO_SUMMARY_TABLE]
)
SELECT w.wo_id,
       f.total_batch_freight * w.weight  AS allocated_freight
FROM   wo_weights w
JOIN   freight_base f ON f.batch_id = w.batch_id;
```

**Skills:** `Weighted Allocation` · `GL Segment Parsing` · `SUBSTR` · `Domestic/Foreign Split` · `Oracle PL/SQL`

---

### 04 · Finance Standard Cost Forecast — 24-Month Rolling

**File:** `sql/04_Finance_Std_Cost_Forecast.pdf`

**Business Problem:** Finance required a 24-month forward-looking standard cost forecast that automatically generated future periods, applied UOM conversion rates, and incorporated LAG-based YoY trend factors — without manual date entry.

**Technical Highlights:**
- `CONNECT BY LEVEL <= 24` to auto-generate 24 future monthly periods
- `LAG()` over item/plant for prior-year same-period standard cost baseline
- `CROSS JOIN` to broadcast UOM conversion factors across all forecast rows
- Compound growth formula: `prior_cost * POWER(1 + trend_rate, period_offset)`

```sql
-- Auto-generate 24 future periods (masked)
WITH calendar AS (
    SELECT ADD_MONTHS(TRUNC(SYSDATE, 'MM'), LEVEL - 1) AS forecast_month
    FROM   DUAL
    CONNECT BY LEVEL <= 24
),
cost_baseline AS (
    SELECT item_id, plant_code, cost_period,
           standard_cost,
           LAG(standard_cost, 12) OVER (PARTITION BY item_id, plant_code
                                         ORDER BY cost_period) AS prior_yr_cost
    FROM   [STANDARD_COST_TABLE]
)
SELECT c.forecast_month, b.item_id,
       b.prior_yr_cost * POWER(1 + [TREND_RATE], 1) AS projected_cost
FROM   calendar c
CROSS JOIN cost_baseline b;
```

**Skills:** `CONNECT BY` · `LAG` · `CROSS JOIN` · `UOM Conversion` · `Oracle PL/SQL`

---

### 05 · Inventory Lot Expiration — Aging Buckets

**File:** `sql/05_Inventory_Lot_Expiration.pdf`

**Business Problem:** Supply chain needed a lot expiration aging report bucketing all on-hand inventory into 6 time bands (expired, 0–30, 31–60, 61–90, 91–180, 180+ days) for FIFO compliance and write-off risk assessment.

**Technical Highlights:**
- `SYSDATE` arithmetic to compute days-until-expiration per lot
- 6-bucket `CASE WHEN` classification with negative days = already expired
- `LEFT JOIN` to on-hand quantities to catch lots with zero balance (fan-out guard)
- Aggregate rollup: total on-hand value by bucket, item class, and warehouse

```sql
-- 6-bucket aging classification (masked)
SELECT
    lot_number,
    item_id,
    warehouse_code,
    expiration_date,
    TRUNC(expiration_date) - TRUNC(SYSDATE)  AS days_remaining,
    CASE
        WHEN expiration_date < SYSDATE              THEN 'EXPIRED'
        WHEN expiration_date < SYSDATE + 30         THEN '0-30 Days'
        WHEN expiration_date < SYSDATE + 60         THEN '31-60 Days'
        WHEN expiration_date < SYSDATE + 90         THEN '61-90 Days'
        WHEN expiration_date < SYSDATE + 180        THEN '91-180 Days'
        ELSE                                             '180+ Days'
    END                                              AS aging_bucket,
    NVL(oh.on_hand_qty, 0)                          AS on_hand_qty
FROM   [LOT_MASTER_TABLE] l
LEFT JOIN [ON_HAND_TABLE]  oh ON oh.lot_number = l.lot_number;
```

**Skills:** `SYSDATE Arithmetic` · `CASE WHEN Buckets` · `LEFT JOIN` · `Aging Analysis` · `Oracle PL/SQL`

---

## 📂 dbt Portfolio — Analytics Engineering & Statistical Modeling

> 14 dbt models built on Snowflake for a healthcare claims analytics platform. Full unmasked code — all models are portfolio-safe infrastructure and statistical implementations. Models progress layer by layer: Configuration → Staging → Intermediate Statistical Models → Combined Mart.

---

### 01 · dbt Project Configuration — dbt_project.yml

**File:** `dbt/01_DBT_Project_Config_YAML.pdf`

Project-level YAML defining model paths, materializations by layer (view for staging, table for marts), and variable defaults. Sets `schema` overrides per subfolder so staging models land in `STAGING_SCHEMA` and marts in `MART_SCHEMA` automatically.

**Skills:** `dbt_project.yml` · `Materialization Strategy` · `Schema Routing` · `Variable Defaults`

---

### 02 · Source Definition — sources.yml

**File:** `dbt/02_DBT_Healthcare_Source_YAML.pdf`

Declares the `healthcare` source pointing to `HEALTHCARE_DB.HEALTHCARE_SCHEMA.CLAIMS`. Defines all 14 columns with descriptions, data types, and `not_null` / `unique` test assertions for `entity_id` and `period_date`.

**Skills:** `sources.yml` · `Schema Tests` · `Column Descriptions` · `Source Freshness`

---

### 03 · Staging Model — stg_healthcare_claims.sql

**File:** `dbt/03_DBT_Staging_Healthcare_Claims.pdf`

Type-safe materialized view casting all 14 source columns to explicit types (`::date`, `::float`, `::int`). Adds a `_loaded_at` audit timestamp. Serves as the single source of truth for all downstream intermediate models.

```sql
-- Type casting in staging (full code)
{{ config(materialized='view') }}

SELECT
    entity_id::VARCHAR         AS entity_id,
    payer::VARCHAR             AS payer,
    state::VARCHAR(2)          AS state,
    specialty::VARCHAR         AS specialty,
    period_date::DATE          AS period_date,
    metric_value::FLOAT        AS metric_value,
    claim_volume::INT          AS claim_volume,
    return_value::FLOAT        AS return_value,
    denial_rate::FLOAT         AS denial_rate,
    days_to_payment::INT       AS days_to_payment,
    is_fraud_flag::INT         AS is_fraud_flag,
    y_value::FLOAT             AS y_value,
    x1_value::FLOAT            AS x1_value,
    x2_value::FLOAT            AS x2_value,
    CURRENT_TIMESTAMP          AS _loaded_at
FROM {{ source('healthcare', 'claims') }}
```

**Skills:** `Staging Layer` · `Type Casting` · `Materialized View` · `Audit Columns` · `dbt Source Ref`

---

### 04 · Source Validation Query

**File:** `dbt/04_DBT_Test_Query.pdf`

Ad-hoc validation SQL run against the raw source before dbt models execute. Checks row counts, null rates per column, date range coverage, and fraud flag distribution — confirming the CSV load completed cleanly before downstream processing begins.

**Skills:** `Data Quality` · `Source Validation` · `NULL Auditing` · `Pre-run Checks`

---

### 05 · Moving Averages — 7-Period & 30-Period Rolling

**File:** `dbt/05_DBT_Moving_Average.pdf`

Computes 7-period and 30-period simple moving averages over `metric_value` partitioned by `entity_id` and ordered by `period_date`. Uses `AVG() OVER (ROWS BETWEEN N PRECEDING AND CURRENT ROW)` window frame syntax in Snowflake SQL.

```sql
{{ config(materialized='table') }}

SELECT
    entity_id, payer, state, specialty, period_date, metric_value,
    AVG(metric_value) OVER (
        PARTITION BY entity_id
        ORDER BY period_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS ma_7,
    AVG(metric_value) OVER (
        PARTITION BY entity_id
        ORDER BY period_date
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) AS ma_30
FROM {{ ref('stg_healthcare_claims') }}
```

**Skills:** `Window Functions` · `Rolling Average` · `Snowflake SQL` · `dbt ref()`

---

### 06 · Simple Linear Regression — OLS via Normal Equations

**File:** `dbt/06_DBT_Simple_Linear_Regression.pdf`

Implements Ordinary Least Squares regression in pure SQL using the normal equations `β = (XᵀX)⁻¹Xᵀy`. Computes slope and intercept from `SUM`, `AVG`, and arithmetic — no external Python or UDF required. Outputs fitted values and residuals per row.

**Skills:** `OLS Regression` · `Normal Equations` · `SQL-Native ML` · `Residuals` · `Snowflake SQL`

---

### 07 · Multiple Linear Regression — 2-Predictor OLS via Cramer's Rule

**File:** `dbt/07_DBT_Multi_Linear_Regression.pdf`

Extends OLS to two predictors (`x1_value`, `x2_value`) using Cramer's Rule to solve the 3×3 normal equation system. All matrix operations (determinants, cofactors) expressed as SQL arithmetic — producing `β₀`, `β₁`, `β₂` coefficients and fitted `ŷ`.

**Skills:** `Multiple Regression` · `Cramer's Rule` · `Matrix Arithmetic in SQL` · `2-Predictor OLS`

---

### 08 · Exponential Smoothing — Alpha=0.3, 10-Lag Recursive

**File:** `dbt/08_DBT_Exponential_Smoothing.pdf`

Implements Holt-Winters single exponential smoothing with `α = 0.3` using a 10-step recursive CTE chain. Each CTE computes one lag: `S_t = α * y_t + (1-α) * S_{t-1}`. Jinja2 `{{ var('alpha') }}` makes the parameter configurable at run time.

**Skills:** `Exponential Smoothing` · `Recursive CTE` · `Jinja2 Variables` · `Time Series` · `dbt`

---

### 09 · ARIMA Feature Engineering — Differencing + AR Lags

**File:** `dbt/09_DBT_ARIMA.pdf`

Constructs ARIMA-ready feature columns: first and second order differencing (`d=1`, `d=2`) plus AR lag features (`LAG(1)` through `LAG(3)`) using Snowflake window functions. Output table feeds into downstream regression for AR coefficient estimation.

```sql
-- First & second differencing + AR lags
metric_value - LAG(metric_value, 1) OVER (PARTITION BY entity_id ORDER BY period_date)  AS diff_1,
metric_value - LAG(metric_value, 2) OVER (PARTITION BY entity_id ORDER BY period_date)  AS diff_2,
LAG(metric_value, 1) OVER (PARTITION BY entity_id ORDER BY period_date)                 AS ar_lag_1,
LAG(metric_value, 2) OVER (PARTITION BY entity_id ORDER BY period_date)                 AS ar_lag_2,
LAG(metric_value, 3) OVER (PARTITION BY entity_id ORDER BY period_date)                 AS ar_lag_3
```

**Skills:** `ARIMA` · `Differencing` · `AR Lags` · `LAG()` · `Time Series Feature Engineering`

---

### 10 · GARCH(1,1) — Conditional Volatility Estimation

**File:** `dbt/10_DBT_GARCH.pdf`

Implements GARCH(1,1) conditional variance estimation: `σ²_t = ω + α·ε²_{t-1} + β·σ²_{t-1}`. Parameters `ω`, `α`, `β` are Jinja2 variables, allowing scenario analysis without code changes. Outputs conditional volatility (`√σ²_t`) per entity per period.

```sql
-- GARCH(1,1) conditional variance (Jinja2 parameters)
{{ var('omega') }}
    + {{ var('alpha_garch') }} * LAG(squared_residual, 1) OVER (PARTITION BY entity_id ORDER BY period_date)
    + {{ var('beta_garch') }}  * LAG(cond_variance,    1) OVER (PARTITION BY entity_id ORDER BY period_date)
AS cond_variance
```

**Skills:** `GARCH(1,1)` · `Conditional Volatility` · `Jinja2 Parameters` · `Financial Risk Modeling`

---

### 11 · Value at Risk — Parametric VaR at 90/95/99%

**File:** `dbt/11_DBT_Value_at_Risk.pdf`

Computes parametric Value at Risk at three confidence levels (90%, 95%, 99%) using `μ - z * σ` formula. Mean and standard deviation computed per entity via `AVG()` and `STDDEV_SAMP()`. Z-scores hardcoded as constants (1.282, 1.645, 2.326).

**Skills:** `Value at Risk` · `Parametric VaR` · `Financial Analytics` · `STDDEV_SAMP` · `Risk Modeling`

---

### 12 · Sharpe Ratio — Risk-Adjusted Return, Annualized

**File:** `dbt/12_DBT_Sharpe_Ratio.pdf`

Computes Sharpe Ratio as `(mean_excess_return / σ_p) * √12` — annualizing monthly periodic returns. Risk-free rate `0.004167` (≈ 5% annual / 12) subtracted per row before aggregation. Uses `NULLIF(σ_p, 0)` guard to prevent division by zero.

```sql
{{ config(materialized='table') }}

WITH excess AS (
    SELECT entity_id, payer, state, specialty,
           return_value - 0.004167 AS excess_return
    FROM {{ source('healthcare', 'claims') }}
),
agg AS (
    SELECT entity_id, payer, state, specialty,
           AVG(excess_return)        AS mean_excess_return,
           STDDEV_SAMP(excess_return) AS sigma_p,
           COUNT(*)                  AS n_periods
    FROM excess
    GROUP BY entity_id, payer, state, specialty
)
SELECT *,
       mean_excess_return / NULLIF(sigma_p, 0)             AS sharpe_periodic,
       (mean_excess_return / NULLIF(sigma_p, 0)) * SQRT(12) AS sharpe_annualised
FROM agg
```

**Skills:** `Sharpe Ratio` · `Risk-Adjusted Return` · `Annualization` · `NULLIF Guard` · `Financial Analytics`

---

### 13 · Straight-Line Forecast — 12-Period Compound Growth

**File:** `dbt/13_DBT_Straight_Line_Forecast.pdf`

Generates a 12-period compound growth forecast using a Jinja2 `{% for %}` loop to emit one `UNION ALL` branch per period. Each branch computes `base_value * POWER(1 + growth_rate, n)` where `n` is the loop index. Zero hardcoded period values.

```sql
-- Jinja2 loop generates 12 UNION ALL branches
{% for n in range(1, 13) %}
SELECT entity_id, payer, state, specialty,
       DATEADD('month', {{ n }}, MAX(period_date)) AS forecast_date,
       AVG(metric_value) * POWER(1 + {{ var('growth_rate') }}, {{ n }}) AS forecast_value
FROM {{ ref('stg_healthcare_claims') }}
GROUP BY entity_id, payer, state, specialty
{% if not loop.last %} UNION ALL {% endif %}
{% endfor %}
```

**Skills:** `Jinja2 For Loop` · `Compound Growth` · `UNION ALL Generation` · `12-Period Forecast` · `dbt`

---

### 14 · Combined Mart — Wide Denormalized Output Table

**File:** `dbt/14_DBT_Mart_Combined.pdf`

Joins all 13 upstream models into a single wide mart table for Tableau consumption. One row per `entity_id` / `period_date` carries: staging columns, MA-7, MA-30, OLS fitted values, MLR coefficients, exponential smoothing, ARIMA lags, GARCH volatility, VaR levels, Sharpe ratio, and 12-period forecast.

**Skills:** `Mart Layer` · `Multi-Model Join` · `Wide Denormalized Table` · `Tableau-Ready Output` · `dbt`

---

## 📂 Snowflake Portfolio — Cloud Data Warehouse Infrastructure

> Full infrastructure setup for the healthcare claims analytics platform. Unmasked — all code is portfolio-safe DDL and pipeline configuration.

---

### 01 · Healthcare Claims Database Setup

**File:** `snowflake/01_Snowflake_Healthcare_DB_Setup.pdf`

**Business Problem:** A clean, typed, properly permissioned Snowflake schema was needed as the foundation for 14 downstream dbt models. The infrastructure had to support CSV ingestion from an external stage, handle NULL variants, and recover from malformed rows without blocking the pipeline.

**Technical Highlights:**
- `CREATE OR REPLACE SCHEMA` for isolated analytics environment
- Named external stage (`claims_stage`) with CSV format options: quoted fields, header skip, date parsing, NULL handling
- 14-column typed `CLAIMS` table — explicit `VARCHAR`, `DATE`, `FLOAT`, `INT` — no `VARIANT` catch-alls
- `COPY INTO` with `ON_ERROR = CONTINUE` for fault-tolerant partial loads
- `GRANT WRITE` on stage with least-privilege role assignment
- Verification `COUNT(*)` and `LIMIT 10` to confirm load before dbt runs
- `DROP SCHEMA DEMO.DBT_TFOSTER` cleanup to retire the development sandbox

```sql
-- Full infrastructure pipeline (unmasked)
CREATE OR REPLACE SCHEMA HEALTHCARE_DB.HEALTHCARE_SCHEMA;

CREATE OR REPLACE TABLE HEALTHCARE_DB.HEALTHCARE_SCHEMA.CLAIMS (
    entity_id       VARCHAR,
    payer           VARCHAR,
    state           VARCHAR(2),
    specialty       VARCHAR,
    period_date     DATE,
    metric_value    FLOAT,
    claim_volume    INT,
    return_value    FLOAT,
    denial_rate     FLOAT,
    days_to_payment INT,
    is_fraud_flag   INT,
    y_value         FLOAT,
    x1_value        FLOAT,
    x2_value        FLOAT
);

CREATE OR REPLACE STAGE HEALTHCARE_DB.HEALTHCARE_SCHEMA.claims_stage
FILE_FORMAT = (
    TYPE                      = 'CSV'
    FIELD_OPTIONALLY_ENCLOSED_BY = '"'
    SKIP_HEADER               = 1
    DATE_FORMAT               = 'YYYY-MM-DD'
    NULL_IF                   = ('NULL', 'null', '')
);

GRANT WRITE ON STAGE HEALTHCARE_DB.HEALTHCARE_SCHEMA.claims_stage
    TO ROLE ACCOUNTADMIN;

COPY INTO HEALTHCARE_DB.HEALTHCARE_SCHEMA.CLAIMS
FROM @HEALTHCARE_DB.HEALTHCARE_SCHEMA.claims_stage/healthcare_claims_timeseries.csv
FILE_FORMAT = (
    TYPE                      = 'CSV'
    FIELD_OPTIONALLY_ENCLOSED_BY = '"'
    SKIP_HEADER               = 1
    DATE_FORMAT               = 'YYYY-MM-DD'
    NULL_IF                   = ('NULL', 'null', '')
)
ON_ERROR = 'CONTINUE';

SELECT COUNT(*) FROM HEALTHCARE_DB.HEALTHCARE_SCHEMA.CLAIMS;
SELECT * FROM HEALTHCARE_DB.HEALTHCARE_SCHEMA.CLAIMS LIMIT 10;

DROP SCHEMA DEMO.DBT_TFOSTER;
```

**Skills:** `Snowflake DDL` · `External Stage` · `COPY INTO` · `ON_ERROR=CONTINUE` · `GRANT/REVOKE` · `CSV Pipeline` · `Cloud Data Warehouse`

---

## 📂 Tableau Portfolio — Data Visualization & Business Intelligence

> Six Tableau workbooks spanning geospatial analysis, LOD expressions, statistical fraud detection, forecasting, and animated global data stories. Each PDF includes dashboard screenshots, technical breakdowns, and field-level calculation documentation.

---

### 01 · Startup Quadrant Analysis — Dynamic Filtering & Scatter

**File:** `tableau/01_Tableau_Startup_Quadrant_Analysis.pdf`

**Business Problem:** Investors and accelerators needed to rapidly identify startups that are both financially healthy (high revenue, controlled expenses) and exhibiting exceptional growth — across hundreds of companies simultaneously.

**4 Views:**
- **Industries Bar** — Average employee count by sector; Food & Beverage leads at 393.4, Travel & Hospitality trails at ~20
- **Dynamic Treemap** — Top N startups ranked by 2015 Growth % (Rankfind: 784%, Matelectronics: 776%, down to Zun-Ex: 622%) — driven by parameter
- **Startup Quadrant Scatter** — Revenue ($8.1M cutoff) × Expenses ($4.3M cutoff) reference lines; top 16 growth leaders highlighted as red diamonds
- **Dual-Encoded Scatter** — Adds shape encoding (diamond = Top Growth, circle = Out) and 4-color quadrant classification

**Skills:** `Tableau Sets` · `Parameters` · `Dual Mark Encoding` · `Reference Lines` · `Treemap` · `Dynamic Filtering`

---

### 02 · LOD Expressions & Geospatial Profit Analysis — INCLUDE / FIXED / EXCLUDE

**File:** `tableau/02_Tableau_LOD_Expressions_Geospatial.pdf`

**Business Problem:** Retail analysts needed profit analysis at multiple geographic granularities simultaneously — city level, state level, and country level — without creating separate views for each aggregation.

**4 Views:**
- **LOD Include Choropleth** — `{INCLUDE [City] : SUM([Profit])}` preserves city-level variation at state view; values range −1,828 to 3,189 vs. plain avg range of −71.7 to 121.6
- **LOD Fixed (1) Choropleth** — `{FIXED [Country] : SUM([Profit])}` as ratio denominator — side-by-side with/without LOD comparison
- **LOD Fixed (2) Bubble Map** — City-level profit ratios relative to state totals; bubble size = ABS(ratio), color = direction
- **Combined Overlay** — LOD Include choropleth + LOD Exclude bubbles on single US map with dual legend

**Key Insight:** The side-by-side maps make LOD impact visually obvious — INCLUDE preserves sub-level variation that simple aggregates collapse and obscure.

**Skills:** `LOD INCLUDE` · `LOD FIXED` · `LOD EXCLUDE` · `Choropleth Maps` · `Proportional Symbol Map` · `Dual-Layer Map` · `Mapbox`

---

### 03 · Store Distance & Geospatial Range Analysis — NYC Satellite Map

**File:** `tableau/03_Tableau_Store_Distance_NYC.pdf`

**Business Problem:** A retail chain needed to identify which NYC store locations fall outside their defined service radius for staffing, delivery routing, and coverage gap analysis — on an intuitive aerial view of the city.

**1 View:**
- **Satellite Aerial Map** — Mapbox satellite tile base; coffee cup custom shapes encode In Range (blue) vs. Out of Range (orange); icon size encodes Minimum NY Great-Circle Distance (0.0724 to 1.0292 units)
- Out-of-range stores cluster along western Manhattan waterfront and lower Manhattan — highlighting service gaps in high-density zones

**Skills:** `Custom Shapes` · `Satellite Background Map` · `Haversine Distance` · `Dual Mark Encoding` · `Geospatial Analysis` · `Mapbox`

---

### 04 · SPC Sales Per Capita Analysis — Australian Retail Forecasting

**File:** `tableau/04_Tableau_SPC_Sales_Analysis.pdf`

**Business Problem:** Retail analysts needed simultaneous regional benchmarking and forward-looking sales planning for 4 Australian regions (NSW, QLD, VIC, WA) from 2000–2019, packaged for C-suite presentation.

**5 Views:**
- **Competitors Analysis** — Box-and-whisker + dot overlay per region; NSW median ~8%, QLD ~6%, VIC ~7%, WA widest spread
- **Industry Analysis** — Faceted time series by region (NSW peaks ~600M turnover); all 4 regions show seasonal oscillation with clear upward trends
- **Sales Per Capita Value** — SPC $ time series; VIC shows steepest growth trajectory, QLD SPC essentially flat despite high turnover
- **SPC Forecast** — Tableau forecast engine with confidence bands extending 2016–2019; shaded uncertainty bands per region
- **SPC Presentation Story** — Fully navigable 4-tab Tableau Story for board-level delivery

**Skills:** `Box Plot` · `Faceted Time Series` · `Tableau Forecasting` · `Confidence Bands` · `Tableau Story` · `Multi-Region Analysis`

---

### 05 · Benford's Law Fraud Detection — Madoff Financial Records

**File:** `tableau/05_Tableau_Benfords_Law_Fraud.pdf`

**Business Problem:** Financial auditors and compliance teams needed a visual framework to detect digit-frequency anomalies in financial records — demonstrated on Bernie Madoff's actual reported returns vs. the expected Benford distribution.

**4 Views:**
- **Base Case** — Natural orders dataset vs. Benford expected curve; digit 1 ≈ 30.10%, digit 9 ≈ 4.58%
- **Madoff Records** — Actual digit '1' frequency ≈ 39% vs. expected 30.10% — **9.29% excess** is the fraud signal; bar chart + Benford line overlay
- **Deviation Chart** — Above/below-zero deviation bars per digit; digit 1 most over-represented (+9.29%), digits 2–5 all under-represented
- **Story Presentation** — Annotated tabbed story: base case → Madoff anomaly → conclusion

**Key Finding:** Madoff's data showed statistically impossible conformance to Benford's Law — a distribution that naturally-occurring financial data always follows.

**Skills:** `Benford's Law` · `Dual-Axis Bar+Line` · `Deviation Chart` · `Fraud Detection` · `Annotated Story` · `Financial Analytics`

---

### 06 · World Life Expectancy — Animated Scatter Story

**File:** `tableau/06_Tableau_World_Life_Expectancy.pdf`

**Business Problem:** Development economists and public health analysts needed an animated visual showing how the global fertility-life expectancy relationship has evolved across decades — across ~180 countries grouped by world region.

**1 Animated View within Tableau Story:**
- **X-axis:** Fertility Rate (1.0 to 8.0+ births/woman)
- **Y-axis:** Life Expectancy (30–85 years)
- **Bubble size:** Population (country-level)
- **Color encoding:** 7 world regions (Sub-Saharan Africa, South Asia, North America, Middle East & North Africa, Latin America & Caribbean, Europe & Central Asia, East Asia & Pacific)
- **Pages shelf:** Animated by Year — steps through decades from ~1960s onward
- **Shown at 1967:** Sharp negative correlation visible; Sub-Saharan Africa and South Asia cluster lower-right (high fertility, low life expectancy); Europe and North America cluster upper-left
- **Show history trail:** Reveals country trajectories over time — the "demographic transition" arc

**Skills:** `Pages Shelf Animation` · `Bubble Chart` · `7-Region Color Encoding` · `Show History Trail` · `Tableau Story` · `Global Health Analytics`

---

## Repository Structure

```
thomas-foster-portfolio/
│
├── README.md                                      ← Master portfolio landing page (this file)
│
├── sql/
│   ├── 01_PPV_Analysis.pdf                        ← Production Performance Variance
│   ├── 02_Labor_Shift_Analytics.pdf               ← Labor Shift Detail Summary
│   ├── 03_Finance_Freight_Costs.pdf               ← Freight Allocation & PPV
│   ├── 04_Finance_Std_Cost_Forecast.pdf           ← 24-Month Rolling Standard Cost Forecast
│   └── 05_Inventory_Lot_Expiration.pdf            ← Lot Expiration Aging Buckets
│
├── dbt/
│   ├── README.md                                  ← dbt portfolio section README
│   ├── 01_DBT_Project_Config_YAML.pdf             ← dbt_project.yml configuration
│   ├── 02_DBT_Healthcare_Source_YAML.pdf          ← sources.yml with 14-column claims table
│   ├── 03_DBT_Staging_Healthcare_Claims.pdf       ← Type-safe staging view
│   ├── 04_DBT_Test_Query.pdf                      ← Source validation query
│   ├── 05_DBT_Moving_Average.pdf                  ← 7-period & 30-period rolling averages
│   ├── 06_DBT_Simple_Linear_Regression.pdf        ← OLS via normal equations
│   ├── 07_DBT_Multi_Linear_Regression.pdf         ← 2-predictor OLS via Cramer's Rule
│   ├── 08_DBT_Exponential_Smoothing.pdf           ← Alpha=0.3 weighted smoothing
│   ├── 09_DBT_ARIMA.pdf                           ← d=1,2 differencing + AR(1-3) lags
│   ├── 10_DBT_GARCH.pdf                           ← GARCH(1,1) conditional volatility
│   ├── 11_DBT_Value_at_Risk.pdf                   ← Parametric VaR at 90/95/99%
│   ├── 12_DBT_Sharpe_Ratio.pdf                    ← Risk-adjusted return, annualized
│   ├── 13_DBT_Straight_Line_Forecast.pdf          ← 12-period compound growth via Jinja2
│   └── 14_DBT_Mart_Combined.pdf                   ← Wide mart joining all models for Tableau
│
├── snowflake/
│   └── 01_Snowflake_Healthcare_DB_Setup.pdf       ← Schema, Stage, Table, COPY INTO pipeline
│
└── tableau/
    ├── 01_Tableau_Startup_Quadrant_Analysis.pdf   ← Industries bar, treemap, quadrant scatter
    ├── 02_Tableau_LOD_Expressions_Geospatial.pdf  ← INCLUDE/FIXED/EXCLUDE choropleth maps
    ├── 03_Tableau_Store_Distance_NYC.pdf           ← Satellite map, Haversine distance
    ├── 04_Tableau_SPC_Sales_Analysis.pdf          ← Box plots, time series, SPC forecast
    ├── 05_Tableau_Benfords_Law_Fraud.pdf          ← Benford's Law, Madoff fraud analysis
    └── 06_Tableau_World_Life_Expectancy.pdf       ← Animated scatter, Pages shelf, 7 regions
```

---

## Education

**PhD in Data Science** *(In Progress)*
Concentration: AI Engineering & Healthcare Claims Analytics

**Bachelor of Science** — Information Systems / Business Analytics

---

## Contact

| | |
|---|---|
| **Email** | thomasfoster79@yahoo.com |
| **Phone** | (540) 570-6094 |
| **Location** | Pea Ridge, AR 72751 |
| **GitHub** | [github.com/thomasfoster](https://github.com/thomasfoster) |

---

*© 2026 Thomas E. Foster — All portfolio work represents original analysis and code. SQL projects use masked identifiers per organizational data governance requirements. dbt, Snowflake, and Tableau projects use full unmasked code.*
