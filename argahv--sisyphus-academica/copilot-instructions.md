## sisyphus-academica

> You are working on an open-source academic research pipeline. The goal is to help researchers find papers, generate novel hypotheses, verify citations, and improve writing quality — using a multi-agent swarm.

# Sisyphus Academica

You are working on an open-source academic research pipeline. The goal is to help researchers find papers, generate novel hypotheses, verify citations, and improve writing quality — using a multi-agent swarm.

## Architecture

- `orchestrator/research-director.md` — Primary agent, the conductor
- `subagents/` — 8 writing pipeline agents (writer, verifier, style-auditor, etc.)
- `novelty-engines/` — 6 novelty generation agents (contrarian, heretic, etc.)
- `reviewers/` — 10 adversarial reviewer personas
- `tools/` — Python CLI tools (literature search, citation verification)
- `skills/` — Portable agent skills (novelty engines, reviewers as standalone SKILL.md files)
- `templates/` — LaTeX venue templates (NeurIPS, ICML, ICLR, Nature, arXiv)

## Key Constraints

1. **No hardcoded paths in code** — Use relative paths or env vars (SISYPHUS_ACADEMICA_DIR)
2. **Python stdlib preferred** — tools/ should work with urllib + json + xml.requests
3. **Tests must pass** — `python -m pytest tests/ -v`
4. **No network in unit tests** — all tests must work offline
5. **README is the product** — keep it clean, one-line install, no gatekeeping

## Making Changes

- README changes: high impact, be careful with framing
- Tool changes: keep CLI backward compatible, add tests
- Agent prompt changes: maintain output format (JSON schemas in examples)
- Always run flake8 before committing: `flake8 tools/ --max-line-length=100 --ignore=E501,W291`

## Build & Test

```bash
pip install -e ".[dev]"
python -m pytest tests/ -v
flake8 tools/ --max-line-length=100 --ignore=E501,W291
```

---
> Source: [argahv/sisyphus-academica](https://github.com/argahv/sisyphus-academica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
