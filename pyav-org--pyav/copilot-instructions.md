## pyav

> Run `source ./scripts/activate.sh` if not in virtualenv

# Dev Workflow

Run `source ./scripts/activate.sh` if not in virtualenv

# 1. Make changes

# 2. Build

make

# 3. Test

make test # All tests
python -m pytest tests/some_file.py # Individual file
python -m pytest -k "substring"

# 4. Lint before commiting

make lint

# Gotchas

- Uses Cython so look out for UB

---
> Source: [PyAV-Org/PyAV](https://github.com/PyAV-Org/PyAV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
