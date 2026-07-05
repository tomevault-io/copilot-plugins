## docs-and-artifacts-path

> alephant-ai-gateway documentation and artifact paths; agents must follow when generating docs, plans, or audits


# Documentation and artifact paths (mandatory)

Documentation and artifacts produced in **alephant-ai-gateway** must live under the repository paths below; **plans and audits must be written in Chinese.** Authoritative detail is in [docs/agent-docs/docs-and-artifacts-path.md](docs/agent-docs/docs-and-artifacts-path.md).

## Path rules

| Kind | Path |
|------|------|
| Agent docs | `alephant-ai-gateway/docs/agent-docs/` |
| Human-facing docs | `alephant-ai-gateway/docs/human-docs/` |
| Plans / audits / reviews | `alephant-ai-gateway/docs/plans/` |

## Agent requirements

- When adding **new agent-facing specs**, write them under `alephant-ai-gateway/docs/agent-docs/`.
- When adding **new human-facing docs**, write them under `alephant-ai-gateway/docs/human-docs/`; do not put them in repo-root `docs-dev/` or `docs/`.
- When adding **new plans** or **audit reports**, write them under `alephant-ai-gateway/docs/plans/`; do not put them in repo-root `docs-dev/plans/`.
- Every **plan** must include an **execution checklist**: checkboxes `[]` / `[x]` that map one-to-one to tasks; mark `[x]` when each task is done.
- When referencing docs or plans, treat `alephant-ai-gateway/docs/agent-docs`, `alephant-ai-gateway/docs/human-docs`, and `alephant-ai-gateway/docs/plans` as canonical locations.

---
> Source: [AlephantAI/AIephant-AI-Agent-Gateway](https://github.com/AlephantAI/AIephant-AI-Agent-Gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
