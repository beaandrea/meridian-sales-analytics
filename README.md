# Meridian IS: Sales Force Capacity & Territory Analytics

> **Tools:** Python (Pandas) · SQL · Power BI (DAX) · Star Schema Modeling

## Executive Summary
Meridian Industrial Solutions' sales revenue flatlined for four consecutive quarters, triggering an executive deadlock: the VP of Sales blamed market conditions, Sales Ops suspected territory imbalances, and Finance refused to approve new headcount without proof the existing team was being fully utilized. I analyzed 9,000+ CRM records across three regional offices to determine the root cause. The data ruled out market cooling and instead exposed a CRM routing failure that had artificially concentrated 194 active deals onto a single rep — locking **$4.93M in stalled pipeline** that could be recovered without adding a single headcount.

## The Business Questions
These three questions represent the direct asks from each executive stakeholder:
 
1. **Territory (VP of Sales):** Which regional offices are structurally over- or under-resourced relative to their revenue output — and is Central's volume advantage actually translating to proportional revenue?
2. **Capacity (Sales Ops / RevOps):** Which reps are carrying a disproportionate active deal load, and does their win rate and deal volume data reflect unsustainable pipeline concentration?
3. **ROI (Finance / FP&A):** What is the minimum quantifiable revenue unlock from redistributing stalled pipeline — without requiring new headcount approval?

> **Data Constraint Note:** The raw dataset contains no lead source, lead score, or BANT qualification metadata. Regional performance differences are therefore evaluated purely on conversion efficiency and volume distribution — a limitation that rules out lead quality as an alternative explanation for the findings.

## Data Source & Methodology
**Data Source:** Data sourced from Maven Analytics' CRM Sales Opportunities dataset (B2B sales pipeline data from a fictitious computer hardware company) and adapted to the Meridian IS scenario, including territory remapping, derived metrics (Revenue per Rep, Pipeline Value at Risk), and stakeholder framing to simulate enterprise sales operations conditions.

1. **Data Triage (Python/Pandas):** Ingested and cleaned raw CRM extracts (`sales_pipeline.csv`, `accounts.csv`, `teams.csv`). *See `audit_log.md` for the full data quality and imputation breakdown.*
2. **Exploratory Data Analysis (SQL):** Executed the SCAN framework *(Stakeholders, Columns, Aggregates, Notable Segments)* to query territory volume vs. revenue efficiency. *See `audit_log.md` for the SQL insight logs.*
3. **Executive Dashboard (Power BI & DAX):** Built an interactive, 3-page reporting layer to visualize the bottlenecks and calculate the exact financial ROI of restructuring.

> **Analytical Note on Pipeline Value at Risk:** The raw dataset contained four stages: *Won, Lost, Engaging,* and *Prospecting*. Because there was no traditional "In Progress" designation, the "Pipeline Value at Risk" was calculated by isolating deals in the *Engaging* and *Prospecting* stages and multiplying them by the historical average value of a *Won* deal. 

## Data Architecture
 
Three raw CSV extracts were modeled into a unified star schema for Power BI:
 
| Table | Role | Grain |
| :--- | :--- | :--- |
| `sales_pipeline` | **Fact Table** | One row per deal opportunity |
| `accounts` | **Dimension** | Company name, sector, assigned region |
| `teams` | **Dimension** | Sales rep, manager, regional office |
 
**Key Engineering Decisions:**
 
- `close_value` nulls (≈2,000 rows) imputed with `$0` for active deals. Active pipeline is valued separately via **Pipeline Value at Risk** (Active Deals × Avg. Won Deal Value) to prevent undercounting.
- `sales_cycle_days` derived as `close_date − engage_date` using Pandas datetime conversion. 50 rows with missing `engage_date` were quarantined from velocity calculations to prevent metric skew.
- `Capacity Score` engineered as `Active Deal Count ÷ Average Win Rate` to surface reps carrying disproportionate workload relative to their actual conversion output.
- Account names standardized via `.str.title()`; misspelled sector values corrected via `.replace()` before loading to Power BI semantic layer.

