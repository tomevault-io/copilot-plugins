## extension-architecture

> Browser extension and web demo - Zustand, views, platform, components


# Extension Architecture

## Entry points

| Mode | HTML | Vite config | Output |
|------|------|-------------|--------|
| Browser MV3 | `newtab.html` | `vite.config.ts` | `extension/dist/` |
| Web demo | `index.html` | `vite.web.config.ts` | `public/demo/` |

Manifest source of truth: `extension/manifest.config.ts` (CRXJS).

## Platform branching

```typescript
// extension/src/lib/platform.ts
isExtensionContext()  // chrome.runtime?.id
isWebAppContext()     // standalone /demo SPA
```

- **Extension:** `chrome.storage.local` for settings
- **Web:** `localStorage` for settings
- **Both:** IndexedDB for pages (separate origins → separate data)

Web-only components: `WebInstallBanner`, `MobileExperienceNotice` (viewport < 768px).

## Zustand store (`useStore.ts` facade + slices)

Call sites keep importing `useStore` from `store/useStore.ts`. Internally it composes slices:

| Slice | Owns |
|-------|------|
| `slices/pagesWorkspace.ts` | pages CRUD, view, workspace move/delete + OPFS GC |
| `slices/themeUi.ts` | theme, custom themes, `applyThemeToDocument` |
| `slices/dialogs.ts` | delete / link / attachment-delete flags (serializable payloads only) |
| `slices/editorBridge.ts` | `pageEditor` + `applyLink` / `removeLink` / confirm attachment delete |

Do **not** put TipTap `Editor` types or `onConfirm` closures in the pages/dialogs slices. Prefer exported **selectors** for derived lists.

## Views

- `Dashboard` - recent pages, quick create
- `PageView` - title + editor, debounced save

View type: `{ kind: "dashboard" }` | `{ kind: "page"; id: string }`

## Components conventions

- `Sidebar.tsx` - workspace tree UI; DnD helpers live in `lib/workspaceDrag.ts`
- `SearchPalette.tsx` - ⌘K, FlexSearch (ephemeral index)
- `ThemeDropdown.tsx` - built-in + custom themes
- Dialogs - delete confirm, etc.

CSS: `ko-` classes, theme via `data-theme` + `--ko-*` variables from `extension/src/styles/`.
**Dark / multi-theme:** never hardcode light-only colors (`white`, `#fff`, mixes with literal white). Use `--ko-bg` / `--ko-surface` / `--ko-surface-2` / `--ko-text` / `--ko-text-muted` / `--ko-border` / `--ko-accent`. Verify new UI in both a light and a dark theme (see `AGENTS.md` §3.7).

## Keyboard shortcuts

Defined in `App.tsx` - check existing bindings before adding new global listeners.

## Build guards

- `guard-production-build.mjs` - blocks prod build during dev
- `clean-dev-artifacts.mjs` - clears stale `dist/`
- Use `npm run dev:reset --prefix extension` if HMR breaks

## Tests

Mirrored under `tests/extension/` (see `tests/README.md`).  
`extension/src/<path>/<file>.ts` → `tests/extension/<path>/<file>.test.ts`.

```bash
npm run test -- tests/extension/store/
npm run test -- tests/extension/lib/attachments/
```

## Attachments vs editor commands (dependency rule)

**`lib/attachments/`** = OPFS I/O + policy only (no TipTap types):

```
lib/attachments/
├── attachmentManager.ts   ← save/read/delete, object URL cache
├── fileSystemManager.ts   ← OPFS root + subdirs (images/, audio/)
├── sanitizeBlockDoc.ts    ← persist sanitizer + ref-count GC helpers
├── voiceRecorder.ts       ← MediaRecorder wrapper + live levels
├── waveform.ts            ← decode peaks for playback UI
├── imageClipboard.ts      ← paste/drop DataTransfer → image Files
└── imageFiles.ts          ← pure image File checks
```

**`editor/commands/`** = TipTap insert/paste adapters that call attachment services:

```
editor/commands/
├── insertImage.ts
├── insertAudioFromFile.ts
├── insertVoiceRecording.ts
└── insertSelection.ts
```

Node views use hooks under `editor/hooks/` (`useVoiceNoteRecording`, `useVoiceNotePlayback`, `useAttachmentImage*`).
Editor paste/drop: `imagePasteDrop.ts` (priority above markdown paste).
Dialogs: `AttachmentDeleteDialog` confirms via store `pendingAttachmentDelete` (path only).

## Constants

**Canonical source:** `shared/constants.ts` (see `.cursor/rules/constants-policy.mdc` and `AGENTS.md` §3.8).

- `extension/src/lib/constants.ts` is a re-export only - edit `shared/constants.ts`
- UI labels, error messages, attachment/OPFS paths, MIME types, debounce/limits → named exports there
- Import via `@/lib/constants` - do not hardcode copy in components or attachment helpers
- Do not add parallel string modules

---
> Source: [aryancodes-tech/my-memos](https://github.com/aryancodes-tech/my-memos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
