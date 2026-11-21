# DAX Documentation – P&C Insurance Analytics

> **Important verification note:** The PBIX report definition exposes the measures used by visuals, but the embedded semantic-model binary could not be decoded into a complete formula export in this environment. The formulas below are the documented build formulas available from the project build record. Before treating this file as a literal formula audit, open the PBIX in Power BI Desktop and compare each expression with the formula bar. Measure names can differ slightly from the names below (for example, `Renewed Rate %` vs `Renewal Rate %`).

## Core measures

### Total Written Premium
```dax
Total Written Premium =
SUM(Fact_Policies[AnnualPremium])
```
Adds annual premium across policies in the current filter context. Used for premium cards, trends, product/channel/region comparisons and agent ranking.

### Total Policies
```dax
Total Policies =
COUNTROWS(Fact_Policies)
```
Counts policy records. This is a policy count, not a distinct-customer count.

### Total Claims
```dax
Total Claims =
COUNTROWS(Fact_Claims)
```
Counts claim records, including Open and Under Review claims.

### Total Approved Claim Amount
```dax
Total Approved Claim Amount =
SUM(Fact_Claims[ClaimAmountApproved])
```
Sums populated approved amounts. DAX SUM ignores blanks; it does not convert unresolved claims to zero approved loss. The dataset contains 379 blank approved amounts, all in Open or Under Review statuses.

### Average Premium per Policy
```dax
Average Premium per Policy =
DIVIDE([Total Written Premium], [Total Policies], 0)
```
Average annual premium per policy in the current filter context.

### Loss Ratio %
```dax
Loss Ratio % =
DIVIDE([Total Approved Claim Amount], [Total Written Premium], 0)
```
Descriptive approved-claim amount divided by written premium. It is not an actuarial incurred loss ratio or earned-premium ratio.

### Rejected Claims %
```dax
Rejected Claims % =
DIVIDE(
    CALCULATE([Total Claims], Fact_Claims[ClaimStatus] = "Rejected"),
    [Total Claims],
    0
)
```
Share of recorded claims with Rejected status. Interpret alongside claim validity, fraud controls and customer outcomes; lower is not automatically better without context.

### Active Policies Only
```dax
Active Policies Only =
CALCULATE([Total Policies], Fact_Policies[PolicyStatus] = "Active")
```
Counts policies currently labelled Active. It should not be described as a historical active-policy count at a past date unless status history is available.

### Premium % of Total
```dax
Premium % of Total =
DIVIDE(
    [Total Written Premium],
    CALCULATE([Total Written Premium], ALL(Fact_Policies)),
    0
)
```
Share of premium relative to the unfiltered Fact_Policies total. Depending on intended analysis, removing all fact-table filters may also remove filters users expect to retain; validate the denominator for each visual.

### Premium Excluding Cancelled
```dax
Premium Excluding Cancelled =
CALCULATE(
    [Total Written Premium],
    Fact_Policies[PolicyStatus] <> "Cancelled"
)
```
Premium excluding cancelled policies while retaining other current filters. The project brief also describes a version that removes filters; that is a different business definition and should not be silently substituted.

### High-Value Claims Count
```dax
High-Value Claims Count =
CALCULATE(
    [Total Claims],
    Fact_Claims[ClaimAmountReported] > 100000
)
```
Counts claims with reported amount above ₹100,000. If the business requirement is approved claim amount above the threshold, replace the column deliberately and label it accordingly.

### Fraud-Flagged Claim Amount
```dax
Fraud-Flagged Claim Amount =
CALCULATE(
    [Total Approved Claim Amount],
    Fact_Claims[FraudFlag] = "Y"
)
```
Sums approved amount for claims flagged Y. A flag is not proof of fraud.

### Renewal Rate % – provisional
```dax
Renewal Rate % =
DIVIDE(
    CALCULATE([Total Policies], Fact_Policies[PolicyStatus] = "Renewed"),
    CALCULATE(
        [Total Policies],
        Fact_Policies[PolicyStatus] IN {"Renewed", "Expired"}
    ),
    0
)
```
This is the provisional definition recorded in the build notes: Renewed / (Renewed + Expired). The brief says policies renewed divided by policies due for renewal. The dataset also contains Lapsed and Cancelled statuses, so the eligible denominator must be confirmed before calling this the definitive renewal rate.

