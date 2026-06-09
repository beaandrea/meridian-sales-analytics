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
| **Aggregates** (Territory Efficiency) | `total_won_revenue` vs `total_deal_volume` by `regional_office` | **Severe Regional Workload Imbalance.** Despite identical headcounts (10 reps per region), the Central office handles 50% more lead volume than the East (3,512 vs 2,291), yet yields lower total revenue and a lower win rate than the West. | VP of Sales / RevOps | Proves the revenue plateau is a territory distribution problem. Headcount is evenly distributed, but lead volume is not. Reps in the Central region are oversaturated, leading to lower conversion efficiency. |
| **Notable Segments** (Pipeline Utilization) | `active_pipeline` by `sales_agent` | **Artificial Pipeline Bloat.** Top-performing reps are structurally overloaded, with Darcel Schlecht holding 194 active deals simultaneously. Half of the top 10 most overloaded reps sit in the Central region. | Finance (FP&A) | Validates Finance's hesitation to add headcount. Meridian doesn't need to hire more reps; they need to take stalled, active leads away from oversaturated reps in the Central region and reassign them. |
| **Notable Segments** (Management Bottlenecks) | `assigned_deals` by `manager` in Central Region | **Mismanaged Lead Routing.** Manager Melvin Marxen is routing 747 deals to a single rep (Darcel Schlecht), which is more than double the volume assigned to his next busiest rep (345 deals). | RevOps / VP of Sales | Burnout is not just regional; it is driven by localized manager routing rules. Top performers are being treated as catch-alls for inbound leads, causing massive pipeline bottlenecks. |

---

### Strategic Recommendations

To ensure these data findings translate into tangible business value, the insights have been categorized based on the levers the executive team can actually pull.

| Category | The Finding | The Business Lever | Recommendation |
| :--- | :--- | :--- | :--- |
| **Directional** | **Central region volume is 50% higher than the East.** | *Medium Control.* Highlights a broken territory mapping model. | **Redraw Territory Lines.** Shift lead routing criteria so that border-state inbound leads default to the underutilized East region rather than the flooded Central region. |
| **Actionable** *(Short-Term)* | **Melvin Marxen is dumping 747 deals onto one rep.** | *High Control.* RevOps controls CRM lead-routing automation. | **Implement Lead Caps.** Institute a hard cap on active pipeline volume (e.g., 80 active deals per rep). Force managers to distribute leads evenly across the team rather than relying solely on top performers. |
| **Actionable** *(Strategic)* | **Finance refuses to approve headcount.** Data shows current reps are hoarding 100+ active deals. | *High Control.* Sales leadership dictates pipeline cleanup protocols. | **Enforce the "Close-Lost" Rule.** Require reps to immediately close-out stalled deals. Once the artificial bloat is cleared, Finance will have an accurate utilization baseline to approve targeted hiring in the Central region. |