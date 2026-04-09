## artometrix-yolia

> **Quant Briefs** is a lightweight, CrewAI-powered SaaS that generates daily “hedge-fund style” quant briefs for a given watchlist of tickers.

# CLAUDE.md — Quant Briefs Project

## Project Overview

**Quant Briefs** is a lightweight, CrewAI-powered SaaS that generates daily “hedge-fund style” quant briefs for a given watchlist of tickers.

Each brief blends:

- **Market data (Yahoo Finance)** — prices, returns, volatility.
- **Quant metrics** — CAGR, Sharpe, Max Drawdown, Momentum.
- **LLM insights** — 5-bullet commentary summarizing market signals, risks, and what to watch next.
- **Sector heatmaps** and **PDF/HTML reports** (Citadel-style).

Goal: create a deployable learning project that blends **CrewAI + FastAPI + GCP + Vercel** to practice LLM orchestration, deployment, and financial data automation.

---

## 🏗️ System Architecture

[Vercel Frontend (Next.js)]
└──> calls
<https://api.quant-briefs.yourdomain.com>

[GCP Cloud Run Backend (FastAPI + CrewAI)]
├── /report?tickers=ASTS,RKLB,KTOS
├── /jobs/daily (triggered by Cloud Scheduler)
├── uses Cloud SQL (Postgres) or Firestore
├── stores reports to GCS bucket
├── sends email via SendGrid/Postmark
└── secrets in Secret Manager

[Cloud Scheduler]
└── daily trigger (07:30 Europe/Paris)

---

## ⚙️ Key Components

### Backend (Python / FastAPI)

- `crew/agents.py` — defines Data, Quant, Writer, QA agents.
- `crew/tasks.py` — defines tasks orchestrated by CrewAI.
- `crew/quant.py` — computes quant metrics (CAGR, Sharpe, etc.).
- `crew/tools.py` — wraps Yahoo Finance (price, metadata, news).
- `crew/crew.py` — builds & runs the Crew pipeline.
- `app.py` — FastAPI entrypoint (serves `/report` + scheduled jobs).
- `templates/report.html.j2` — HTML template rendered into PDF.

### Frontend (Next.js / Vercel)

- Minimal dashboard to enter watchlist, trigger report, and view output.
- Uses `NEXT_PUBLIC_API_BASE` to call backend API.
- Optional paywall with Stripe or Lemon Squeezy.

---

## ☁️ Deployment Setup

### Hybrid deployment (recommended)

| Component | Platform | Notes |
|------------|-----------|-------|
| **Frontend** | Vercel | Next.js static site; preview deploys |
| **Backend** | GCP Cloud Run | Dockerized FastAPI + CrewAI |
| **DB** | Cloud SQL (Postgres) or Firestore | User/watchlist storage |
| **Storage** | GCS Bucket | PDF & report archive |
| **Scheduler** | Cloud Scheduler | Calls `/jobs/daily` |
| **Secrets** | Secret Manager | API keys, DB creds |
| **Domain** | Cloud Run domain mapping | `api.quant-briefs.yourdomain.com` |

### Backend environment variables

### Frontend environment variables (Vercel)

---

## 🧩 Data Schema

**User**

- id, email, plan, created_at

**Watchlist**

- id, user_id, name

**Ticker**

- id, watchlist_id, symbol, note

**PriceCache**

- symbol, date, open, high, low, close, volume

**Report**

- id, user_id, watchlist_id, created_at, html_path, pdf_path, summary_json

---

## 💡 Key Workflows

### `/report?tickers=ASTS,RKLB,KTOS`

1. CrewAI pipeline runs.
2. Metrics + commentary generated.
3. HTML rendered via Jinja2.
4. PDF generated with WeasyPrint → saved to GCS.
5. HTML returned to browser.

### Daily cron (Cloud Scheduler)

1. Triggers `/jobs/daily` on Cloud Run.
2. Iterates all active watchlists.
3. Regenerates briefs.
4. Emails summaries to users.

---

## 📊 Quant Metrics

| Metric | Formula | Description |
|:--|:--|:--|
| **CAGR** | `(P_end / P_start)^(1/years) - 1` | Compound annual growth |
| **Volatility** | `σ(daily returns) × √252` | Annualized risk |
| **Sharpe** | `(CAGR - rf) / Vol` | Risk-adjusted return |
| **Max Drawdown** | `min(P / cummax(P) - 1)` | Largest drop from peak |
| **Momentum 6M/12M** | `P_t / P_t-L - 1` | Relative strength |

---

## 🧰 Tech Stack

| Layer | Stack |
|-------|-------|
| Frontend | Next.js + Tailwind (Vercel) |
| Backend | FastAPI + CrewAI + WeasyPrint |
| Data | yfinance, pandas, numpy |
| DB | Cloud SQL (Postgres) |
| Orchestration | Cloud Scheduler + Cloud Run |
| Deployment | Docker + GitHub Actions + Artifact Registry |
| Auth | HMAC between Vercel and API |
| PDF Rendering | WeasyPrint (Cairo/Pango) |

---

## 🚀 Quickstart (local)

```bash
# 1. Run locally
cd backend
pip install -r requirements.txt
uvicorn app:app --reload

# 2. Visit
http://localhost:8000/report?tickers=ASTS,RKLB,KTOS

# 3. (optional) Build container
docker build -t quant-briefs-api .




## Collaboration Guidelines (for Claude)

When contributing or asked for help:
 1. Prefer concise, deterministic JSON outputs when chaining agents.
 2. Maintain clear separation between quant logic (Python) and narrative (LLM).
 3. Keep reproducibility: every report must re-run from stored price data + config.
 4. Avoid hallucinating data — if missing, return "N/A".
 5. When writing text, match the Citadel / hedge-fund tone (analytical, unemotional).
 6. Always validate metrics with QA agent before output.
 7. Respect secrets and never expose API keys or credentials.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Skoomy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Skoomy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