📄 *[Full Data Quality & SQL Audit Log →](docs/Data_Audit.md)*

## Insights Deep-Dive
 
### Finding 1 — Territory Imbalance Is Real, But Not Where the VP Expected
 
Central carries **50% more deal volume** than East (3.5K vs. 2.3K deals), yet generates lower revenue per rep ($0.33M) than West ($0.36M) — despite all three regions posting nearly identical win rates (~63%). The volume advantage is not translating to proportional revenue. This rules out win rate as the differentiator and points directly to pipeline structure: Central reps are processing more deals per head than the regional model was designed to support.
 
> *The data does not support a market plateau hypothesis. Win rates are stable across all three regions. The revenue gap is a capacity problem, not a demand problem.*
 
### Finding 2 — A Single Routing Rule Created a Concentration Bomb
 
Manager Melvin Marxen (Central) routed **747 total deals to one rep** — Darcel Schlecht — more than double the volume assigned to any other rep on his team. Darcel currently holds **194 simultaneous active deals**, 2.5× the Central team average. This is not a performance failure on Darcel's part (her win rate holds at 63.11%), but a structural routing failure: her pipeline has scaled past any realistic closing capacity, and the deals at the bottom of her queue are statistically unlikely to ever receive attention.
 
A secondary concentration risk exists under a different manager: Anna Snelling (Dustin Brinkmann's team) holds 112 active deals, confirming the routing failure is a CRM default behavior, not an isolated management style.

Velocity data confirms this: Darcel's average close cycle of 45.84 days sits below the Central team average of 48.49 days, meaning her throughput is holding above the team norm despite carrying 2.5× the active deal load. The stalled pipeline is a routing problem, not a rep performance problem.
 
### Finding 3 — $4.93M Is Stalled. $169K Is the Conservative, Immediate Floor.
 
Company-wide Pipeline Value at Risk totals **$4.93M** in active/prospecting deals that are stalled due to rep overload. As a conservative, actionable proof of concept: redistributing just Darcel's 114 excess active deals (above a proposed 80-deal cap) to underutilized East-region reps — applying East's historical win rate of 63.02% and average deal value of $2,360.91 — projects an **immediate revenue unlock of $169,627**.
 
This figure is intentionally conservative. It models only one rep's overflow, applied to one receiving region, and does not account for additional pipeline recovery from Anna Snelling or other overloaded reps across the Central and West offices.

---

## Operational Recommendations
 
| Priority | Finding | Recommended Action | Expected Impact |
| :--- | :--- | :--- | :--- |
| **Immediate** | Darcel holds 194 active deals via automated CRM routing | Implement a hard cap of 80 active deals per rep in CRM lead-routing automation | Releases 114 excess deals for redistribution; projected $169K+ immediate unlock |
| **Short-Term** | $4.93M in pipeline is stalled across all regions | Enforce a "Close-Lost" rule: require reps to formally close stalled deals within 30 days | Gives Finance an accurate utilization baseline; unblocks headcount approval process |
| **Strategic** | Central volume is 50% above East despite lower revenue efficiency | Redraw lead routing logic so border-state inbound leads default to East region | Rebalances regional load without adding headcount; positions East for growth |
 
> **On headcount:** Finance's hesitation is data-validated. The current sales force is not being utilized efficiently — it is being concentrated. Approving new headcount before fixing the routing logic would replicate the same bottlenecks at higher cost.

## Dashboard
 
| Page | What It Answers |
| :--- | :--- |
| [**1. Territory Overview**](images/page1_territory.png) | Proves Central's volume surplus is eroding revenue efficiency, not generating it |
| [**2. Rep Utilization**](images/page2_utilization.png) | Exposes pipeline concentration at rep and manager level; confirms routing failure spans teams |
| [**3. ROI & Recommendations**](images/page3_roi.png) | Translates the bottleneck into a recoverable dollar figure and a concrete operational action plan |