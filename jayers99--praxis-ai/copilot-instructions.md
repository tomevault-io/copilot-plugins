## praxis-ai

> Praxis: policy-driven AI workflow system. Deterministic behavior resolution: **Domain + Stage + Privacy + Environment → Behavior**

# Praxis AI - Claude Code Instructions

## Overview

Praxis: policy-driven AI workflow system. Deterministic behavior resolution: **Domain + Stage + Privacy + Environment → Behavior**

Projects live in `$PRAXIS_HOME/projects/`, separate from this framework repo.

## Critical Rules

1. **Formalize is a hard boundary** — No execution without formalization artifacts
2. **Follow Guardrail G1** — Bidirectional Consistency Propagation (see below)
3. **Policy validation is deterministic** — No skipping required artifacts
4. **Hexagonal architecture** — Domain (pure logic) → Application (orchestration) → Infrastructure (external)

## Lifecycle

**Stages:** Capture → Sense → Explore → Shape → **Formalize** → Commit → Execute → Sustain → Close

| From    | Can Regress To    |
|---------|-------------------|
| Execute | Commit, Formalize |
| Sustain | Execute, Commit   |
| Close   | Capture           |

**Iteration meaning changes at Formalize:** Before = discovery (what is this?). After = refinement (how good?). Scope change during Execute → regress to Formalize.

## Domains & Artifacts

| Domain  | Purpose            | Formalize Artifact | Subtypes |
|---------|--------------------|--------------------|-----------------------------------------|
| Code    | Functional systems | `docs/sod.md`      | cli, library, api, webapp, infrastructure, script |
| Create  | Aesthetic output   | `docs/brief.md`    | visual, audio, video, interactive, generative, design |
| Write   | Structured thought | `docs/brief.md`    | technical, business, narrative, academic, journalistic |
| Learn   | Skill formation    | `docs/plan.md`     | skill, concept, practice, course, exploration |
| Observe | Raw capture        | (none)             | notes, bookmarks, clips, logs, captures |

## Privacy Levels

1. Public → 2. Public–Trusted → 3. Personal → 4. Confidential → 5. Restricted

## Guardrail G1: Bidirectional Consistency Propagation

When modifying **canonical files** (domains.md, lifecycle.md, privacy.md, layer-model.md, CLAUDE.md):

1. Identify affected canonical documents
2. Check parent constraints for conflicts
3. Find dependent children
4. Verify no stale references or contradictions
5. Update all affected documents OR flag for review

**Triggers:** Adding/removing/renaming canonical concepts, changing constraints, modifying document relationships, altering validation rules.

See `core/governance/guardrails.md` for dependency graph.

## Validation (ADR-002)

| Rule                         | Severity | Trigger |
|------------------------------|----------|---------|
| Unknown domain/stage/privacy | Error    | Value not in allowed list |
| Missing formalize artifact   | Error    | stage ≥ commit AND artifact missing |
| Invalid stage regression     | Warning  | Transition not in allowed table |
| Privacy downgrade            | Warning  | privacy_level decreased |

## AI Permissions by Domain

| Operation | Code | Create | Write | Learn | Observe |
|-----------|:----:|:------:|:-----:|:-----:|:-------:|
| suggest   |  ✓   |   ✓    |   ✓   |   ✓   |    ✓    |
| complete  |  ✓   |   ✓    |   ✓   |   ✓   |    ✗    |
| generate  |  ?   |   ✓    |   ?   |   ✓   |    ✗    |
| transform |  ?   |   ✓    |   ?   |   ✓   |    ✗    |
| execute   |  ?   |   —    |   —   |   —   |    —    |

✓ = Allowed, ✗ = Blocked, ? = Ask first

## praxis.yaml Schema

```yaml
domain: code|create|write|observe|learn
stage: capture|sense|explore|shape|formalize|commit|execute|sustain|close
privacy_level: public|public-trusted|personal|confidential|restricted
environment: Home|Work
subtype: cli|library|api|...  # Optional
coverage_threshold: 0-100     # Optional
```

## Project Structure

```
src/praxis/
  cli.py           # Typer entry point
  domain/          # Business models
  application/     # Services
  infrastructure/  # External concerns (YAML, git, filesystem)

tests/
  features/        # Gherkin files
  step_defs/       # pytest-bdd steps

core/
  spec/            # Normative: lifecycle.md, domains.md, privacy.md, sod.md
  governance/      # layer-model.md, opinions-contract.md, guardrails.md
  checklists/      # Stage entry/exit criteria

opinions/          # Advisory guidance by domain
research-library/  # Cataloged research (see CATALOG.md)
adr/               # Architecture Decision Records
```

## Commands

```bash
# Development
uv run pytest && uv run ruff check . && uv run mypy .

# CLI
praxis new <name> --domain <d> --privacy <p>
praxis init --domain <d>
praxis validate [--strict]
praxis stage <stage>
praxis status
praxis audit
praxis opinions [--prompt]
```

## Issue Workflow

**Labels:** `maturity: raw|shaped|formalized`, `size: small|medium|large`, `type: feature|spike|chore`, `priority: high|medium|low`

```bash
gh issue list --label "maturity: formalized" --label "type: feature"
```

**Maturity → Lifecycle mapping:** raw (Capture/Sense) → shaped (Explore/Shape) → formalized (Formalize+)

See [CONTRIBUTING.md](CONTRIBUTING.md) for full workflow.

## Opinions Framework

1. Check for `docs/opinions/` directory
2. Read praxis.yaml for domain, stage, subtype
3. Resolve chain: `_shared/` → `{domain}/principles.md` → `{domain}/{stage}.md` → `subtypes/`
4. Apply as guidance (advisory, not binding)
5. User instructions override opinions

Run `praxis opinions --prompt` for formatted AI context.

## References

- **Specs:** `core/spec/` (lifecycle.md, domains.md, privacy.md, sod.md)
- **Governance:** `core/governance/` (guardrails.md, opinions-contract.md)
- **Guides:** `docs/guides/` (user-guide.md, ai-setup.md)
- **ADRs:** `adr/` (001-policy-engine.md, 002-validation-model.md)
- **Research:** `research-library/CATALOG.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jayers99) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
