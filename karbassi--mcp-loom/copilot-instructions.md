## mcp-loom

> uv run --with pytest pytest tests/ -v

# Project Instructions

## Testing

```sh
uv run --with pytest pytest tests/ -v
```

## Releasing

- Tag versions with annotated tags: `git tag -a v1.2.3 -m "v1.2.3"`
- Do NOT create GitHub releases — tags + CHANGELOG.md is sufficient
- Bump version in both `pyproject.toml` and `src/loom_mcp/server.py`

---
> Source: [karbassi/mcp-loom](https://github.com/karbassi/mcp-loom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
