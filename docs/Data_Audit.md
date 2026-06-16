# Phase 1: Data Quality Triage & Issue Log

**Project:** Sales Force Capacity & Territory Analytics (Meridian IS)  
**Dataset:** Maven CRM Sales Opportunities (Raw)  

### Executive Summary
The initial audit of the raw CRM extract revealed legacy data entry errors and formatting inconsistencies. Categorical dimensions required text standardization, while core pipeline dates required type conversion. The data was successfully cleaned, augmented with a custom `Capacity Score`, and exported for analysis.

### Issue Log

| Table | Column | Issue | Row Count | Magnitude | Solvable? | Resolution Plan |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `sales_pipeline` | `engage_date`, `close_date` | **Invalid Data Type:** Loaded as strings instead of datetime objects. | ~8,800 | High | Yes | Force standardization via Pandas `to_datetime()` to enable duration math. |
| `sales_pipeline` | `close_value` | **Expected Nulls:** Missing values represent active deals. | ~2,000 | High | Yes | Impute with `$0` to prevent aggregation errors. *Methodology Note: This field is only used for closed deals; active deals are valued separately using the Pipeline Value at Risk methodology, so this $0 imputation does not undercount active pipeline calculations.* |
| `sales_pipeline` | `account` | **Missing Core Dimension:** Leads not yet linked to accounts. | ~1,420 | Medium | Yes | Impute with `"Unassigned"` to prevent data loss when grouping. |
| `sales_pipeline` | `engage_date` | **Missing Velocity Metric:** Deals missing start dates. | ~50 | Medium | Yes | Quarantine these specific rows when calculating `sales_cycle_days` so they do not skew velocity metrics. |
| `accounts` | `account` | **Inconsistent Formatting:** Mixed casing across entity names. | ~1,200 | Low | Yes | Apply Pandas `.str.title()` to standardize all account names. |
| `accounts` | `sector` | **Data Entry Error:** Misspelled categorical value ("technology"). | ~150 | Low | Yes | Map and overwrite the misspelled string using Pandas `.replace()`. |

<br>

# Phase 2: Strategic SQL EDA & Business Insights

### Requirements Gathering
* **Stakeholder Goals:** Resolve executive gridlock. The VP of Sales suspects a market plateau, Sales Ops suspects territory imbalance, and Finance requires utilization proof before approving headcount. 
* **Columns & Coverage:** Analysis leverages the standardized pipeline dataset, specifically focusing on `regional_office`, `sales_agent`, and `deal_stage` to map territory volume, alongside engineered metrics (`sales_cycle_days`, `active_deals`) to evaluate individual capacity ceilings.

---

### Insights Log

| SQL Query Focus | Metric & Dimension | The Finding | Relevant Stakeholder | Domain Context (Why it matters) |
| :--- | :--- | :--- | :--- | :--- |
| **Aggregates** (Territory Efficiency) | `total_won_revenue` vs `total_deal_volume` by `regional_office` | **Regional Imbalance.** The Central office handles 50% more lead volume than the East, yet yields lower total revenue and a lower win rate than the West. | VP of Sales / RevOps | Indicates that the Central region is operating beyond its current capacity ceiling, eroding overall revenue efficiency. |
| **Notable Segments** (Pipeline Utilization) | `active_pipeline` by `sales_agent` | **Artificial Bloat.** Top-performing reps are structurally overloaded, with Darcel Schlecht holding 194 active deals simultaneously. | Finance (FP&A) | Validates Finance's hesitation to add headcount. The presence of massive unclosed pipelines masks the true utilization rate of the existing sales force. |
| **Notable Segments** (Management Bottlenecks) | `assigned_deals` by `manager` in Central Region | **Mismanaged Lead Routing.** Manager Melvin Marxen routed 747 deals to a single rep, more than double the volume assigned to his next busiest rep. | RevOps / VP of Sales | Whether this is intentional manager behavior or a CRM routing default is unclear without further investigation, but the volume concentration itself is a structural risk regardless. |