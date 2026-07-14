# Power BI Forge — Claude Code marketplace

This repository is a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces)
hosting **Power BI Forge**, a plugin that lets Claude build Power BI dashboards by
driving the open-source [`pbi-cli`](https://github.com/MinaSaad1/pbi-cli) tool —
and encourages it to feed those dashboards from free open-source APIs and
databases.

## Install

```
/plugin marketplace add rchase999/powerbi-forge
/plugin install powerbi-forge@powerbi-forge
```

(Or use a local path: `/plugin marketplace add ./` from the repo root.)

For local development without installing:

```bash
claude --plugin-dir ./plugins/powerbi-forge
```

## Layout

```
powerbi/
├── .claude-plugin/
│   └── marketplace.json          # marketplace entry
└── plugins/
    └── powerbi-forge/            # the plugin
        ├── .claude-plugin/plugin.json
        ├── skills/               # powerbi-dashboard, open-data-sources,
        │                         #   pbi-cli-reference, dashboard-design
        ├── commands/             # setup, dashboard, add-source
        ├── agents/               # powerbi-architect
        ├── README.md · LICENSE · CHANGELOG.md
```

See [`plugins/powerbi-forge/README.md`](plugins/powerbi-forge/README.md) for full
usage, requirements, and the quick start.

## Requirements

Windows · Python 3.10+ · Power BI Desktop · `pbi-cli` (`pipx install
pbi-cli-tool`). The `/powerbi-forge:setup` command handles the tool install and
connection for you.

## License

MIT (this plugin). `pbi-cli` and the Microsoft Analysis Services libraries it
bundles are distributed separately under their own terms. Not affiliated with
Microsoft.
