# 🚗 Malaysia Vehicle Registration Trends

A data engineering pipeline built on **Databricks** that ingests, transforms, and analyses vehicle registration data from Malaysia's official open data portal ([data.gov.my](https://data.gov.my)).

---

## 📌 Project Overview

This project follows the **Medallion Architecture** (Bronze → Silver → Gold) to process vehicle registration data published by the Road Transport Department (JPJ) of Malaysia. The goal is to surface trends in car registrations by manufacturer, fuel type, and state over time.

This is a learning project built to practise real-world data engineering workflows using PySpark, Delta Lake, and Databricks.

---

## 🗂️ Project Structure

```
malaysia-vehicle-trends/
├── notebooks/
│   ├── 01_bronze_ingest.ipynb       # Raw data ingestion from data.gov.my API
│   ├── 02_silver_transform.ipynb    # Cleaning, type casting, standardisation
│   └── 03_gold_aggregate.ipynb      # Business-ready aggregations and metrics
├── data/
│   └── README.md                    # Notes on data sources and schema
└── README.md
```

---

## 📊 Data Source

All data is sourced from **[data.gov.my](https://data.gov.my)** — Malaysia's official open government data portal, maintained by the Department of Statistics Malaysia (DOSM) and the Road Transport Department (JPJ).

| Dataset | Source | Update Frequency |
|---|---|---|
| Vehicle Registration Transactions (Cars) | JPJ via data.gov.my | Monthly |
| Vehicle Registrations by Fuel Type | JPJ via data.gov.my | Monthly |

Data is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

---

## 🏗️ Architecture

```
data.gov.my API / Parquet files
        │
        ▼
  [Bronze Layer]
  Raw data, no transformations
  Stored as Delta tables
        │
        ▼
  [Silver Layer]
  Cleaned, typed, standardised
  Joined with lookup tables
        │
        ▼
  [Gold Layer]
  Aggregated, business-ready
  Monthly trends, market share, YoY growth
```

---

## 🛠️ Tech Stack

- **Platform:** Databricks Free Edition (Serverless)
- **Language:** Python (PySpark) + Spark SQL
- **Storage Format:** Delta Lake
- **Orchestration:** Databricks Workflows
- **Version Control:** GitHub

---

## 📈 Key Questions This Pipeline Answers

- Which car manufacturers have the highest registrations in Malaysia?
- How are electric vehicle (EV) registrations trending year over year?
- Which states have the highest vehicle registration volumes?
- How has the market share of each manufacturer changed over time?

---

## 🚀 Getting Started

### Prerequisites
- A [Databricks Free Edition](https://www.databricks.com/learn/free-edition) account
- A GitHub account with this repo connected as a Git folder in Databricks

### Running the Pipeline

Run the notebooks in order inside Databricks:

1. `01_bronze_ingest.ipynb` — pulls raw data from data.gov.my and saves to Delta
2. `02_silver_transform.ipynb` — cleans and standardises the raw data
3. `03_gold_aggregate.ipynb` — produces aggregated tables ready for analysis

Each notebook is self-contained and can be run independently after Bronze is populated.

---

## 📝 Notes

- This project uses **serverless compute** on Databricks Free Edition, so no cluster configuration is needed.
- Daily compute limits apply on the Free Edition — if your compute pauses, it resets the next day.
- Data from data.gov.my is updated monthly; re-running `01_bronze_ingest.ipynb` will refresh the data.

---

## 🙋 Author

Built by Muhammad Syafiq Farhan as a hands-on learning project for data engineering with Databricks.