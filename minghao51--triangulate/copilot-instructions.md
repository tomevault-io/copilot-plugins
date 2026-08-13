## triangulate

> - **Analyze First:** Read relevant files before proposing solutions. Never hallucinate.

## 1. Workflow
- **Analyze First:** Read relevant files before proposing solutions. Never hallucinate.
- **Approve Changes:** Present a plan for approval before modifying code.
- **Minimal Scope:** Change as little code as possible. No new abstractions.

## 2. Output Style
- High-level summaries only.
- No speculation about code you haven't read.

## 3. Technical Stack
- **Python:**
  - Package manager: `uv`.
  - Execution: Always `uv run <command>`. Never `python`.
  - Sync: `uv sync`.
- **Frontend:**
  - Verify: Run `npm run check` and `npm test` after changes.
- **Docs:** Update `ARCHITECTURE.md` if structure changes.
- **Files:** Markdown files must follow `YYYYMMDD-filename.md` format.

---
> Source: [minghao51/triangulate](https://github.com/minghao51/triangulate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
