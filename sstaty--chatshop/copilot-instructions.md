## chatshop

> - Keep your context windows focus on the current goal. Ingest only relevant information.

# ChatShop — Claude Code Instructions

- Keep your context windows focus on the current goal. Ingest only relevant information.

## Communication style
- ALWAYS default to conversation over directly implementing plan or code. First conversation, then agreement, then action. Prioritize conversation, asking for extra information, suggesting better solution, brainstorming. 
- Push back when something is wrong or suboptimal. Say so directly. Don't assume I am right. Challenge my assumptions when warranted.
- Play devil's advocate on architectural decisions when relevant. If I propose something technically flawed, say "that's wrong because X"

## Project structure, usage & workflow
- Refer to all files under /documentations when planning or coding a new feature. This folder contains all plans, architecture, requirements.
- After completing each implementation task, update the respective documentation .md files to reflect what was actually built. Note any deviations from the original design and why. Do not update mid-task, only when done.
- We use "uv" for this project, so always use "uv" (instead of pip, etc.). When trying to run python, you must use "uv run python ..."

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sstaty) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
