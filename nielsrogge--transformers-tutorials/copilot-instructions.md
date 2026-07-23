## transformers-tutorials

> Use `uv` alongside the virtual environment when running Python scripts or notebooks.

# Python enviromnent

Use `uv` alongside the virtual environment when running Python scripts or notebooks.

When requiring environment variables, use:

```bash
uv run --env-file keys.env main.py
```

Do not edit the `pyproject.toml` file manually. Always use `uv add` to let uv add Python packages to the virtual environment.

---
> Source: [NielsRogge/Transformers-Tutorials](https://github.com/NielsRogge/Transformers-Tutorials) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
