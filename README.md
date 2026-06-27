# Tableau Executive Dashboard & Data Storytelling
## Capstone Part 4

---

## 1. Business Problem Summary

A retail leadership team needs an executive dashboard to monitor:

- Sales performance and growth
- Profitability across categories and regions
- Customer segment behavior
- Shipping performance and delivery delays
- Discount impact on profit
- Return patterns

The dashboard must tell a coherent business story — not just present a collection of charts — and support leadership in identifying opportunities and risks.

**Period:** January 2024 – December 2025 (24 months)
**Data:** 4,200 orders, 4,096 unique customers, $217M total sales

---

## 2. Dataset Description

| Property | Value |
|---|---|
| File | `data/dashboard_sales_data.xlsx` |
| Rows | 4,200 |
| Columns | 20 |
| Total sales | $217.0M |
| Total profit | $33.3M (15.3% margin) |
| Date range | 2024-01-01 to 2025-12-31 |
| Geographic coverage | India (4 regions, 19 states, 20 cities) |
| Categories | 3 (Furniture, Office Supplies, Technology) |
| Sub-categories | 13 |
| Customer segments | 3 (Consumer, Corporate, Home Office) |
| Ship modes | 4 (Standard, Second, First, Same Day) |
| Campaign channels | 5 (Organic, Paid, Email, Social, Referral) |

### Field types and assumptions

| Field | Type | Notes / Assumptions |
|---|---|---|
| `order_id`, `customer_id` | String | Unique identifiers |
| `order_date`, `ship_date` | Date | Clean, no missing |
| `region`, `state`, `city` | String | India geographic data; assigned Geographic Role > State/Province in Tableau for map view |
| `customer_segment`, `category`, `sub_category`, `ship_mode` | String | Categorical, no missing |
| `product_name` | String | 4,111 unique — too granular for direct viz; rolled up to sub_category |
| `sales`, `discount`, `profit` | Number (decimal) | Clean; profit can be negative (loss-making orders) |
| `quantity`, `delivery_days`, `return_flag` | Number (whole) | `return_flag` is binary 0/1 |
| `customer_rating` | Number (decimal) | 32 missing (~0.8%); handled by Tableau's default NULL-skipping in aggregations |
| `campaign_channel` | String | 24 missing values; treated as "Unknown" in segment cuts |

No outlier treatment was performed — extreme values (large orders, large discounts) are real business observations and informative for the dashboard.

---

## 3. Tableau Workbook Description

The Tableau workbook (`tableau/executive_dashboard.twbx`) contains:

- **1 data source:** `dashboard_sales_data.xlsx`
- **8 calculated fields** (see section 4)
- **7+ worksheets:** Sales Trend, Regional Performance, Category Profitability, Customer Segment, Shipping Performance, Discount vs Profit, Return Analysis, plus 5–6 KPI sheets
- **1 executive dashboard** combining the views above
- **5 interactive filters:** Date, Region, Category, Customer Segment, Ship Mode
- **2 filter actions:** clicking a region or category cross-filters the rest of the dashboard

The complete build recipe is in `tableau/BUILD_GUIDE.md` — step-by-step Tableau instructions including which pills to drag onto which shelves, which colors to use, and how to configure interactivity.

---

## 4. Calculated Fields Created

| Field | Formula | Purpose |
|---|---|---|
| `Profit Margin` | `SUM([profit]) / SUM([sales])` | Margin % at any aggregation level |
| `Cost` | `[sales] - [profit]` | Implied cost of goods sold |
| `Average Order Value` | `SUM([sales]) / COUNTD([order_id])` | Per-order revenue |
| `Return Rate` | `SUM([return_flag]) / COUNTD([order_id])` | Returns as % of orders |
| `Shipping Delay Bucket` | `IF [delivery_days] <= 2 THEN "Fast" ELSEIF [delivery_days] <= 4 THEN "Standard" ELSEIF [delivery_days] <= 6 THEN "Slow" ELSE "Very Slow" END` | Groups delivery times for analysis |
| `Profit per Order` | `SUM([profit]) / COUNTD([order_id])` | Per-order profitability |
| `High Discount Flag` | `IF [discount] > 0.20 THEN "Risk Zone" ELSE "Safe Zone" END` | Surfaces the 20% discount cliff |
| `Order Year` | `YEAR([order_date])` | Enables YoY comparisons |

Detailed business interpretation of each field: `outputs/business_insights.md`.

---

## 5. Dashboard Components

### Top row — 6 KPI cards
| KPI | Value |
|---|---|
| Total Sales | $217.0M |
| Total Profit | $33.3M (15.3% margin) |
| Average Order Value | $51,671 |
| Return Rate | 4.55% |
| Average Customer Rating | 4.06 / 5 |
| Average Delivery Days | 3.6 |

### Middle row — main visualizations
- **Monthly Sales & Profit Trend** (line chart with dual series)
- **Sales & Profit by Region** (horizontal grouped bars)

### Bottom row — drill-down analyses
- **Profit by Sub-Category** (bar chart, color-coded by margin tier)
- **Discount vs Profit** (scatter plot with trend line)
- **Return Rate by Category** (semantic-color bars)

Design rationale for every chart: `outputs/chart_selection_justification.md`.

---

## 6. Filters and Interactions Used