## Time-intelligence measures

### Premium YTD / MTD / QTD
```dax
Premium YTD = TOTALYTD([Total Written Premium], Dim_Date[Date])
Premium MTD = TOTALMTD([Total Written Premium], Dim_Date[Date])
Premium QTD = TOTALQTD([Total Written Premium], Dim_Date[Date])
```
Cumulative written premium for the current year, month or quarter in the active date context.

### Premium Previous Year
```dax
Premium Previous Year =
CALCULATE(
    [Total Written Premium],
    SAMEPERIODLASTYEAR(Dim_Date[Date])
)
```
Returns premium for the comparable prior-year date range.

### YoY Growth %
```dax
YoY Growth % =
DIVIDE(
    [Total Written Premium] - [Premium Previous Year],
    [Premium Previous Year],
    0
)
```
Percentage change from the comparable prior-year period.

### Premium Previous Month
```dax
Premium Previous Month =
CALCULATE(
    [Total Written Premium],
    PREVIOUSMONTH(Dim_Date[Date])
)
```
Returns the previous month's premium for month-over-month comparison.

### Rolling 3-Month Premium
```dax
Rolling 3-Month Premium =
CALCULATE(
    [Total Written Premium],
    DATESINPERIOD(
        Dim_Date[Date],
        MAX(Dim_Date[Date]),
        -3,
        MONTH
    )
)
```
Returns the total premium in the trailing three-month date window. Despite an earlier brief description mentioning AVERAGEX, this expression is a rolling total, not a rolling average; label it accurately.

### Claims by Reported Date
```dax
Claims by Reported Date =
CALCULATE(
    [Total Claims],
    USERELATIONSHIP(Dim_Date[Date], Fact_Claims[ReportedDate])
)
```
Activates the alternate ReportedDate relationship for this calculation. The report's default claim-date analysis uses ClaimDate.

### Average Days to Settle – current build formula
```dax
Average Days to Settle =
AVERAGEX(
    FILTER(
        Fact_Claims,
        NOT ISBLANK(Fact_Claims[SettlementDate])
    ),
    DATEDIFF(
        Fact_Claims[ClaimDate],
        Fact_Claims[SettlementDate],
        DAY
    )
)
```
Average elapsed days from ClaimDate to SettlementDate for records with a settlement date. Dataset result is about 40.3 days. **Brief discrepancy:** the project brief specifies SettlementDate minus ReportedDate; that calculation is about 35.4 days. Choose one definition and state it in the dashboard/documentation.

### Cumulative Policies – Year slicer-aware
```dax
Cumulative Policies =
CALCULATE(
    [Total Policies],
    FILTER(
        ALLSELECTED(Dim_Date),
        Dim_Date[Date] <= MAX(Dim_Date[Date])
    )
)
```
Running policy count over the dates retained by external selections. `ALLSELECTED` preserves slicer selections, including the selected year.

## Advanced / analytical measures

### Premium Rank by Agent
```dax
Premium Rank by Agent =
RANKX(
    ALL(Dim_Agents[AgentID]),
    [Total Written Premium],
    , DESC,
    DENSE
)
```
Ranks agents by premium, ignoring the individual agent row filter while retaining other report filters.

### Weighted Average Deductible
```dax
Weighted Avg Deductible =
DIVIDE(
    SUMX(
        Fact_Policies,
        Fact_Policies[Deductible] * Fact_Policies[AnnualPremium]
    ),
    [Total Written Premium],
    0
)
```
Weights each policy deductible by annual premium. This is a portfolio weighted average, not the simple mean deductible.

### Loss Ratio Using VAR
```dax
Loss Ratio Using VAR =
VAR ApprovedClaims = [Total Approved Claim Amount]
VAR WrittenPremium = [Total Written Premium]
RETURN
    DIVIDE(ApprovedClaims, WrittenPremium, 0)
```
Same business result as Loss Ratio %, expressed with variables for readability and easier debugging.

### Customers with Multiple Product Lines
```dax
Customers with Multiple Product Lines =
SUMX(
    VALUES(Dim_Customers[CustomerID]),
    IF(
        CALCULATE(DISTINCTCOUNT(Fact_Policies[ProductID])) > 1,
        1,
        0
    )
)
```
Counts customers with policies in more than one distinct ProductID. Dataset result: 933 customers (77.8%).

