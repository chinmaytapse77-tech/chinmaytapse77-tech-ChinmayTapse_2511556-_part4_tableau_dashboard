# Chart Selection Justification

This document explains the chart-type, encoding, and design decisions for each major view in the executive dashboard. Every choice is motivated by the **business question being asked**, the principle of **least cognitive load**, and a deliberately avoided mistake.

---

## Chart 1 — Monthly Sales & Profit Trend

**Question answered:** How is the business performing over time? Where are the peaks and troughs?

**Chart type:** Dual line chart (two series on a shared time axis)

**Why this type:**
- Time-series data with continuous ordering → lines are the canonical choice
- Two related measures (sales and profit) on the same axis lets the reader visually compare them
- Showing 24 monthly data points lets the eye spot seasonal patterns and inflection points
- Filled area under the sales line gives a visual "volume" cue without adding a separate chart

**Encoding choices:**
- X-axis: month (continuous date)
- Y-axis: $M (shared between sales and profit; both measured in dollars)
- Color: blue for sales, orange for profit (categorical distinction, no inherent meaning to the order)
- Annotations: peak and trough markers point to the high-leverage data points; the reader doesn't have to find them
- Background shading: 2024 vs 2025 distinction reinforces the YoY comparison

**Design principle applied:** *Direct the eye to the insight.* The most important moments (peak Aug 2025, trough Aug 2024) are explicitly annotated. The user does not have to scan the chart to find what matters.

**Mistake avoided:** Showing sales on a secondary right-hand axis would have allowed both lines to span the full chart height — but it introduces dual-axis distortion, making relative changes ambiguous. By using a single y-axis (with profit visibly lower than sales), the relative magnitudes are honest.

---

## Chart 2 — Sales & Profit by Region

**Question answered:** Which regions are the largest, and how does profit compare to sales across them?

**Chart type:** Horizontal grouped bar chart

**Why this type:**
- Comparing 4 categorical values (regions) → bar chart is the right tool
- Horizontal orientation: region labels are easier to read horizontally; also works well for ranking
- Grouped (paired) bars let sales and profit be compared per region without requiring the reader to cross-reference two separate charts

**Encoding choices:**
- Y-axis: regions, sorted by sales descending (so the eye moves naturally from biggest to smallest)
- X-axis: $M
- Color: blue (sales), orange (profit) — consistent with the trend chart
- Direct labels on each bar (e.g., "$64.7M") so users don't need to estimate from axis

**Design principle applied:** *Sort by the measure that matters.* Sorting by sales descending makes the dominant region obvious. An alphabetical sort would have buried the lead.

**Mistake avoided:** A pie chart would have been the wrong choice. With only 4 segments at similar sizes (15.1%–15.6% margins make them visually indistinguishable on a pie), the differences would be impossible to discern. Bars allow precise length comparison.

---

## Chart 3 — Profit Margin by Region

**Question answered:** Setting aside sales volume, which region is the most efficient at converting sales into profit?

**Chart type:** Horizontal bar chart (single measure)

**Why this type:**
- Single metric across 4 categories → simple bar chart
- Allows ranking and direct comparison of efficiency, separate from volume
- Pairs naturally with Chart 2 to give "volume × efficiency" picture

**Encoding choices:**
- Color is meaningful here: above 15% margin = green, below = purple. This adds a quick "good/bad" read.
- Sorted ascending so the highest performer is at the top
- Direct labels with one decimal precision

**Mistake avoided:** Combining margin and sales on a single chart (e.g., bubble with size = sales, x = margin) would have crammed two stories into one — the dashboard separates them so leadership can ask "who's biggest?" and "who's most efficient?" independently.

---

## Chart 4 — Profit by Sub-Category

**Question answered:** Which products carry the business, and which are marginal?

**Chart type:** Horizontal bar chart with color encoding for margin tier

**Why this type:**
- 13 sub-categories → bar chart (a treemap would also work, but bars allow precise reading of small values, which matter here because the bottom 9 sub-categories are all small)
- Adding a color dimension (margin tier) lets the chart answer two questions at once: "how much profit?" and "at what margin?"

**Encoding choices:**
- Y-axis: sub-categories, sorted by profit descending
- X-axis: profit in $M
- Color: green (>15% margin), orange (10-15%), light-orange (5-10%), red (<5%) — semantic traffic-light coloring
- Length of bar shows profit magnitude; color shows efficiency tier
- Legend in the corner shows the margin tiers

**Design principle applied:** *Two visual variables, two questions.* The eye reads bar length AND color simultaneously. A reader can instantly see "Copiers makes a lot of profit AND is in the healthy margin tier" — both data points conveyed in one glance.

**Mistake avoided:** A treemap would have made the small-profit categories (Paper, Binders, etc.) almost invisible. They matter for the SKU-rationalization conversation, so they need to be readable. Bars give them equal visual real estate.

---

## Chart 5 — Discount vs Profit Scatter Plot

**Question answered:** What is the relationship between discount level and order-level profit? Where is the break-even?

**Chart type:** Scatter plot with overlaid trend line (binned mean)

**Why this type:**
- Two continuous variables → scatter plot is the standard
- 4,200 data points → scatter shows the *distribution* of outcomes (not just the average)
- An overlaid binned-mean line gives the trend without obscuring the variance

**Encoding choices:**
- X-axis: discount %
- Y-axis: profit per order ($K)
- Color: green (positive profit), red (negative profit) — semantic, immediate
- Trend line: dark blue (high-contrast, reads as "summary signal" over the scatter)
- Reference line at y=0 (the break-even line): visually critical, drawn dashed

