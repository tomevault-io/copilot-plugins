## mentor

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## What This Is

Mentor is a Codex skill (`SKILL.md` with YAML frontmatter) that generates docs-first mentorship compasses for technical topics. It runs on Codex.ai and Codex, and relies on current source lookup to verify resource URLs and freshness.

The intended behavior is direction-setting, not tutorial generation:
- read this documentation first
- check this page next
- inspect this repo or artifact for a pattern
- notice this concept before moving on
- avoid this rabbit hole for now

## Repository Structure

- `SKILL.md` - The skill definition. This is the primary artifact. All behavior rules, output format, source ranking, and anti-patterns live here.
- `references/schema.json` - JSON Schema contract for machine-readable compass output.
- `examples/` - Gold-standard example outputs that demonstrate correct behavior:
  - `example-output-rust.md` - broad topic with user background
  - `example-output-cloud-run.md` - broad topic, default assumptions
  - `example-output-terraform-modules.md` - narrow topic, compressed compass
  - `example-output-react-server-components.md` - clarification trigger + fast-moving topic
- `evals/evals.json` - 10 test cases with machine-gradable assertions covering structure, source quality, mode variety, dedup, content, clarification, personalization, freshness, time constraints, weak-docs handling, schema compliance, and tutorial-drift guards.
- `scripts/validate.mjs` - Dependency-free validation harness for schema, eval, example, and drift checks.
- `TESTING.md` - Test strategy and validation notes.

## How the Artifacts Relate

`SKILL.md` is the source of truth. The schema, examples, docs, and evals must stay consistent with it:
- If you change the output format in `SKILL.md`, update `references/schema.json`.
- If you change decision rules, update affected examples and eval assertions.
- If you change resource modes, update schema enums and every example.
- If you change public positioning, update `README.md` and `CONTRIBUTING.md`.
- If you add or change behavior contracts, update `scripts/validate.mjs` when the rule can be checked statically.

## Validation

Run:

```bash
npm test
```

This validates JSON syntax, schema contract behavior, enum alignment, example structure, resource deduplication, eval shape, and stale terminology. For behavior changes, also forward-test at least one broad topic and one narrow topic in a fresh agent/thread.

## Key Design Constraints

- **Mentorship compass, not tutorial**: Default output points to docs and inspection targets; it does not generate setup steps or exercises.
- **Source hierarchy is strict**: Official docs > maintainer/vendor sources > official sample repos as inspection targets > justified community sources.
- **Broad vs. narrow topics produce different compass sizes**: Broad topics usually use 4-5 iterations; narrow topics usually use 2-4; time-boxed topics under 3 hours usually use 2-3.
- **Resource modes are constrained**: Use `Read first`, `Read next`, `Skim for context`, `Inspect`, `Bookmark`, and `Practice later`.
- **Practice is opt-in**: Quickstarts, tutorials, labs, and commands belong outside the main compass unless the user explicitly asks for hands-on practice.
- **Dedup rule**: A resource in the Learning Compass must not appear again in Explore Later.
- **Clarification policy is minimal**: Ask only when the answer would change the first three resources.

---
> Source: [ayhammouda/mentor](https://github.com/ayhammouda/mentor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
