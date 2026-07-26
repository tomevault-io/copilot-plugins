## camunda-platform-helm

> Follow the root `AGENTS.md` and the path-scoped guides in `.github/instructions/`.

# GitHub Copilot — Repository Instructions

Follow the root `AGENTS.md` and the path-scoped guides in `.github/instructions/`.

## Comments

Write comments only to explain non-obvious **how**. Never add reasoning, rationale,
"why", or narration comments. Architectural rationale belongs in an ADR under `docs/adr/`,
authored by a human — never create or edit ADRs proactively. Tactical rationale (bug-fix
defaults, timeouts, label choices) goes in the PR body or commit message. Keep only required
structured comments: Apache license headers, `## @param`/`## @extra` values docs, the
`{{- /* NOTE */ -}}` helper convention, and lint/build pragmas (`//nolint`, `//go:build`,
`# yamllint disable`, `# yamllint disable-line`, `# shellcheck disable`).

---
> Source: [camunda/camunda-platform-helm](https://github.com/camunda/camunda-platform-helm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
