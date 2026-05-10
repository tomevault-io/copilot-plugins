## build-your-own-agents-skill

> This repo is designed so an AI coding agent can read a small set of files and immediately produce a serious, production-oriented agent design.

# AGENTS.md

## Purpose
This repo is designed so an AI coding agent can read a small set of files and immediately produce a serious, production-oriented agent design.

## Canonical Entry Points
- Skill: [`skills/production-agent-architecture/SKILL.md`](skills/production-agent-architecture/SKILL.md)
- Reference: [`skills/production-agent-architecture/reference.md`](skills/production-agent-architecture/reference.md)
- Scaffolder: [`skills/agent-scaffolder/SKILL.md`](skills/agent-scaffolder/SKILL.md)
- Example: [`examples/marketing-agent/agent-build-spec.md`](examples/marketing-agent/agent-build-spec.md)
- Human quickstart: [`docs/getting-started.md`](docs/getting-started.md)

## Required Output
If you use this repo to design an agent, output all of the following:
- `Agent Build Spec`
- controller loop pseudocode
- tool contract table
- session state schema
- permissions and approval matrix
- failure and retry strategy
- observability minimums
- evaluation plan
- rollout phases

## Design Rules
- Prefer loop-based runtimes over one-shot chatbot wrappers.
- Separate tool schemas from tool execution.
- Treat permissions as runtime policy, not prompt text.
- Make approval flows explicit.
- Include evaluation and observability early.
- Keep v1 narrow and testable.

## Recommended Read Order
1. [`skills/production-agent-architecture/SKILL.md`](skills/production-agent-architecture/SKILL.md)
2. [`skills/production-agent-architecture/reference.md`](skills/production-agent-architecture/reference.md)
3. [`examples/marketing-agent/agent-build-spec.md`](examples/marketing-agent/agent-build-spec.md)
4. production docs under [`docs/`](docs/)

---
> Source: [xuanhieu2611/build-your-own-agents-skill](https://github.com/xuanhieu2611/build-your-own-agents-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
