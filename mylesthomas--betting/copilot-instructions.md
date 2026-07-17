## betting

> You are an AI agent working on a data-driven NBA/NFL betting analytics codebase.

# AGENTS.md

You are an AI agent working on a data-driven NBA/NFL betting analytics codebase.

This repository analyzes sports betting markets, finds value opportunities, and automates data pipelines. Your role is to understand the system, make changes, and validate your work.

## 🗺️ Navigation

**Start here:**
- **Architecture overview:** `docs/ARCHITECTURE.md`
- **Full documentation index:** `docs/index.md`

**Core knowledge bases:**
- **Betting domain knowledge:** `docs/domain/` - Props, odds, line movement, market mechanics
- **Design decisions:** `docs/design-docs/` - Why things are built this way
- **API references:** `docs/references/` - DraftKings, The Odds API, NBA API, AWS
- **Quality tracking:** `docs/QUALITY_SCORE.md` - Current health of each domain
- **Active work:** `docs/exec-plans/active/` - What's currently in progress

## 🎯 Core Principles

From `docs/design-docs/core-beliefs.md`:

1. **Fail fast** - Don't check for keys that should exist. Let it fail if data is malformed.
2. **No fake data** - Never create mock/test data without explicit permission.
3. **Readable paths** - Use config/root detection, not relative parent paths (`../../`).
4. **Explicit over implicit** - Only check existence for truly optional items.
5. **Agent legibility first** - Optimize for discoverability. If you can't find it in-repo, it doesn't exist.

See `.cursor/rules/cursor_rules.mdc` for detailed coding standards.

## 🏗️ System Architecture

### Key Domains

```
DATA INGESTION          STORAGE              ANALYSIS              EXECUTION
────────────────        ───────────          ────────────          ─────────
lambda/                 data/                analysis/             automation/
scripts/fetch_*.py      ├─ 01_input/         backtesting/          (bet placement)
                        ├─ 02_cache/         implementation/
                        ├─ 03_intermediate/
                        └─ 04_output/
                        
                        src/                 (utilities - cross-cutting)
                        ├─ config_loader.py
                        ├─ odds_utils.py
                        ├─ team_utils.py
                        └─ s3_utils.py
```

### Dependency Rules

**Enforced by tests in `tests/test_architecture.py`:**

1. Analysis CANNOT directly call external APIs → Use Ingestion layer
2. Lambda functions CANNOT import from `analysis/`
3. Utils (`src/`) are pure functions with no business logic
4. Data flows: **Ingestion → Storage → Analysis**

See `docs/design-docs/dependency-boundaries.md` for complete rules.

## 📋 Before Making Changes

**1. Understand the context:**
- What domain does this touch? (Ingestion, Analysis, Utils, etc.)
- Read relevant design docs in `docs/design-docs/`
- Check current quality: `docs/QUALITY_SCORE.md`

**2. Check for active work:**
- Review `docs/exec-plans/active/` to avoid conflicts
- Check `docs/exec-plans/tech-debt-tracker.md` for known issues

**3. Understand betting concepts:**
- If touching props/odds/lines → Read `docs/domain/betting-fundamentals.md`
- If analyzing line movement → Read `docs/domain/market-mechanics.md`
- If handling data quality → Read `docs/domain/data-quality-standards.md`
- **Modeling / odds math / ROI / data quirks** → `knowledge-base/wiki/` (see index)

**4. Know the edge cases:**
- Postponed games, injuries, line freezes → `docs/domain/edge-cases.md`
- Sport-specific patterns → `docs/domain/nba-vs-nfl.md`

## ✅ Validation & Testing

**Before committing:**

Run the pre-commit check:
```bash
python scripts/agent_precommit_check.py
```

This validates:
- ✅ All pytest tests pass
- ✅ Architectural boundaries enforced
- ✅ Data validation harnesses pass
- ✅ Golden principles compliance
- ✅ Docstrings present for new functions

**Available validation harnesses:**
- `scripts/validate_props_data.py` - Check props data quality
- `scripts/validate_line_movement.py` - Detect significant moves
- `scripts/validate_cache_consistency.py` - Ensure caches are current

See `docs/validation/` for validation rules.

## 🔧 Common Tasks

### Fetching Data
```python
# Use existing utilities in src/
from src.config_loader import load_config
from src.s3_utils import read_from_s3, write_to_s3

# Never hardcode API keys - use config
config = load_config()
api_key = config['apis']['draftkings']['key']
```

### Normalizing Names
```python
# Always use utilities for consistency
from src.team_utils import normalize_team_name
from src.player_name_utils import normalize_player_name

team = normalize_team_name("LA Lakers")  # → "Los Angeles Lakers"
```

### Working with Odds
```python
# Use odds_utils for conversions
from src.odds_utils import american_to_probability, implied_vig

prob = american_to_probability(-110)  # → 0.5238
```

## 📖 Getting Help

**Betting concepts unclear?**
→ `docs/domain/betting-fundamentals.md`
→ `knowledge-base/wiki/index.md` — odds math, ROI, edge, model evaluation, data quirks, NBA season structure

**API usage questions?**
→ `docs/references/` (DraftKings, The Odds API, NBA API)

**Querying or inspecting S3 data (e.g. schema, sample rows)?**
→ `docs/references/duckdb-s3-queries.md` — how to run DuckDB with httpfs and credentials for `duckdb -c "..."` calls.

**Architecture questions?**
→ `docs/design-docs/` or `docs/ARCHITECTURE.md`

**Data quality issues?**
→ `docs/domain/data-quality-standards.md`

**Code style questions?**
→ `.cursor/rules/cursor_rules.mdc`

## 🚀 Agent Workflow

1. **Understand the task** - Read relevant docs
2. **Check dependencies** - Review architecture boundaries
3. **Make changes** - Follow golden principles
4. **Validate** - Run validation harnesses and tests
5. **Self-review** - Use `scripts/agent_self_review.py`
6. **Commit** - With descriptive message following `.cursor/rules/gitflow.mdc`
7. **Log work** - Update `docs/exec-plans/agent-work-log.md`

## 📝 Key Files

| File | Purpose |
|------|---------|
| `docs/ARCHITECTURE.md` | System map |
| `docs/references/duckdb-s3-queries.md` | How to run DuckDB queries against S3 (httpfs, region, credentials) |
| `docs/QUALITY_SCORE.md` | Current health metrics |
| `docs/design-docs/golden-principles.md` | Coding standards (enforced) |
| `docs/design-docs/dependency-boundaries.md` | Layer rules |
| `config/config.yaml` | Main configuration |
| `tests/test_architecture.py` | Boundary enforcement |

---

**Remember:** Repository knowledge is the only knowledge. If it's not documented here, it doesn't exist for you. When in doubt, read the docs, don't guess.

**Philosophy:** We optimize for agent legibility and fast iteration with strong guardrails. Corrections are cheap, waiting is expensive.

---
> Source: [MylesThomas/betting](https://github.com/MylesThomas/betting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
