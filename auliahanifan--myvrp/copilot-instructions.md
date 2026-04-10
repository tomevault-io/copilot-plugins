## myvrp

> 2. Treat UI and backend logic as one vertical slice

# Segarloka Auto-Route Service

# How to Use

1. always use `uv`

# Coding Rules:

1. Keep it simple
2. Treat UI and backend logic as one vertical slice
3. Do not change `app.py` without checking the matching application service, test, and output flow

## Architecture Rules

1. `app.py` is a Streamlit UI shell only
2. Business orchestration belongs in `src/application/`
3. Streamlit state and display helpers belong in `src/presentation/`
4. `app.py` must not import solver, parser, distance-calculation, output-generator, or map-visualizer modules directly
5. Any change to route generation flow must be reflected in:
   - UI entrypoint
   - application service contract or orchestration
   - relevant tests
6. Prefer vertical-slice changes over layer-only edits

### High-Level Purpose

This project is to automate the routing process for Segarloka's delivery service.
Currently, the routing process is done manually by the operations team.

### Matrix

Matrix done by OSRM.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/auliahanifan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/auliahanifan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
