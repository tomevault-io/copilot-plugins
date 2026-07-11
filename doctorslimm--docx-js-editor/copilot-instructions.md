## docx-js-editor

> You are running inside a Ralph autonomous loop. Each iteration you must:

# Ralph Loop — Eigenpal DOCX Editor

## Your job

You are running inside a Ralph autonomous loop. Each iteration you must:

1. Read the current plan file in `.ralph/` (highest numbered `##_*.md` file).
2. Find the **first** unchecked task (`- [ ]`).
3. If all tasks are checked, output the exit signal and stop.
4. Research the ECMA-376 spec and existing code to understand the problem.
5. Implement ONLY that one task.
6. Run the **fast verify**: `bun run typecheck` (catches most errors in <5s)
7. Run **targeted tests only** - see "Test Strategy" below
8. If tests pass, mark task done, commit, update `.ralph/progress.txt`
9. Output RALPH_STATUS block.

```
RALPH_STATUS: {
  "status": "in_progress" | "complete",
  "current_task": "<task title>" | "none",
  "exit_signal": false | true
}
```

---

## Progress Tracking — So User Knows What's Happening

**UPDATE `.ralph/progress.txt` FREQUENTLY** — every 2-3 minutes minimum, or after each significant action. The user is watching this file to know what you're doing.

**Update after EACH of these actions:**

- Starting a new task
- Reading a key file (note which file and what you learned)
- Making an edit (note which file and what changed)
- Running a command (note result)
- Encountering an issue or insight
- Completing a task

Format (append to file):

```
[TIMESTAMP] TASK: <task title>
[TIMESTAMP] STATUS: starting | researching | reading | editing | running | testing | done | failed
[TIMESTAMP] NOTES: <brief description - include file names and specifics>
```

Example:

```
[2024-02-02 10:15] TASK: Fix getSelectionRange for cursor-only
[2024-02-02 10:15] STATUS: starting
[2024-02-02 10:16] STATUS: researching
[2024-02-02 10:16] NOTES: Checking ECMA-376 spec for selection handling
[2024-02-02 10:20] STATUS: reading
[2024-02-02 10:20] NOTES: Reading AIEditor.tsx:105
[2024-02-02 10:21] STATUS: editing
[2024-02-02 10:21] NOTES: Edited AIEditor.tsx - implemented paragraph expansion
[2024-02-02 10:22] STATUS: running
[2024-02-02 10:22] NOTES: Running typecheck - passed
[2024-02-02 10:23] STATUS: testing
[2024-02-02 10:23] NOTES: Tests pass - 11/11 cursor-only tests green
[2024-02-02 10:23] STATUS: done
```

**User can monitor progress with:**

```bash
tail -f .ralph/progress.txt
```

---

## Subagents — Use For Complex Tasks

You can spin up **subagents** for parallel work. Use the Task tool with appropriate agent types:

- **Explore agent** - For codebase exploration, finding files, understanding architecture
- **Plan agent** - For designing implementation approaches
- **Bash agent** - For running commands, git operations

**When to use subagents:**

- Searching across multiple files for a pattern
- Investigating how a feature works across the codebase
- Running parallel test suites
- Complex research tasks

**Example:**

```
Use Task tool with subagent_type="Explore" to find all files that handle selection
```

---

## SPEED OPTIMIZATIONS — Read This First

### Fast Verification Cycle

**DO NOT run the full test suite.** Run targeted tests only:

```bash
# Step 1: Type check (fast, catches 90% of issues)
bun run typecheck

# Step 2: Run ONLY the relevant test file(s)
npx playwright test tests/<relevant>.spec.ts --timeout=30000 --workers=4

# Step 3: If fixing a specific test, use --grep
npx playwright test --grep "test name pattern" --timeout=30000
```

### Test File Mapping

