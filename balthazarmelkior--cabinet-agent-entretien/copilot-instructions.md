## cabinet-agent-entretien

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LangGraph-powered agent for French accounting firms. It analyzes client balance sheets (FEC or PDF), benchmarks ratios against sector norms, detects financial signals, and generates a personalized interview brief in Word format.

## Running the Application

```bash
# Setup
python -m venv .venv
source .venv/Scripts/activate  # Windows: .venv\Scripts\activate

pip install -r requirements.txt

cp .env.example .env  # then fill in OPENAI_API_KEY

# Launch Streamlit UI
streamlit run app/main.py
```

## Environment Variables (`.env`)

| Variable | Required | Default | Purpose |
|---|---|---|---
| `OPENAI_API_KEY` | Yes | — | GPT-4o for all LLM calls |
| `INSEE_API_KEY` | No | — | INSEE Esane API (benchmark fallback) |
| `LLM_MODEL` | No | `gpt-4o` | LLM model to use |
| `EMBEDDING_MODEL` | No | `text-embedding-3-small` | RAG embeddings |
| `CATALOGUE_PATH` | No | `data/catalogue_missions.json` | Mission catalog location |
| `BDF_CACHE_DIR` | No | `data/bdf_cache` | Banque de France API cache |
| `RAG_THRESHOLD` | No | `50` | Catalog size above which RAG is used instead of direct LLM matching |

## Architecture

### LangGraph Pipeline (`graph.py`)

The main orchestrator is `prepare_entretien_bilan(fichier_path, catalogue_path, code_naf)`. It builds and runs a `StateGraph` with this topology:

```
extract_financial_data
    ├─→ detect_signals ──────────┐
    └─→ benchmark_sectoriel ─────┤
                                  ▼
                            match_missions → generate_interview_plan → END
```

The two middle nodes run in parallel. State flows through a `BillanState` TypedDict holding the intermediate and final Pydantic models.

### Key Pydantic Models (`models.py`)

- `DonneesFinancieres` — normalized financial data (income statement + balance sheet items as `PosteComptable`)
- `Signal` — detected risk/opportunity with type, severity, code, and leviers (talking points)
- `RatioSectoriel` — a ratio with client value, sector median/quartiles, and interpretation
- `BenchmarkSectoriel` — full benchmark result with list of `RatioSectoriel`
- `Mission` / `MissionRecommandee` — catalog entry and scored recommendation
- `FicheEntretien` — final output: exec summary, vigilance points, interview plan, missions, checklist

### Document Parsing (`parsers/`)

- **FEC** (`fec_parser.py`): Standard DGFiP tab-separated file (latin-1), mapped to French PCG account codes using pandas. Returns `DonneesFinancieres`.
- **PDF** (`pdf_parser.py`): Extracts text via pdfplumber, then sends to GPT-4o with a JSON schema for structured extraction.

### Sector Benchmarking (`benchmark/`)

Three-source **fallback chain** managed by `BenchmarkOrchestrator`:

1. **Banque de France** (`sources/bdf.py`) — Queries SDMX-JSON Webstat API, caches results in `data/bdf_cache/`. Provides 7 ratios. Highest reliability.
2. **INSEE Esane** (`sources/insee.py`) — Bearer token required; provides client/supplier delays only.
3. **LLM Estimation** (`sources/llm_source.py`) — GPT-4o generates plausible sector norms from NAF code as last resort.

`BenchmarkSource` (abstract base in `base.py`) defines the interface each source implements.

### Signal Detection (`nodes/detect_signals.py`)

1. Computes `Ratios` from `DonneesFinancieres` (`analysis/ratios.py`)
2. Applies deterministic rule-based detection (`analysis/rules.py`) — 10 rules covering EBE, debt, liquidity, client delays, BFR, etc.
3. Enriches with GPT-4o for qualitative/soft signals (sectoral, governance, regulatory)
4. Sorts by severity descending

To add a new deterministic signal: add a rule function in `analysis/rules.py` following the existing pattern and append it to the rules list.

### Mission Matching (`matching/`)

- **≤50 missions** (`llm_matcher.py`): All missions passed directly to GPT-4o for scoring (0.0–1.0).
- **>50 missions** (`rag_matcher.py`): Chroma vectorstore retrieves top-k candidates per signal, then LLM scores the shortlist.

The threshold is controlled by `RAG_THRESHOLD` env variable. The mission catalog lives in `data/catalogue_missions.json`.

### Interview Plan Generation (`nodes/generate_interview_plan.py`)

Receives all upstream context and calls GPT-4o (temperature=0.2) to produce a structured `FicheEntretien`: executive summary, vigilance points, thematic interview plan (context + open question + associated mission per theme), argumentaires, and a document checklist.

### Word Export (`output/word_generator.py`)

`generate_word_doc(fiche, client_name, annee)` → returns a `BytesIO` `.docx`. Uses python-docx with styled headings and tables matching a corporate template.

### Streamlit UI (`app/main.py`)

Single-page app with two phases:
1. **Form**: Upload FEC/PDF, enter client name + NAF code, click "Lancer l'analyse"
2. **Dashboard**: KPI cards + 4 tabs (Benchmark, Signaux, Missions, Fiche Entretien) + Word download

UI subcomponents: `app/components/charts.py` (Plotly radar + bar), `app/components/cards.py` (HTML signal/mission cards), `app/components/download.py` (BytesIO helper).

## Testing

No tests exist yet. `pytest` and `pytest-asyncio` are in `requirements.txt`. To run when tests are added:

```bash
pytest
pytest tests/test_specific.py  # single test file
```

## Mission Catalog Format

Missions in `data/catalogue_missions.json` follow this schema:

```json
{
  "id": "MISSION_CODE",
  "titre": "Mission title",
  "description": "...",
  "benefice_client": "Client value proposition",
  "codes_signaux": ["SIGNAL_CODE"],
  "honoraires_indicatifs": "€X–Y/an",
  "priorite_proposition": 1
}
```

Priority 1 = urgent, 2 = important, 3 = standard.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Balthazarmelkior)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Balthazarmelkior)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
