## mic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI-powered event validation system for government educational/training event submissions (workshops, seminars, hackathons, etc.). Processes CSV/XLSX files containing event data (titles, PDFs, images, metadata) and validates them against a 100-point scoring system using Google Gemini AI with Groq Cloud as fallback. There is also an `event_validator_ollama/` variant that runs entirely locally using Ollama (not the primary codebase).

## Commands

```bash
# Setup
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt

# Run CLI (interactive mode)
python main.py

# Run CLI (non-interactive)
python main.py --non-interactive input.csv --output-csv output.csv

# Run FastAPI server (port 8000, auto-reload)
python run_api.py

# Install as package
pip install -e .
```

No formal test suite exists (test files are gitignored).

## Architecture

### Entry Points
- `main.py` → `event_validator/main.py` — CLI with interactive prompts or `--non-interactive` flag
- `run_api.py` → `event_validator/api/app.py` — FastAPI server with `/validate/batch` POST endpoint

### Core Pipeline (per submission)
```
CSV/XLSX input
  → column_mapper.py (normalize field names, resolve blob URLs)
  → downloader.py (fetch PDFs/images from Azure Blob Storage)
  → extractors/ (pdf_extractor.py, image_extractor.py)
  → Heuristic pre-scoring gate (quick rule-based checks, saves 30-50% API calls)
  → Validators (each returns List[ValidationResult]):
      1. theme_validator.py   — AI semantic alignment (10pts) + rule-based level/duration (11pts) + participants>15 (12pts)
      2. pdf_validator.py     — 5 checks in 1 unified API call (25pts total)
      3. image_validator.py   — 4 checks in 1 vision API call (20pts total)
      4. duplicate_validator.py — SHA256 + pHash, no AI (15pts)
  → Kill switch enforcement (event_driven types 1,2,4: if PDF fails, zero theme+image scores)
  → Score aggregation → Status (Accepted/Rejected/Reopen)
  → Enriched CSV output with score breakdown columns
```

### AI Client Chain
`gemini_client.py` (primary, uses `gemini-1.5-flash`) → `groq_client.py` (fallback, uses `llama-3.1-8b-instant` text / `llama-4-scout` vision). Both use response caching, rate limiting, and circuit breakers.

### Concurrency & Rate Limiting Stack
- `utils/concurrency.py` — Provider-level semaphores (`GEMINI_MAX_CONCURRENT=6`, `GROQ_MAX_CONCURRENT=1`)
- `utils/rate_limiter.py` — Token bucket algorithm per provider
- `utils/circuit_breaker.py` — Opens on sustained 429 errors, prevents cascading failures
- `orchestration/runner.py` — `ThreadPoolExecutor` with configurable `DEFAULT_MAX_WORKERS` (default 12)
- `api/app.py` — Additional request-level semaphore, auto-switches to sequential mode on rate limit detection

### Key Data Types (`types.py`)
- `EventSubmission` — Main container: row_data, pdf_data, images, validation_results, overall_score, status, kill_switch
- `ValidationResult` — criterion, passed (bool), points_awarded, message
- `ValidationConfig` — acceptance_threshold (default 60), duplicate_phash_threshold (default 5)

### Scoring Rules (`config/rules.py`)
- Theme: 40pts (33 effective — year alignment disabled)
- PDF: 25pts
- Image: 20pts (14 effective — geotag disabled)
- Similarity: 15pts
- Max effective: 87pts. Threshold: 60pts
- Binary scoring only (full points or zero per rule)

### Event-Driven Types
The `event_driven` field (1, 2, 3, 4) controls validation behavior:
- Types 1, 2, 4: Validate against system-generated canonical titles; kill switch applies
- Type 3: Validate user-provided titles against theme; no kill switch

### File Downloads
PDFs and images are downloaded from Azure Blob Storage via `utils/blob_path_resolver.py` (deterministic URL mapping by event_driven type) into `downloaded_files/`. Periodic cleanup thread runs in API mode.

### PDF Extraction
Multi-layer fallback: pdfplumber → PyPDF2 → EasyOCR (for scanned PDFs). OCR reader must be initialized in main thread on Linux to avoid `[Errno 5]` I/O errors.

## Environment Variables

| Variable | Purpose | Default |
|----------|---------|---------|
| `GEMINI_API_KEY` | Google Gemini API key (primary) | required |
| `GROQ_API_KEY` | Groq Cloud API key (fallback) | optional |
| `ACCEPTANCE_THRESHOLD` | Score threshold for acceptance | 60 |
| `PHASH_THRESHOLD` | Hamming distance for duplicate detection | 5 |
| `GEMINI_MAX_CONCURRENT` | Max concurrent Gemini API calls | 6 |
| `GROQ_MAX_CONCURRENT` | Max concurrent Groq API calls | 1 |
| `DEFAULT_MAX_WORKERS` | ThreadPoolExecutor workers | 12 |
| `GEMINI_RPM_LIMIT` | Gemini requests per minute limit | 148 |
| `USE_GPU` | EasyOCR GPU usage (set "0" to disable) | 1 |

## Important Patterns

- The `groq_api_key` field in `ValidationConfig` actually stores the Gemini API key due to backward compatibility naming — be careful when modifying config handling.
- `_original_row_data` is stored as a dynamic attribute on `EventSubmission` to preserve original CSV column names in output while using standardized names internally.
- Output CSVs use `utf-8-sig` encoding (BOM) for Excel compatibility.
- All score breakdown columns use camelCase (e.g., `themeAlignmentScore`, `pdfTitleScore`) and format as `"X.0/Max"`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hariV0078) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
