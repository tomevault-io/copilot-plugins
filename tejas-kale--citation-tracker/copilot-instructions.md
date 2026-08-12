## citation-tracker

> `citation-tracker` is a Python-based CLI tool designed to track citations of academic papers and analyze how citing works engage with the tracked research using Large Language Models (LLMs).

# Citation Tracker - Project Context

`citation-tracker` is a Python-based CLI tool designed to track citations of academic papers and analyze how citing works engage with the tracked research using Large Language Models (LLMs).

## Project Overview

*   **Purpose**: Automates the pipeline of discovering new citations, fetching PDFs, extracting text, and generating LLM-powered analyses of the relationship between citing and cited papers.
*   **Main Technologies**:
    *   **Python 3.12+**: Core language.
    *   **Click**: CLI framework.
    *   **SQLite**: Local data persistence (using WAL mode for concurrency and lightweight migrations).
    *   **LLM Backends**: OpenRouter (OpenAI-compatible).
    *   **PDF Extraction**: `pymupdf4llm` for converting PDFs to Markdown-rich text.
    *   **Metadata Extraction**: Fallback LLM-powered extraction for papers not yet indexed in academic databases.
    *   **API Sources**: Semantic Scholar, OpenAlex, and **NASA ADS** (for astronomy papers).
    *   **Reporting**: Generates Markdown and HTML reports (with interactive vertical Mermaid.js influence graphs) saved locally.
    *   **PWA**: A lightweight, client-side PWA with progress indicators for quick paper lookups.
    *   **Package Management**: `uv` is the preferred tool for installation and development.

## Architecture

*   **`src/citation_tracker/cli.py`**: Entry point for all commands. Uses `ThreadPoolExecutor` for high-throughput processing.
*   **`src/citation_tracker/db.py`**: Manages the SQLite schema and migrations. Supports short UUIDs, DOIs, SSIDs, OpenAlex IDs, and ADS Bibcodes.
*   **`src/citation_tracker/sources/`**: Contains API clients for Semantic Scholar, OpenAlex, and NASA ADS. Includes a `deduplicator.py` for merging results across multiple sources.
*   **`src/citation_tracker/fetcher.py`**: Handles PDF downloads with fallbacks for ADS Link Gateway, Unpaywall, Crossref, and arXiv.
*   **`src/citation_tracker/analyser.py`**: Orchestrates LLM interactions. Implements a **map-reduce** strategy for long papers and specialized metadata extraction for new works.
*   **`src/citation_tracker/report.py`**: Assembly of analysis reports, featuring **categorized vertical Mermaid.js trees** to visualize citation relationships (EXTENDS, CHALLENGES, etc.).
*   **`src/pwa/`**: A simplified, standalone PWA for mobile/web interaction.

## Building and Running

### Installation
```bash
# Install with uv (recommended)
uv tool install .

# Development mode
uv tool install --editable .
```

### Core Commands
*   **Add a paper**: `citation-tracker add <URL|DOI|SSID>` (Resolves metadata via content-peeking if needed)
*   **List papers**: `citation-tracker list` (Displays numeric IDs, status, and Source URL/DOI)
*   **Process citations**: `citation-tracker run [--id <ID|DOI>] [--workers 8]` (High-throughput pipeline)
*   **Show status**: `citation-tracker status` (Summary of database and pending items)
*   **Manage papers**: `pause`, `resume`, or `remove` papers by ID or DOI.
*   **List citations**: `citation-tracker citations --id <ID>` (Shows citing papers and status)
*   **Show report**: `citation-tracker show --id <ID>` (Displays the analysis report for a paper)
*   **Ingest manual PDF**: `citation-tracker ingest <path.pdf> --id <ID>` (Links a PDF to a tracked paper)

## Development Conventions

*   **Configuration**: Managed via `config.yaml` and `.env`. Keys include `OPENROUTER_API_KEY` and `ADS_DEV_KEY`.
*   **Concurrency**: High-throughput steps (fetch, parse, analyse) use thread pools. SQLite uses WAL mode to handle concurrent writes.
*   **Resolution Strategy**: Robust identification using DOIs, arXiv IDs, and content-based peeking (PDF downloading + LLM extraction) for ambiguous URLs.
*   **Analysis Strategy**: Large papers are split into chunks, summarised (map), and then synthesised (reduce) before final engagement analysis.
*   **Error Handling**: Uses `tenacity` for retrying API calls. Sources are robust to rate-limiting (429), falling back to alternative providers or LLM extraction.
*   **Data Storage**: By default, all data (DB, PDFs, and HTML reports) is stored in `~/.citation-tracker/`.

## Key Files
*   `pyproject.toml`: Dependency definitions and script entry points.
*   `config.yaml`: Main application configuration (model choice, API settings).
*   `.env.example`: Template for required environment variables.
*   `src/citation_tracker/db.py`: The "source of truth" for data structures and migrations.

---
> Source: [tejas-kale/citation-tracker](https://github.com/tejas-kale/citation-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
