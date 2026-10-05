# PhonePe UPI Analytics Dashboard (Power BI)

An interactive Power BI dashboard built on 300,000 simulated UPI transactions. It covers payment success rate, service-wise value, user demographics, and usage patterns, with a custom-designed UI.

![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-00599C?style=for-the-badge&logo=microsoft&logoColor=white)
![Power Query](https://img.shields.io/badge/Power_Query-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white)
![Excel](https://img.shields.io/badge/Excel-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white)
![Figma](https://img.shields.io/badge/Figma-F24E1E?style=for-the-badge&logo=figma&logoColor=white)

![Dashboard overview](images/01-dashboard-overview.png)

---

## At a Glance

| | |
|---|---|
| **Scale** | 300K transactions, ~108K users, ₹3.47bn total value, calendar year 2024 |
| **Model** | Star schema: 1 fact table, 2 dimensions, 2 one-to-many relationships |
| **Deliverables** | 1 dashboard page, 2 hover-tooltip pages, 11 DAX measures, 1 custom date table |
| **Tools** | Power BI Desktop, DAX, Power Query (M), Excel, Figma |

---

## Skills Demonstrated

| Skill | What I did | Where to see it |
|---|---|---|
| **Data cleaning (Power Query)** | Profiled column quality and distribution, removed duplicate transaction IDs, blank user rows, and standardized text values | `All_Transactions`, `All_Users` queries |
| **Data modeling** | Built a star schema with single-direction, one-to-many relationships | [Data Model](#data-model) |
| **DAX: calculated table** | Wrote a custom date dimension (year, month, quarter, weekday, weekend flag) and marked it as the date table | `Date_Table` |
| **DAX: measures** | Built KPI measures (sum, count, distinct count, ratio) in a dedicated `Measures` table | [Key DAX Measures](#key-dax-measures) |
| **DAX: time intelligence** | Previous-month and MoM % measures using `DATEADD` | KPI cards |
| **Feature engineering** | Created generational age segments with a conditional column | `All_Users[Age Segment]` |
| **Sort-by-column** | Sorted Month and Weekday by their numeric keys so axes render chronologically | `Date_Table` |
| **Conditional formatting** | Rule-based callout colors (green/red) and gradient bars | KPI cards, service bar chart |
| **Report tooltips** | Built hover pages for sub-service and age-segment breakdowns | `Tooltip1`, `Tooltip3` pages |
| **Dashboard UI design** | Applied a Figma-designed layout, removed default visual chrome, aligned to a grid | Report canvas |
| **Dynamic text** | Embedded a Q&A-driven value into the Insights card | Insights card |
| **Data critique** | Audited the finished dashboard and documented its flaws | [Limitations](#limitations--what-id-fix) |

---

## Business Questions

| # | Question | Answered by |
|---|---|---|
| 1 | How many transactions and how much value are processed? | KPI cards |
| 2 | What share of transactions succeed? | Success Rate card, Payment Status slicer |
| 3 | How do volume and value trend over time? | Transactions Over Time (area chart), Month slicer |
| 4 | Which service category carries the most value? | Service Transaction Value Analysis |
| 5 | Which age group makes up the user base? | Age Segment Contribution |
| 6 | Weekday vs weekend activity? | Weekday vs Weekend donut |
| 7 | Who are the highest-value users? | Top 5 Users chart |

---

## Dataset

Simulated PhonePe-style UPI data in one Excel workbook (`Phonepe-Final-Dataset.xlsx`), two sheets. Transactions span **January 2024 onward** [VERIFY: end date, expected Dec 2024].

| Table | Grain | Rows | Columns |
|---|---|---|---|
| `All_Users` | 1 row per user | ~108K [VERIFY: exact count] | `User_ID`, `Name`, `Age`, `Join_Date` |
| `All_Transactions` | 1 row per transaction | 300,000 [VERIFY] | `Transaction_ID`, `Amount`, `User_ID`, `Service`, `Service_Type`, `Payment_Status`, `Reason`, `Date` |

Services: Loans, Insurance, Money Transfer, Recharge Bills. Payment statuses: Successful, Failed, Pending.

---

## What I Did

1. **Cleaned the data in Power Query before loading**: checked column quality and distribution, removed duplicate transaction IDs and blank user rows, replaced `Recharge_Bills` → `Recharge Bills` and `Money_Transfer` → `Money Transfer`.
2. **Built a custom date table in DAX**, sorted Month by Month No. and Weekday by Day No., and marked it as the date table.
3. **Modeled the data** as a star schema (`All_Users` and `Date_Table` → `All_Transactions`).
4. **Created a `Measures` table** and wrote KPI and time-intelligence measures.
5. **Engineered age segments** (Gen Z, Millennials, Gen X, Boomers) as a conditional column.
6. **Built the visuals and two tooltip pages** for hover drill-downs.
7. **Designed the UI**: Figma layout as page background, stripped visual borders/backgrounds, conditional colors, gradient bars, tile slicer, dynamic insight text.

---

## Data Model

![Data model](images/02-data-model.png)

| Relationship | Cardinality | Cross-filter | Active |
|---|---|---|---|
| `All_Users[User_ID]` → `All_Transactions[User_ID]` | One to many | Single | Yes |
| `Date_Table[Date]` → `All_Transactions[Date]` | One to many | Single | Yes |

---

## Key DAX Measures

```dax
Total Transaction Value = SUM(All_Transactions[Amount])
```

```dax
Successful Transactions =
CALCULATE(
    [Total Transactions],
    All_Transactions[Payment_Status] = "Successful"
)
```

```dax
Success Rate = DIVIDE([Successful Transactions], [Total Transactions])
```

```dax
Transaction Value Previous Month =
CALCULATE(
    [Total Transaction Value],
    DATEADD(Date_Table[Date], -1, MONTH)
)
```

```dax
Transaction Value MoM % =
DIVIDE(
    [Total Transaction Value] - [Transaction Value Previous Month],
    [Transaction Value Previous Month],
    0
)
```

```dax
Date_Table =
ADDCOLUMNS(
    CALENDAR(MIN(All_Transactions[Date]), MAX(All_Transactions[Date])),
    "Year", YEAR([Date]),
    "Month No.", MONTH([Date]),
    "Month", FORMAT([Date], "MMM"),
    "Quarter", "Q" & FORMAT([Date], "Q"),
    "Weekday", FORMAT([Date], "dddd"),
    "Day No.", WEEKDAY([Date], 2),
    "Weekend", IF(WEEKDAY([Date], 2) >= 6, "Weekend", "Weekday")
)
```

---

## Dashboard Walkthrough

| Visual | What it shows |
|---|---|
| **KPI cards** | 300K transactions, ₹3.47bn total value, 108K unique users, 96.00% success rate |
| **Transactions Over Time** | Monthly transaction count and value as an area chart, Jan to Dec |
| **Service Transaction Value Analysis** | Value by service, gradient-colored |
| **Age Segment Contribution** | Donut of the four generational segments |
| **Weekday vs Weekend Usage** | Donut of transaction share by day type |
| **Top 5 Users (by Transaction Value)** | Column chart of the highest-value users |
| **Insights card** | Narrative text with a dynamic value for the top service |
| **Slicers** | Month dropdown and Payment Status buttons filter the whole page |
| **Tooltips** | Hovering service and user charts shows top sub-services and age-segment value |

![Tooltip example](images/03-tooltip.png)

---

## Key Findings

| # | Finding | Reading |
|---|---|---|
| 1 | **Loans account for ~₹2.5bn of ₹3.47bn (~72%)**; Insurance ~₹0.5bn, Money Transfer ~₹0.4bn, Recharge Bills ~₹0.1bn | Value is concentrated in one service. A separate average-ticket-size view would show whether that is volume or large tickets |
| 2 | **Success rate is 96.00%**, leaving 4% failed or pending | Failure reasons are merged into Pending (see limitations), so the failure breakdown is not usable |
| 3 | **Gen X (37.4%) and Millennials (37.3%) make up ~75% of users**; Gen Z 20.74%, Boomers 4.56% | Share of users only. The segments span unequal age ranges (Gen Z 9 years, Millennials and Gen X 16 each, Boomers 2), so the split mostly reflects bucket width, not behavior |
| 4 | **Weekdays carry 71.6% of transactions, which equals weekdays' share of 2024's calendar** (262 of 366 days) | Daily volume is roughly flat. This is not a weekday-heavy pattern, it is calendar math |
| 5 | **Top 5 users hold ₹1.2M to ₹1.8M each, about ₹7.1M combined (~0.2% of total value)** | No whale concentration, so a top-user retention program has limited leverage |

---

## Limitations & What I'd Fix

- **MoM cards are misleading with "All" months selected.** A year-long total is compared against the same window shifted back one month, which only partially exists in the data, so the figure (about 9%) reflects window size, not growth. Selecting a single month gives a real MoM value.
- **Failure reasons merged into Pending.** `Wrong Pin`, `Server Error`, and `Insufficient Amount` are failures, not pending states. A proper `Failed` category would make the success-rate analysis useful.
- **Weekday vs weekend is not normalized.** Raw share mirrors the calendar. Average transactions per day by day type would be the correct metric.
- **Age segments are unequal-width buckets.** Comparing their user shares without adjusting for bucket width overstates Gen X and Millennials.
- **Unique Users MoM card omitted** in the tutorial because of a relationship issue.
- **Synthetic data.** Names and values are generated, so findings illustrate technique, not real user behavior.
- **Single dashboard page** with no drill-through or bookmarks.

---

## What I Learned

- Clean and profile data in Power Query before loading, not after visuals break.
- Month and weekday text must be sorted by numeric keys or charts render alphabetically.
- A central `Measures` table and a marked date table keep the model maintainable and time intelligence reliable.
- A shifted-window measure can produce plausible-looking but wrong numbers. Always test a measure with a single filter context.
- Percentages need a baseline: 71.6% weekday share looks like an insight until you compare it with 5/7.

---

## My Additions Beyond the Tutorial

[ADD MANUALLY: what you built or fixed yourself, e.g. a Failed status category, an Average Transactions per Day measure, a corrected MoM measure, a new page. Delete this section only if you changed nothing, and say so honestly.]

---

## Repository Structure

```text
├── data/
│   └── Phonepe-Final-Dataset.xlsx
├── dashboard/
│   └── PhonePe_Analysis.pbix
├── images/
│   ├── 01-dashboard-overview.png
│   ├── 02-data-model.png
│   └── 03-tooltip.png
└── README.md
```

**To open:** download `PhonePe_Analysis.pbix`, open it in Power BI Desktop, and if prompted, repoint the source under *Transform data → Data source settings* to `data/Phonepe-Final-Dataset.xlsx`.

---

## Credits & Contact

- **Tutorial:** [ADD: creator name] ([Part 1]([ADD: URL]), [Part 2]([ADD: URL]))
- **Figma layout template:** [ADD: link or credit]
- **Author:** [ADD: Piyush Panthi] | [LinkedIn]([ADD: URL]) | [Email]([ADD: email])
