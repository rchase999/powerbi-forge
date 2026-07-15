---
name: powerbi-dashboard
description: >-
  Build a complete Power BI dashboard or report end to end with the pbi-cli
  tool. Use whenever the user wants to create, design, or automate a Power BI
  (.pbix / .pbip / PBIR) dashboard, semantic model, DAX measures, report pages,
  or visuals — especially when the data should come from an open-source API or
  database. Triggers on "power bi", "powerbi", "pbi", "dashboard", "dax",
  "semantic model", "tmdl", "report page", "visuals".
---

# Power BI Forge — Dashboard Build Orchestration

You are building a real Power BI dashboard by driving the **pbi-cli** command
line tool. This skill is the master workflow. Two companion skills carry the
detail — load them as needed:

- **`open-data-sources`** — free APIs and databases to feed the dashboard, and
  the exact Power Query (M) recipes to ingest each one. Prefer open data over
  fabricated numbers.
- **`pbi-cli-reference`** — the full command map and the `--help` / `--json`
  discovery protocol. Never guess a flag; discover it.
- **`dashboard-design`** — the layout grid, theme JSON, visual-selection matrix,
  and DAX patterns that make the result look professional instead of default.

## Golden rules

1. **Never fabricate data.** Wire a real open-source API or database (see
   `open-data-sources`). Use sample data only if the user explicitly asks, and
   say so plainly.
2. **Discover, don't guess.** Before using any pbi-cli subcommand you are unsure
   about, run `pbi <group> <cmd> --help`. Add `--json` to any read command so you
   get machine-readable output you can act on.
3. **Validate every layer before moving on.** `dax validate` before committing a
   measure, `report validate` after editing report JSON, `model stats` to
   confirm the model shape. Report what you actually observed — never claim a
   step passed without seeing its output.
4. **Snapshot before risky edits.** `pbi database export-tmdl <path>` gives you a
   restore point for the semantic model. Use `pbi database diff-tmdl` to show the
   user exactly what changed.
5. **Work in small, reversible steps** and keep the user's intent front and
   center: what decision should this dashboard help them make?

## Preconditions

pbi-cli is **Windows-only**, needs **Python 3.10+**, and needs **Power BI Desktop
running** for anything that touches the semantic model. If the environment is not
ready, run the `/powerbi-forge:setup` command (or walk the user through it):

```bash
pipx install pbi-cli-tool      # once (or: pip install pbi-cli-tool)
pbi-cli skills install         # register pbi-cli's own skill hub, once
pbi connect                    # attach to the running Power BI Desktop instance
```

Report-layer work (PBIR JSON: pages, visuals, filters, bookmarks, themes) needs
**no connection** — you can scaffold and edit a report project offline. Only the
model layer (tables, measures, DAX, relationships) needs `pbi connect`.

## The build workflow

Think of it as five phases. Announce the plan, then execute phase by phase,
validating as you go.

### 1. Frame the dashboard
Clarify with the user (one focused question at a time if anything is ambiguous):
- **Audience & decision** — who reads it and what action it drives.
- **3–7 key questions** the dashboard must answer (these become the visuals).
- **Data source** — pick a concrete open API/DB from `open-data-sources` if the
  user has none.
- **Grain** — the level of one fact row (e.g. one order line, one day, one sensor
  reading).

### 2. Ingest the data (model layer — needs `pbi connect`)
Turn the source into tables. The ingestion path is a Power Query (M) expression
attached to a table partition:
- Use `pbi expression` to manage shared M queries and `pbi partition` to attach an
  M partition to a table; `pbi table create` to declare tables.
- For a REST API: an M `let` block using `Web.Contents` + `Json.Document`.
- For a database: `Sql.Database`, `PostgreSQL.Database`, or `Odbc.DataSource`
  (DuckDB/SQLite). Exact recipes live in `open-data-sources`.
- If an API is messy, stage it first: pull with a tiny Python/DuckDB script into a
  local SQLite/DuckDB/Parquet file, then point Power BI at that clean store.

### 3. Model it as a star schema (needs `pbi connect`)
A good model is what separates an "awesome" dashboard from a slow, confusing one.
- One or more **fact** tables (measures/events) surrounded by **dimension** tables
  (who/what/when/where). Keep facts narrow, dimensions descriptive.
