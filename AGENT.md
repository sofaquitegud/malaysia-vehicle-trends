# AGENT.md

This file provides full project context for AI coding agents (OpenAI Codex) working in this repository. Read this before making any changes.

---

## Project Summary

This is a **data engineering learning project** built on Databricks Free Edition (serverless). It ingests Malaysia vehicle registration data from [data.gov.my](https://data.gov.my) — the official Malaysian government open data portal — and processes it through a Bronze → Silver → Gold medallion architecture using PySpark and Delta Lake.

The primary author is learning Databricks and data engineering. Prioritise **clarity and readability** over cleverness. Always explain what the code does in comments.

---

## Repository Structure

```
malaysia-vehicle-trends/
├── notebooks/
│   ├── 01_bronze_ingest.ipynb       # Raw ingestion from data.gov.my
│   ├── 02_silver_transform.ipynb    # Cleaning and standardisation
│   └── 03_gold_aggregate.ipynb      # Aggregations and business metrics
├── data/
│   └── README.md                    # Data source notes and schema docs
├── AGENT.md                         # This file
└── README.md                        # Project overview
```

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Databricks Free Edition | Compute platform (serverless) |
| PySpark | Data processing |
| Delta Lake | Storage format for all tables |
| Spark SQL | Querying Gold tables |
| Python 3 | Scripting and notebook code |
| data.gov.my API | Primary data source |
| GitHub | Version control |

---

## Data Sources

All data comes from **data.gov.my** and is licensed under CC BY 4.0.

### Primary datasets

| Dataset | API ID | Parquet URL |
|---|---|---|
| Vehicle Registration Transactions (Cars) | `registration_transactions_car` | `https://storage.data.gov.my/transportation/cars_{YEAR}.parquet` |

### API base URL
```
https://api.data.gov.my/data-catalogue
```

### Example API call
```python
import requests
response = requests.get(
    "https://api.data.gov.my/data-catalogue",
    params={"id": "registration_transactions_car", "limit": 50}
)
```

### Preferred ingestion method
Load from **Parquet files** where available — they contain the full dataset without pagination and load faster than the API. Only use the API for incremental or filtered fetches.

```python
import pandas as pd
df = pd.read_parquet("https://storage.data.gov.my/transportation/cars_2025.parquet")
```

---

## Architecture: Medallion Pattern

Every data transformation follows this strict layering. **Never skip a layer or mix responsibilities between layers.**

### Bronze — Raw Ingestion
- Load data exactly as-is from the source (no transformations)
- Store as Delta table: `bronze_vehicle_registrations`
- Keep all original column names, types, and values — even dirty ones
- Append a `ingested_at` timestamp column only
- Write mode: `overwrite` (full refresh for now, incremental later)

### Silver — Cleaned & Standardised
- Cast columns to correct types (especially dates)
- Rename columns to snake_case and meaningful names
- Drop rows where critical fields are null
- Add derived columns: `year`, `month`
- Store as Delta table: `silver_vehicle_registrations`
- No aggregations here — row-level only

### Gold — Aggregated & Business-Ready
- Aggregations, joins, and metrics go here
- Each Gold table answers a specific business question
- Store as Delta tables with descriptive names, e.g.:
  - `gold_monthly_by_manufacturer`
  - `gold_yearly_market_share`
  - `gold_ev_registration_trend`

---

## Coding Conventions

### General
- Use **PySpark DataFrame API** as the primary transformation tool
- Use **Spark SQL** (`%sql` magic or `spark.sql()`) for Gold layer queries and ad-hoc analysis
- Avoid pandas for large transformations — use it only for small lookups or API calls in the ingestion step
- All notebooks must run **top to bottom without errors** before committing

### Naming
- Table names: `{layer}_{description}` in snake_case — e.g. `silver_vehicle_registrations`
- Column names: snake_case — e.g. `total_registrations`, `manufacturer_name`
- Variables: snake_case — e.g. `df_bronze`, `df_silver`
- Notebook prefixes: `01_`, `02_`, `03_` to indicate execution order

### Comments
- Every major transformation block must have a comment explaining **why**, not just what
- Add a markdown cell at the top of each notebook explaining its purpose and inputs/outputs
- Add a markdown cell at the top of each major section (Ingest, Clean, Write)

### PySpark style
```python
# Preferred: method chaining with one operation per line
df_silver = (
    df_bronze
    .withColumn("date", F.to_date("date"))
    .withColumn("year", F.year("date"))
    .withColumn("month", F.month("date"))
    .filter(F.col("date").isNotNull())
    .withColumnRenamed("maker", "manufacturer")
)

# Avoid: deeply nested one-liners that are hard to read
```

### Imports
Always import PySpark functions as `F`:
```python
from pyspark.sql import functions as F
from pyspark.sql import SparkSession
```

---

## What to Avoid

- **Do not use `df.toPandas()` on large datasets** — this pulls everything into driver memory and will crash on Free Edition
- **Do not hardcode file paths** — use variables or constants at the top of the notebook
- **Do not transform data in the Bronze layer** — Bronze is read-only after ingestion
- **Do not use `.show()` as the only debugging tool** — use `.printSchema()`, `.describe()`, and `.count()` together
- **Do not write to paths outside Delta tables** — all outputs should be Delta tables, not raw CSVs written to DBFS
- **Do not use `spark.read.csv()` for the main datasets** — Parquet is the preferred format
- **Do not commit notebooks with cell outputs** — clear all outputs before committing (Databricks does this automatically on push, but double-check)

---

## Delta Table Operations

### Writing tables
```python
# Full overwrite (use during development)
df.write.format("delta").mode("overwrite").saveAsTable("silver_vehicle_registrations")

# Append (use for incremental loads later)
df.write.format("delta").mode("append").saveAsTable("bronze_vehicle_registrations")
```

### Reading tables
```python
df = spark.table("silver_vehicle_registrations")
```

### Checking what tables exist
```python
spark.sql("SHOW TABLES").show()
```

---

## Environment Notes

- **Platform:** Databricks Free Edition — serverless compute only, no classic clusters
- **Spark session:** Already available as `spark` in every notebook — do not create a new one
- **Python version:** Whatever Databricks serverless provides — do not pin to a specific version
- **No local environment** — this project runs entirely on Databricks. If a VS Code + Databricks Connect setup is added later, this file will be updated
- **Compute limits:** Free Edition has daily compute quotas. If compute is unavailable, it resets the next day. Data and tables are not deleted when compute pauses

---

## Git Workflow

- **Main branch:** `main` — always stable and runnable
- **Feature branches:** `feature/notebook-name` — e.g. `feature/bronze-ingest`
- Commit after each notebook runs successfully end-to-end
- Commit messages should be descriptive: `Add silver transformation with date casting and null handling`
- Never commit broken notebooks to `main`

---

## Current Project Status

| Layer | Notebook | Status |
|---|---|---|
| Bronze | `01_bronze_ingest.ipynb` | ✅ Completed |
| Silver | `02_silver_transform.ipynb` | ✅ Completed |
| Gold | `03_gold_aggregate.ipynb` | ✅ Completed |

Update this table as notebooks are completed.

---

## Useful References

- [data.gov.my Data Catalogue](https://data.gov.my/data-catalogue)
- [data.gov.my API Docs](https://developer.data.gov.my)
- [Databricks Free Edition Docs](https://docs.databricks.com/aws/en/getting-started/free-edition)
- [Delta Lake Documentation](https://docs.delta.io/latest/index.html)
- [PySpark API Reference](https://spark.apache.org/docs/latest/api/python/)
- [Medallion Architecture Guide](https://www.databricks.com/glossary/medallion-architecture)