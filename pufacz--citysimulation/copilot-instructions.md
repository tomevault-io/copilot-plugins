## citysimulation

> This file documents project-specific guidance for Claude Code and defines available skills/tools.

# Claude Code Configuration

This file documents project-specific guidance for Claude Code and defines available skills/tools.

## Project Overview

**City Simulation** — A realistic 3D city simulation with LLM-powered autonomous agents.

- **Architecture**: Godot 4 (C# / GDScript) + Python sidecar (asyncio, websockets, Anthropic SDK)
- **Status**: M1–M5 complete (city scene, living world, visual polish, scripted agents, LLM brains)
- **Repository**: [pufacz/citysimulation](https://github.com/pufacz/citysimulation)
- **Tech Stack**: Godot 4.3 (Forward+), Python 3.11+, asyncio, websockets, Anthropic Claude

## Skills

### GitHub Sync Skill

**Purpose**: Sync local git repository to GitHub with customizable organization and repository name.

**Usage**:
```bash
python3 sync_repo_skill.py --org <organization> --repo <repo_name> [options]
```

**Parameters**:
- `--org, --organization`: GitHub organization or username (required)
- `--repo, --repository`: Repository name on GitHub (required)
- `--private`: Make repository private (default: public)
- `--force`: Force push to overwrite remote history
- `--branch, -b`: Branch to push (default: current branch)
- `--no-push`: Create repository without pushing

**Examples**:
```bash
# Sync to organization repo (public)
python3 sync_repo_skill.py --org pufacz --repo citysimulation

# Private repository
python3 sync_repo_skill.py --org myorg --repo myproject --private

# Force push a specific branch
python3 sync_repo_skill.py --org pufacz --repo test --force --branch main
```

**Location**: `./sync_repo_skill.py` (Python) or `./sync-to-github.sh` (Bash)
**Documentation**: See `SYNC_SKILL_README.md`

## Development Workflow

### M1–M5 Complete — What's Implemented

| Milestone | Status | Focus | Files |
|-----------|--------|-------|-------|
| **M1** | ✅ | City scene (buildings, roads, lighting, camera) | `godot/scripts/city_*.gd` |
| **M2** | ✅ | Living world (traffic, pedestrians, HUD) | `godot/scripts/vehicle.gd`, `pedestrian*.gd` |
| **M3** | ✅ | Visual polish (weather, LOD, props, follow camera) | `godot/scripts/weather.gd`, `free_fly_camera.gd` |
| **M4** | ✅ | Scripted agents + sidecar skeleton | `godot/scripts/agent*.gd`, `brain/citybrain/*.py` |
| **M5** | ✅ | LLM-powered agent decisions (Anthropic) | `brain/citybrain/llm.py`, `budget.py` |

### M6 — Next Phase

**Goal**: Long-term memory, agent conversations, player chat, multi-provider support.

**Key tasks**:
- Agent memory persistence (reflection, summarization)
- Inter-agent conversations (when agents meet)
- Player ↔ agent chat interface
- Multi-provider abstraction (Anthropic, OpenAI, etc.)
- Save/load world state

### Running the Project

**Godot Editor** (interactive):
```bash
godot --path godot -e
```

**Godot Headless** (for testing, automation):
```bash
godot --headless --path godot --quit-after 300
```

**Python Sidecar**:
```bash
cd brain

# With Anthropic API key (real LLM decisions)
uv run python -m citybrain

# Or, mock LLM mode (no API calls, canned decisions)
CITYBRAIN_MOCK_LLM=1 uv run python -m citybrain
```

**Full Stack** (Godot + sidecar in parallel):
```bash
# Terminal 1: Start sidecar
cd brain && uv run python -m citybrain

# Terminal 2: Run Godot
godot --path godot
```

### Configuration

**Agent LLM Profiles**: [`godot/agent_profiles.json`](godot/agent_profiles.json)
- Named presets: `default`, `premium`, `flagship`
- Per-agent assignments with overrides
- Resolution order: built-in defaults → `default` profile → assigned profile → agent-specific overrides → in-game UI edits

**API Keys**: [`brain/.env`](brain/.env) (git-ignored)
- `ANTHROPIC_API_KEY=sk-ant-...` for real LLM decisions
- `CITYBRAIN_MOCK_LLM=1` for offline development

**In-Game Settings**: Persisted to `user://agent_llm.json` (highest precedence)
- Select agent → Inspector → LLM brain section
- Edit model, enabled state, tokens/day

## Coding Conventions

### GDScript (Godot)

- **File naming**: `snake_case.gd` for scripts, `PascalCase` for class names
- **Indentation**: Tabs (Godot default)
- **Signals**: Use `decision_needed.emit(agent, trigger)` pattern for event-driven logic
- **Determinism**: World logic in `_process()` with `SimClock.sim_delta()`; never block on sidecar

### Python (Sidecar)

- **File naming**: `snake_case.py` modules
- **Classes**: `PascalCase` for class names
- **Type hints**: Use `from __future__ import annotations` + full type hints
- **Async**: All I/O uses `asyncio`; avoid `time.sleep()` except in tests
- **Pydantic**: Models for all protocol messages (type safety + validation)

## Testing

### Godot

```bash
# Headless test run (no rendering)
godot --headless --path godot --quit-after 300

# With movie-writer frames (visual validation)
godot --path godot --write-movie /tmp/frames.png --quit-after 60
```

### Python

```bash
cd brain
uv run pytest -v
uv run pytest tests/test_budget.py  # Budget metering tests
uv run pytest tests/test_llm.py      # LLM integration tests
uv run pytest tests/test_server.py   # Decision routing tests
```

### End-to-End

1. Start sidecar: `cd brain && CITYBRAIN_MOCK_LLM=1 uv run python -m citybrain`
2. Run Godot: `godot --headless --path godot --quit-after 600 ++ --speed=30`
3. Check logs:
   ```bash
   # Sidecar: agent configs received, decisions logged (llm / rules)
   grep "config agent" /tmp/citybrain.log
   grep "(llm)" /tmp/citybrain.log | wc -l
   ```

## Documentation

- [`app.md`](app.md) — Full specification (architecture, protocols, milestones)
- [`godot/README.md`](godot/README.md) — Godot project guide (controls, scripts, implementation notes)
- [`SYNC_SKILL_README.md`](SYNC_SKILL_README.md) — GitHub sync skill documentation

## Git & GitHub

**Repository**: https://github.com/pufacz/citysimulation

**Branch Strategy**:
- `main` — stable, all tests passing
- Feature branches: `feature/...` → PR → merge to `main`
- Use the sync skill to keep it in sync: `python3 sync_repo_skill.py --org pufacz --repo citysimulation`

**Committing**:
- Always include `Co-Authored-By: Claude <noreply@anthropic.com>` when working with Claude Code
- Descriptive messages referencing the ticket/milestone
- One logical change per commit

## Known Limitations & Future Work

1. **Agent Memory** (M6): Currently stateless; single-shot LLM calls. Next: add reflection/compaction.
2. **Navigation**: Sidewalk graphs only; no full navmesh. Could improve with Godot 4's NavigationServer3D.
3. **Multi-Provider**: Currently Anthropic-only. M6 will add abstraction for OpenAI, Google, etc.
4. **Persistence**: No save/load yet. M6 will add world snapshots and agent state export.
5. **Player Interaction**: HUD is read-only; no chat with agents yet. M6 will add full chat interface.

## Support & Troubleshooting

**Godot won't start**:
```bash
godot --path godot -e &  # Editor mode for error messages
```

**Sidecar won't connect**:
- Check `brain/.env` has `ANTHROPIC_API_KEY` or `CITYBRAIN_MOCK_LLM=1`
- Verify port 8765 is free: `lsof -i :8765`
- Restart sidecar: `pkill -f citybrain` then restart

**Agent decisions look wrong**:
- Disable LLM (use fallback): `CITYBRAIN_MOCK_LLM=1`
- Check agent config in `godot/agent_profiles.json`
- Review sidecar logs: `grep agent_N /tmp/citybrain.log`

**Tests failing**:
```bash
# Clear Python cache
find . -type d -name __pycache__ -exec rm -rf {} + 2>/dev/null

# Reinstall deps
cd brain && uv sync && uv run pytest
```

## Questions?

See [app.md](app.md) for architecture depth, [godot/README.md](godot/README.md) for Godot specifics, or each module's docstrings for code-level details.

---
> Source: [pufacz/citysimulation](https://github.com/pufacz/citysimulation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
