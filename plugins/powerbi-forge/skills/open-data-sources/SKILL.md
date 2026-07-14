---
name: open-data-sources
description: >-
  Catalog of free, open-source data for Power BI dashboards — public REST APIs,
  open databases and sample datasets — plus the exact Power Query (M) recipes to
  ingest each into a pbi-cli semantic model, and open-source tools (DuckDB, dbt,
  powerbi-visuals-tools, D3) for staging and visuals. Use when choosing or wiring
  a data source for a Power BI dashboard, or when the user has no data yet.
---

# Open Data Sources & Ingestion Recipes

**Default to open data.** A dashboard is only as good as what feeds it. Prefer a
real, free, public source over invented numbers. This skill lists sources that
need no paid key (or a free one) and the M recipe to load each into Power BI via
pbi-cli's `expression` / `partition` / `table` commands.

## How ingestion works in pbi-cli

A table's data comes from a **partition** whose source is a **Power Query (M)**
expression. So "add a data source" = "write an M `let` block and attach it to a
table."

```bash
# 1. discover the exact flags first
pbi expression --help
pbi partition --help
pbi table --help

# 2. typical flow (confirm flag names via --help):
pbi table create --name Sales
pbi partition ...      # attach the M expression below as the table's partition
pbi --json model stats # verify the table loaded
```

Two ingestion strategies:
- **Direct** — Power BI calls the API/DB itself via the M recipe. Best for clean,
  stable sources.
- **Staged** — pull with a small open-source script (Python `requests`/`httpx` +
  `pandas`, or `duckdb`) into a local **SQLite / DuckDB / Parquet / CSV** file,
  then point Power BI at that clean store. Best for messy APIs, pagination,
  auth, or heavy transforms. Staging keeps the model fast and refreshable.

---

## Public REST APIs (no/low friction)

All return JSON and load with the `Web.Contents` + `Json.Document` pattern below.

| Domain | Source | Base URL | Key? |
| --- | --- | --- | --- |
| Countries / demographics | REST Countries | `https://restcountries.com/v3.1/all` | No |
| Global development | World Bank | `https://api.worldbank.org/v2/country/all/indicator/<code>?format=json` | No |
| Weather / climate | Open-Meteo | `https://api.open-meteo.com/v1/forecast?...` | No |
| FX / currency | Frankfurter (ECB) | `https://api.frankfurter.app/latest?from=USD` | No |
| Crypto markets | CoinGecko | `https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd` | No |
| Earthquakes | USGS | `https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson` | No |
| Air quality | OpenAQ | `https://api.openaq.org/v3/...` | Free key |
| Space / astronomy | NASA Open APIs | `https://api.nasa.gov/...` | Free key (`DEMO_KEY`) |
| Software / repos | GitHub REST | `https://api.github.com/...` | Free token (higher limits) |
| Public transport | TfL Unified | `https://api.tfl.gov.uk/...` | Free key |
| Curated world data | Our World in Data | CSV/JSON exports per dataset | No |
| Government open data | data.gov / EU / Singapore data.gov.sg | portal-specific | Mostly no |
| API testing / demo | JSONPlaceholder | `https://jsonplaceholder.typicode.com/...` | No |

### M recipe — REST API → table

```m
let
    Source     = Web.Contents(
        "https://restcountries.com",
        [ RelativePath = "v3.1/all", Query = [ fields = "name,population,region" ] ]
    ),
    Json       = Json.Document(Source),
    ToTable    = Table.FromList(Json, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    Expanded   = Table.ExpandRecordColumn(ToTable, "Column1", {"name","population","region"}),
    Typed      = Table.TransformColumnTypes(Expanded, {{"population", Int64.Type}})
in
    Typed
```

Notes:
- Use `RelativePath` + `Query` (not string concatenation) so refresh and
  credentials work correctly.
- For a **paginated** API, stage it instead (see below) — looping pagination in M
  is fragile.
