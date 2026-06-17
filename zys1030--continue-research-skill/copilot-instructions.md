## continue-research-skill

> Restore the current state of an ongoing research code project before starting a new discussion. Use when the user explicitly invokes $continue-research or asks to read the project's progress and understand where the research currently stands. Read the project's local handoff documents, summarize the active state, and stop for the user's next question. Do not begin analysis, propose a new plan, edit files, or run experiments unless the user asks afterward.


# Continue Research

## Purpose

Build a concise research handoff from the project's own documents. This skill is intentionally lightweight: it restores context but does not freeze project-specific methods or statistical conventions.

## Workflow

1. Start from the current project root. Read `AGENTS.md` first when present and follow its local instructions.
2. Read `README.md` when present to recover the project entry points and document map.
3. Read `docs/progress.md` when present. Treat it as the primary source for the current research state.
4. Read the decisions index, such as `docs/decisions/README.md`, when present. Open only ADRs directly referenced by the active sections of `progress.md`, or the most recent ADRs needed to understand the current state.
5. Read other documents only when they are directly linked from the active progress notes or needed to disambiguate the current state. Avoid broad code scans, raw-data scans, and loading all historical ADRs by default.
6. Return a concise handoff summary and stop. Wait for the user to initiate the next discussion.

## Summary Format

Keep the response short and use these sections when applicable:

- **Current State**: the active research direction and the latest confirmed facts.
- **Open Questions**: unresolved questions explicitly recorded in the documents.
- **Relevant Records**: the small set of progress sections or ADRs used to build the summary.

End by stating that context has been restored and that you are waiting for the user's next question.

## Boundaries

- Do not modify files.
- Do not run tests, scripts, or experiments.
- Do not inspect large output directories or raw run data.
- Do not turn unresolved ideas into conclusions.
- Do not propose next steps unless the user asks for them.
- If `docs/progress.md` is missing, say so and summarize only the documents that are available.

---
> Source: [zys1030/continue-research-skill](https://github.com/zys1030/continue-research-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
