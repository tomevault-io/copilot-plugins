## knowledge-distillery

> Knowledge Distillery — A system that delivers only verified knowledge to AI coding agents.

# AGENTS.md

Knowledge Distillery — A system that delivers only verified knowledge to AI coding agents.
3-layer architecture with convention-based air gap (operational isolation).

- Design documents in `docs/`.

## Project Nature

This project builds the Knowledge Distillery tool AND dogfoods it on itself.
The repository uses its own distillation pipeline to capture knowledge about its own development.

- MUST: All implementation artifacts must be built in a form deployable/applicable to other projects
- Delivery format: Claude Code Plugin (Skill + CLI + schema shipped as a single plugin). Distribution is Claude-first; runtime CLI (`knowledge-gate`) is vendor-neutral (`sqlite3`-based)
- Dogfooding: This repo has its own `.knowledge/vault.db` and GitHub Actions workflows. Plugin assets live under `plugins/knowledge-distillery/`

## Implementation Philosophy

The current implementation serves as a proof of concept. Do not over-engineer for edge cases or enforce rigid policies. Keep the design open to change — tight constraints and exhaustive safeguards make future iteration harder. Focus on experiencing real-world usability first and refining incrementally based on what you learn.

## Knowledge Vault
- A UserPromptSubmit hook reminds you to query the vault when active entries exist
- When the hook fires and the task involves code modifications, query before planning:
  - Single file: `knowledge-gate query-paths <file-path>` (summary index by default)
  - Multiple files: `knowledge-gate domain-resolve-path <path>` → `knowledge-gate query-domain <domain>` (summary index by default)
  - Topic search: `knowledge-gate search <keyword>` (summary index by default)
  - Fetch full details only for the specific entries you need: `knowledge-gate get <id>` or `knowledge-gate get-many <id...>`
- MUST/MUST-NOT rules from returned entries must be strictly followed
- For structural changes in areas without related rules, confirm with a human first
- Do not hand-read vault knowledge: never read `.knowledge/vault.db` or its entries directly — always go through `knowledge-gate`. (Pipeline skills that own `.knowledge/` — `mark-evidence`, `batch-refine`, `triage-backtest` — may read and write the changeset and report files they generate.)

## Memento
- After every git commit, attach a memento session summary as a git note on `refs/notes/commits`
- The summary follows the 7-section format: Decisions Made, Problems Encountered, Constraints Identified, Open Questions, Context, Recorded Decisions, Vault Entries Referenced
- See `/knowledge-distillery:memento-commit` for the full workflow and format specification
- If the PostToolUse hook fires a reminder, follow it — generate the summary and attach the note

## Decision Recording
- When a clear project decision is made during the session, automatically record it using `/knowledge-distillery:record-decision`
- Trigger conditions: scope decisions ("X is out of scope"), architectural choices ("use Y for Z"), established constraints ("we can't do X because Y"), direction confirmations after deliberation
- Do NOT trigger for: user preferences (use auto-memory instead), temporary debugging choices, implementation details obvious from code
- Decision commits do not require a memento note — the decision file itself provides the session context
- Decision files are committed on the current branch and enter the vault through the normal pipeline (PR merge → mark-evidence → batch-refine)

## Versioning

The `bumping-version` skill (in `skill-set`) reads this section.

- **Base branch**: main
- **Commit message**: chore: bump version to {version}
- **Extra version files**: (none — `plugins/knowledge-distillery/.claude-plugin/plugin.json` is auto-detected)
- **Changelog categories**: Added, Improved, Fixed

---
> Source: [ether-moon/knowledge-distillery](https://github.com/ether-moon/knowledge-distillery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
