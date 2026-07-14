# Power BI Forge

A Claude Code plugin that turns Claude into a Power BI dashboard builder. It wraps
the open-source [`pbi-cli`](https://github.com/MinaSaad1/pbi-cli) tool so Claude
can model a star schema, author DAX, and lay out polished report pages — and it
steers Claude toward **free open-source APIs and databases** for the data instead
of made-up numbers.

## What it gives Claude

**Skills** (auto-loaded when relevant)
- `powerbi-dashboard` — the five-phase build workflow: frame → ingest → model →
  measure → report, with validation gates at every layer.
- `open-data-sources` — a catalog of free public REST APIs (World Bank,
  Open-Meteo, REST Countries, CoinGecko, USGS, GitHub…) and open databases
  (Chinook, Northwind, Pagila, DuckDB, Postgres, SQLite) with the exact Power
  Query (M) recipe to ingest each.
- `pbi-cli-reference` — the full pbi-cli command map plus a `--help` / `--json`
  discovery protocol so flags are never guessed.
- `dashboard-design` — layout grid, a color-blind-safe theme JSON, a
  visual-selection matrix, and a layered DAX pattern library.

**Slash commands**
- `/powerbi-forge:setup` — install/verify pbi-cli and connect to Power BI Desktop.
- `/powerbi-forge:dashboard <goal + source>` — build a dashboard end to end.
- `/powerbi-forge:add-source <api | db | dataset>` — wire an open data source into
  the model.

**Agent**
- `powerbi-architect` — a specialist subagent (Opus) for delegating full builds.

## Requirements

- **Windows**, **Python 3.10+**, and **Power BI Desktop** (model work needs it
  running; report/PBIR edits can happen offline).
- `pbi-cli` itself:
  ```bash
  pipx install pbi-cli-tool      # or: pip install pbi-cli-tool
  pbi-cli skills install
  pbi connect                    # with Power BI Desktop open
  ```
  Or just run `/powerbi-forge:setup` and let Claude handle it.

## Install the plugin

From the marketplace in this repo:

```
/plugin marketplace add rchase999/powerbi-forge
/plugin install powerbi-forge@powerbi-forge
```

Or point Claude Code at a local checkout for development:

```bash
claude --plugin-dir ./plugins/powerbi-forge
```

## Quick start

```
/powerbi-forge:setup
/powerbi-forge:dashboard  a sales overview from the Chinook sample database
```

Claude will pick or confirm an open data source, build a star schema, write the
DAX, lay out the report, validate each layer, and hand you a dashboard plus an
optional TMDL snapshot.

## How it works

pbi-cli drives Power BI over two layers — the **semantic model** (via .NET / TOM /
ADOMD into a running Power BI Desktop) and the **report** (direct PBIR JSON
edits). This plugin doesn't reimplement any of that; it gives Claude the workflow,
the open-data recipes, the command reference, and the design system to use pbi-cli
well.

## Notes

- Not affiliated with or endorsed by Microsoft. Power BI is a Microsoft trademark.
- pbi-cli is a separate project under its own license; this plugin only
  orchestrates it. See `LICENSE`.
