## tax-co-pilot

> <!-- SPDX-License-Identifier: GPL-3.0-or-later -->

<!-- SPDX-License-Identifier: GPL-3.0-or-later -->
# Agent Entry Point (Strict)

Before starting any task, an agent **MUST** read the relevant file(s) in `.agent_tools/`.

## Required Routing
- Global constraints: `.agent_tools/00_master_directives.md`
- Coding style: `.agent_tools/01_style_guide.md`
- Architecture boundaries: `.agent_tools/02_architecture.md`
- Validation requirements: `.agent_tools/03_testing_rules.md`
- Documentation maintenance: `.agent_tools/04_doc_updater.md`
- Session log protocol: `.agent_tools/05_session_log.md`

## Repository Mapping Rule
- Use the repository file tree in `README.md` as the primary map for project structure before manually traversing directories.
- If any task adds, moves, renames, or deletes files/directories, the same task **MUST** update the file tree in `README.md`.

## Non-Negotiable Completion Rule
After finishing any task, the agent **MUST** append exactly one new line to `.agent_tools/05_session_log.md` in this exact format:

`- [YYYY-MM-DD] {File Changed}: {Technical Summary}`

No rewrites, no deletions, and no reordering of prior log entries are allowed.

---
> Source: [DesignForFailure/Tax_Co-Pilot](https://github.com/DesignForFailure/Tax_Co-Pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
