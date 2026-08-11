## upmio-clickhouse-hackathon

> - Scope: Repository-wide engineering rules for AI coding agents

# AGENTS.md

- Version: 0.8
- Date: 2026-06-05
- Status: Confirmed
- Scope: Repository-wide engineering rules for AI coding agents

## 1. Purpose

This file defines repository-wide engineering rules for AI coding agents.

It is not a product design document. It must not duplicate project decisions or phase-specific implementation details.

Follow sealed decisions in `docs/master-spec.md`. Do not reinterpret or override them here.

## 2. Required Reading

Before starting any phase work, read only:

1. `docs/master-spec.md`
2. `docs/evidence-summary.md`
3. the current phase spec under `docs/phases/`
4. architecture/design documents explicitly referenced by the current phase spec

Do not read or rewrite all raw `docs/discovery/` or `docs/runtime/` files by default. Open raw evidence only when the phase spec requires it or implementation conflicts with expected behavior.

## 3. Spec-First Rule

Do not start feature code without a confirmed phase spec or explicit user approval.

If a phase spec is missing, vague, or inconsistent with `docs/master-spec.md`, stop and ask for clarification.

## 4. Scope Control

Implement only what the current phase spec requires.

Do not add adjacent features, refactors, dashboards, CRDs, APIs, SQL operations, scripts, or dependencies unless the phase spec explicitly includes them.

If a useful improvement is found outside scope, record it as Future Work or Open Question.

## 4.1 UPM API Server Integration Rule

`upm-api-server` is the product API entry point.

Any new product feature that exposes user-facing or automation-facing behavior
must be integrated and registered in `upm-api-server` unless the current phase
explicitly defines the work as package-only, operator-only, evidence-only, or
internal implementation-only.

When an API is added or changed, update the repository API reference in the
same phase so every supported endpoint, parameter, request body, response
shape, and usage example is documented.

## 5. Safety Rules

- Never commit secrets.
- Do not weaken or delete tests to make work pass.
- Do not push directly to `main` unless explicitly instructed.
- Do not install application services directly on Kubernetes host nodes.
- Do not bypass the product path defined in `docs/master-spec.md`.
- Do not automate destructive or high-risk operations without explicit human approval.

## 6. Quality Gate Commands

Use commands that match the files/tools touched by the current phase.

```bash
# Secret scan
gitleaks detect --source . --redact

# Markdown
markdownlint "docs/**/*.md"

# Go
go fmt ./...
go vet ./...
go test ./...
go build ./...

# Helm / package charts
helm lint <chart-dir>
helm template <release-name> <chart-dir> --values <values-file> >/tmp/rendered.yaml

# Kubernetes server-side validation
kubectl apply --dry-run=server -f <manifest.yaml>

# Repository-level gate, if Makefile is added later
make quality
```

If a command is not available yet, record it as skipped with the reason.

Do not claim completion without evidence. Command output, tests, diffs, and runtime checks are acceptance evidence; AI completion statements are not.

## 7. Commit Rules

Use one branch per phase. Do not mix unrelated phase work.

Each commit must have a single purpose.

Commit message format:

```text
phase-XX: concise description
```

## 7.1 Phase Branch Merge Rules

Use `main` as the stable third-party entry branch.

After a phase passes implementation, validation, review, accepted repair, and
closeout, merge that phase branch back into `main`.

Keep historical `phase-XX` branches after merge as process evidence unless the
owner explicitly asks to delete them.

Start the next phase branch from updated `main`, not from an older phase branch.

Do not merge an unfinished phase branch into `main`.

Direct pushes to `main` are allowed only for explicit owner-approved phase
closeout merge work or repository-management fixes.

## 8. Handling Uncertainty

When uncertain:

1. Check `docs/master-spec.md`.
2. Check `docs/evidence-summary.md`.
3. Check the current phase spec.
4. Open raw evidence only if needed.
5. Ask the user if still unclear.

Runtime evidence overrides source-code assumptions.

## 9. Completion Report

When finishing a phase, report:

- files changed
- commands run
- commands skipped and why
- tests/results
- known risks
- out-of-scope findings
- whether the phase acceptance criteria are met

## 10. Phase Start Review

Before starting implementation for any repository phase, first read the current phase spec and explicitly report:

- what capability the phase implements;
- what the phase boundaries and non-goals are;
- whether the phase spec is reasonable for the hackathon second-stage goal;
- risks, ambiguities, or scope conflicts that need human decision.

Do not start phase implementation until the owner confirms the phase interpretation or explicitly asks to proceed.

Terminology rule:

- "Hackathon second stage" means the competition development / Demo stage.
- "Repository Phase 02" means `docs/phases/phase-02-clickhouse-topology.md`.

If the wording is ambiguous, default to the hackathon-stage meaning and ask only when the distinction affects implementation.

---
> Source: [weiqinzhou3/upmio-clickhouse-hackathon](https://github.com/weiqinzhou3/upmio-clickhouse-hackathon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
