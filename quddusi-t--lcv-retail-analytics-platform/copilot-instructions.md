## lcv-retail-analytics-platform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LCV Retail Analytics Platform simulates LC Waikiki's global retail data infrastructure: PostgreSQL OLTP source → GCS Parquet staging → BigQuery data warehouse → dbt transformations → analytics/ML consumption. It's a portfolio project demonstrating enterprise data engineering patterns.

## Development Commands

### Setup
```bash
uv install                          # Install dependencies (preferred)
cp .env.template .env               # Configure environment variables
```

### Data Pipeline
```bash
# 1. Seed PostgreSQL with synthetic data (50 stores, 500 products, 10K customers, 1M sales)
python src/postgres/seed_synthetic_data.py

# 2. Extract PostgreSQL → GCS Parquet (use --local for testing without GCS)
python src/etl/postgres_to_gcs.py --local

# 3. Create BigQuery datasets
python src/etl/setup_bigquery.py

# 4. Load GCS → BigQuery raw layer
python src/etl/gcs_to_bigquery.py
```

### dbt Transformations
```bash
cd src/etl/dbt_project
dbt run          # Run all models (staging → marts)
dbt test         # Run data quality tests
dbt docs generate && dbt docs serve
```

### ML API
```bash
uvicorn src.ml.api:app --reload --host 0.0.0.0 --port 8000
# Docs at http://localhost:8000/docs
```

### Code Quality
```bash
black src/
ruff check src/ --fix
mypy src/
pytest tests/ -v
pytest tests/ --cov=src
pre-commit run --all-files
```

## Architecture

### Data Flow
```
PostgreSQL (star schema) → GCS (Parquet, date-partitioned) → BigQuery raw → dbt staging → dbt marts → BI/ML
```

### BigQuery Layers
- **retail_analytics_raw**: Raw tables loaded directly from GCS Parquet files
- **staging**: dbt views — deduplication, null validation, type coercion (materialized as VIEWS for cost)
- **marts**: dbt tables — pre-aggregated analytics ready for BI/ML (materialized as TABLES, partitioned + clustered)

### dbt Model Structure
`src/etl/dbt_project/models/`
- **staging/**: `stg_sales_clean`, `stg_customer_clean`, `stg_product_clean`, `stg_store_clean`, `stg_date_clean`
- **marts/**: `fct_daily_sales_trends`, `fct_store_performance`, `fct_product_performance`, `fct_regional_sales`, `fct_customer_lifetime_value`

Variable `raw_dataset = "retail_analytics_raw"` set in `dbt_project.yml` is used in staging models to reference source tables.

### PostgreSQL Star Schema
Defined in `SCHEMA/star_schema.sql`. Fact table `fact_sales` (~1M rows, grain = one transaction) references `dim_date`, `dim_product`, `dim_store`, `dim_customer`. `dim_product` has SCD Type 2 columns (`valid_from`, `valid_to`, `is_current`).

### Analytics SQL Views
`src/analytics/queries.sql` defines views on PostgreSQL: YoY sales growth, RFM customer segments, inventory turnover, churn risk detection (90+ day inactivity). These are separate from dbt models and run against PostgreSQL directly.

### ML Service (Planned)
`src/ml/` — FastAPI app with endpoints `/api/predict/churn` and `/api/predict/demand`. Churn uses scikit-learn (Logistic Regression/Random Forest); demand forecasting uses XGBoost/Prophet.

## Environment Variables

Required (see `.env.template`):
- `POSTGRES_*`: Connection details for source database
- `GCP_PROJECT_ID`, `GCP_KEY_PATH`, `GCS_BUCKET`: GCP credentials and bucket
- `BIGQUERY_DATASET`: Target dataset (default: `retail_analytics`)

Data generation scale (optional, defaults provided):
- `NUM_STORES=50`, `NUM_PRODUCTS=500`, `NUM_CUSTOMERS=10000`, `NUM_SALES=1000000`, `RANDOM_SEED=42`

## Key Reference Files
- `SCHEMA/data_dictionary.md` — Complete column definitions, business rules, data lineage
- `docs/GOVERNANCE.md` — Data quality checks and SCD handling
- `docs/BEST_PRACTICES.md` — SQL optimization patterns used in this project
- `src/etl/dbt_project/models/*/schema.yml` — dbt model tests and column descriptions

---
> Source: [quddusi-t/lcv-retail-analytics-platform](https://github.com/quddusi-t/lcv-retail-analytics-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
