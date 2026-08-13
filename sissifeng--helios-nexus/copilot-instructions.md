## helios-nexus

> Nexus: An intelligent optimization platform that automatically selects, adapts, and explains optimization strategies for scientific experiments.

# CLAUDE.md — Nexus Development Guide

## Project Overview
Nexus: An intelligent optimization platform that automatically selects, adapts, and explains optimization strategies for scientific experiments.

## Quick Commands
```bash
# Backend
pip install -e ".[dev]"
nexus server start                        # API at :8000

# Frontend
cd optimization_copilot/web && npm install && npm run dev   # UI at :5173

# Tests
pytest                                    # All 6,300+ tests
pytest tests/test_api.py -v              # Specific file
ruff check optimization_copilot/         # Lint
```

## Architecture
- `optimization_copilot/core/` — CampaignSnapshot, Observation, ParameterSpec
- `optimization_copilot/diagnostics/` — 14-signal diagnostic engine
- `optimization_copilot/backends/` — 10+ optimization algorithm backends
- `optimization_copilot/meta_controller/` — Phase orchestration and strategy switching
- `optimization_copilot/api/` — FastAPI routes (campaigns, chat, analysis, loop)
- `optimization_copilot/web/` — React 18 + TypeScript frontend
- `tests/` — 155 test files

## Tech Stack
- Python 3.10+, FastAPI, Pydantic v2, Click
- React 18, TypeScript strict, Vite 5
- Claude API (optional, for AI chat features)
- Zero external ML dependencies

## Development Rules
- All code must be deterministic and reproducible (given same seed)
- Every optimization decision must produce a full audit trail
- Use type hints throughout Python code
- TypeScript strict mode for all frontend code
- Keep implementations minimal — YAGNI principle
- Each module should be independently testable

## Model
- Default: claude-opus-4-6
- API key via environment variable `MODEL_API_KEY` (fallback: `ANTHROPIC_API_KEY`)

---
> Source: [SissiFeng/HELIOS_Nexus](https://github.com/SissiFeng/HELIOS_Nexus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
