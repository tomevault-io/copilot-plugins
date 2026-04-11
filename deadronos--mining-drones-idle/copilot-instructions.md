## mining-drones-idle

> use 'sequentialthinking' tool eagerly.

# AI Agent Instructions

use 'sequentialthinking' tool eagerly.

Look at `.github/instructions/spec-driven-workflow-v1.instructions.md` for guidance on spec/design/tasks.

also look into `.github/instructions/*.instructions.md` files and which glob patterns of files they should guide.

the initial idea was '/idea.md'
progressive designs are at '/memory/designs'

Do not create explainer documents or other documentation unless specifically asked to.

## Important

Run before handing off code changes:

pnpm typecheck
pnpm lint
pnpm test

## 🧠 Memory Bank System

**CRITICAL**: This project uses a structured Memory Bank for context preservation. Review these files before starting work:

- **Memory Location**: `/memory/` folder
- **Task Management**: `/memory/tasks/` with `_index.md` master list
- **Design Documentation**: `/memory/designs/` for architectural decisions
- **Context Files**: `activeContext.md`, `progress.md`, `projectbrief.md`

**ID Management**: Ensure unique Task/Design IDs across both active designs and tasks folders and `/COMPLETED/` subfolders.
📖 **Full Details**: See `.github/instructions/memory-bank.instructions.md`

## 📝 Before You Start

1. ✅ Check Memory Bank files in `/memory/`
2. ✅ Review existing tasks in `/memory/tasks/_index.md`
3. ✅ Update `activeContext.md` with current focus
4. ✅ Follow unique ID allocation rules
5. ✅ Document progress in memory system

- **Memory Updates**: Document significant changes in memory bank

---

📚 **Reference**: `.github/copilot-instructions.md` for detailed technical patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deadronos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/deadronos)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