- Create a **date dimension** and mark it as the date table — use `pbi calendar`.
  Time intelligence DAX depends on it.
- Define **relationships** (`pbi relationship`), typically one-to-many from
  dimension to fact, single-direction filtering unless you have a reason.
- Hide raw key/technical columns; surface friendly names. Add `pbi hierarchy` for
  drill paths (e.g. Year → Quarter → Month).
- Verify with `pbi --json model stats`.

### 4. Author DAX measures (needs `pbi connect`)
Measures are the analytics. Build up in layers (see `dashboard-design` for the
full pattern library):
1. **Base measures** — `Total Sales = SUM(...)`, distinct counts, etc.
2. **Time intelligence** — YTD, MTD, YoY %, rolling averages, off the date table.
3. **Ratios & rates** — margin %, conversion %, share of total.
4. Optionally **calculation groups** (`pbi calc-group`) to apply time-intelligence
   variants across many measures without duplicating them.

For every measure: `pbi dax validate "<expr>"` first, then create it, then sanity
-check the number with `pbi dax execute "EVALUATE ROW(\"v\", [Measure])"`.

### 5. Build the report (report layer — no connection needed)
Assemble the pages and visuals (see `dashboard-design` for layout + theme):
- `pbi report create` to scaffold a PBIR project.
- Apply a **corporate theme** (`pbi report set-background`, `pbi format ...`) up
  front so every visual inherits a consistent look.
- Add pages (`pbi report add-page`), then visuals (`pbi visual add`) placed on a
  clean grid — KPI cards across the top, a trend line, a breakdown bar, a detail
  table, slicers down one side.
- Bind visuals to model fields (`pbi visual bind` / `visual bulk-bind`).
- Add slicers and page/visual filters (`pbi filters add-categorical`,
  `add-topn`, `add-relative-date`).
- Add `pbi bookmarks` for guided states or reset buttons where useful.
- For a chart Power BI can't do natively, author a custom visual with the
  open-source `powerbi-visuals-tools` scaffold (D3 / Lodash / date-fns allowed) —
  see `open-data-sources` for the toolchain.
- Finish with `pbi report validate` and `pbi report preview`, and report what you
  saw.

## Custom / advanced visuals (open-source)

pbi-cli can embed a custom visual you build with Microsoft's open-source SDK:

```bash
npx --yes powerbi-visuals-tools@^5.6.0 new my-visual
# edit src/visual.ts (D3, Lodash, date-fns are within the npm allowlist)
npx --yes powerbi-visuals-tools@^5.6.0 package
pbi visual import-custom dist/my-visual.pbiviz --replace
```

## Packaging a standalone `.pbip` (critical gotcha)

`pbi report create --dataset-path` references the model folder as a **PBIP
semantic-model artifact**, but `pbi database export-tmdl` writes only *raw* TMDL.
Power BI Desktop then refuses to open the `.pbip` with
`RequiredArtifactMissing: definition.pbism`. A valid semantic-model artifact needs:

```
MyModel.Dataset/            (or .SemanticModel)
├── .platform               ← {"metadata":{"type":"SemanticModel","displayName":"MyModel"},
│                              "config":{"version":"2.0","logicalId":"<guid>"}}
├── definition.pbism        ← {"version":"4.2","settings":{}}
└── definition/             ← ALL exported TMDL moves under here
    ├── model.tmdl · database.tmdl · relationships.tmdl
    ├── tables/*.tmdl
    └── cultures/*.tmdl
```

After `export-tmdl`, create `definition/`, move every `.tmdl` (and the `tables/`
and `cultures/` folders) into it, then add `.platform` and `definition.pbism` at
the artifact root. Keep the folder name the same so the report's
`definition.pbir` `byPath` (`../MyModel.Dataset`) still resolves. Verify the
`.pbip` actually opens (launch it, confirm no `FrownSnapShot*.zip` appears in
`%LOCALAPPDATA%\Microsoft\Power BI Desktop\`) before claiming done — do not rely
on `report validate` alone, which only checks the report layer.

## Deliver

When done, summarize for the user: which data source is wired, the model shape
(facts/dims/measures from `model stats`), the pages and visuals, and how to open
the result in Power BI Desktop. Offer a TMDL snapshot (`pbi database export-tmdl`)
so they can version-control or restore the model.
