# Changelog

All notable changes to Power BI Forge are documented here. This project follows
[Semantic Versioning](https://semver.org).

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