| Feature Area          | Test File                      | Quick Verify Pattern    |
| --------------------- | ------------------------------ | ----------------------- |
| Bold/Italic/Underline | `formatting.spec.ts`           | `--grep "apply bold"`   |
| Alignment             | `alignment.spec.ts`            | `--grep "align text"`   |
| Lists                 | `lists.spec.ts`                | `--grep "bullet list"`  |
| Colors                | `colors.spec.ts`               | `--grep "text color"`   |
| Fonts                 | `fonts.spec.ts`                | `--grep "font family"`  |
| Enter/Paragraphs      | `text-editing.spec.ts`         | `--grep "Enter"`        |
| Undo/Redo             | `scenario-driven.spec.ts`      | `--grep "undo"`         |
| Line spacing          | `line-spacing.spec.ts`         | `--grep "line spacing"` |
| Paragraph styles      | `paragraph-styles.spec.ts`     | `--grep "Heading"`      |
| Toolbar state         | `toolbar-state.spec.ts`        | `--grep "toolbar"`      |
| **Cursor-only ops**   | `cursor-paragraph-ops.spec.ts` | `--grep "cursor only"`  |

### Avoid Hanging

- **Never run all 500+ tests at once** unless explicitly validating final results
- Use `--timeout=30000` (30s max per test)
- Use `--workers=4` for parallel execution
- If a command takes >60s, Ctrl+C and retry with narrower scope
- Avoid `git log` with large outputs; use `--oneline -10`

---

## ECMA-376 Official Spec — Reference

The DOCX format is standardized as ECMA-376 / ISO-29500. Local reference docs:

```bash
# Quick reference
reference/quick-ref/wordprocessingml.md   # Paragraphs, runs, formatting
reference/quick-ref/themes-colors.md      # Theme colors, fonts, tints

# XML Schemas
reference/ecma-376/part1/schemas/wml.xsd      # WordprocessingML
reference/ecma-376/part1/schemas/dml-main.xsd # DrawingML (themes, colors)

# Full spec PDFs (5000+ pages)
reference/ecma-376/part1/*.pdf
```

**Online resources:**

- ECMA-376: https://ecma-international.org/publications-and-standards/standards/ecma-376/
- Microsoft Open Specs: https://learn.microsoft.com/en-us/openspecs/office_standards/ms-oe376/

---

## WYSIWYG Fidelity — Hard Rule

This is a WYSIWYG editor. Output must look identical to Microsoft Word.

**Must preserve:**

- **Fonts:** Custom/embedded fonts render correctly
- **Theme colors:** Theme slots (`dk1`, `lt1`, `accent1`) resolve to correct colors
- **Styles:** styles.xml definitions apply (headings, body, character styles)
- **Character formatting:** Bold, italic, font size/family/color, highlight, underline, strikethrough
- **Tables:** Borders, cell shading, merged cells
- **Headers/footers:** Render on each page
- **Section layout:** Margins, page size, orientation

---

## ProseMirror Editor Architecture

The editor uses **ProseMirror** as the editing core.

### Key Files

```
src/prosemirror/
├── schema/
│   ├── nodes.ts          # paragraph, table, image, hardBreak, tab nodes
│   ├── marks.ts          # bold, italic, underline, color, font, hyperlink
│   └── index.ts          # Schema export
├── conversion/
│   ├── toProseDoc.ts     # Document → ProseMirror (with style resolution)
│   └── fromProseDoc.ts   # ProseMirror → Document (round-trip)
├── plugins/
│   ├── keymap.ts         # Keyboard shortcuts (Enter, Backspace, Tab, etc.)
│   └── selectionTracker.ts # Emits selection context for toolbar
├── commands/
│   ├── formatting.ts     # toggleBold, setFontSize, setTextColor, etc.
│   ├── paragraph.ts      # setAlignment, setLineSpacing, applyStyle
│   └── lists.ts          # toggleBulletList, indent/outdent
├── styles/
│   ├── styleResolver.ts  # OOXML style chain resolution
│   └── index.ts          # Exports createStyleResolver
├── utils/
│   └── tabCalculator.ts  # Tab width calculation utilities
├── ProseMirrorEditor.tsx # React wrapper component
└── editor.css            # Editor styling
```

