## agent-quality-ratchet

> This project uses a quality ratchet — code quality can only improve, never regress.

# CLAUDE.md

## Quality Ratchet

This project uses a quality ratchet — code quality can only improve, never regress.

### Mandatory Gates

Before handoff or PR:
```bash
python ratchet.py check    # Must pass — blocks if quality regressed
```

After improving quality:
```bash
python ratchet.py measure  # Lock in improvements as new floor
```

### Agent-Native Output

All commands support `--json` for structured output:
```bash
python ratchet.py check --json     # {status, metrics[], failures[]}
python ratchet.py debt --json      # {items[{file, impact, score}]}
python ratchet.py orient --json    # {lint_violations, test_count, coverage_percent, top_debt}
```

### Key Files

| File | Purpose |
|------|---------|
| `ratchet.py` | Core engine (single file, zero deps) |
| `.ratchet.yaml` | Config: language, metrics, priority weights |
| `.ratchet-state.json` | Enforced floors (violations down, tests up) |
| `.ratchet-history.jsonl` | Weekly trend data |

### Tech Debt Prioritization

```bash
python ratchet.py debt            # Ranked by business impact
python ratchet.py debt --growth   # Growth-path items only
```

### Skills

| Skill | Command | When |
|-------|---------|------|
| Quality Ratchet | `/quality-ratchet` | Enforce floors, measure baselines, scan debt |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Emuthmartinez)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Emuthmartinez)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
