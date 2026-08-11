## activate-venv-before-python

> Activate venv before python/pip


# Python venv

Before any `python` or `pip` (or `pytest` etc.), activate the project venv. Folder: `.venv` or `ambiente_virtual`.

```bash
source .venv/bin/activate && python scripts/script.py
```

Use `ambiente_virtual/bin/activate` if that's the folder name.

If neither exists: try to find a venv (e.g. other common names, or dirs containing `pyvenv.cfg`). If none or multiple are found, prompt the user.

This project handles dependencies in a `pyproject.toml` file instead of requirements.txt file.
Install it with `pip install -e .` (from the `llm/` directory).

---
> Source: [leanseefeld/8iadt-tc-fase3-assistente-medico](https://github.com/leanseefeld/8iadt-tc-fase3-assistente-medico) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
