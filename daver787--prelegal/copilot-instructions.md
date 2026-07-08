## prelegal

> This is a SaaS product to allow users to draft legal agreements based on templates in the templates directory.

# Prelegal Project

## Overview

This is a SaaS product to allow users to draft legal agreements based on templates in the templates directory.
The user can carry out AI chat in order to establish what document they want and how to fill in the fields.
The available documents are covered in the catalog.json file in the project root, included here:

@catalog.json

PL-7 is the latest completed feature: real authentication (sign up / sign in against the SQLite users table), document persistence (drafts saved to the database and restored on reload), UI polish, and a legal disclaimer. The app is now feature-complete.

## Development process

When instructed to build a feature:
1. Use your Atlassian tools to read the feature instructions from Jira
2. Develop the feature - do not skip any step from the feature-dev 7 step process
3. Thoroughly test the feature with unit tests and integration tests and fix any issues
4. Submit a PR using your github tools

## AI design

When writing code to make calls to LLMs, use your Cerebras skill to use LiteLLM via OpenRouter to the `openrouter/openai/gpt-oss-120b` model with Cerebras as the inference provider. You should use Structured Outputs so that you can interpret the results and populate fields in the legal document.

There is an OPENROUTER_API_KEY in the .env file in the project root.

## Technical design

The entire project should be packaged into a Docker container.  
The backend should be in backend/ and be a uv project, using FastAPI.  
The frontend should be in frontend/  
The database should use SQLLite and be created from scratch each time the Docker container is brought up, allowing for a users table with sign up and sign in.  
The frontend is statically built (`output: 'export'` in `next.config.ts`) and served by FastAPI via `StaticFiles`. All API routes must be registered on `app` **before** the `app.mount("/", ...)` call in `backend/main.py`.  
There should be scripts in scripts/ for:  
```bash
# Mac
scripts/start-mac.sh    # Start
scripts/stop-mac.sh     # Stop

# Linux
scripts/start-linux.sh
scripts/stop-linux.sh

# Windows
scripts/start-windows.ps1
scripts/stop-windows.ps1
```
Backend available at http://localhost:8000

## Color Scheme
- Accent Yellow: `#ecad0a`
- Blue Primary: `#209dd7`
- Purple Secondary: `#753991` (submit buttons)
- Dark Navy: `#032147` (headings)
- Gray Text: `#888888`

Brand colors are defined in `frontend/tailwind.config.ts` as `brand.yellow`, `brand.blue`, `brand.purple`, `brand.navy`, `brand.gray`. Use Tailwind classes (`bg-brand-purple`, `text-brand-navy`, etc.) rather than inline styles.

## Implementation status

### Done (all features complete)
- **PL-3**: Mutual NDA creator — form + live preview + PDF export (client-side, Next.js)
- **PL-4**: V1 foundation — Docker container, FastAPI backend (`backend/`), SQLite DB (`data/prelegal.db`, fresh each run), static frontend served by FastAPI, fake login page (any credentials accepted, `localStorage` session flag), start/stop scripts for Mac/Linux/Windows
- **PL-5**: AI chat for Mutual NDA — freeform chat replaces the form panel; `POST /api/chat` calls LiteLLM/Cerebras with structured output to extract NDA fields; live preview updates in real-time as fields are populated
- **PL-6**: Expand to all supported document types — AI auto-detects document type; union `FieldUpdates` schema covers all 8 primary document types; `DocumentPreview` dispatches to per-document preview components; `previewHelpers.tsx` shares common UI; unsupported addenda handled gracefully
- **PL-7**: Real auth (sign up / sign in against `users` table with hashed passwords), document persistence (drafts saved to DB and restored on reload), UI polish, legal disclaimer

---
> Source: [daver787/prelegal](https://github.com/daver787/prelegal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
