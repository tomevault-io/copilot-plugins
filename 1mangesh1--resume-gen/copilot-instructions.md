## resume-gen

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**RESUME.GEN** is an AI-powered resume tailoring system built as a collection of n8n workflows orchestrated via webhooks. It accepts a user's resume (PDF/TXT) and a job description (URL or text), then outputs an ATS-optimized PDF resume compiled from a LaTeX template.

This is **not** a traditional Node.js/TypeScript codebase — there are no `package.json`, test runners, or build tools. All logic lives in n8n workflow JSON files and a vanilla HTML frontend.

## Development Commands

All commands are via `make`:

```bash
make setup       # Copy .env.example → .env (first time only)
make build       # Build the local LaTeX service Docker image
make up          # Build (if needed) and start all services
make down        # Stop all services
make restart     # Restart all services
make import      # Import all 6 workflows into the running n8n instance
make logs        # Tail n8n container logs
make logs-latex  # Tail LaTeX service logs
make status      # Show docker compose ps output
make open        # Open n8n UI (port 5678) and frontend (port 3000) in browser
make clean       # Stop and remove volumes (destructive)
```

**First-time setup:**
1. `make setup` → edit `.env` with `GEMINI_API_KEY`
2. `make up`
3. `make import`
4. Open n8n UI → toggle all 6 workflows to **Active**

## Environment Variables

Required in `.env` (see `.env.example`):
- `GEMINI_API_KEY` — from aistudio.google.com/app/apikey

Optional:
- `CF_API_TOKEN` + `CF_ACCOUNT_ID` — Cloudflare Browser Rendering (primary JD scraper; falls back to Jina AI if absent)
- `N8N_HOST`, `N8N_PORT`, `N8N_PROTOCOL`, `WEBHOOK_URL`, `TIMEZONE`

## Architecture

### Services (Docker)

| Service | Port | Purpose |
|---|---|---|
| `n8n` | 5678 | Workflow orchestration |
| `latex` | 5000 (internal) | Local PDF compilation (pdflatex) |
| `frontend` | 3000 | Static nginx frontend |

### 6-Workflow Pipeline

JD Scraper and Resume Parser run **in parallel** after input preparation:

```
POST /webhook/resume-gen  (main-orchestrator)
  │
  ├── POST /webhook/jd-scraper     ← (parallel) Cloudflare → Jina AI fallback
  └── POST /webhook/resume-parser  ← (parallel) Gemini extracts structured JSON
  │         [Merge JD + Parser]
  │
  └── POST /webhook/ai-tailoring-engine  ← Gemini re-ranks, rewrites bullets
        │
        └── POST /webhook/latex-generator   ← Builds .tex, compiles via local latex service
              │
              └── POST /webhook/output-handler  ← Formats final JSON response
```

### Workflow Responsibilities

| Workflow | Webhook | Key Service |
|---|---|---|
| `main-orchestrator.json` | `/webhook/resume-gen` | — |
| `resume-parser.json` | `/webhook/resume-parser` | Gemini 2.5 Flash |
| `jd-scraper.json` | `/webhook/jd-scraper` | Cloudflare Browser Rendering, Jina AI |
| `ai-tailoring-engine.json` | `/webhook/ai-tailoring-engine` | Gemini 2.5 Flash |
| `latex-generator.json` | `/webhook/latex-generator` | `http://latex:5000/compile` (local) |
| `output-handler.json` | `/webhook/output-handler` | — |

### Local LaTeX Service

`latex-service/` contains a minimal Docker image (Debian + TeX Live + Python).

- `Dockerfile` — installs `texlive-latex-base/recommended/extra` + `texlive-fonts-recommended`
- `server.py` — Python HTTP server: `POST /compile { tex_source }` → `{ pdf_base64, error }`

The n8n container waits for this service to be healthy before starting (`depends_on: latex: condition: service_healthy`).

### Resume JSON Schema (passes between workflows)

```json
{
  "contact": { "name", "email", "phone", "linkedin", "github" },
  "summary": "",
  "skills": [{ "category", "items": [] }],
  "experience": [{ "company", "title", "dates", "bullets": [] }],
  "projects": [{ "name", "tech": [], "bullets": [] }],
  "education": [{ "institution", "degree", "dates", "gpa" }]
}
```

### Response Schema

```json
{
  "pdf_base64": "...", "tex_source": "...", "ats_score": 0-100,
  "matched_keywords": [], "compile_error": null,
  "generated_at": "ISO8601", "status": "success|partial"
}
```

## Testing Workflows

```bash
# Full pipeline
curl -X POST http://localhost:5678/webhook/resume-gen \
  -F "resume=@/path/to/resume.pdf" \
  -F "jd_url=https://jobs.example.com/position"

# JD Scraper only
curl -X POST http://localhost:5678/webhook/jd-scraper \
  -H "Content-Type: application/json" \
  -d '{"url": "https://jobs.example.com/position"}'

# LaTeX service directly
curl -X POST http://localhost:5000/compile \
  -H "Content-Type: application/json" \
  -d '{"tex_source": "\\documentclass{article}\\begin{document}Hello\\end{document}"}' | jq .
```

## Workflow Import Script

`scripts/import-workflows.sh` patches workflow JSONs with sequential IDs (1–6), copies them into the n8n container, imports via `n8n import:workflow --separate`, and restarts n8n. Re-run with `make import` after modifying any workflow JSON.

## LaTeX Template

`templates/jake-resume.tex` is the Jake's Resume reference template. The `latex-generator` workflow dynamically generates `.tex` from the AI-tailored resume JSON and compiles it via the local `latex` service. On compile failure, raw `.tex` is returned so the user can fix it in Overleaf.

## Key Design Decisions

- **Local LaTeX compilation** — `latex-service/` eliminates the `latex.ytotech.com` dependency; all PDF generation is self-hosted
- **Parallel JD + resume parsing** — main-orchestrator fans out to both sub-workflows simultaneously via n8n Merge node, then combines results
- **Modular sub-workflows** — each workflow is independently testable via its own webhook
- **Jina AI as scraper fallback** — free, no auth required at `r.jina.ai/<url>`
- **Gemini strict prompting** — anti-hallucination constraints, temperature 0.1–0.2, JSON mode
- **n8n 2.x compatibility** — workflows must be published (not just imported) to activate webhooks; the import script handles this
- **CF keys optional** — if `CF_API_TOKEN`/`CF_ACCOUNT_ID` are blank, the JD scraper skips Cloudflare and goes straight to Jina AI

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/1Mangesh1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
