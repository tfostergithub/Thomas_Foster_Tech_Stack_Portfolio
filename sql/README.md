# Thomas Foster — SQL Analytics Portfolio

> **Data Analyst · Oracle Fusion ERP · Manufacturing & Finance Analytics**
> 📧 thomasfoster79@yahoo.com | 📍 Pea Ridge, AR

---

## About This Portfolio

This repository showcases production-grade SQL queries built inside **Oracle Fusion ERP** across manufacturing, finance, maintenance, and warehouse domains. Every query in this portfolio was written to solve a real business problem — replacing manual Excel reconciliations, enabling cross-functional drill-through reporting, and delivering parameterized, reusable analytics for operations and finance teams.

Each project includes:
- ✅ Business problem statement
- ✅ Technical design explanation
- ✅ Masked SQL (sensitive schema details removed)
- ✅ Sample output
- ✅ Full PDF portfolio document

---

## Skills & Technologies

![Oracle SQL](https://img.shields.io/badge/Oracle%20SQL-F80000?style=for-the-badge&logo=oracle&logoColor=white)
![Oracle Fusion ERP](https://img.shields.io/badge/Oracle%20Fusion%20ERP-F80000?style=for-the-badge&logo=oracle&logoColor=white)
![PL/SQL](https://img.shields.io/badge/PL%2FSQL-F80000?style=for-the-badge&logo=oracle&logoColor=white)

| Category | Skills |
|---|---|
| **Query Design** | CTEs, Window Functions, GROUPING SETS, CONNECT BY, CROSS JOIN |
| **Analytics** | Variance Analysis, Aging Buckets, Rolling Forecasts, Weighted Allocation |
| **ERP Modules** | Manufacturing (WIE), Accounts Payable, GL, Costing (CST), Inventory, Purchasing |
| **Techniques** | ROW_NUMBER(), LAG(), DENSE_RANK KEEP, REGEXP_SUBSTR, LISTAGG, NULLIF, NVL |
| **Data Handling** | Timezone conversion, UOM conversion, Parameterized queries, NULL-safe guards |

---

## Projects

---

### 1. 📊 PPV Analysis — Production Performance Variance
📄 [`01_PPV_Analysis.pdf`](./01_PPV_Analysis.pdf)

**Business Problem**
Production teams needed a single, reliable report to quantify how much they produced vs. what was planned per work order — and where cases were lost to rework. Previously, analysts reconciled four separate Excel exports, a process that took hours and introduced errors.

**What It Does**
- Computes **planned vs. actual cases produced** per work order, item, and plant
- Calculates **production variance** (absolute and %) and **package yield %** (good cases ÷ total output)
- Layers in **rework counts** (dented and scrapped cases) from quality category assignments
- Resolves **container size** attributes per item using a plant-preferred category hierarchy

**Key SQL Techniques**
```sql
-- ROW_NUMBER() to pick best container-size category (plant-specific preferred)
ROW_NUMBER() OVER (
  PARTITION BY csa.inventory_item_id
  ORDER BY CASE WHEN csa.organization_id = :PLANT_CODE THEN 0
                WHEN csa.organization_id IS NULL        THEN 1
                ELSE 2 END, csa.can_size_code
) rn

-- Package yield with NULL-safe division
ROUND(cases_produced / NULLIF(cases_produced + cases_scrapped + cases_dented, 0), 4)
  AS pkg_yield


### 2. 👷 Labor Analytics — Shift Detail Summary
📄 ['02_Labor_Shift_Analytics.pdf']

**Business Problem**
Maintenance supervisors needed a hierarchical view of labor hours charged per technician, per shift, and per plant — including work-order on-time flags — without pulling from multiple systems and reconciling manually.

**What It Does**
- Joins work orders → operation resources → labor instances across Oracle Fusion Maintenance
- Parses shift identifier from the labor instance code string using REGEXP_SUBSTR
- Converts minutes-based UOM to hours where applicable
- Produces four roll-up levels in a single query: technician → shift → plant → grand total

**Key SQL Techniques**
```sql

Copy
-- Parse shift character from encoded labor instance code
CASE WHEN REGEXP_COUNT(WL.LABOR_INSTANCE_CODE, '-') >= 2 THEN
  SUBSTR(WL.LABOR_INSTANCE_CODE, INSTR(WL.LABOR_INSTANCE_CODE,'-',1,2)+1, 1)
END AS SHIFT

-- GROUPING SETS for four subtotal levels in one pass
GROUP BY GROUPING SETS (
  (PLANT_CODE, SHIFT, UNIQUE_RESOURCE),  -- detail
  (PLANT_CODE, SHIFT),                   -- shift subtotal
  (PLANT_CODE),                          -- plant subtotal
  ()                                     -- grand total
)