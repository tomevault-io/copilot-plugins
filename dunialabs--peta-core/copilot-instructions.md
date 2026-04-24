## peta-core

> - **Claude Code** = Surgeon + Analyst + Toolchain Manager

# Cursor IDE Scheduling Rules

## Role Positioning

- **Claude Code** = Surgeon + Analyst + Toolchain Manager
  - Responsible for: Precise, controllable tasks with clear objectives
  
- **Codex** = System Architect + Refactoring Expert + Multi-round Reasoning Engineer
  - Responsible for: Complex, open-ended, large-scale development tasks

---

## Task Assignment Rules

### → Assign to Claude Code (Precise, Controllable)

| Task Type | Tools to Use |
|---------|---------|
| File search/location | `Glob`, `Grep` |
| Precise code modification | `Edit` (string replacement) |
| Testing/Build/Git | `Bash` |
| Task breakdown | `TodoWrite` |
| Document processing | `Read`, `Write`, `WebFetch` |
| Code review | `Task` agent |

**Suitable scenarios**:
- Small-scale precise modifications
- Variable renaming, batch replacement
- Running tests, build commands
- Finding code, understanding structure
- Updating existing documents

### → Assign to Codex (Complex, Large-scale)

| Task Type | Tools to Use |
|---------|---------|
| Large-scale code generation | `codex` |
| Complex refactoring (cross-file) | `codex` |
| Multi-round reasoning | `codex` + `codex-reply` |
| Long workflow tasks | `codex` |
| Repository context reasoning | `codex` |

**Suitable scenarios**:
- New module development
- Cross-file large-scale refactoring
- Complex tasks requiring multi-round reasoning
- Understanding project architecture and making system-level modifications

---

## Collaboration Workflow

```
1. Read context
   ├── CLAUDE.md (project architecture)
   ├── AGENTS.md (development guidelines)
   └── mcp-tools-guide.md (tool inventory)

2. Determine task type
   ├── Precise/controllable → Claude Code
   └── Complex/large-scale → Codex

3. Execute task

4. Update knowledge base
   ├── New feature → docs/implementation/FeatureName_IMPLEMENTATION.md
   ├── API changes → docs/api/API.md
   └── Architecture changes → CLAUDE.md

5. Continue to next task
```

---

## Knowledge Base Rules

- ❌ Prohibit creating duplicate documents
- ✅ Single source of truth: Each feature has only one authoritative document
- ✅ Categorized storage: docs/api/, docs/architecture/, docs/implementation/, docs/migration/

---

## Reference Documentation

- `CLAUDE.md` - Project architecture (Required reading for Claude Code)
- `AGENTS.md` - Development guidelines (Required reading for Codex)
- `mcp-tools-guide.md` - Tool capability inventory
- `PROJECT_COLLABORATION.md` - Complete collaboration documentation

---
> Source: [dunialabs/peta-core](https://github.com/dunialabs/peta-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
