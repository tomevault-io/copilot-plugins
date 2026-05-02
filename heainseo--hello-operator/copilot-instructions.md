## hello-operator

> `hello-operator` is the canonical consumer validation repo for `kube-slint`.

# AGENTS.md

## Repo Identity

`hello-operator` is the canonical consumer validation repo for `kube-slint`.

It may look like a standard Kubebuilder sample operator, but for Codex work its primary role is validating consumer integration friction, reporting DX findings, and preserving a reproducible reference consumer path.

## Source Of Truth Priority

1. `AGENTS.md`
2. `docs/CODEX_OPERATING_RULES.md`
3. `docs/KUBE_SLINT_DX_AUDIT.md` as audit/evidence, not rule-setting authority
4. `docs/KUBE_SLINT_CONSUMER_UX_TEST_REPORT.md` as audit/evidence, not rule-setting authority
5. `docs/PROGRESS_LOG.md`
6. `Tiltfile`
7. `hack/run-slint-gate.sh`
8. `.slint/policy.yaml`
9. `test/e2e/sli_*`
10. `README.md`
11. legacy/sample-oriented Kubebuilder defaults as reference-only unless aligned above

## Change Principles

- No product feature development unless explicitly requested.
- Keep this repo interpretable from the consumer side, not the product-author side.
- Preserve friction evidence instead of silently optimizing it away.
- Prefer the smallest diff that clarifies the repo's role.
- If a behavior or environment assumption is uncertain, document it as an assumption or TODO rather than normalize it.

## Test And Docs Update Rules

- If consumer validation scope changes, update `docs/CODEX_OPERATING_RULES.md` and the relevant audit/report docs together.
- Keep `docs/KUBE_SLINT_DX_AUDIT.md` and `docs/KUBE_SLINT_CONSUMER_UX_TEST_REPORT.md` as primary evidence documents for consumer-friction claims, below the repo rule documents.
- Update `docs/PROGRESS_LOG.md` for real task/status changes, not as the sole contract document.
- Do not rewrite Kubebuilder-derived test assets as product claims; they are fixtures unless a higher-priority doc says otherwise.

## Hard Boundaries

- No controller logic changes, reconcile changes, or operator feature work in operating-rules tasks.
- No test semantics changes unless the task is explicitly about semantics.
- No workflow logic rewrites during identity/documentation cleanup.
- Do not edit sibling repos from this repo's task context.

## Read First

1. `docs/KUBE_SLINT_DX_AUDIT.md`
2. `docs/CODEX_OPERATING_RULES.md`
3. `docs/KUBE_SLINT_CONSUMER_UX_TEST_REPORT.md`
4. `docs/PROGRESS_LOG.md`
5. `Tiltfile`
6. `hack/run-slint-gate.sh`
7. `.slint/policy.yaml`
8. `test/e2e/sli_integration_test.go`
9. `test/e2e/sli_e2e_test.go`
10. `README.md`

## Reporting Format

- Start with confirmed consumer-side facts.
- Separate "consumer friction observed" from "consumer fixture intentionally retained".
- Name the document or code path that supports each important conclusion.
- If sample-operator assets remain for fixture reasons, say so explicitly.

## Agent Roles

- Consumer integration / friction audit agent: analyze onboarding, path mismatch, packaging gaps, and operator-consumer attachment friction.
- Docs / reporting agent: maintain repo identity wording, audit/report authority, and progress-log discipline.

Parallel agents are limited to read-heavy exploration and analysis. All write changes must be integrated by the main thread.

---
> Source: [HeaInSeo/hello-operator](https://github.com/HeaInSeo/hello-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
