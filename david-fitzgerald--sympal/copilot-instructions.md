## sympal

> PRINCIPLES.md is the authority for all design decisions. User writes all code — LLM teaches, not generates. Hibernating until ~November 2026 (post S2S Intensive).

PRINCIPLES.md is the authority for all design decisions. User writes all code — LLM teaches, not generates. Hibernating until ~November 2026 (post S2S Intensive).

# CONTEXT.md — SymPAL Project Context

> **LLM-Agnostic**: This file provides project context for any LLM (Claude, GPT, Gemini, etc.)

Version 0.3.0 · updated 2026-04-20.

**Phase:** paused — hibernating until ~November 2026 (post S2S Intensive) | **Harness:** L0

## Serves
- **Primary:** `sales-to-systems/` — human-AI symbiosis platform; build capability + portfolio piece

## Docker

```bash
.docker/run.sh projects/paused/SymPAL
```

## Quick Start

1. Read this file
2. Read `foundations/implementation-plan.md` for current milestone
3. Check [GitHub Issues](https://github.com/david-fitzgerald/sympal/issues) for current tasks
4. Reference PRINCIPLES.md for design decisions
5. Use personas for review/challenge work, not creation

## Personas

Use for reviewing, challenging, and red-teaming. Load by reading the persona file from `prompts/`, write AS the persona in first person, follow the persona's rubric for scoring.

| Persona | Function | Core Question |
|---------|----------|---------------|
| Vale | Philosophy | "Is this coherent?" |
| Kael | Implementation | "Will this survive reality?" |
| Ryn | Systems/security | "How will this fail?" |
| Seren | Code craft | "Is this well-crafted?" |
| Orin | User advocacy | "Are users better off?" |
| Adversary | Red team | "What's wrong with this?" |
| Vero Certus | Final reviewer | Catches coherent-but-systematically-wrong errors |
| Solas-Venn | Meta-persona | Creates/validates personas |

**Phase intensity**: Foundations → Vale, Adversary heavy. Implementation → Kael, Ryn, Seren heavy. Orin + Adversary always.

**Roadmap governance**: Vale (principle alignment) + Orin (user value) when roadmap changes. Vero before ratification. Adversary shadows all three.

## Working Patterns

### Versioning

Semantic versioning: always MAJOR.MINOR.PATCH. Git tags for milestones.

| Type | Filename | Version Location |
|------|----------|------------------|
| Foundational docs | No version in name (e.g., `prd.md`) | Header + Version History |
| Personas/prompts | Version in filename (e.g., `vale-v1.1.md`) | Filename + header |
| Reviews | Reference doc version (e.g., `vero-review-prd-v0.2.md`) | Filename |

### Voice Registers

| Document Type | Voice |
|---------------|-------|
| PRINCIPLES.md, philosophical-foundations.md | Formal, rigorous |
| project-context.md, README | Personal, colloquial |
| Persona outputs | Follow prompt spec |

### Reviews

Vero Certus review required before any foundational document is ratified. Persist reviews to `foundations/reviews/` as `[reviewer]-review-[document]-v[X.Y.Z].md`.

### Git Workflow

**Repo**: https://github.com/david-fitzgerald/sympal

**Commit style**:
```
[area]: Brief description

Co-Authored-By: Claude <noreply@anthropic.com>
```

Areas: `feat`, `fix`, `refactor` (code), `foundations`, `prompts`, `docs`, `config`, `meta`

Push after each commit. Commit after completing a task, after significant file changes, and before ending session.

## Resume Point

M3 DSL Compilation — ready to start. See `foundations/implementation-plan.md` and `foundations/tdd.md`.

---
PRINCIPLES.md is the authority. User writes all code. Personas review, not create.

---
> Source: [david-fitzgerald/sympal](https://github.com/david-fitzgerald/sympal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
