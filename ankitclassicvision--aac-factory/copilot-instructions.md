## aac-factory

> This file mirrors `CLAUDE.md` for Codex, Gemini, Copilot, and other agents. Read `CLAUDE.md` for

# AGENTS.md — AAC Factory operating instructions (all AI coding agents)

This file mirrors `CLAUDE.md` for Codex, Gemini, Copilot, and other agents. Read `CLAUDE.md` for
the full operating instructions; the short version:

1. The Factory turns a one-sentence idea into a certified AI Agent Workflow blueprint:
   scaffold with `python3 scripts/new_agent_package.py "Idea Name"`, interview the owner in plain
   language into `concepts/<slug>/atlas/atlas.json`, then run
   `python3 scripts/run_pipeline.py concepts/<slug> --improve` and report the readiness ladder
   honestly (R0 structure / R1 completeness / R2 human-graded goldens / R3-R4 human gates).
2. Hard rules: never author golden examples (harvest real records, humans grade); never fill
   unknown fields with plausible content (TODO + provenance `assumed`); never read `.holdout/`
   from selection code; never overwrite human-edited cards or graded goldens; never take external
   actions — the Factory designs and certifies only.
3. User preferences live in `factory.config.json`; if the user has no model ladder set, run `python3 scripts/scan_models.py --write-config` first (multi-provider detection: Anthropic/OpenAI/Gemini/Mistral/DeepSeek/Ollama) (harness: anthropic-api / claude-code-headless /
   hermes-skill / local-model; auth: oauth-cli vs api-key; model ladder incl. optional local
   models; deploy runtime_target). Per-node card values override preferences.
4. Models are swappable per node via `scripts/improve_node.py`: challengers from the ladder,
   adoption only on better-or-equal-and-cheaper with a sealed-holdout pass, ledgered, re-gated.
5. Verify before claiming anything works: run `tests/test_agent_pipeline.py` and
   `tests/test_improvement_loop.py`.

---
> Source: [AnkitClassicVision/aac-factory](https://github.com/AnkitClassicVision/aac-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
