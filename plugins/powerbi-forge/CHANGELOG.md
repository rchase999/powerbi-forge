# Changelog

All notable changes to Power BI Forge are documented here. This project follows
[Semantic Versioning](https://semver.org).

## [0.2.0] - 2026-07-15

### Added
- `web-base-to-powerbi` skill: a design-base workflow — author an animated web
  page as the visual spec, verify it in a real browser, then convert it
  element-by-element into a native Power BI dashboard (semantic model + PBIR)
  using Power BI's full capabilities. Includes the web-element → native-visual
  mapping table and a "wow" capability checklist (bookmarks/navigation,
  drill-through, field parameters, decomposition tree, key influencers, tooltips).

## [0.1.3] - 2026-07-15

### Fixed
- `pbi-cli-reference` skill: documented that `visual bind --field` on a slicer
  column writes it as a `"Measure"` reference, causing Power BI's
  "Missing_References" error; the fix is to change `"Measure"`→`"Column"` in the
  slicer's `visual.json`. `--field` is correct for cards, wrong for slicer columns.
- `dashboard-design` skill: added the root cause of the theme-load failure (PBIR
  preview forbids externally added RegisteredResources, per Microsoft docs) and a
  supported automated alternative — baking `dataPoint.defaultColor` into each
  chart's `visual.json`.

## [0.1.2] - 2026-07-15

### Fixed
- `dashboard-design` skill now documents the `pbi report set-theme` gotcha: the
  embedded PBIR `RegisteredResources` custom theme can make Power BI Desktop fail
  to open the report ("Something went wrong — Failed to load the report") while
  the model still loads. Adds the isolate-by-stripping step and the reliable
  re-apply path (import the theme through Desktop's View → Themes → Browse UI, or
  bake colors per visual). Also notes the invalid `BaseThemes/` path prefix
  `set-theme` may emit for custom themes.

## [0.1.1] - 2026-07-15

### Fixed
- `powerbi-dashboard` skill now documents the PBIP semantic-model packaging
  gotcha: `database export-tmdl` writes raw TMDL, but a standalone `.pbip`
  requires the model folder to be a valid artifact (`.platform` +
  `definition.pbism` at the root, TMDL under `definition/`). Without it Power BI
  Desktop fails to open with `RequiredArtifactMissing: definition.pbism`. Added
  the fix steps and a launch-and-check-for-FrownSnapShot verification step.

## [0.1.0] - 2026-07-14

### Added
- Initial release.
- `powerbi-dashboard` skill: end-to-end orchestration workflow for building a
  dashboard with pbi-cli (connect → model → measure → report → validate).
- `open-data-sources` skill: catalog of free open-source APIs and databases plus
  the exact Power Query (M) recipes to ingest them into the semantic model.
- `pbi-cli-reference` skill: condensed, verified map of every pbi-cli command
  group with a `--help`/`--json` discovery protocol so no flags are guessed.
- `dashboard-design` skill: layout grid, color theme JSON, visual-selection
  matrix, and DAX measure patterns that make a report look professional.
- `/powerbi-forge:setup`, `/powerbi-forge:dashboard`, and
  `/powerbi-forge:add-source` slash commands.
- `powerbi-architect` subagent for delegating full dashboard builds.
