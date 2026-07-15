---
name: web-base-to-powerbi
description: >-
  Design-base workflow for a wow-factor Power BI dashboard: first author an
  animated web page (HTML/CSS/JS) as the visual "design base", verify it in a real
  browser, then convert that base element-by-element into a NATIVE Power BI
  dashboard (semantic model + PBIR) using Power BI's full capabilities. Use when
  the user wants a stunning/impressive dashboard, a mockup-first approach, or to
  "turn a webpage into a Power BI dashboard".
---

# Web Design-Base → Native Power BI Dashboard

Build the *look* fast in the web (where layout, color, and motion are cheap to
iterate), then rebuild it as a **native Power BI dashboard** — the web page is the
**spec**, not the deliverable. The native report is what ships, because only it
gives live data, cross-filtering, DAX, and drill. Pairs with the
`powerbi-dashboard`, `dashboard-design`, and `pbi-cli-reference` skills.

> Why not just ship HTML? A web page can't do what a BI tool does: a real semantic
> model, DAX measures, cross-highlight, drill-through, slicers, RLS, refresh. The
> base captures the *design*; Power BI supplies the *engine*.

## Workflow

### 1. Author the design base (web)
Create one self-contained HTML file — the "wow" mock — grounded in the subject:
- A KPI row, 2–4 chart panels, and any hero/timeline element, laid out on a grid.
- Real numbers from the intended data source (never lorem).
- Subject-specific palette + type; tasteful motion (count-up, bar-grow, path-draw,
  hover). Respect `prefers-reduced-motion`.
- Keep it ASCII-safe: use HTML entities (`&mdash;` `&middot;` `&rarr;`) in markup
  and `\uXXXX` escapes in JS strings, so it survives any charset.

### 2. Verify the base in a real renderer (ground it)
Serve it (`python -m http.server`, since `file:` is often blocked in headless
browsers) and screenshot it with Playwright. Look at the actual pixels; fix what
the screenshot reveals (mojibake, overflow, contrast); re-render. A static lint is
not observation. Optionally publish it as an artifact for the user to see/approve.

### 3. Convert each element to its native Power BI equivalent
Build the model first (`powerbi-dashboard` skill), then translate the base:

| Web design-base element | Native Power BI equivalent |
| --- | --- |
| KPI tile / count-up number | **Card** (or **KPI** visual) bound to a measure |
| Number + delta vs target | **KPI visual** (`--indicator` + `--goal`) or card + conditional color |
| Animated line / area chart | **Line / area chart** (`--category` date, `--value` measure) |
| Growing bar chart | **Bar / column chart** (sorted; Top-N filter) |
| Donut / pie | **Donut / pie chart** (few categories) |
| Filter chips / segmented control | **Slicer** (bind the **column** with `--field`, then confirm it's a `Column`, not `Measure`, or you get `Missing_References`) |
| Nav buttons / tabs | **pageNavigator** visual, or **buttons + bookmarks** |
| Tooltip popovers | **Report-page tooltips** (a hidden tooltip page) |
| "Explore / what-if" controls | **Field parameters** + **what-if parameter** |
| Drill / expand-on-click | **Drill-through page** + **decomposition tree** |
| "Why did X change" callout | **Key influencers** visual |
| Timeline ribbon | **Slicer (timeline)** or a small-multiple / scrollable list |
| Hover glow / active states | Cross-highlight (native) + conditional formatting |
| Page theme / colors | Theme via Desktop UI import, or bake `dataPoint.defaultColor` per visual (see `dashboard-design` — file-embedding a custom theme is blocked in the PBIR preview) |

### 4. Use Power BI's full capabilities (the "wow" checklist)
Go beyond static charts — this is what makes it impress:
- **Interactivity:** slicers, cross-highlight, drill-down, drill-through pages.
- **Navigation:** bookmarks + buttons / pageNavigator; a bookmark "story" sequence.
- **Guided analysis:** decomposition tree, key influencers, Q&A visual.
- **Parameters:** field parameters (swap measures/dimensions live), what-if sliders.
- **Polish:** custom report-page tooltips, KPI/gauge, conditional formatting,
  a consistent theme, a clean 16:9 grid.
- **Model depth:** star schema, marked date table, time-intelligence DAX,
  calculation groups.

### 5. Validate + verify in Desktop
`pbi report validate` after edits, then open the `.pbip` and confirm it renders
(the report layer's failures — `Failed to load the report`, `Missing_References` —
don't show in `validate`; only opening reveals them). Fix, reload, confirm.

## Deliverables
Ship both: the **web base** (design reference / approval mock) and the **native
`.pbip`** (the working dashboard). Keep the base in the project so the intended
look is documented and re-derivable.
