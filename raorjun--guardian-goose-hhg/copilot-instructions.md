## guardian-goose-hhg

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Guardian Goose** is a jargon-free Solana wallet analyzer with voice output. The core loop is: user pastes a Solana signature → auto-detection triggers → Helius + Gemini analyze it → ElevenLabs reads the result aloud. Total round-trip must be **under 3 seconds**.

## Repo Structure

```
guardian-goose-HHG/
├── frontend/   # Astro 5.0 + Tailwind v4
├── backend/    # FastAPI + Python 3.12
└── docs/       # System specs, persona guidelines, API schemas
```

## Agent Boundaries

- **@Frontend-Honker** — lives in `/frontend` only. Never touches backend files.
- **@Solana-Scout** — lives in `/backend` only. Never touches frontend files.

## Frontend (`/frontend`)

**Stack:** Astro 5.0, Tailwind v4, ElevenLabs voice synthesis.

**Key rules:**
- Use **Astro Actions** for all backend communication — no manual `fetch` boilerplate.
- UI style is "Bubble UI": rounded, playful, oversized tap targets.
- Auto-detect Solana transaction signatures on paste — no submit button needed.
- Replace all crypto jargon: `slippage` → "Price Change", `gas` → "Network Toll", `RPC` → "Network Connection", etc.

**Commands (once scaffolded):**
```bash
cd frontend
pnpm install
pnpm run dev      # dev server
pnpm run build    # production build
pnpm run preview  # preview production build
```

## Backend (`/backend`)

**Stack:** FastAPI, Python 3.12, Pydantic v2, Helius API, Google Gemini API.

**Key rules:**
- All Helius and Solana data structures must have **Pydantic models** — no raw dicts passed around.
- Helius handles on-chain data fetching; Gemini handles natural-language explanation generation.
- ElevenLabs TTS is triggered from the frontend after receiving Gemini's text response.

**Commands (once scaffolded):**
```bash
cd backend
python -m venv .venv && source .venv/Scripts/activate  # Windows
pip install -r requirements.txt
uvicorn main:app --reload   # dev server
pytest                      # run all tests
pytest tests/test_foo.py    # run single test file
```

## Core Design Principles

1. **Invisible UX** — interactions are triggered automatically, not by button clicks.
2. **Jargon-free** — every user-facing string must use plain English equivalents.
3. **Accessibility first** — built for users who rely on voice output (ElevenLabs).
4. **3-second SLA** — paste-to-voice round-trip must complete in under 3 seconds end-to-end.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/raorjun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/raorjun)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
