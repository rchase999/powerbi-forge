---
name: dashboard-design
description: >-
  Design patterns that make a Power BI dashboard look professional — page layout
  grid, a corporate color theme JSON, a visual-selection matrix (which chart for
  which question), KPI card design, and a layered DAX measure pattern library
  (base, time intelligence, ratios). Use when laying out report pages, choosing
  visuals, theming, or writing DAX for a Power BI dashboard.
---

# Dashboard Design — Make It Awesome, Not Default

A dashboard earns "awesome" through restraint and structure, not clutter. Apply
these patterns while building the report and model.

## Layout: one screen, one grid

Design each page on a **1280×720** canvas as a grid. A reliable executive layout:

```
┌──────────────────────────────────────────────────────────┐
│  Title / logo                          date range slicer   │  header (~60px)
├──────────┬──────────┬──────────┬──────────┬───────────────┤
│  KPI 1   │  KPI 2   │  KPI 3   │  KPI 4   │               │  cards row (~110px)
├──────────┴──────────┴──────────┴──────────┤   slicers     │
│                                            │   (category,  │
│         Trend over time (line)             │    region,    │
│                                            │    etc.)      │
├───────────────────────┬────────────────────┤              │
│   Breakdown (bar)      │   Detail (table)    │              │
└───────────────────────┴────────────────────┴───────────────┘
```

Principles:
- **Z-pattern reading**: most important number top-left; supporting detail
  bottom-right.
- **Top row = KPIs** (the "so what" at a glance), then **trend**, then
  **breakdown**, then **detail**.
- **Align and pad consistently.** Snap visuals to the grid; equal gutters. Give
  everything room to breathe.
- **One accent color** for the primary metric; muted neutrals for the rest.
- **≤ 6–8 visuals per page.** More than that, add a page and use bookmarks or
  drill-through to navigate.
- Every visual gets a **clear title that states the insight**, not just the field
  name ("Revenue up 12% YoY", not "Sum of Revenue").

## Color theme (apply first, so visuals inherit it)

Author a Power BI theme JSON and apply it before adding visuals. Use a
**color-blind-safe** palette and consistent typography:

```json
{
  "name": "Forge Corporate",
  "dataColors": ["#2E6BE6", "#12A594", "#F2A93B", "#E5534B", "#7A5AF8", "#64748B"],
  "background": "#FFFFFF",
  "foreground": "#0F172A",
  "tableAccent": "#2E6BE6",
  "good": "#12A594", "neutral": "#64748B", "bad": "#E5534B",
  "textClasses": {
    "title":    { "fontFace": "Segoe UI Semibold", "fontSize": 14, "color": "#0F172A" },
    "label":    { "fontFace": "Segoe UI",          "fontSize": 10, "color": "#334155" },
    "callout":  { "fontFace": "Segoe UI",          "fontSize": 28, "color": "#0F172A" }
  }
}
```

Apply and reinforce with `pbi report set-background`, `pbi format
background-gradient|background-conditional|background-measure`. Keep the same
palette across pages. Reserve red strictly for "bad"; don't decorate with it.

## Visual selection matrix — chart for the question

| The question is about… | Use | Avoid |
| --- | --- | --- |
| A single headline number + trend | **KPI / Card** with sparkline | 3D anything |
| Change over time | **Line** (or area for volume) | Pie over time |
| Compare categories | **Bar / column** (sorted!) | Unsorted bars |
| Part-to-whole (few parts) | **Stacked bar / 100% bar / donut ≤5 slices** | Pie with many slices |
| Ranking / top-N | **Bar sorted + Top-N filter** | Tables the user must scan |
| Two measures' relationship | **Scatter** | Dual-axis combo unless necessary |
| Geographic pattern | **Map / filled map** | Map when a bar would be clearer |
| Row-level detail / audit | **Table / matrix** with conditional formatting | Overusing tables for trends |
| Progress to target | **Gauge / KPI with target** | Gauge for non-goal metrics |

Rules: **sort bars by value**, start axes at zero for bars, limit series to what
the eye can track (≤ ~5 lines), turn on data labels only where they aid reading.
Add Top-N and relative-date filters so pages stay focused (`pbi filters add-topn`,
`add-relative-date`).

## KPI card design

Each card: big **callout** number, a short label, and a **delta vs comparison**
(prior period or target) colored good/neutral/bad. Drive the delta color with a
measure and `pbi format background-measure` / conditional formatting. Four to five
cards max in the top row.

## DAX measure pattern library

Build measures in layers. Validate each with `pbi dax validate` before creating
it; sanity-check the value with `pbi dax execute`.

```dax
-- 1. Base measures
Total Sales   = SUM ( Sales[Amount] )
Order Count   = DISTINCTCOUNT ( Sales[OrderID] )

-- 2. Time intelligence (needs a marked date table — pbi calendar)
Sales YTD     = TOTALYTD ( [Total Sales], 'Date'[Date] )
Sales PY      = CALCULATE ( [Total Sales], SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
Sales YoY %   =
    VAR cur = [Total Sales]
    VAR py  = [Sales PY]
    RETURN DIVIDE ( cur - py, py )
Rolling 3M    =
    CALCULATE ( [Total Sales],
        DATESINPERIOD ( 'Date'[Date], MAX ( 'Date'[Date] ), -3, MONTH ) )

-- 3. Ratios & rates (always DIVIDE, never "/", to dodge divide-by-zero)
Margin %      = DIVIDE ( [Total Profit], [Total Sales] )
% of Total    = DIVIDE ( [Total Sales], CALCULATE ( [Total Sales], ALL ( Sales ) ) )

-- 4. Delta vs target, for KPI card coloring
Sales vs Target = [Total Sales] - [Sales Target]
```

Conventions:
- Use `DIVIDE()` for every ratio.
- Use variables (`VAR`) for readable, single-scan measures.
- Format measures explicitly (currency, %, thousands) so cards read cleanly.
- Consider a **calculation group** (`pbi calc-group`) to apply YTD / PY / YoY
  across many base measures instead of writing each variant by hand.

## Accessibility & polish checklist

- Color-blind-safe palette; never encode meaning by color alone (add labels/icons).
- Sufficient text contrast; readable font sizes (≥10pt labels).
- Consistent number formatting and units everywhere.
- Descriptive, insight-led titles on every visual and page.
- Tooltips add context, not clutter; alt text on key visuals where supported.
- Test the page at the target resolution with `pbi report preview` before
  declaring it done.
