## pdffigures-mcp-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture

This is a FastAPI service that wraps **PDFFigures 2.0**, a Scala/JVM tool from Allen AI, to extract figures, tables, and captions from scholarly PDFs. It also exposes a **FastMCP** tool so AI agents can invoke extraction programmatically.

```
Client (HTTP/MCP) → FastAPI (app.py) → service.py (subprocess) → pdffigures2 JAR (Scala/JVM)
                                                      ↓
                                              Rendered PNGs + JSON metadata
```

- **`app/app.py`** — FastAPI server + FastMCP tool. Three entry points: REST API (`/api/extract`), MCP (`/mcp`), and static file serving (`/resources`). The core extraction logic (`extract_pdf_logic`) is shared between the REST and MCP paths. The MCP tool is `extract_figures_from_pdf(pdf_url)`.
- **`app/service.py`** — Runs `pdffigures2.jar` via `subprocess.run` with a 180s timeout. Reads the output JSON metadata file after the JAR completes. This runs synchronously, so callers must use `run_in_threadpool`.
- **`app/utils.py`** — File I/O helpers (`save_uploaded_file`, `read_output_file`).
- **`figure_extractor.py`** — Standalone CLI client that POSTs a local PDF to the API and downloads the rendered figures. Run with `python figure_extractor.py <path_to_pdf>`.
- **`skills/pdffigures2/`** — Agent Skill providing a non-MCP alternative. The CLI script (`scripts/pdffigures2`) is an API client that POSTs a local PDF to the extraction server and downloads rendered figures. Reads server URL from `.env` (copy `.env.example` to `.env`).

## Commands

```sh
# Build and run with Docker
docker build -t pdf-extraction .
docker run -p 5001:5001 pdf-extraction

# Run locally (requires pdffigures2 JAR at the expected path)
python -m uvicorn app.app:app --host 0.0.0.0 --port 5001

# Test extraction via CLI
python figure_extractor.py <path-to-pdf>

# API docs
open http://localhost:5001/docs
```

## Environment Variables

| Variable | Default | Purpose |
|---|---|---|
| `JAVA_OPTS` | `-XX:MaxRAMPercentage=75.0` | JVM flags for pdffigures2 |
| `PDFFIGURES_JAR_PATH` | `/pdffigures2/pdffigures2.jar` | Path to the assembly JAR |
| `PDFFIGURES_WORK_DIR` | `/pdffigures2` | Working directory for the subprocess |
| `OUTPUT_DIR` | `/app/outputs` | Where rendered PNGs + JSON are written |
| `RESOURCE_BASE_URL` | `http://localhost:5001` | Prepended to relative `renderURL` paths in responses |
| `LOG_LEVEL` | `INFO` | Python logging level |
| `PARALLEL_DOWNLOAD_CHUNKS` | `4` | Number of parallel byte-range download chunks |

## Docker Build

Multi-stage build (`Dockerfile`):
1. **Stage 1 (builder):** Clones `allenai/pdffigures2` from GitHub and runs `sbt assembly` to produce the fat JAR.
2. **Stage 2 (runtime):** Python 3.11 slim image with JRE headless + Tesseract OCR. Copies the JAR from the builder stage and installs Python deps from `requirements.txt`. Runs the uvicorn server on port 5001.

## Key Constraints

- The pdffigures2 JAR requires Java (JRE headless is sufficient). The subprocess call in `service.py` is synchronous and CPU-bound — always wrap it in `run_in_threadpool` to avoid blocking the async event loop.
- The output JSON is written to `OUTPUT_DIR` with the filename pattern `{pdf_basename}.json`. The `-m` flag passed to the JAR expects the prefix to end with a path separator.
- PDF inputs are validated by checking for the `%PDF-` magic bytes header. Temporary files are cleaned up in a `finally` block after extraction.
- The MCP mount path is `/mcp`, which is separate from the REST API prefix `/api`.
- License: Apache 2.0.

---
> Source: [vlln/pdffigures-mcp-server](https://github.com/vlln/pdffigures-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
