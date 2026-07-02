## lattice

> This is the Lattice source repository. It builds a repo-local AI Coding harness that users install into their own projects.

# Lattice Agent Guide

## Role

This is the Lattice source repository. It builds a repo-local AI Coding harness that users install into their own projects.

Do not treat this repository as a target project already using Lattice. The installable artifact is `harness-template/` plus `prismspec/`.

## Product Direction

Optimize for a Chinese-first, AI-friendly, commercially credible developer tool:

- Chinese docs are the primary public entry; English docs are secondary but should not contradict Chinese docs.
- PrismSpec is the standalone Spec Coding skill pack.
- Lattice is the repo-local harness that adds context, verification, evidence, loop, and learn.
- Claims must be grounded in current files, commands, examples, or test output.
- Prefer fewer concepts with stronger contracts over broad platform language.

## Source Of Truth

| Surface | Canonical Location |
|---------|--------------------|
| Public product docs | `README.md`, `README.en.md` |
| System design docs | `docs/wiki/` |
| PrismSpec workflow | `prismspec/skills/*/SKILL.md` |
| PrismSpec package contract | `prismspec/skillpack.yaml` |
| PrismSpec templates | `prismspec/templates/` |
| Lattice install template | `harness-template/` |
| CI artifact/comment workflow template | `harness-template/.github/workflows/lattice-eval.yml` |
| Delivery pipeline, doctor, gates, and eval summary | `harness-template/lattice/kernel/`, `harness-template/lattice/kernel/delivery/` |
| Context layer | `harness-template/lattice/context/`, `harness-template/lattice/kernel/context/` |
| Target-project Claude import | `harness-template/CLAUDE.lattice.md` |

## Architecture Vocabulary

Use these component names consistently in product docs and agent-facing docs:

| Term | Meaning |
|------|---------|
| PrismSpec | Spec Coding workflow and skill pack. |
| Orchestrator | Agent control plane for stage routing, spec status, task selection, and evidence gating. |
| Context | Project context supply: map, project knowledge, external references, and selected facts recorded in `spec.md`. |
| Verification | Deterministic command execution: pipeline and gates. |
| Evidence / Eval | Structured records, summaries, history, outcomes, central sink, and dashboard. |
| Loop / Learn | Retry, escalation, learn drafts, reviewer evidence, and knowledge promotion. |

Do not use `Eval` as a synonym for running tests. Verification runs commands; Evidence / Eval records and analyzes the results.

## Design Rules

- Keep the public product experience Chinese-first; keep English docs as secondary entry points.
- Keep PrismSpec independent. Do not duplicate SDD workflow logic under `harness-template/lattice/skills/`.
- Use directory specs as the only default shape: `lattice/specs/<spec-id>/spec.md`, `plan.md`, `review.md`, and `verify.md`.
- Preserve the install boundary: `kernel/` is framework code; `manifest.yaml`, `context/`, and `specs/` are user assets.
- Do not overwrite project-owned files on upgrade unless the user explicitly asks.
- Prefer small shell contracts for install, routing, and gates. Move only genuinely complex parsing into separate tools.
- Keep docs current with implementation. Do not leave obsolete names, legacy paths, or one-off planning notes in public docs.
- Context is not a bulk loader. Treat `lattice/context/README.md` as the agent-readable map; treat scripts as optional helpers.
- Learn is governed. New durable knowledge should have source, review, and promotion evidence when the harness provides it.
- Shell is appropriate for install, CI, gates, deterministic lint, sync, and evidence generation. Semantic context selection and architectural judgment belong to the Agent and the spec skills.

## Common Tasks

| Task | Edit |
|------|------|
| User-facing positioning | `README.md`, `README.en.md` |
| Design explanation | `docs/wiki/*.md` |
| Spec workflow behavior | `prismspec/skills/*/SKILL.md`, `prismspec/bin/guide.sh`, `prismspec/bin/lint.sh` |
| Spec templates | `prismspec/templates/` |
| Install/init behavior | `install.sh`, `init.sh`, `harness-template/` |
| Pipeline behavior | `harness-template/lattice/kernel/delivery/` |
| Context behavior | `harness-template/lattice/context/`, `harness-template/lattice/kernel/context/` |
| Runnable examples | `examples/` |

## AI-Friendly Maintenance Checklist

Before changing docs or skills, check:

- Does the instruction point to an existing file or command?
- Can a fresh Agent resume from artifacts instead of conversation memory?
- Are generated artifacts separated from project-owned assets?
- Are Plan Mode and TDD Mode described as implementation policies, not separate workflows?
- Is each new concept backed by a path, command, or example?

## Verification

Run these before reporting completion:

```bash
bash -n init.sh install.sh tests/smoke-test.sh $(find harness-template prismspec/bin -name '*.sh')
shellcheck --severity=warning init.sh install.sh tests/smoke-test.sh $(find harness-template prismspec/bin -name '*.sh')
bash tests/smoke-test.sh
bash examples/go-gin-gorm/try-it.sh
git diff --check
```

For docs-only changes, at least run:

```bash
git diff --check
rg -n "\\bscaffold\\b|scaffold-template|create-item-api\\.md|test-feature\\.md|Eval Evidence|lattice/skills/sdd|kernel/knowledge/loader\\.sh" README.md README.en.md docs prismspec harness-template examples SKILL.md -S
```

## Release Hygiene

- Root `README.md` is the Chinese default entry.
- `README.en.md` is the English entry.
- Avoid extra redirect-only README files.
- Keep `CLAUDE.md` small; long-lived repo instructions belong here.
- Keep target-project instructions in `harness-template/CLAUDE.lattice.md`.

---
> Source: [zdolphin07-dotcom/lattice](https://github.com/zdolphin07-dotcom/lattice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
