## llm-council-ranking-members

> > **IMPORTANT**: This project is based on [Andrej Karpathy's llm-council](https://github.com/karpathy/llm-council). The original repository does not yet have a LICENSE file. A license has been requested in [issue #58](https://github.com/karpathy/llm-council/issues/58). Once a license is added to the original repo, it will be added here as well.

# CLAUDE.md - Technical Notes for LLM Council Ranking Members

> **IMPORTANT**: This project is based on [Andrej Karpathy's llm-council](https://github.com/karpathy/llm-council). The original repository does not yet have a LICENSE file. A license has been requested in [issue #58](https://github.com/karpathy/llm-council/issues/58). Once a license is added to the original repo, it will be added here as well.

## Quick Start

**Run Backend:** `python -m backend.main` (port 8001)
**Run Frontend:** `cd frontend && npm run dev` (port 5173)
**Environment:** Requires `OPENROUTER_API_KEY` in `.env`

## What This Is

LLM Council Ranking Members is a 3-stage deliberation system where multiple LLMs collaboratively answer questions through anonymized peer review:

1. **Stage 1**: Multiple LLMs answer the same question in parallel
2. **Stage 2**: Each LLM evaluates and ranks all responses (anonymized as "Response A, B, C...")
3. **Stage 3**: A chair LLM synthesizes the final answer using all responses and rankings

**Key Innovations**:

- **Anonymization**: Stage 2 responses are anonymized to prevent model bias - models can't favor their own or specific other models' responses
- **Human Participation**: Users can join the council by adding their own responses (Stage 1) and rankings (Stage 2) - evaluated blindly alongside AI responses
- **Polished UX/UI**: Clean, modern interface with light/dark themes, interactive visualizations (similarity heatmaps, chord diagrams, Plackett-Luce rankings), and real-time response streaming

## Core Principles

- **Always use free models** for testing (never paid models)
- **Run backend as module**: `python -m backend.main` from project root (not from backend dir)
- **Use relative imports** in backend files (e.g., `from .config import ...`)
- **Graceful degradation**: System continues with successful responses if some models fail
- **Transparency**: All raw model outputs visible in UI tabs for inspection

---
> Source: [gmgeorg/llm-council-ranking-members](https://github.com/gmgeorg/llm-council-ranking-members) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