**Design principle applied:** *Show the data, not just the summary.* If we showed only the binned-mean line, we would hide the fact that even at low discounts some orders lose money, and even at high discounts some orders are highly profitable. The scatter shows the variance; the line shows the trend.

**Mistake avoided:** Aggregating to a single bar per discount band (which would show only the band averages from `business_insights.md`) would hide the variance. Leadership might think "discounts below 20% always make money" when in fact some 5% discounts still produce losses. The scatter prevents that overclaim.

---

## Chart 6 — Return Rate by Category

**Question answered:** Which category has the highest return risk?

**Chart type:** Horizontal bar chart with semantic color

**Why this type:**
- 3 categories → bar chart
- Single measure (return rate %)
- Compact: takes minimal dashboard real estate

**Encoding choices:**
- Y-axis: categories, sorted ascending
- X-axis: return rate %
- Color: green (<4%), orange (4-7%), red (>7%) — traffic light. The reader sees "Furniture is red" before reading the number.
- Direct labels with 2 decimal precision (returns are sub-10% values, decimals matter)

**Mistake avoided:** A 100% stacked bar (returned vs not-returned per category) would have made the differences nearly invisible — a 5% slice vs 8% slice are hard to distinguish visually when both are dwarfed by the 92-95% "not returned" portion. The standalone metric, in absolute terms, makes the gap obvious.

---

## Chart 7 — KPI Cards (Header Row)

**Question answered:** What are the headline numbers at a glance?

**Chart type:** Card / number display (not a chart in the strict sense, but a visualization choice)

**Why this type:**
- Single-number summaries are easier to read than mini-charts when the goal is "remember this number"
- 6 KPI cards across the top frame the entire dashboard
- Each card has a sub-line (e.g., "+4.3% YoY", "15.3% margin") that adds the *context* a number alone lacks

**Encoding choices:**
- Color-coded borders match the metric type (blue = sales, green = profit, etc.)
- Large bold number is the headline; smaller subtitle is the contextual fact
- Consistent layout (6 equally-sized cards) lets the reader scan them rhythmically

**Design principle applied:** *Hierarchy.* The KPIs are physically the largest text on the dashboard, signalling "read these first."

**Mistake avoided:** Using sparklines (small inline trend lines) inside the KPI cards would have been visually busy. The trend chart below already shows the trend; the cards should be calm and immediately readable.

---

## Chart 8 — Filter Interaction View (filtered to Technology / 2025)

**Question answered:** Does the dashboard remain coherent under filtering? What does the business look like through a narrowed lens?

**Chart type:** Same as full dashboard, but with all views responding to the filter

**Why this design:**
- Filter interaction is itself an analytical feature, not just a UI feature
- Demonstrating it on a screenshot proves the dashboard is connected (not a static set of charts)
- The Technology + 2025 filter is the most interesting subset because Technology is the profitable engine and 2025 is the more recent year

**Encoding choices:**
- A visible "FILTERS APPLIED" banner makes the filter state obvious
- KPI cards recompute under the filter (sales drops from $217M to the filtered subset)
- All views shift to show only the filtered records

**Design principle applied:** *Interactivity makes a dashboard a tool, not a poster.* A static dashboard answers prepared questions; an interactive one lets leadership ask their own.

---

## Cross-Cutting Principles Applied to the Whole Dashboard

| Principle | How it's applied |
|---|---|
| **Consistent color language** | Blue = sales, Orange = profit, Green = healthy, Red = problem. Same meaning everywhere. |
| **Sort by what matters** | All ranked views are sorted by the measure of interest (sales, profit, margin) — never alphabetically by default. |
| **Direct labels** | Most charts have value labels on the bars so users don't squint at axes. Reduces error. |
| **One question per chart** | Each view answers one focused business question. No "mixed-purpose" charts. |
| **Minimal chrome** | Removed top and right spines, faded gridlines. The data is the figure-ground, not the framing. |
| **Annotations for the lead** | Peak and trough months are labeled directly. Margin tier shown on category bars. Insights are surfaced, not hidden in the chart. |
| **Honest scales** | All bar charts start at zero. No truncated y-axes that exaggerate small differences. The exception is the scatter plot, where the natural range is centered on the break-even line. |
| **Avoid pie charts** | Not used anywhere. Bars allow precise comparison; pies do not. |
| **Avoid dual axes** | Not used. Two-measure charts use grouped bars on a shared axis instead. |

---

## Filter Design

Three primary filters were chosen based on which slices leadership most commonly needs:

| Filter | Why it's there |
|---|---|
| **Date range** | Single-select year vs all-time view (24 months) |
| **Region** | Most common executive question: "What does this look like in West?" |
| **Category** | Lets users isolate the Technology vs Furniture story |
| **Customer segment** | Required for segment-specific insight |
| **Ship mode** | Operational filter for shipping diagnostics |

The dashboard uses a **filter action** so that clicking a region bar (in the regional chart) automatically filters all other views — a tactile demonstration of interactivity that goes beyond pull-down filters.

---

## Tableau Implementation Notes

For the Tableau workbook (`tableau/executive_dashboard.twbx`):
- Each chart above corresponds to one Tableau worksheet
- The composite dashboard layout pulls those worksheets into a tiled dashboard with the KPI cards at top, charts arranged in 2 rows below
- The 5 filters are placed in a right-side panel that applies to all worksheets
- Filter actions are configured: clicking a region or category cross-filters the dashboard
