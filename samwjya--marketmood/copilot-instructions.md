## marketmood

> Real-time pipeline that correlates financial news

# MarketMood

Real-time pipeline that correlates financial news 
sentiment with crypto price movements.

## What We're Building
Ingest top 5 crypto prices + financial news headlines,
score sentiment, store in AWS S3, transform with dbt,
load into BigQuery, serve via FastAPI, display on
Streamlit dashboard.

## Stack
- Python 3.11
- Airflow 2.8
- dbt-bigquery
- FastAPI
- Streamlit
- Terraform (AWS only — S3, EC2, networking)
- Docker
- GitHub Actions CI/CD

## Coins We Track
BTC, ETH, BNB, SOL, XRP

## Data Sources
- CoinGecko API (crypto prices, free, no key needed)
- NewsAPI (financial headlines, free tier)

## Project Structure
MarketMood/
├── terraform/
│   └── modules/
│       ├── networking/
│       ├── storage/
│       └── compute/
├── ingestion/
│   ├── crypto_ingestion.py
│   ├── news_ingestion.py
│   └── sentiment_scorer.py
├── airflow/
│   └── dags/
│       ├── crypto_pipeline_dag.py
│       └── news_pipeline_dag.py
├── dbt/
│   ├── models/
│   │   ├── staging/
│   │   │   ├── stg_crypto_prices.sql
│   │   │   └── stg_news_sentiment.sql
│   │   └── marts/
│   │       └── mart_price_sentiment_correlation.sql
│   └── dbt_project.yml
├── api/
│   └── main.py
├── dashboard/
│   └── app.py
├── docker/
│   ├── Dockerfile.ingestion
│   ├── Dockerfile.api
│   └── docker-compose.yml
└── .github/
    └── workflows/
        └── ci.yml

## Environment Variables Needed
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=us-east-1
S3_BUCKET_NAME=marketmood-raw
NEWSAPI_KEY=
GOOGLE_APPLICATION_CREDENTIALS=path/to/key.json
BIGQUERY_PROJECT=marketmood
BIGQUERY_DATASET=marketmood

## Build Progress
- [ ] Terraform skeleton
- [ ] CoinGecko ingestion → S3
- [ ] NewsAPI ingestion → S3
- [ ] Sentiment scoring
- [ ] Airflow DAGs
- [ ] dbt models
- [ ] BigQuery loading
- [ ] FastAPI endpoints
- [ ] Streamlit dashboard
- [ ] Docker
- [ ] CI/CD

## Architecture
CoinGecko + NewsAPI
    → Python ingestion services
    → S3 raw storage (AWS)
    → Airflow DAGs (hourly)
    → dbt transformation
    → BigQuery (GCP free tier)
    → FastAPI serving layer
    → Streamlit dashboard
```

**Copy this into .gitignore:**
```
.env
*.json
__pycache__/
*.pyc
.terraform/
*.tfstate
*.tfstate.backup
.terraform.lock.hcl
venv/
.venv/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samwjya) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
