## python

> How to run Python and skillsaw in this project


This project uses a `.venv` virtualenv. Never invoke bare `python` or
`python3` — the system interpreter won't have skillsaw installed. Use:

- `.venv/bin/skillsaw` to run the linter directly
- `.venv/bin/python3` when you need the project's interpreter
- `make` targets when they exist (`make test`, `make lint`, `make format`)

---
> Source: [stbenjam/skillsaw](https://github.com/stbenjam/skillsaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
