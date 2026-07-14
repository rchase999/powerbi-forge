# Changelog

All notable changes to Power BI Forge are documented here. This project follows
[Semantic Versioning](https://semver.org).

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
