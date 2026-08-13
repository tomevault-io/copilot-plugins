## editor-markdown

> Tiptap editor, slash menu, markdown paste, and task list behavior


# Editor & Markdown Rules

## Stack

- **Tiptap 2** + ProseMirror
- **lowlight** for code blocks
- **tiptap-markdown** for paste serialization (v0.8.x - peer compatibility)
- Custom extensions in `extension/src/editor/`

## Save path

```
Editor onUpdate → debounce (EDITOR_SAVE_DEBOUNCE_MS)
  → store updatePage → encode doc → IndexedDB
```

Never bypass store to write IDB directly from editor components.

## Markdown paste

Key modules:

- `markdownPaste.ts` - detection + conversion
- `taskListMarkdown.ts` - `- [x]` input rules
- `HighlightWithMarkdown.ts` - highlight syntax
- `tests/extension/editor/markdownPaste.test.ts` - regression tests

### Task list vs bullet list

`- [ ]` and `- [x]` must become **task items**, not bullet lists.

- Bullet input rule must **skip** lines matching `- [`
- Task item rule handles `- [x] ` explicitly
- Order of input rules matters - task before bullet

### Supported paste features

Headings, GFM task lists, tables, links, images, highlights, code fences.

## List Backspace

- `listBackspace.ts` - at the start of a bullet / ordered / task item, Backspace **lifts** to a normal paragraph (does not merge into the previous item)
- Registered in `editorExtensions.ts` via `ListBackspace`
- Tests: `tests/extension/editor/listBackspace.test.ts`

## Slash menu

- `SlashMenu.tsx` - command palette inside editor
- Add commands via existing extension patterns
- Keyboard-first UX - preserve ⌘/Ctrl bindings

## Toolbar

- `EditorToolbar.tsx` - inline formatting + attachment insert controls
- Color presets from `@/lib/constants` → `shared/constants.ts` (`EDITOR_TEXT_COLORS`, etc.) - never hardcode hex/labels in toolbar components

## Adding editor features

1. Check if Tiptap extension already exists in `package.json`
2. Register in editor setup file alongside peers
3. Add markdown paste handling if feature has text syntax
4. Add test case in `tests/extension/editor/markdownPaste.test.ts` for paste scenarios

## Manual verification

Paste from external sources (Notion export, GitHub markdown, VS Code) after changes.

---
> Source: [aryancodes-tech/my-memos](https://github.com/aryancodes-tech/my-memos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