### Target / KPI measures
```dax
Target Premium Value = 50000000
Target Policies = 4000
Target Active Policies = 2500
Target Loss Ratio = 0.70
Target Rejected Claim % = 0.10
```
These are illustrative practice targets set for the KPI visuals, not externally validated industry benchmarks. The report uses cumulative actual measures for date-axis KPI visuals.

### Cumulative Written Premium – Year slicer-aware
```dax
Cumulative Written Premium =
CALCULATE(
    [Total Written Premium],
    FILTER(
        ALLSELECTED(Dim_Date),
        Dim_Date[Date] <= MAX(Dim_Date[Date])
    )
)
```
Running premium within the selected date range. The user verified that the Year slicer changes the value.

### Cumulative Active Policies – Year slicer-aware
```dax
Cumulative Active Policies =
CALCULATE(
    [Active Policies Only],
    FILTER(
        ALLSELECTED(Dim_Date),
        Dim_Date[Date] <= MAX(Dim_Date[Date])
    )
)
```
Cumulative count of policies whose current status is Active, grouped by policy start date. It is not a historical as-of active count because policy status is not time-versioned.

### Cumulative Loss Ratio – Year slicer-aware
```dax
Cumulative Loss Ratio =
VAR CurrentDate = MAX(Dim_Date[Date])
VAR CumulativeClaims =
    CALCULATE(
        [Total Approved Claim Amount],
        FILTER(ALLSELECTED(Dim_Date), Dim_Date[Date] <= CurrentDate)
    )
VAR CumulativePremium =
    CALCULATE(
        [Total Written Premium],
        FILTER(ALLSELECTED(Dim_Date), Dim_Date[Date] <= CurrentDate)
    )
RETURN
    DIVIDE(CumulativeClaims, CumulativePremium, 0)
```
Combines claim-date filtered approved amount and policy-start-date filtered premium under the shared date table. This is a useful dashboard comparison but not a matched exposure cohort.

### Cumulative Rejected Claim % – Year slicer-aware
```dax
Cumulative Rejected Claim % =
VAR CurrentDate = MAX(Dim_Date[Date])
VAR RejectedClaims =
    CALCULATE(
        [Total Claims],
        Fact_Claims[ClaimStatus] = "Rejected",
        FILTER(ALLSELECTED(Dim_Date), Dim_Date[Date] <= CurrentDate)
    )
VAR AllClaims =
    CALCULATE(
        [Total Claims],
        FILTER(ALLSELECTED(Dim_Date), Dim_Date[Date] <= CurrentDate)
    )
RETURN
    DIVIDE(RejectedClaims, AllClaims, 0)
```
Cumulative rejected share by ClaimDate, preserving Year slicer selections. The five KPI visuals were manually tested across 2023, 2024 and 2025.

### Dynamic Top N – pattern
```dax
Product Rank =
RANKX(
    ALL(Dim_Products[ProductLine]),
    [Total Written Premium],
    , DESC,
    DENSE
)

Show Top N Products =
IF([Product Rank] <= [Top N Products Value], 1, 0)
```
Uses a What-If parameter to control the number of product lines displayed. Exact parameter table/measure names should be confirmed in the PBIX.

### Loss Ratio Color
```dax
Loss Ratio Color =
SWITCH(
    TRUE(),
    [Loss Ratio %] < 0.5, "#008000",
    [Loss Ratio %] <= 1, "#FFC000",
    "#FF0000"
)
```
Returns green below 50%, amber from 50% to 100%, and red above 100%. These are project-specific thresholds, not universal insurance standards.

## Measures/techniques visible in the PBIX that need exact formula export from Power BI
- `Target Premium` and What-If `Growth Target %` value.
- `Agent Premium Rank`.
- `Rolling 3-Month Avg Claim Amount`.
- `Claim Trend Anomaly Flag`.
- Field parameter `View By`.
- `Show in Top N` and Top N parameter.
- The final visual-facing measure names `Renewed Rate %`, `Total Claim Amount(Approved)`, `Rejected Claims %`, `Avg. Days to Settle`, `Average Premium Per Policy`, `Customers with 2+ Product Line`.

Before publishing, use Power BI Desktop's formula bar to confirm the exact final formulas, because visual metadata confirms names and field usage but not the DAX expressions.
