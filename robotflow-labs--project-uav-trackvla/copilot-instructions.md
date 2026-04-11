## project-uav-trackvla

> **UAV-Track VLA: Embodied Aerial Tracking**

# UAV-TRACKVLA

## Paper
**UAV-Track VLA: Embodied Aerial Tracking**
arXiv: https://arxiv.org/abs/2604.02241

## Module Identity
- Codename: UAV-TRACKVLA
- Domain: UAV
- Part of ANIMA Intelligence Compiler Suite

## Structure
```
project_uav_trackvla/
├── pyproject.toml
├── configs/
├── src/anima_uav_trackvla/
├── tests/
├── scripts/
├── papers/          # Paper PDF
├── AGENTS.md        # This file
├── NEXT_STEPS.md
├── ASSETS.md
└── PRD.md
```

## Commands
```bash
uv sync
uv run pytest
uv run ruff check src/ tests/
uv run ruff format src/ tests/
```

## Conventions
- Package manager: uv (never pip)
- Build backend: hatchling
- Python: >=3.10
- Config: TOML + Pydantic BaseSettings
- Lint: ruff
- Git commit prefix: [UAV-TRACKVLA]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RobotFlow-Labs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RobotFlow-Labs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
