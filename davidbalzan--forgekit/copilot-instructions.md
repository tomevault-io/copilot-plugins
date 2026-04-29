## forgekit

> This project uses **ForgeKit** - a documentation-driven approach for building software with AI assistance.

# ForgeKit - Copilot Instructions

This project uses **ForgeKit** - a documentation-driven approach for building software with AI assistance.

## Project Structure

```
project-root/
├── apps/
│   ├── web/                   ← Frontend (React + Vite + Tailwind)
│   └── api/                   ← Backend (Node.js + Hono)
├── packages/
│   ├── shared/                ← Shared types & utilities
│   └── ui/                    ← UI component library
├── docs/
│   ├── PRODUCTION_ROADMAP.md  ← Single source of truth (Current Focus at top)
│   ├── DECISIONS.md           ← Architectural Decision Records
│   ├── TECH_STACK.md          ← Technology choices
│   ├── ARCHITECTURE_GUIDE.md  ← System design rationale
│   └── phases/                ← Phase-based task planning
└── docs/CURRENT_FOCUS.md          ← Quick session context
```

## Key Documents to Read First

1. **docs/CURRENT_FOCUS.md** - What's actively being worked on
2. **docs/PRODUCTION_ROADMAP.md** - Overall project status
3. **docs/phases/phaseN/PHASEN_TASKS.md** - Detailed task breakdowns

## Status Indicators

- ✅ Complete
- 🚧 In Progress
- ⏳ Not Started
- 🔴 Critical Priority
- 🟡 Medium Priority
- 🟢 Low Priority

## Task Checkbox Format

```markdown
- [ ] Uncompleted task
- [x] Completed task
```

## When Working on Tasks

1. Check docs/CURRENT_FOCUS.md for active work
2. Find the task in the relevant phase TASKS file
3. Update checkboxes as you complete sub-tasks
4. Update progress metrics when tasks complete

## Coding Preferences

- Clean architectures, modular approach
- Tailwind CSS for styling
- Latest stable versions of dependencies
- Avoid unnecessary code duplication
- Maintain consistency across the codebase

---
> Source: [davidbalzan/forgeKit](https://github.com/davidbalzan/forgeKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
