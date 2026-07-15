---
name: pbi-cli-reference
description: >-
  Command reference for the pbi-cli tool that drives Power BI Desktop — every
  command group (dax, table, column, measure, relationship, report, visual,
  filters, format, bookmarks, database/TMDL, security, trace, etc.), the global
  --json flag, the REPL, and a discovery protocol so flags are never guessed. Use
  when running any pbi / pbi-cli command for a Power BI dashboard or model.
---

# pbi-cli Command Reference

pbi-cli exposes ~27 command groups over two layers: a **semantic model** layer
(direct .NET / TOM / ADOMD interop into a running Power BI Desktop — needs
`pbi connect`) and a **report** layer (direct PBIR JSON edits — no connection).

The invocation is `pbi <group> <command> [options]`. The distributed console
scripts are `pbi` and `pbi-cli` (`pbi-cli` also carries `setup` / `skills`).

## Discovery protocol — do this, don't guess flags

The exact flags per subcommand are **not memorized here on purpose**, so you never
invent one. Discover them:

```bash
pbi --help                 # top-level: list all groups
pbi <group> --help         # e.g. pbi visual --help
pbi <group> <cmd> --help   # e.g. pbi visual add --help  → real flags
```

Add `--json` before the group on any **read** command for machine-readable output
you can parse and act on:

```bash
pbi --json measure list
pbi --json visual list --page "Overview"
pbi --json dax execute "EVALUATE TOPN(5, Sales)"
pbi --json model stats
```

## Setup, connection, REPL

```bash
pbi-cli skills install          # register pbi-cli's own skill hub (one-time)
pbi-cli skills list|uninstall
pbi setup                       # initialize pbi-cli
pbi connect                     # attach to running Power BI Desktop
pbi connect --data-source localhost:54321   # attach to a specific instance
pbi disconnect
pbi connections list|last       # named / most-recent connections
pbi repl                        # interactive REPL: tab-complete, history
```

## Semantic model layer (requires `pbi connect`)

| Group | Purpose |
| --- | --- |
| `dax execute` / `dax validate` / `dax clear-cache` | run, syntax-check, and cache-clear DAX |
| `table` | create / manage tables |
| `column` | create / manage columns (incl. calculated) |
| `measure` | create / manage measures |
| `relationship` | create / manage relationships |
| `hierarchy` | create / manage hierarchies (drill paths) |
| `calc-group` | calculation groups (reusable time-intelligence, etc.) |
| `partition` | table partitions (where the M source lives) |
| `expression` | shared M (Power Query) queries |
| `calendar` | configure & mark the date table |
| `advanced culture` | culture / locale |
| `security-role` | row-level security (RLS) roles |
| `perspective` | perspectives (scoped model views) |
| `model stats` | model shape / size diagnostics |
| `trace start|stop|fetch|export` | performance tracing |

### Deployment & snapshots

```bash
pbi database export-tmdl <path>   # snapshot the model as TMDL (version-controllable)
pbi database import-tmdl <path>
pbi database export-tmsl <path>   # TMSL (JSON) scripting format
pbi database diff-tmdl <a> <b>    # show what changed between snapshots
pbi transaction ...               # group model edits atomically
```

Take a TMDL snapshot **before** risky model edits; `diff-tmdl` shows the user
exactly what you changed.

## Report layer (no connection needed — PBIR JSON)

```bash
pbi report create                 # scaffold a PBIR project
pbi report info|validate|preview|reload
pbi report add-page|delete-page|get-page
pbi report set-background|set-visibility
```

### Visuals (32 built-in visual types + custom)

```bash
pbi visual add|get|list|update|delete
pbi visual bind|bulk-bind          # bind visual(s) to model fields
pbi visual bulk-update|bulk-delete
pbi visual where                   # select visuals by criteria (for bulk ops)
pbi visual import-custom <file.pbiviz> --replace
pbi visual list-custom|remove-custom
```

### Filters, formatting, bookmarks

```bash
pbi filters list
pbi filters add-categorical|add-topn|add-relative-date
pbi filters remove|clear

pbi format get|clear
pbi format background-gradient|background-conditional|background-measure

pbi bookmarks list|get|add|delete|set-visibility
```

## Working discipline

- Prefer `--json` reads → parse → act, over eyeballing human output.
- After any model edit, `pbi --json model stats`. After any report edit,
  `pbi report validate`. Report the real output; never claim success unseen.
- Bulk commands (`visual bulk-*`, `visual where`) exist for a reason — use them
  instead of looping single edits when touching many visuals.
- If a command errors, read the exact error text, run its `--help`, and adjust —
  do not retry the same call blindly.

## Binding gotchas (verified)

- **Slicer columns → `Missing_References`.** `pbi visual bind <slicer> --field
  "Table[Column]"` writes the projection as a `"Measure"` reference, but a slicer
  needs a **column**. Power BI then fails to render with "Underlying Error:
  Missing_References". Fix: in the slicer's `visual.json`, change the projection's
  `"Measure"` wrapper to `"Column"` (same `Expression`/`Property`). `--field` is
  correct for *cards* (which show measures), wrong for slicers (which need columns).
  After binding slicers, grep the visual JSON to confirm each column is under
  `"Column"`, not `"Measure"`.
- Chart `--category` writes columns correctly (as `"Column"`); only `--field` on a
  column is the trap.
