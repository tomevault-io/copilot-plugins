## optimize-anything

> Optimize any text artifact -- prompts, code, configs, skills -- using evolutionary search with gepa

optimize-anything uses [gepa](https://gepa-ai.github.io/gepa/) to evolve text artifacts
through LLM-guided mutations scored by your custom evaluator. gepa proposes changes,
evaluates them, reflects on what worked, and iterates — like a researcher improving
a draft through structured feedback cycles.

## What Can Be Optimized

Any text artifact with a measurable quality signal:
- **Prompts and instructions** — system prompts, agent SOPs, few-shot examples
- **Code** — algorithms, configs, SQL queries, GPU kernels
- **Skills and tools** — SKILL.md files, tool descriptions, agent architectures
- **Content** — templates, documentation, structured output formats

## Quick Start

1. **Prepare a seed** — the text you want to optimize
2. **Create an evaluator** — use the **generate-evaluator** skill to build one matched to your objective
3. **Run optimization** — use `/optimize-anything:optimize` for guided mode, or `/optimize-anything:quick` for zero-config

Or skip evaluator setup entirely — the guided workflow handles it:

```
/optimize-anything:quick my-prompt.txt "make it clearer and more specific"
```

## Available Skills

- **generate-evaluator** — Choose the right evaluator pattern (judge, command, composite) and generate a script
- **optimization-guide** — Full workflow walkthrough covering optimization modes, configuration, budget, and result interpretation
- **evaluator-patterns** — Library of ready-to-use evaluator templates for prompts, code, docs, and agent instructions

## Key Concept: Evaluator Quality = Optimization Quality

gepa's reflection LM reads your evaluator's output to decide what to change next. Return rich feedback — sub-scores, error messages, improvement hints — not just a number. An evaluator returning `{"score": 0.4, "clarity": 0.8, "completeness": 0.2, "missing": "no error handling instructions"}` drives far better optimization than one returning `{"score": 0.4}`.

---
> Source: [ASRagab/optimize-anything](https://github.com/ASRagab/optimize-anything) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