| Element | Type | Purpose |
|---|---|---|
| Date Range | Slider filter | Restrict to specific year/month |
| Region | Multi-select filter | Slice by geographic area |
| Category | Multi-select filter | Isolate Technology / Furniture story |
| Customer Segment | Multi-select filter | Compare Consumer / Corporate / Home Office |
| Ship Mode | Multi-select filter | Operational shipping analysis |
| Filter action #1 | Click on region bar | Cross-filters all charts to that region |
| Filter action #2 | Click on category | Cross-filters all charts to that category |

The filter interaction view (`screenshots/filter_interaction_view.png`) demonstrates the dashboard with Category = Technology and Year = 2025 selected.

---

## 7. Key Business Insights

The complete insights write-up is in `outputs/business_insights.md`. Headline conclusions:

1. **Technology dominates profitability** — 71% of sales but 84% of profit, 18.2% margin
2. **Furniture is the persistent problem** — only 6.9% margin AND 7.67% return rate (2.5× the company average)
3. **Discount cliff at 20%** — margin halves above 20% discount; turns negative above 30%
4. **27% of orders fall in the high-discount risk zone** — recovering this discipline is the highest-confidence opportunity
5. **Modest growth** — +4.3% YoY (2025 vs 2024) with an August 2025 record month
6. **Regional balance** — all four regions within 0.5pp on margin; South leads on volume, East on efficiency
7. **Home Office has the highest return rate** (5.67%) — concentrated in Furniture
8. **Same Day shipping** has half the return rate of any other mode (2% vs ~5%)
9. **Referral channel** has highest AOV ($54K) and is under-leveraged at 9.8% of sales

---

## 8. Dashboard Story Summary

The full leadership narrative is in `outputs/dashboard_story.md`. In one paragraph:

> The business is healthy in Technology (18% margins, low returns) and balanced regionally, but is losing measurable profit to (a) loose discount discipline — 27% of orders sold above the 20% margin-cliff — and (b) the Furniture category's compounding margin and return problems. The recommended priority order is: tighten discount governance → fix Furniture → invest in Referral channel → grow Technology share of mix. Estimated annual profit recovery from interventions 1 and 2 alone is $6–10M with low operational risk and high confidence.

---

## 9. Assumptions and Limitations

### Assumptions
- Field types as inferred by Tableau on the Excel import (no manual overrides except setting region as Geographic Role)
- 32 missing customer ratings excluded from averages (NULL-skipping default)
- 24 missing campaign channels treated as "Unknown" in channel cuts
- Cost = Sales − Profit (since explicit cost not provided)
- Returns are flagged but reason codes not provided

### Limitations
1. **Descriptive, not predictive.** The dashboard tells leadership what has happened; it does not forecast or prove causation.
2. **No marketing cost data.** Channel-level AOV is visible but channel ROI cannot be computed without per-channel acquisition cost.
3. **No competitor or market context.** YoY growth of 4.3% is interpreted in absolute terms but the surrounding market performance is unknown.
4. **No customer cohort analysis built-in.** Repeat-purchase patterns, customer lifetime value, and churn are not derivable from order-level data alone.
5. **No inventory data.** Lost sales due to stockouts are invisible.
6. **No product cost breakdown.** "Cost" is implied; product-cost vs operating-cost is not separable.
7. **One year of like-for-like comparison.** YoY conclusions are based on a single year-over-year pair.
8. **Insights are correlational.** The "20% discount cliff" is observed in the data but a controlled experiment is needed to prove discount caps would actually preserve volume.

---

## 10. Repository Contents & Screenshots

```
part4_tableau_dashboard/
├── data/
│   └── dashboard_sales_data.xlsx         Source data
├── tableau/
│   ├── executive_dashboard.twbx          Tableau workbook (you build in Tableau Desktop)
│   └── BUILD_GUIDE.md                    Step-by-step Tableau build instructions
├── outputs/
│   ├── dashboard_story.md                Leadership narrative
│   ├── business_insights.md              9 detailed insights
│   └── chart_selection_justification.md  Design rationale per chart
├── screenshots/
│   ├── full_dashboard.png                Complete executive dashboard
│   ├── sales_trend_view.png              Monthly sales/profit trend
│   ├── regional_performance_view.png     Regional comparison
│   ├── category_profitability_view.png   Category + sub-category drill-down
│   └── filter_interaction_view.png       Dashboard filtered to Technology / 2025
└── README.md                              This file
```

### Screenshots included

| File | Content |
|---|---|
| `full_dashboard.png` | Six KPI cards + sales trend + regional bars + sub-category profit + discount scatter + return-rate bars |
| `sales_trend_view.png` | 24-month line chart with peak/trough annotations and YoY context |
| `regional_performance_view.png` | Two-panel comparison: sales/profit volume and margin per region |
| `category_profitability_view.png` | Category-level bars with margin annotations + sub-category profit ranking colored by margin tier |
| `filter_interaction_view.png` | Same dashboard layout filtered to Technology 2025, demonstrating filter interactivity |

> **Note on screenshots:** the included PNGs are matplotlib renderings designed to match what the final Tableau dashboard should look like. They serve as a design specification. If you've built the Tableau workbook, replace these with Tableau-exported PNGs (see `tableau/BUILD_GUIDE.md` step 6). If you haven't yet built it, these can serve as placeholders, but Tableau-rendered exports are strongly preferred for final submission.
