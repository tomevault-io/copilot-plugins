## aware-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AWARE-AI is a full-stack web application for AI-powered personal styling analysis. Users upload a photo and receive styling recommendations with an "animal personality type" (동물상), style score, and AI-generated styled images.

## Running the Application

```bash
# Start the backend (FastAPI on http://127.0.0.1:8000)
python3 test.py

# Frontend is served as static files at /frontend/
# Access via http://127.0.0.1:8000/frontend/index.html (or use any HTTP server)
```

### Environment Setup

Requires a `.env` or `AI.env` file with:
- `GEMINI_API_KEY` — Google Gemini API key (required)
- `OPENAI_API_KEY` — OpenAI API key (required)
- `GEMINI_IMAGE_MODEL` — defaults to `gemini-2.5-flash-image`
- `OPENAI_TEXT_MODEL` — defaults to `gpt-5-mini`
- `DEBUG` — `1` enables debug logging (default)

No test suite, linter, or build system is configured.

## Architecture

**Backend** (`test.py`): Single-file FastAPI application with two main endpoints:
- `POST /api/analyze` — Takes a photo + metadata (name, gender, age), sends to OpenAI for analysis, returns structured JSON (`AnalyzeResult`) with style score, animal vibe, styling tips, and 3 style options.
- `POST /api/apply` — Takes a photo + edit prompt, sends to Google Gemini image model, returns a streamed styled image.

**Frontend** (`frontend/`): Vanilla JS single-page app with three screens:
1. **Upload** — Photo upload with name/gender/age inputs
2. **Result** — Displays analysis (style score, animal vibe, tips, style options)
3. **Styled** — Shows AI-generated styled images with customization

State is managed via a global `state` object in `app.js`. The frontend talks to `http://127.0.0.1:8000` (hardcoded as `API_BASE`).

## Key Implementation Details

- AI model outputs are sanitized in `_sanitize_analyze_dict()` to handle common model errors: normalizes `style_score` from 0-100 to 0-10, matches vibe text to allowed animal types, fills fallback style options if fewer than 3 are returned.
- Allowed animal vibes (동물상): 강아지상, 고양이상, 토끼상, 여우상, 사슴상, 곰상, 공룡상.
- The `/api/analyze` endpoint retries with a larger `max_output_tokens` if the first OpenAI call returns incomplete.
- The `/api/apply` endpoint wraps user prompts with identity-preservation instructions to prevent Gemini from changing the person's face.
- All UI text and AI prompts are in Korean.

---
> Source: [optimist-AWARE/AWARE-AI](https://github.com/optimist-AWARE/AWARE-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
