# Meridian IS: Sales Force Capacity & Territory Analytics

## Executive Summary
Meridian Industrial Solutions, a B2B distributor, hit a sudden revenue plateau, causing an executive deadlock over the root cause. I analyzed 9,000+ CRM records using Python and SQL to determine whether the issue was a market cooling, territory imbalances, or artificial pipeline inflation. I delivered a 3-page executive Power BI dashboard that identified severe pipeline concentration and provided a mathematically backed strategy to unlock $4.93M in stalled pipeline value.

## The Business Questions
* **The Territory Question:** Which territories are structurally over-resourced or under-resourced relative to their revenue potential?
* **The Capacity Question:** Which sales reps are carrying a disproportionate quota load, and is their performance (e.g., win rate, deal velocity) reflecting that strain?
* **The Utilization Question:** What percentage of each rep's active pipeline is realistically winnable given their current win rate and cycle length, and where is the pipeline being artificially inflated by deals that will never close?
* **The ROI Question:** What does a rebalanced territory and quota allocation look like, and what is the projected revenue impact?

## Data Source & Methodology
**Data Source:** Data sourced from Maven Analytics' CRM Sales Opportunities dataset (B2B sales pipeline data from a fictitious computer hardware company) and adapted to the Meridian IS scenario, including territory remapping, derived metrics (Revenue per Rep, Pipeline Value at Risk), and stakeholder framing to simulate enterprise sales operations conditions.

1. **Data Triage (Python/Pandas):** Ingested and cleaned raw CRM extracts (`sales_pipeline.csv`, `accounts.csv`, `teams.csv`). *See `audit_log.md` for the full data quality and imputation breakdown.*
2. **Exploratory Data Analysis (SQL):** Executed the SCAN framework *(Stakeholders, Columns, Aggregates, Notable Segments)* to query territory volume vs. revenue efficiency. *See `audit_log.md` for the SQL insight logs.*
3. **Executive Dashboard (Power BI & DAX):** Built an interactive, 3-page reporting layer to visualize the bottlenecks and calculate the exact financial ROI of restructuring.

> **Analytical Note on Pipeline Value at Risk:** The raw dataset contained four stages: *Won, Lost, Engaging,* and *Prospecting*. Because there was no traditional "In Progress" designation, the "Pipeline Value at Risk" was calculated by isolating deals in the *Engaging* and *Prospecting* stages and multiplying them by the historical average value of a *Won* deal. 

## Data Architecture
Three raw CSV extracts were cleaned and modeled into a unified analytical dataset:

* `sales_pipeline`: Core fact table (grain: one row per deal opportunity)
* `accounts`: Account dimension (company name, sector, region)
* `teams`: Sales rep and manager dimension

**Key Engineering Decisions:**
* `close_value` nulls imputed with $0 for active deals, valued separately via Pipeline Value at Risk methodology to avoid undercounting active pipeline.
* `sales_cycle_days` derived as `close_date` minus `engage_date` using Pandas datetime conversion. 50 rows with missing `engage_date` were quarantined to prevent velocity metric skew.
* `Pipeline Value at Risk` calculated as: Active/Prospecting deals multiplied by historical average Won deal value (no "In Progress" stage existed in this dataset).

📄 **[View the complete Data Quality & SQL Insights Audit Here](docs/Data_Audit.md)**

---

## The Executive Dashboard

### 1. Territory Overview
![Territory Overview](images/page1_territory.png)
*Proves that the Central Region's high lead volume is heavily eroding revenue efficiency.*

### 2. Representative Utilization
![Rep Utilization](images/page2_utilization.png)
*Exposes pipeline concentration bottlenecks (194 active deals for a single rep) and visually connects the risk back to CRM management routing rules.*

### 3. Executive Recommendations & ROI Math
![Executive ROI](images/page3_roi.png)
*Provides the mathematical derivation proving that redistributing stalled pipeline to underutilized reps can unlock immediate revenue without expanding the sales team.*

---

## Strategic Recommendations & ROI

The data strongly links Meridian IS's revenue plateau to internal territory imbalances and localized routing bottlenecks, rather than external market cooling. By pulling the right operational levers, the business can unblock stalled deals and unlock an estimated **$4.93M** in total pipeline value at risk.

| Category | The Finding | The Business Lever | Recommendation |
| :--- | :--- | :--- | :--- |
| **Actionable** *(Short-Term)* | **Melvin Marxen routed 747 deals to one rep.** | *High Control.* RevOps controls CRM lead-routing automation. | **Implement Lead Caps.** Institute a hard cap on active pipeline volume (e.g., 80 active deals per rep). Force managers to distribute leads evenly. |
| **Actionable** *(Strategic)* | **Finance refuses to approve headcount.** Data shows reps holding 100+ active deals. | *High Control.* Sales leadership dictates pipeline cleanup protocols. | **Enforce the "Close-Lost" Rule.** Require reps to immediately close-out stalled deals. Once the bloat is cleared, Finance will have an accurate utilization baseline to approve targeted hiring. |
| **Directional** | **Central region volume is 50% higher than the East.** | *Medium Control.* Highlights a broken territory mapping model. | **Redraw Territory Lines.** Shift lead routing criteria so that border-state inbound leads default to the underutilized East region rather than the flooded Central region. |