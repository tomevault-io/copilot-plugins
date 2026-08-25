## persona-lightsim

> Lightweight persona market-research & simulation harness on Nemotron-Personas 10-country data.

# persona-lightsim

Lightweight persona market-research & simulation harness on Nemotron-Personas 10-country data.

**Triggers:** For persona sampling, use-case analysis, willingness-to-pay, segment quantification → use the `persona-research` skill (orchestrator). For dialogue-free reaction simulation, card distillation, pack loading → use the `persona-lightsim` skill. Simple questions (data location, previous results) can be answered directly.

**Data:** `data/nemotron-personas-*` — if missing, run `python3 scripts/setup_data.py` first (downloads the 63MB lite pack from HuggingFace and creates `data/.venv-personas`). Full-data override: `NEMOTRON_PERSONAS_BASE` env var. The sampler must run through `data/.venv-personas/bin/python` (needs pyarrow).

**Language:** Active skill/agent docs are English by default; `python3 scripts/set_language.py ko` switches to Korean. Sources live in `locales/{en,ko}/` — edit there, then rerun set_language; never edit `.claude/` copies directly.

**한국어:** 페르소나 표본 추출·유즈케이스 분석·지불 의사·세그먼트 정량화는 `persona-research` 스킬, 대화 없는 반응 시뮬·카드 증류·팩 적재는 `persona-lightsim` 스킬을 사용하라. 데이터가 없으면 먼저 `python3 scripts/setup_data.py`. 스킬 문서 한국어 전환: `python3 scripts/set_language.py ko` (원본은 `locales/`에서 수정 후 재적용).

---
> Source: [Dongkyu-ES/persona-lightsim](https://github.com/Dongkyu-ES/persona-lightsim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
