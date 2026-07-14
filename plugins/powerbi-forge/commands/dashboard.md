---
description: Build a complete Power BI dashboard end to end with pbi-cli
argument-hint: "<goal + data source>, e.g. 'sales KPIs from the Chinook SQLite DB'"
---

Build a Power BI dashboard for this request:

**$ARGUMENTS**

Delegate the build to the `powerbi-architect` subagent, or run it yourself
following the `powerbi-dashboard` skill workflow. Either way:

1. **Frame it** — confirm the audience, the decision it drives, and the 3–7
   questions it must answer. Ask one focused question only if something essential
   is ambiguous; otherwise proceed with sensible defaults and state them.
2. **Source the data** — if no data source is named, pick a concrete free
   open-source API or database from the `open-data-sources` skill and say which,
   with its license/terms. Never fabricate numbers.
3. **Model** — ingest via a Power Query (M) partition, build a star schema, mark a
   date table (`pbi calendar`), set relationships. Verify with
   `pbi --json model stats`.
4. **Measure** — author layered DAX (base → time intelligence → ratios), each one
   `pbi dax validate`'d before creation.
5. **Report** — apply the theme first, then lay out pages on the design grid from
   the `dashboard-design` skill (KPI cards, trend, breakdown, detail, slicers),
   bind visuals, add filters/bookmarks. Finish with `pbi report validate` and
   `pbi report preview`.
6. **Deliver** — summarize the data source, model shape, pages, and how to open
   it; offer a TMDL snapshot (`pbi database export-tmdl`).

Ensure Power BI Desktop is running and `pbi connect` succeeds before model work
(run `/powerbi-forge:setup` if not). Report what you actually observe at each
gate — no unverified completion claims.
