## skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains agent skills for neural network interpretability using [NNsight](https://nnsight.net/). Skills are packaged for both Claude Code (via `.claude-plugin/`) and OpenAI Codex (via `.codex/skills/`).

## Architecture

Skills follow the [Agent Skills Specification](https://agentskills.io/):

- **Skill files**: `plugins/nnsight/skills/<skill-name>/SKILL.md` - Markdown files with YAML frontmatter (`name`, `description`) containing technique documentation and code examples
- **Claude Code plugin**: `plugins/nnsight/.claude-plugin/plugin.json` - Plugin metadata
- **Marketplace config**: `.claude-plugin/marketplace.json` - Declares available plugins
- **Codex symlinks**: `.codex/skills/` - Symlinks to skill directories for Codex compatibility

Each skill is a self-contained guide for a specific interpretability technique: nnsight-basics, logit-lens, activation-patching, attribution-patching, causal-tracing, and model-steering.

## Adding a New Skill

1. Create `plugins/nnsight/skills/<skill-name>/SKILL.md` with:
   ```yaml
   ---
   name: skill-name
   description: When to use this skill (appears in agent selection).
   ---
   ```
2. Add technique documentation with Python code examples using NNsight patterns
3. Create symlink: `cd .codex/skills && ln -s ../../plugins/nnsight/skills/<skill-name> .`

## NNsight Patterns Used Across Skills

All skills use consistent NNsight patterns:
- `LanguageModel("model-name", device_map="auto", dispatch=True)` for loading
- `model.trace(prompt)` context for interventions
- `.save()` to persist values outside trace
- `tracer.invoke(prompt)` for batched processing
- `model.transformer.h[layer].output[0]` for GPT-2 hidden states (adjust path for other architectures)
- `remote=True` for NDIF remote execution on large models

---
> Source: [ndif-team/skills](https://github.com/ndif-team/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
