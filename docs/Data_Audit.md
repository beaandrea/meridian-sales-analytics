# Phase 1: Data Quality Triage & Issue Log

**Project:** Sales Force Capacity & Territory Analytics (Meridian IS)
**Dataset:** Maven CRM Sales Opportunities (Raw)
**Objective:** Clean raw B2B pipeline data, engineer custom capacity metrics, standardize categorical dimensions, and prepare a unified data model for SQL analysis.

### Executive Summary
The initial audit of the raw CRM extract revealed several legacy data entry errors and formatting inconsistencies. Categorical dimensions (Account Names, Sectors) required text standardization, while core pipeline dates required type conversion to allow for velocity calculations. The data was successfully cleaned, merged with the dimension tables, augmented with a custom `Capacity Score`, and exported to a pristine processed dataset for analysis.

### Issue Log

| Table | Column | Issue | Magnitude | Solvable? | Resolution Plan |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `sales_pipeline` | `engage_date`, `close_date` | **Invalid Data Type:** Loaded as strings instead of datetime objects. | High | Yes | Force standardization via Pandas `to_datetime()` to enable duration math. |
| `sales_pipeline` | `close_value` | **Expected Nulls:** Missing values represent active deals that haven't closed. | High | Yes | Impute with `0` so downstream aggregation math (like SUMs) doesn't break on NaN values. |
| `sales_pipeline` | `account` | **Missing Core Dimension:** Leads not yet linked to formal CRM accounts. | Medium | Yes | Impute with `"Unassigned"` to prevent data loss when grouping by account. |
| `sales_pipeline` | `engage_date` | **Missing Velocity Metric:** Deals missing their start dates. | Medium | Yes | Quarantine these specific rows when calculating the `sales_cycle_days` average so they do not skew the velocity metrics. |
| `accounts` | `account` | **Inconsistent Formatting:** Mixed casing (lowercase, sentence case) across entity names. | Low | Yes | Apply Pandas `.str.title()` to standardize all account names to Title Case. |
| `accounts` | `sector` | **Data Entry Error:** Misspelled categorical value ("technology"). | Low | Yes | Map and overwrite the misspelled string using Pandas `.replace()`. |

<br>

# Phase 2: Strategic SQL EDA & Business Insights

**Project:** Sales Force Capacity & Territory Analytics (Meridian IS)
**Dataset:** Cleaned `meridian_pipeline_clean` table (SQLite)
**Objective:** Execute business-focused exploratory data analysis (EDA) using the SCAN framework to test competing stakeholder hypotheses (territory saturation vs. rep burnout).

### Requirements Gathering
Before writing SQL aggregations, the domain context and North Star metrics were established to guide the analysis:
* **Stakeholder Goals:** Resolve the executive gridlock. The VP of Sales suspects a market plateau, Sales Ops suspects territory imbalance, and Finance requires utilization proof before approving headcount. 
* **Columns & Coverage:** Analysis leverages the fully standardized Phase 1 output, specifically isolating `regional_office` and `sales_agent` against our engineered metrics (`sales_cycle_days` and `capacity_score`).

---

### Insights Log

| SQL Query Focus | Metric & Dimension | The Finding | Relevant Stakeholder | Domain Context (Why it matters) |
| :--- | :--- | :--- | :--- | :--- |
| **Notable Segments** (Rep Capacity) | `capacity_score` by `sales_agent` | **Severe isolated rep burnout.** Sales Agent Darcel Schlecht is carrying a massive operational load with a capacity score (5.28M) over 2.5x higher than the next highest rep (2.02M). | Sales Ops / VP of Sales | Darcel is managing 747 total deals while maintaining a competitive 46% win rate. This is textbook structural overload and indicates a severe misallocation of accounts to a single top-performer. |