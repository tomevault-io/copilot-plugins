## club-5060ti

> This repo is a public-facing community guide for RTX 5060 Ti local LLM setups.

# AGENTS.md

This repo is a public-facing community guide for RTX 5060 Ti local LLM setups.

## Rules

- Keep claims evidence-backed. If a number came from one machine, say that.
- Do not include private IPs, API keys, bearer tokens, SSH hostnames, or personal infrastructure details.
- Prefer short reproducible commands over long narrative.
- Keep the repo tidy: docs in docs/, scripts in scripts/, sanitized snippets in examples/, result tables in data/.
- Keep comparisons factual and directly relevant to reproducing 5060 Ti results.
- When adding benchmark data, include method, context length, model, quant, runtime, and caveats.
- Scripts must use standard-library Python or clearly document dependencies.

## Verification

~~~bash
python3 -m py_compile scripts/*.py
bash -n scripts/*.sh examples/*.sh
./scripts/check_repo.sh
~~~

---
> Source: [5p00kyy/club-5060ti](https://github.com/5p00kyy/club-5060ti) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
