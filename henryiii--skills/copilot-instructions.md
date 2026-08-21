## skills

> A collection of agent skills following the open <https://agentskills.io> specification, plus a few helper scripts. Skills are tested with OpenCode and open models (Kimi, GLM, Gemma), so write skill instructions to be harness- and model-agnostic — do not assume Claude-specific features. Note that some harnesses (like `pi`) don't tell the model what model it is, so skills that need a model name must derive it another way.

# Agent instructions

## What this repo is

A collection of agent skills following the open <https://agentskills.io> specification, plus a few helper scripts. Skills are tested with OpenCode and open models (Kimi, GLM, Gemma), so write skill instructions to be harness- and model-agnostic — do not assume Claude-specific features. Note that some harnesses (like `pi`) don't tell the model what model it is, so skills that need a model name must derive it another way.

## Structure

- Each skill is a top-level directory containing a `SKILL.md` with YAML frontmatter: `name`, `description`, `license`, and optionally `compatibility` (required tools, e.g. "Requires uv"). The body is step-by-step instructions for an agent.
- `description` should say when to trigger, phrased like "Use when dropping a Python version from a project".
- Optional subdirectories: `resources/` for docs the skill tells the agent to read on demand (see `drop-python/`), `assets/` for files the skill copies into target repos (see `sp-recommends/`), and `scripts/` for helper scripts the skill runs (see `explore-issues/`).
- Python scripts (top-level `scripts/` and inside skills) use only the standard library — run with `python3` directly; GitHub access goes through the `gh` CLI.
- The README lists every skill with a one-line summary; update it when adding or renaming a skill.

## Conventions

- Skills consistently prefer `uv`/`uvx` for running Python tools and `prek -a --quiet` (a Rust pre-commit rewrite) for linting in target repos.
- There is no build, test suite, or lint config in this repo itself.

---
> Source: [henryiii/skills](https://github.com/henryiii/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
