---
description: Wire a free open-source API or database into the Power BI model
argument-hint: "<api url | db connection | dataset name>"
---

Add this data source to the current Power BI semantic model:

**$ARGUMENTS**

Follow the `open-data-sources` skill:

1. **Classify** the source: REST API, database (Postgres/MySQL/SQLite/DuckDB/SQL
   Server), or a file/dataset (CSV/Parquet). If `$ARGUMENTS` is vague or empty,
   recommend a concrete free open source that fits the user's goal.
2. **Pick the strategy**:
   - *Direct* — clean, stable source → Power BI queries it via M.
   - *Staged* — paginated / auth'd / messy → pull with an open-source script
     (Python `requests`/`httpx` + `pandas`, or `duckdb`) into a local
     SQLite/DuckDB/Parquet file, then load that. Keep any key in an env var, never
     in the model.
3. **Write the M recipe** (`Web.Contents`+`Json.Document` for APIs;
   `PostgreSQL.Database`/`Sql.Database`/`Odbc.DataSource`/`Parquet.Document` for
   stores) using the exact patterns in `open-data-sources`.
4. **Attach it**: discover flags with `pbi table --help`, `pbi partition --help`,
   `pbi expression --help`, then create the table and attach the M partition.
5. **Verify** the table loaded and typed correctly: `pbi --json model stats` and a
   quick `pbi dax execute "EVALUATE TOPN(5, '<Table>')"`. Report row counts and
   column types you actually observed.
6. **Tell the user** the source, its license/terms, refresh cadence, and how it
   fits the model (fact or dimension, and its grain).
