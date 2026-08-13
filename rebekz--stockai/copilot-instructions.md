## stockai

> This project uses `uv` for dependency management and running the CLI.

# Claude Code Instructions

## Running the CLI

This project uses `uv` for dependency management and running the CLI.

### Python Path

Use the Miniconda Python installation:
```
/Users/fitrakacamarga/miniconda3/bin/python
```

### Command Format

**Preferred (with uv):**
```bash
uv run stockai <command>
```

**Alternative (if uv not available):**
```bash
/Users/fitrakacamarga/miniconda3/bin/python -m stockai.cli.main <command>
```

### Available Commands

- `uv run stockai evening` - Run the evening briefing
- `uv run stockai morning` - Run the morning briefing
- `uv run stockai autopilot` - Run autopilot mode
- `uv run stockai backtest` - Run backtesting
- `uv run stockai monitor` - Monitor portfolio

### Important

- Always use the Miniconda Python path: `/Users/fitrakacamarga/miniconda3/bin/python`
- Do NOT use system `python` or `python3` directly
- Prefer `uv run stockai` when available for proper dependency resolution

---
> Source: [rebekz/stockai](https://github.com/rebekz/stockai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
