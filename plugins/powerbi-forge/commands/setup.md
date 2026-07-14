---
description: Install and verify pbi-cli, then connect to Power BI Desktop
argument-hint: "(optional) data source, e.g. localhost:54321"
---

Get the environment ready to build Power BI dashboards with pbi-cli.

Requirements: **Windows**, **Python 3.10+**, and **Power BI Desktop must be
running** for model work.

Do this, reporting the real output of each step (do not claim success you did not
observe):

1. Check whether pbi-cli is installed: `pbi --version` (or `pbi --help`).
2. If missing, install it:
   ```bash
   pipx install pbi-cli-tool      # preferred; falls back to: pip install pbi-cli-tool
   ```
3. Register pbi-cli's own skill hub (one-time): `pbi-cli skills install`.
4. Connect to the running Power BI Desktop instance:
   - `pbi connect` — or, if the user passed a data source in `$ARGUMENTS`,
     `pbi connect --data-source $ARGUMENTS`.
5. Confirm the connection and model shape: `pbi --json model stats`.
6. Report status: installed version, connection state, and model summary. If
   Power BI Desktop is not open, tell the user to open it (or a `.pbix`) first —
   the report layer can still be scaffolded offline, but model edits cannot.

If anything fails, quote the exact error and suggest the fix; do not retry blindly.
