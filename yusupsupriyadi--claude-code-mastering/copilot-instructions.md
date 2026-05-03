## claude-code-mastering

> **MANDATORY**: All user communication in **BAHASA INDONESIA**

# CLAUDE.md - Claude Code Rules

## 🎯 CORE RULES

### 1. COMMUNICATION LANGUAGE

**MANDATORY**: All user communication in **BAHASA INDONESIA**

- Explanations, feedback, questions → Bahasa Indonesia
- Only code and technical commands → English

### 2. THINK FIRST, THEN ACT

**ALWAYS analyze before execution:**

- Understand context and goals
- Identify required steps
- Verify approach before implementation
- Use TodoWrite for complex tasks (>3 steps)

---

## 🔧 MCP TOOLS (MANDATORY)

This project uses 3 MCP servers. All are mandatory and must be used according to their purpose.

### 1. Sequential Thinking — Analysis & Planning

**MUST use before any action.** This is the thinking engine for all tasks.

**When to use:**
- **START of every request** → Analyze what user is asking
- **BEFORE coding** → Plan implementation in detail
- **DEBUG/FIX** → Analyze root cause step-by-step
- **COMPLEX LOGIC** → Break down algorithms or flows
- **REFACTORING** → Analyze impact of changes

**How to use:**
1. Call `sequentialthinking` with initial analysis
2. Use multiple thoughts (2-6) to explore the problem
3. Revise thoughts if understanding changes
4. Verify solution before executing
5. Set `nextThoughtNeeded: false` when analysis is complete

**Example — Bug Fix:**
```
Thought 1: Analyze error message and context
Thought 2: Identify root cause
Thought 3: Explore alternative solutions
Thought 4: Choose best solution and verify impact
```

**Example — New Feature:**
```
Thought 1: Understand user requirements
Thought 2: Analyze existing codebase structure
Thought 3: Design architecture approach
Thought 4: Break down implementation steps
Thought 5: Consider edge cases
Thought 6: Plan testing strategy
```

### 2. Serena — Semantic Code Intelligence

**PRIMARY tool for all code operations.** Uses LSP for precise, type-aware code manipulation.

**When to use:**
- **Project onboarding** → `onboarding()` to understand structure
- **Code navigation** → `find_symbol`, `get_symbols_overview`, `find_referencing_symbols`
- **Code editing** → `replace_symbol_body`, `insert_after_symbol`, `insert_before_symbol`, `replace_content`
- **Refactoring** → `rename_symbol` for safe cross-codebase renames
- **Memory/context** → `write_memory`, `read_memory`, `list_memories` for session persistence
- **File operations** → `read_file`, `create_text_file`, `find_file`, `list_dir`
- **Search** → `search_for_pattern` for flexible pattern matching

**Key workflow:**
1. **Start session** → `list_memories()` + `read_memory()` to load previous context
2. **Navigate code** → `find_symbol` + `get_symbols_overview` (avoid reading entire files)
3. **Understand impact** → `find_referencing_symbols` before editing
4. **Edit precisely** → Use symbol-level tools for structural edits, `replace_content` for line-level changes
5. **End session** → `write_memory()` to save important context

**Memory naming convention:** `descriptive_name_YYYY_MM` (e.g., `auth_system_2025_01`, `project_structure`)

**Memory rules:**
- ALWAYS load memories FIRST before starting work
- ALWAYS save memories AFTER completing significant work
- Keep memories CONCISE — focus on key points
- Use `edit_memory()` to update, `delete_memory()` to clean up old ones

### 3. Context7-MCP — Library Documentation

**MUST use when working with external libraries/frameworks.** Fetches up-to-date documentation directly.

**When to use:**
- Need API docs for any library (React, FastAPI, SQLAlchemy, Prisma, Next.js, etc.)
- Looking for best practices or recommended patterns
- Checking configuration options or defaults
- Need code examples for specific features
- Working with unfamiliar library versions

**How to use (2-step process):**
1. **Resolve library ID first:**
   ```
   resolve-library-id(libraryName="react")
   ```
2. **Then fetch docs with topic:**
   ```
   get-library-docs(context7CompatibleLibraryID="/facebook/react", topic="hooks useEffect cleanup")
   ```

**IMPORTANT:** Always resolve the library ID first. Never guess the ID.

---

## ⚡ EXECUTION WORKFLOW

### Simple Tasks (1-3 steps):

1. **LOAD CONTEXT** → `list_memories()` + `read_memory()` from Serena
2. **THINK** → Sequential Thinking for analysis
3. **LOOKUP** → Context7 if external libraries involved
4. **EXECUTE** → Serena tools for code operations
5. **VERIFY** → Check results
6. **SAVE** → `write_memory()` to Serena

### Complex Tasks (>3 steps):

1. **LOAD CONTEXT** → `list_memories()` + `read_memory()` from Serena
2. **THINK** → Sequential Thinking (4-10 thoughts)
3. **LOOKUP** → Context7 for relevant library docs
4. **PLAN** → Create todo list with TodoWrite
5. **EXECUTE** → Work step by step with Serena
6. **VERIFY** → Ensure all completed
7. **SUMMARIZE** → Work result summary
8. **SAVE** → `write_memory()` to Serena

---

## 📝 WORK SUMMARY (MANDATORY)

**EVERY task MUST end with summary in Bahasa Indonesia:**

```
📊 RINGKASAN KERJA:

✅ Yang Dikerjakan:
- [List detail apa yang diselesaikan]
- [Perubahan yang dibuat]

📁 File yang Terpengaruh:
- [path]: [deskripsi perubahan]

📌 Hasil Akhir:
[Ringkasan singkat hasil dan status]

⚠️ Catatan Penting (jika ada):
[Hal yang perlu diperhatikan user]
```

**When to provide:** After completing task, after bug fix, after refactoring, after feature implementation, or when blocked.

---

## ✅ WORKING PRINCIPLES

**ALWAYS:**
- Load Serena memories BEFORE starting any task
- Use Sequential Thinking FIRST for analysis
- Use Context7-MCP for external library documentation
- Communicate in Bahasa Indonesia
- Verify results before finishing
- Provide work summary after completion
- Save context to Serena memory after task

**NEVER:**
- Skip Sequential Thinking analysis
- Guess library APIs without checking Context7
- Read entire files when symbol-level tools suffice
- Skip work summary
- Skip saving context to memory
- Create new files without strong reason
- Commit without user request

---

## 🚫 GIT RULES

**FORBIDDEN in commit messages and PR descriptions:**
- ❌ "🤖 Generated with Claude Code"
- ❌ "Co-Authored-By: Claude <noreply@anthropic.com>"
- ❌ Any AI-related references or robot emojis

**GIT ADD:**
- ✅ Always `git add` per file, not `git add -A` or `git add .`
- ✅ Group only directly related files

---

_All rules above are mandatory for every interaction._

---
> Source: [yusupsupriyadi/claude-code-mastering](https://github.com/yusupsupriyadi/claude-code-mastering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
