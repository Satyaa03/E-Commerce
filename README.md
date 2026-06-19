# E-Commerce Data Pipeline & Analytics

An end-to-end **Medallion Architecture (Bronze → Silver → Gold)** ETL pipeline built in Python on a Supabase (PostgreSQL) backend, with the resulting Gold-layer tables visualized in a Power BI dashboard.

## Overview

This project takes a raw e-commerce orders dataset (`train.csv`, in the style of the classic "Superstore" dataset) and processes it through three progressively refined layers of a data warehouse:

| Layer | Purpose |
|-------|---------|
| **Bronze** | Loads the raw CSV as-is (no cleaning) into Supabase, tagging every row with batch/lineage metadata (`batch_id`, `source_file_name`, `data_source`, `load_date`, `is_active`) for full auditability. |
| **Silver** | Cleans and standardizes the Bronze data — renames columns to `snake_case`, enforces primary-key and business-rule data quality checks, and applies **SCD Type 2** (Slowly Changing Dimension) logic to track historical changes to customers and products. |
| **Gold** | Builds business-ready, aggregated analytical tables: sales summary, product performance, customer analysis, and regional analysis — orchestrated by a single ETL function. |

The final Gold tables power the included Power BI dashboard (`E-commerce.pbix` / `dashboard.png`).

## Repository Contents

| File | Description |
|------|-------------|
| `E_Commerce.ipynb` | Main Jupyter/Colab notebook containing the full Bronze → Silver → Gold ETL pipeline. |
| `train.csv` | Raw source dataset (orders, customers, products). |
| `E-commerce.pbix` | Power BI report built on top of the Gold-layer tables. |
| `dashboard.png` | Screenshot/preview of the Power BI dashboard. |

## Architecture

```
train.csv
   │
   ▼
┌─────────────┐     raw load + lineage tags      ┌─────────────┐
│   BRONZE    │ ───────────────────────────────► │  Supabase   │
│  (raw data) │                                   │  (Postgres) │
└─────────────┘                                   └─────────────┘
   │
   ▼ clean, validate, SCD Type 2
┌─────────────┐
│   SILVER    │
│ (clean data)│
└─────────────┘
   │
   ▼ aggregate
┌─────────────┐
│    GOLD     │ ──► Sales Summary, Product Performance,
│ (analytics) │     Customer Analysis, Region Analysis
└─────────────┘
   │
   ▼
Power BI Dashboard (E-commerce.pbix)
```

## Tech Stack

- **Python** — pandas, numpy, hashlib, datetime
- **SQLAlchemy** — database connectivity and SQL execution
- **Supabase (PostgreSQL)** — cloud data warehouse storing the Bronze/Silver/Gold schemas
- **Google Colab** — notebook execution environment (uses `google.colab.userdata` for secrets)
- **Power BI** — final dashboarding/visualization layer

## Setup & Usage

1. **Create a Supabase project** and note your project reference, host, and database password.
2. **Set Colab secrets** (the notebook reads credentials via `google.colab.userdata`):
   - `SUPABASE_PASSWORD`
   - `SUPABASE_PROJECT_REF`
   - `SUPABASE_HOST`
3. **Upload `train.csv`** to your Colab/working environment.
4. **Run the notebook top to bottom** — it will:
   - Load and tag the raw data into Bronze tables.
   - Clean, validate, and apply SCD Type 2 logic to produce Silver tables.
   - Aggregate Silver data into Gold analytical tables via `run_gold_etl()`.
5. **Open `E-commerce.pbix`** in Power BI Desktop (connected to the same Supabase database) to explore the dashboard, or view `dashboard.png` for a quick preview.

## Pipeline Details

### Bronze Layer
- Reads `train.csv` exactly as-is (preserving original formatting/strings).
- Splits the raw file into three staging tables: `bronze_customers`, `bronze_products`, `bronze_orders`.
- Adds lineage columns (`batch_id`, `source_file_name`, `data_source`, `load_date`, `is_active`) for traceability.

### Silver Layer
- Renames raw column headers to clean `snake_case` (e.g., `Customer ID` → `customer_id`).
- Runs data quality checks: non-null primary keys, valid foreign key references, and business-rule validation (e.g., valid order dates, non-negative sales).
- Implements **SCD Type 2** for `silver_customers` and `silver_products` to preserve historical attribute changes.
- Produces validated `silver_orders`, `silver_customers`, and `silver_products` tables.

### Gold Layer
Reads from the Silver tables and builds four analytical views:
- **Sales Summary** — total sales, total orders, average order value.
- **Product Performance** — sales and order metrics grouped by product.
- **Customer Analysis** — sales and order metrics grouped by customer.
- **Region Analysis** — sales and order metrics grouped by region/state/city.

All four are orchestrated together by `run_gold_etl()`.

## Notes

- This notebook is designed to run in **Google Colab** (it uses `google.colab.userdata` for secret management); to run elsewhere, replace this with environment variables or another secrets manager.
- Database credentials should never be hard-coded — always use a secrets manager (Colab Secrets, `.env`, etc.).
