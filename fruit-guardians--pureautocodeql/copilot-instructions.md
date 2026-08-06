## pureautocodeql

> Runtime application code lives under `pure_auto_codeql/`.

# Agent notes

Runtime application code lives under `pure_auto_codeql/`.

- Prefer: `from pure_auto_codeql.configuration import get_llm_config, LLMRole`
- Prefer: `from pure_auto_codeql.utils...`, `pure_auto_codeql.services...`, etc.
- User secrets: repo-root `config/keys.toml` (template: `config/keys.example.toml`)
- CLI: `uv run pure-auto-codeql ...` or `python Analyze.py ...`
- API: `uvicorn pure_auto_codeql.api.server:app`

See `docs/package_architecture.md` for the package layout.

---
> Source: [Fruit-Guardians/PureAutoCodeql](https://github.com/Fruit-Guardians/PureAutoCodeql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
