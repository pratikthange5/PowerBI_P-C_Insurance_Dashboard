# Dashboard Documentation

## Scope and page inventory
The supplied PBIX contains seven report pages. Five are the requested main analytical pages; two are practice pages (`Parameters(Practice)` and `KPI(Practice)`) and are not included in the five-page business report inventory.

| Page | Purpose | Key visuals / interactions identified in PBIX |
|---|---|---|
| Executive Overview | Executive snapshot of premium and policy portfolio | KPI cards for premium, policies, loss ratio and renewed rate; monthly premium line with prior-year comparison; premium by product; premium by channel; state map; slicers for year, product, sales channel and state; reset action |
| Claims & Loss Analysis | Claims volume, cost and risk indicators | Claim KPI cards; claim status by month; loss ratio by product; loss ratio by region; reported-vs-approved claim scatter by claim type; claim detail table; bookmark buttons to switch product/region view |
| Sales & Agent Performance | Distribution and agent productivity | Agent premium ranking table with region/tier; premium and approved-claim amount by region; policy count trend by channel; renewal-rate KPI; growth-target What-If parameter |
| Customer Insights | Customer mix and cross-sell | Total customers, average premium per policy, customers with 2+ product lines; customer segment mix; credit-score visual; customer ID table for drillthrough selection |
| Customer 360 | Customer-level detail | Customer name/ID and KPI cards; selected customer's policies and claims tables; drillthrough detail page |

## 1. Executive Overview
**Business audience:** Chief Underwriting Officer, Finance and senior leadership.

**Questions answered:** What is written premium? How many policies are represented? What is the aggregate approved-claims-to-premium ratio? Which products, channels and states contribute most premium? How does monthly premium compare with the prior year?

**Visual interpretation:**
- KPI cards provide headline values for Total Written Premium, Total Policies, Loss Ratio % and Renewed Rate %.
- Monthly written-premium line compares current period with prior year.
- Product and channel visuals show premium mix.
- Filled map displays premium by customer state.
- Slicers allow filtering by year, product line, sales channel and state.
- Reset Filters button returns the page to its saved default state.

**Caution:** State is sourced from customer attributes; this visual represents premium by policyholder state, not necessarily risk/property location.

## 2. Claims & Loss Analysis
**Business audience:** Head of Claims and underwriting.

**Questions answered:** How many claims are recorded? What is the approved amount? What is the rejection share? How do claim statuses move over time? Which product/region combinations show higher aggregate loss ratios? How do reported and approved amounts compare?

**Visual interpretation:**
- KPI strip contains total claims, approved claim amount, rejected-claim percentage and average days to settle.
- Monthly stacked columns show claim status.
- Product and regional loss-ratio views are switched with bookmark buttons.
- Scatter plot compares ClaimAmountReported with ClaimAmountApproved and distinguishes ClaimType.
- Claim table supports inspection of claim IDs, policy IDs, claim types and reported/approved amounts.

**Caution:** Blank approved amounts occur for Open and Under Review claims and should remain blank. The simple aggregate loss ratio should not be presented as a fully developed actuarial loss ratio.

## 3. Sales & Agent Performance
**Business audience:** Regional Sales Heads and sales operations.

**Questions answered:** Which agents contribute the most written premium? How does premium vary by region? How do policy counts change by channel? How does the displayed renewal KPI compare with a selected growth target?

**Visual interpretation:**
- Agent table ranks agents by written premium and includes Region and AgentTier.
- Regional comparison includes written premium and approved claim amount.
- Monthly policy trend is split by SalesChannel.
- Renewal-rate KPI is compared with a What-If growth/target parameter.

**Caution:** A What-If parameter changes a scenario/target; it does not change historical business performance. Renewal-rate interpretation depends on the denominator definition.

## 4. Customer Insights
**Business audience:** Customer strategy, marketing and distribution.

**Questions answered:** How many customers are represented? How many hold policies across multiple product lines? How does customer mix vary by segment and credit-score band?

**Visual interpretation:**
- Cards show Total Customers, Average Premium per Policy and Customers with 2+ Product Lines.
- Segment chart shows Individual, Small Business and Corporate mix.
- Credit-score visual displays the selected premium/customer metric by credit-score band.
- Customer ID table provides a starting point for Customer 360 drillthrough.

**Caution:** Credit-score bands are descriptive sample attributes and should not be used to infer creditworthiness or make underwriting decisions without validation and governance.

## 5. Customer 360
**Business audience:** Service, account management and analyst investigation.

**Questions answered:** What policies and claims are associated with a selected customer? What is the customer's premium and claim footprint?

**Visual interpretation:**
- Customer identity and summary cards provide context.
- Policy table includes policy ID, status, annual premium and start date.
- Claim table includes claim ID, status, reported amount and approved amount.
- The intended entry is drillthrough from a customer selection elsewhere in the report.

**Validation item before publishing:** The saved PBIX page definition contains a CustomerID filter value of `CUST00001`. Confirm that the drillthrough filter is dynamic for multiple customer IDs and that this value is not unintentionally fixed as a page-level filter.

## Cross-page interactions and report features
- Report-page tooltip: mini context for premium data points.
- Reset Filters bookmark/button on Executive Overview.
- Bookmark toggle between Loss Ratio by Product Line and Loss Ratio by Region.
- Customer 360 drillthrough.
- Year/product/channel/state slicers and cross-filtering.
- What-If growth target, Dynamic Top-N products and Field Parameter are available in the practice/parameter page.
- KPI Practice page contains 10 summary cards and 5 KPI visuals. The user manually tested the five KPI visuals for 2023, 2024 and 2025 year selections; this practice page is supplementary, not one of the five main pages.

## Report model overview
The model uses six tables: `Dim_Date`, `Dim_Customers`, `Dim_Agents`, `Dim_Products`, `Fact_Policies`, and `Fact_Claims`. The documented active relationships include Date to PolicyStartDate, Date to ClaimDate, and customer/agent/product dimensions to policies; Customer also filters claims directly. Policy-to-claim relationship is inactive to avoid ambiguous filter paths. Date relationships to ReportedDate and SettlementDate are used as alternate date roles where required.
