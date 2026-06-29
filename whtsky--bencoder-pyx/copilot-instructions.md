## bencoder-pyx

> python3 -m venv .venv

# Local Setup

## Build + test from wheel

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -U pip wheel setuptools
python -m pip install -r test-requirements.txt
python -m pip install build asv
python -m build --wheel --no-isolation
python -m pip install --force-reinstall dist/*.whl
python -m pytest -q tests
```

## Benchmarks (asv)

Uses `tests/debian-8.3.0-amd64-netinst.iso.torrent` as input.

```bash
asv machine --yes
asv run --quick -E existing:.venv/bin/python --set-commit-hash $(git rev-parse HEAD)
asv continuous origin/master HEAD --factor 1.10 -E existing:.venv/bin/python
```

---
> Source: [whtsky/bencoder.pyx](https://github.com/whtsky/bencoder.pyx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
