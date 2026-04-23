## agentictrading

> Prevent any reference to external execution repos in factory code


# No External Repo References

This repo is fully standalone. Any reference to an external execution
repository is a bug.

## Forbidden patterns in Python code

- `EXECUTION_REPO_ROOT` used as a non-empty path
- `os.environ.get("EXECUTION_REPO_ROOT")` with a non-empty fallback
- Importing modules from any path outside this repo
- References to `/Documents/Coding/Arbitrage` or similar paths

## If you encounter these

Remove them. The factory must resolve all data from `data/` and all
portfolio state from `data/portfolios/` within this repo. There is
no external execution repo.

## Correct data resolution

```python
data_root = project_root / "data"
portfolio_root = project_root / "data" / "portfolios"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BenPomme) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
