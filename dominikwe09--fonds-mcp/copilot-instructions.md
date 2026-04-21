## fonds-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

**fonds-mcp** is a German investment fund recommendation system implemented as an MCP (Model Context Protocol) server. It helps find matching investment funds (Union Investment / DZ Bank) based on customer risk profiles using semantic search.

## Commands

All commands use the project's virtual environment at `/opt/fonds-mcp/venv/`.

**Run the MCP server:**
```bash
python3 /opt/fonds-mcp/mcp_server.py
```
Starts FastMCP on `http://127.0.0.1:8000/sse` (SSE transport).

**Run the data indexer (re-index funds):**
```bash
python3 /opt/fonds-mcp/scraper.py
```
Fetches fund PDFs from Union Investment, extracts text, generates embeddings, and stores in ChromaDB. Logs to `/var/log/fonds-update.log` and stdout.

There are no test or lint commands defined for this project.

## Architecture

Two loosely coupled components share a ChromaDB vector database at `/opt/fonds-mcp/db/`.

### `scraper.py` — ETL Pipeline

1. Fetches the full fund list (~242 funds) from Union Investment's internal JSON API (`FUNDDATA_API`)
2. Downloads product information sheet PDFs in parallel (max 5 concurrent via asyncio semaphore)
3. Validates PDFs contain actual product info (must contain keywords: `Anlagestrategie`, `Produktinformation`, `Anlageziel`, `Fondsinformation`) — rejects KID documents
4. Extracts text via PyMuPDF, splits into 600-word chunks with 30-word overlap
5. Generates multilingual sentence embeddings (`paraphrase-multilingual-MiniLM-L12-v2`)
6. Stores embeddings + metadata (name, isin, risikoklasse, url, MD5 hash) in ChromaDB collection `"fonds"`
7. Uses hash-based change detection — skips unchanged PDFs

### `mcp_server.py` — MCP Server

Loads the embeddings model and ChromaDB once at startup. Exposes two MCP tools:

- **`fonds_suchen`**: Takes a free-text customer profile description + optional `max_risikoklasse` (1–7) and `anzahl` (1–10). Encodes the query, performs vector similarity search with optional metadata filter on `risikoklasse`, and returns ranked fund names, risk class, and PDF URL.
- **`fonds_liste`**: Lists all available funds, optionally filtered by risk class. Deduplicates by fund name and returns alphabetically sorted results.

### Key Constants

| Constant | Value |
|----------|-------|
| `DB_PATH` | `/opt/fonds-mcp/db` |
| `MODEL_NAME` | `paraphrase-multilingual-MiniLM-L12-v2` |
| Risk classes | 1 (conservative) → 7 (aggressive) |
| Chunk size | 600 words, 30-word overlap |
| Max parallel downloads | 5 |
| Max retries per operation | 3 |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dominikwe09) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
