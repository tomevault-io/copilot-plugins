## pubkey

> Guidelines for using notes with todo.ai tasks

When creating or working with todo.ai tasks, add notes to provide implementation context. Do not use notes for status updates on parent tasks.

**Command:** `./todo.ai note <task-id> "Your note here"`

**Examples:**
- ✅ `./todo.ai note 42 "Modify archive_task() lines 2600-2850. Pattern from delete_task() lines 2926-2935"`
- ✅ `./todo.ai note 42.1 "Depends on task#42 completing first - needs collect_notes() function"`
- ✅ `./todo.ai note 50 "See docs/CODE_SIZE_ANALYSIS.md lines 66-67 for migration cleanup details"`
- ❌ `./todo.ai note 42 "Fix it"` - Too vague, no actionable context

Notes appear as blockquotes (>) and move with tasks when archived.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fxstein) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
