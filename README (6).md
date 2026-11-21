# P&C Insurance Analytics – Business Intelligence Dashboard

**Personal portfolio project | Power BI | DAX | Excel sample data**

## Project overview
This project analyses a sample Property & Casualty (P&C) insurance portfolio across policies, customers, agents, products, dates and claims. It is designed as an executive-facing self-service report for underwriting, claims, sales and finance stakeholders.

The Power BI report contains five main pages: Executive Overview, Claims & Loss Analysis, Sales & Agent Performance, Customer Insights, and Customer 360 (drillthrough). Two additional practice pages are retained in the PBIX for parameter and KPI experiments; they are not counted as the five main report pages.

## Business questions
- How much written premium was generated, and how did it change by policy start year?
- Which product lines, regions and sales channels contribute most premium?
- What is the approved-claim amount relative to written premium?
- How do claim status, rejection rate and settlement duration look?
- Which agents rank highest by written premium?
- How are customers distributed by segment and credit-score band, and how much cross-sell is present?
- Can a user drill through to a customer's policies and claims?

## Dataset
The supplied workbook contains six analytical tables plus a README/data-dictionary sheet:

| Table | Rows | Grain |
|---|---:|---|
| Dim_Date | 1,096 | One row per date, 2023-01-01 to 2025-12-31 |
| Dim_Customers | 1,200 | One row per customer |
| Dim_Agents | 60 | One row per agent |
| Dim_Products | 18 | One row per product |
| Fact_Policies | 3,600 | One row per policy |
| Fact_Claims | 1,900 | One row per claim |

The fact tables total 5,500 rows; all six tables total 7,874 rows. The workbook intentionally contains 379 blank `ClaimAmountApproved` values (Open / Under Review claims) and 645 blank `SettlementDate` values. These are meaningful workflow states and should not be deleted or blindly imputed.

## Key dataset findings (all-time unless stated)
- Total written premium: **₹164.11 million** across 3,600 policies.
- Approved claim amount: **₹251.11 million** across claims with a populated approved amount.
- Aggregate approved-claims-to-written-premium ratio: **153.0%**. This is a descriptive portfolio ratio, not an actuarial/pricing conclusion; claim dates and policy start dates differ.
- Commercial Property contributes **33.6%** of written premium; Marine contributes **21.3%**.
- North is the largest region by written premium (**26.6%**); East contributes **24.6%**.
- Agent channel contributes **38.2%** of premium; Online contributes **30.0%**.
- 277 of 1,900 claims are Rejected (**14.6%**); 379 claims are Open or Under Review.
- 83 claims have `FraudFlag = Y`; their populated approved amount totals ₹7.97 million. A fraud flag is an indicator, not proof of fraud.
- 933 customers have policies across more than one product ID (77.8% of 1,200 customers), using the supplied policy-customer-product links.

Year comparisons use `PolicyStartDate` for premium/policy metrics and `ClaimDate` for claim metrics. They are different event populations and should not be presented as same-cohort comparisons.

## Report pages
1. **Executive Overview** – headline KPIs, premium trend vs prior year, premium by product, channel and state, with year/product/channel/state slicers and reset action.
2. **Claims & Loss Analysis** – claim status trend, loss ratio by product and region, claim amount scatter, claim detail table, claim KPIs and bookmark view toggle.
3. **Sales & Agent Performance** – agent premium ranking, premium and approved-claim amounts by region, policy trend by channel, renewal-rate KPI and growth-target parameter.
4. **Customer Insights** – customer count, average premium per policy, multi-product customer count, segment mix, credit-score view and customer detail selection.
5. **Customer 360** – selected customer profile, policy list, claim list and related totals.

## Interactivity
The PBIX includes a report-page tooltip, reset-filters bookmark/button, product-vs-region loss-ratio bookmark toggle, Customer 360 drillthrough page, slicers, a What-If growth target, a dynamic Top-N product parameter, a field parameter, and a KPI Practice page. The Year slicer on KPI Practice was manually tested for all five KPI visuals across 2023, 2024 and 2025.

## Important interpretation and validation notes
1. **Renewal rate definition:** the current project measure in the build notes uses Renewed / (Renewed + Expired). The brief defines the denominator as policies due for renewal. The dataset also has Lapsed and Cancelled statuses. Treat this as a provisional proxy until the business denominator is formally defined.
2. **Settlement duration:** the current dashboard build measure uses `SettlementDate - ClaimDate` (average about 40.3 days among rows with a settlement date). The project brief specifies `SettlementDate - ReportedDate` (about 35.4 days). State which definition is used; do not mix the two.
3. **Date coverage:** the source date dimension ends 2025-12-31, but 37 claims have ReportedDate in 2026 (latest 2026-01-10). The PBIX build history indicates the date table was extended to 2026-01-10. The source workbook itself remains 2023–2025.
4. **Loss ratio:** total approved claims / total written premium is 153.0% for the supplied sample. This is a simple aggregate comparison across different event dates, not an earned-premium actuarial loss ratio and not evidence of company solvency.
5. **Customer 360 filter:** the saved report definition contains a CustomerID filter value (`CUST00001`) on the Customer 360 page. Confirm that the page responds dynamically to drillthrough for other customers before publishing screenshots or a live demo.
6. **Synthetic data:** names and records are sample data. Do not describe this as a real insurer/client engagement or claim that the records represent actual customers.

## Repository layout
```text
PnC_Insurance_Analytics_Portfolio/
├── README.md
├── data/
│   └── raw/PnC_Insurance_Capstone_Dataset.xlsx
├── powerbi/
│   └── PnC_Insurance_Dashboard.pbix
├── docs/
│   ├── Dashboard_Documentation.md
│   ├── DAX_Documentation.md
│   └── Repository_Structure.md
├── reports/
│   ├── Business_Insights_Report.pdf
│   └── Project_Summary.pdf
└── presentation/
    └── Five_Minute_Project_Walkthrough.pptx
```

## Tools
- Microsoft Power BI Desktop
- Power Query
- DAX
- Microsoft Excel

## How to use
1. Download the repository.
2. Open `powerbi/PnC_Insurance_Dashboard.pbix` in Power BI Desktop.
3. If Power BI asks for a data source path, reconnect it to `data/raw/PnC_Insurance_Capstone_Dataset.xlsx` and refresh.
4. Use the report navigation, slicers, bookmarks and drillthrough interactions.

## Limitations and next steps
- Validate the renewal-rate denominator with a clearly defined renewal-eligible population.
- Align the settlement-duration definition with the brief or clearly label the current calculation.
- Validate Customer 360 dynamic drillthrough for multiple customer IDs.
- Consider a true earned-premium and incurred-loss methodology if policy exposure/earned premium and claim development data become available.
- Add row-level security, incremental refresh and calculation groups only if the project scope requires them; these are outside the current capstone scope.

**Project classification:** Personal portfolio project using synthetic/sample data. No real insurer or client is represented.
