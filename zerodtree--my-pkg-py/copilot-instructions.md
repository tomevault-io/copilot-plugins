## my-pkg-py

> Use uv run or activate venv when running Python commands


# Python Environment

Always use one of the following when running Python commands or scripts:

- Prefix commands with `uv run`: `uv run python script.py`, `uv run pytest`, etc.
- Or activate the virtual environment first: `source .venv/bin/activate`

```bash
# ✅ GOOD
uv run python src/ls_mlkit/util/scheduler.py
uv run pytest tests/

# ✅ ALSO GOOD
source .venv/bin/activate
python src/ls_mlkit/util/scheduler.py

# ❌ BAD
python script.py
pip install package
```

Prefer `uv run` for one-off commands and `source .venv/bin/activate` for interactive sessions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zeroDtree) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