- Put a free API key in a query parameter only for demo work; for anything shared,
  stage the pull so the key never lives in the model.

---

## Open databases & sample datasets

Great for realistic star-schema dashboards without hunting for data.

| Dataset | What it is | Get it as |
| --- | --- | --- |
| **Chinook** | Music store (invoices, tracks, customers) | SQLite / Postgres / MySQL |
| **Northwind** | Classic orders / products / customers | SQLite / Postgres |
| **Pagila / Sakila** | DVD-rental (Postgres / MySQL) | Postgres / MySQL |
| **AdventureWorks** | Microsoft sample retail warehouse | SQL Server / CSV |
| **NYC Taxi** | Huge public trips dataset | Parquet (ideal for DuckDB) |
| **Our World in Data** | COVID, energy, CO₂, population | CSV |
| Kaggle / Hugging Face datasets | Community datasets, any domain | CSV / Parquet |

**Open-source database engines** you can point Power BI at: PostgreSQL, MySQL/
MariaDB, SQLite, and **DuckDB** (excellent for local analytics on Parquet/CSV).
Supabase gives a free hosted open-source Postgres if you want it in the cloud.

### M recipes — database → table

```m
// PostgreSQL (native connector)
let Source = PostgreSQL.Database("localhost:5432", "chinook"),
    Invoice = Source{[Schema="public", Item="invoices"]}[Data]
in Invoice
```

```m
// SQL Server / AdventureWorks
let Source = Sql.Database("localhost", "AdventureWorks"),
    Sales  = Source{[Schema="Sales", Item="SalesOrderHeader"]}[Data]
in Sales
```

```m
// SQLite or DuckDB via ODBC (install the driver once)
let Source = Odbc.DataSource("dsn=DuckDB", [HierarchicalNavigation=true])
in Source
```

```m
// Parquet / CSV file (great with DuckDB-exported extracts)
let Source = Parquet.Document(File.Contents("C:\data\nyc_taxi.parquet"))
in Source
```

---

## Staging with open-source tools (recommended for anything non-trivial)

When an API needs pagination, auth, joins, or cleanup, stage it with free tools,
then load the clean output. Keep secrets in env vars / a `.env`, never in the model.

- **Python** — `requests`/`httpx` to pull, `pandas` to shape, write to SQLite,
  Parquet, or CSV.
- **DuckDB** — query CSV/Parquet/JSON/HTTP directly with SQL; blazing fast, zero
  server. `duckdb.sql("SELECT * FROM read_json_auto('https://...')").write_parquet(...)`.
- **dbt** — version-controlled SQL transforms if the model gets complex.
- **Singer / Meltano** — open-source EL taps for hundreds of sources.

Example (Python → SQLite the model then reads via ODBC/`Sql`):

```python
import requests, pandas as pd, sqlite3
rows = requests.get("https://api.frankfurter.app/2024-01-01..2024-12-31?from=USD").json()
df = pd.DataFrame(rows["rates"]).T.reset_index(names="date")
sqlite3.connect("fx.db").executescript("")  # ensure file
df.to_sql("fx_rates", sqlite3.connect("fx.db"), if_exists="replace", index=False)
```

---

## Open-source tooling for visuals

- **`powerbi-visuals-tools`** (Microsoft, MIT) — scaffold custom visuals in
  TypeScript; pbi-cli embeds the packaged `.pbiviz`.
- **D3.js, Lodash, date-fns** — within pbi-cli's npm allowlist for custom-visual
  code, so you can render bespoke charts (chord, sankey, radial, network) that
  the built-in visual set lacks.

## Choosing well

- Match the source to the **decision** the dashboard drives, not to whatever is
  easiest to fetch.
- Prefer a source with a **time dimension** (dates) — it unlocks trend and
  time-intelligence analysis, the backbone of a good dashboard.
- State the source, its license/terms, and refresh cadence to the user so the
  dashboard is honest about where its numbers come from.
