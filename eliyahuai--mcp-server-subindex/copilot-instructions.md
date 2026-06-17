## mcp-server-subindex

> Generate, validate, and fact-check research tables with AI-sourced citations and per-cell confidence scores. Upload an Excel/CSV to verify every cell, build a new table from a natural language prompt, or fact-check claims in any document against authoritative sources. New accounts get $20 free at subindex.ai/account.


You are connected to the Subindex AI research and validation platform via MCP.

## When to activate

- User wants to **validate or enrich** an existing Excel, CSV, or research table
- User wants to **build a new research table** from a natural language description
- User wants to **fact-check claims, citations, or statistics** in a document or report
- User asks about accuracy, sources, or confidence scores for structured data
- User wants to re-run or update a previous validation job

## What you can do

### 1. Validate an existing table (upload_file → start_table_validation → wait_for_job → approve_validation → get_results)
Upload an Excel or CSV file and Subindex will verify every cell against authoritative sources, returning per-cell confidence scores (High / Medium / Low), corrections, and citations. Results arrive as a markdown table (`results.markdown_table`) — read this first. Full per-cell reasoning is in `results.metadata.rows[].cells[col].comment`.

### 2. Generate a new research table (start_table_maker → wait_for_conversation → wait_for_job → approve_validation → get_results)
Describe the table you want in natural language (entities, columns, row count). Subindex builds it row by row with AI-sourced, citation-backed values. Preview is free; full validation is charged per cell.

### 3. Fact-check a document (start_reference_check → wait_for_job → approve_validation → get_results)
Submit text or a document. Subindex extracts factual claims and verifies each one, returning SUPPORTED / PARTIAL / UNSUPPORTED / UNVERIFIABLE ratings with citations. Pass `auto_approve=True` to run straight through.

## Key rules

- **job_id = session_id** — these are the same string throughout the entire workflow. Use whichever parameter name the tool requires.
- **The preview is always auto-queued** after `start_table_validation` or `start_table_maker` — call `wait_for_job(job_id=session_id)` directly.
- **Preview is free; full validation is charged.** Always review `preview_table` and `cost_estimate` before calling `approve_validation`.
- **`results.markdown_table`** — the complete validated table is embedded directly in the `get_results` response. Read it before parsing any other field.
- After `approve_validation`, call `wait_for_job` with `timeout_seconds = max(900, estimated_validation_time_seconds * 2)`.
- If `wait_for_job` times out, call it again — the job is still running.

## Setup

Users need a Subindex API key: [subindex.ai/account](https://subindex.ai/account) — new accounts get $20 free, no credit card required.

---
> Source: [EliyahuAI/mcp-server-subindex](https://github.com/EliyahuAI/mcp-server-subindex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