### Supported Features

| Feature                 | Status | Notes                         |
| ----------------------- | ------ | ----------------------------- |
| Bold/Italic/Underline   | ✅     | Marks with proper toDOM       |
| Strikethrough           | ✅     | Single and double strike      |
| Superscript/Subscript   | ✅     | Mutually exclusive            |
| Text color              | ✅     | RGB and theme colors          |
| Highlight               | ✅     | Word highlight colors         |
| Font size               | ✅     | Half-points to pt conversion  |
| Font family             | ✅     | ASCII and hAnsi fonts         |
| Paragraph alignment     | ✅     | Left, center, right, justify  |
| Line spacing            | ✅     | Single, 1.5, double, exact    |
| Indentation             | ✅     | Left, right, first-line       |
| Lists (bullet/numbered) | ✅     | With indent levels            |
| Paragraph styles        | ✅     | Style resolution from OOXML   |
| Tables                  | ✅     | Basic editing, Tab navigation |
| Images                  | ✅     | Inline images                 |
| Hyperlinks              | ✅     | With href and tooltip         |
| Undo/Redo               | ✅     | ProseMirror history           |

### Known Limitations

- **Tab stops**: Uses fixed 0.5 inch width (full dynamic positioning deferred)
- **Table editing**: Basic navigation only (no row/column insert/delete yet)
- **Headers/Footers**: Rendered but not editable
- **Cut/Paste**: Some edge cases with cross-browser clipboard

---

## Known Issues (Current)

### 1. End Key Navigation

The End key doesn't always move to the end of the line in all scenarios.

### 2. Cut/Paste Edge Cases

Some clipboard operations may not work correctly in all browsers.

### 3. Complex Table Operations

Row/column insertion, cell merging, and table deletion require prosemirror-tables integration.

---

## Verify Commands

**Fast cycle (use this 95% of the time):**

```bash
bun run typecheck && npx playwright test --grep "<pattern>" --timeout=30000 --workers=4
```

**Single test file:**

```bash
bun run typecheck && npx playwright test tests/formatting.spec.ts --timeout=30000
```

**Cursor-only operations (new critical tests):**

```bash
npx playwright test tests/cursor-paragraph-ops.spec.ts --timeout=30000 --workers=4
```

**Full suite (only for final validation):**

```bash
bun run typecheck && npx playwright test --timeout=60000 --workers=4
```

---

## Rules

- **Screenshots:** Save to `screenshots/` folder
- Work on exactly ONE task per iteration
- Do NOT modify other tasks in the plan
- Do NOT delete files from previous tasks unless required
- Client-side only. No backend.
- No collaboration, comments, tracked changes, or PDF export

---

## Project Context

Minimal Bun + React (TSX) app for EigenPal:

1. **Display DOCX** — render with full WYSIWYG fidelity per ECMA-376 spec
2. **Insert docxtemplater variables** — `{variable}` mappings with live preview (standard docxtemplater syntax)

Target users: Non-technical clients at European banks/insurance companies.

---

## Commit Message Format

```bash
git commit -m "$(cat <<'EOF'
feat: <task title>

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

---

## When Stuck

1. **Type error?** Read the actual types, don't guess
2. **Test failing?** Run with `--debug` and check console output
3. **Selection bug?** Add `console.log` in `getSelectionRange()` to trace
4. **OOXML spec question?** Check `reference/quick-ref/` or ECMA-376 schemas
5. **Timeout?** Kill command, narrow test scope, retry
6. **Complex task?** Spin up a subagent with Task tool

---
> Source: [DoctorSlimm/docx-js-editor](https://github.com/DoctorSlimm/docx-js-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
