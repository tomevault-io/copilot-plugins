## mockstudio

> Guidance for AI agents working in the mockstudio repo.

# AGENTS.md

Guidance for AI agents working in the mockstudio repo.

## Agent skills

### Issue tracker

Issues and PRDs live as markdown files under `.scratch/<feature>/`. See `docs/agents/issue-tracker.md`.

### Triage labels

Five canonical triage roles, recorded as `Status:` values using the default strings. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.

### UI work

Before building or refining `ui/`, load the `local-uidotsh` MCP skill (`uidotsh://ui` → design guidelines, and its ui-picker workflow for exploratory direction choices). Precedence: project decisions win over its generic guidance — semantic token system, Base UI primitives (ADR-0011), dark-only chrome, and the fixed Inspector shell (ADR-0019) are settled; uidotsh is the markup-quality and exploration layer.

---
> Source: [lenxism/mockstudio](https://github.com/lenxism/mockstudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
