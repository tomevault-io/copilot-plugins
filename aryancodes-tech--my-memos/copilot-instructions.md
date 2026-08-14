## my-memos

> > **Audience:** Cursor agents, Claude Code, Copilot Workspace, and human reviewers evaluating AI-assisted engineering depth (e.g. Forward Deployed Engineer workflows).

# AGENTS.md - MyMemos AI Operating Manual

> **Audience:** Cursor agents, Claude Code, Copilot Workspace, and human reviewers evaluating AI-assisted engineering depth (e.g. Forward Deployed Engineer workflows).
>
> **Product:** **MyMemos** - local-first personal knowledge OS that replaces your browser's New Tab. Treat **MyMemos** / `mymemos` as canonical.

This document is the **source of truth** for how an AI agent should reason about, modify, and verify this codebase. It complements human docs (`README.md`, `CONTRIBUTING.md`) with machine-oriented invariants, decision trees, and verification contracts.

---

## 1. Repository topology (three surfaces, one product)

```
┌──────────────────────────────────────────────────────────────────────────┐
│ SURFACE A - Landing site (`src/`, TanStack Start, React 19, Tailwind 4) │
│   Routes: `/` (marketing + download), SSR via Nitro                      │
│   Constants: `@/lib/constants` → `shared/constants.ts`                 │
│   Do NOT import extension code directly                                │
└───────────────────────────────┬──────────────────────────────────────────┘
                                │ `/demo/` embeds built SPA
┌───────────────────────────────▼──────────────────────────────────────────┐
│ SURFACE B - Web demo (`extension/` → `public/demo/`, React 19)           │
│   Entry: `extension/index.html`, build: `vite.web.config.ts`             │
│   Settings: `localStorage` · Pages: IndexedDB · Attachments: OPFS        │
│   (separate origin from extension - data does not sync)                │
└───────────────────────────────┬──────────────────────────────────────────┘
                                │ same `extension/src/` source
┌───────────────────────────────▼──────────────────────────────────────────┐
│ SURFACE C - Browser extension (`extension/` → `dist/`, MV3 + CRXJS)     │
│   Entry: `extension/newtab.html`, overrides New Tab                    │
│   Settings: `chrome.storage.local` · Pages: IndexedDB · Attachments: OPFS│
└──────────────────────────────────────────────────────────────────────────┘
```

### Path alias warning

Aliases resolve differently depending on which `tsconfig` is active:

| Package        | Alias `@/` →      | Alias `@shared/` → |
| -------------- | ----------------- | ------------------ |
| Root / landing | `src/*`           | `shared/*`         |
| Extension      | `extension/src/*` | `../shared/*`      |

Before adding imports, confirm which package you are editing. Product constants: edit `shared/constants.ts` only.

### Generated artifacts - never hand-edit

| Path                                                         | Produced by                                                 |
| ------------------------------------------------------------ | ----------------------------------------------------------- |
| `public/demo/**`                                             | `npm run build:app` (extension web build)                   |
| `public/mymemos-extension.zip`                               | `npm run package:extension`                                 |
| `public/robots.txt`, `public/sitemap.xml`, `public/llms.txt` | `npm run generate:seo` (also `predev:web` / `prebuild:web`) |
| `src/routeTree.gen.ts`                                       | TanStack Router codegen                                     |
| `extension/dist/**`                                          | `npm run dev` or `npm run build:extension`                  |
| `.output/**`, `dist/**` (root)                               | `npm run build:web`                                         |

---

## 2. Mandatory pre-change protocol

Every non-trivial agent session **must** follow this sequence before writing code:

### Phase A - Classify the task

| Task class               | Examples                                   | Primary docs                                                      |
| ------------------------ | ------------------------------------------ | ----------------------------------------------------------------- |
| **Extension product**    | Sidebar, editor, storage, themes, search   | `extension/README.md`, `.cursor/rules/extension-architecture.mdc` |
| **Landing / marketing**  | Hero, scroll video, download flow          | `src/routes/README.md`, `.cursor/rules/landing-site.mdc`          |
| **Landing SEO / GEO**    | Meta tags, FAQ schema, `llms.txt`, sitemap | `src/lib/seo.ts`, `.cursor/SKILLS.md` → `landing-seo`             |
| **Build / CI / tooling** | Vite, workflows, scripts                   | `package.json`, `.cursor/rules/testing-ci.mdc`                    |
| **Cross-cutting**        | Constants, naming, security                | This file §3–§5                                                   |

### Phase B - Trace data flow

Ask explicitly:

1. **Where does state live?** (Zustand vs IndexedDB vs chrome.storage/localStorage)
2. **What gets persisted?** (Block JSON only - see §3.1)
3. **Which surface(s) are affected?** (extension-only, web demo, landing, or build pipeline)
4. **Is there an existing pattern?** (grep siblings before inventing abstractions)

### Phase C - Scope the diff

- **Minimize blast radius.** One logical change per PR.
- **Match conventions** in the nearest sibling file.
- **Constants first.** Before adding any user-facing string, error message, label, path segment, MIME type, debounce, or other tunable: open `shared/constants.ts`, reuse or add a JSDoc'd export, then import via `@/lib/constants`. Never leave new literals in components. See §3.8 and `.cursor/rules/constants-policy.mdc`.
- **Empty strings:** use `len(value) === 0` in extension code (never `!value` for strings).

### Phase D - Verify

| Change type        | Minimum verification                                                                                |
| ------------------ | --------------------------------------------------------------------------------------------------- |
| Extension UI/logic | `npm run test` + manual new-tab check                                                               |
| Storage/schema     | `tests/extension/storage/` + migration review                                                       |
| Landing            | `npm run dev:web` + visual check at `/`                                                             |
| Landing SEO        | `npm run test -- tests/landing/lib/seo.test.ts` + `curl` `/robots.txt`, `/sitemap.xml`, `/llms.txt` |
| Build/scripts      | `npm run ci`                                                                                        |

**CI contract:** `npm run ci` (`check` + builds) must pass before merge. It mirrors `.github/workflows/ci.yml`.

---

## 2.5 Product capabilities (shipped vs not)

Document only what users can do today. Schema fields or storage helpers without UI are **not** product features.

### Shipped (extension + web demo)

| Area               | Capabilities                                                                                                                                                                              |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Workspace          | Nested pages & folders, drag-and-drop reparent/reorder, inline rename, delete with descendant warning, collapsible sidebar & folders                                                      |
| Favorites / Recent | Star/unstar pages (`favorite` boolean persisted); Favorites and Recent are sidebar **views** (not separate sections)                                                                      |
| Dashboard          | Recent pages list, quick-create page/folder                                                                                                                                               |
| Editor             | Tiptap blocks: headings H1–H4, bullet/ordered/task lists, blockquote, HR, tables, code blocks (lowlight); inline bold/italic/underline/strike/code/link; text/highlight colors; alignment |
| Slash + toolbar    | `/` slash menu; formatting toolbar; voice note, audio file, image insert                                                                                                                  |
| Markdown paste     | GFM-oriented paste (headings, task lists, tables, links, images, highlights, code fences)                                                                                                 |
| Images             | Picker, drag-drop, paste file/screenshot/webpage `<img>` → OPFS; align, caption, lightbox, replace, copy, alt text, delete                                                                |
| Voice notes        | Inline record (mic on start), pause/resume, waveform playback + speed cycle, rename, download, delete; attach existing audio file                                                         |
| Search             | ⌘K FlexSearch over **title + body text** (in-memory; rebuilt on demand)                                                                                                                   |
| Themes             | 7 built-ins + user custom themes (persisted in settings tier)                                                                                                                             |
| Persistence        | Debounced autosave; BlockDoc JSON in IndexedDB (`doc_c`); settings in chrome.storage / localStorage; binaries in OPFS                                                                     |

### Landing site

| Area      | Capabilities                                                                                                                                      |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Marketing | Hero, scroll-linked launch video (Cloudinary-hosted), feature bento (CSS mockups; per-feature clips gated off by `LANDING_FEATURE_CLIPS_ENABLED`) |
| Install   | ZIP download (`mymemos-extension.zip`), get-started steps, FAQ accordion                                                                          |
| Demo      | `/demo/` embeds the web SPA (separate origin from the extension)                                                                                  |
| SEO / GEO | Meta + OG/Twitter, JSON-LD, `robots.txt`, `sitemap.xml`, `/llms.txt`                                                                              |

### Present in code but **not** user-facing today

Do **not** claim these as product features in README, FAQ, SEO, or agent summaries:

| Item                                      | Reality                                                           |
| ----------------------------------------- | ----------------------------------------------------------------- |
| `Page.tags`                               | Schema + FlexSearch index field; **no UI** to add/edit tags       |
| `Page.archived`                           | Schema + filtered from selectors; **no UI** to archive            |
| `exportWorkspace` / `importWorkspace`     | Implemented + tested in `db.ts`; **no UI** entry point            |
| Cloud sync, accounts, backlinks/wikilinks | Out of scope / roadmap only (`extension/README.md` Roadmap-ready) |

---

## 3. Architectural invariants (non-negotiable)

### 3.1 Storage contract

```
┌─────────────────────────────────────────────────────────────┐
│ IndexedDB (`mymemos`)                                       │
│   pages  → LZ-compressed BlockDoc (`doc_c` field)           │
│   images → legacy blob store (no longer written; OPFS for attachments) │
├─────────────────────────────────────────────────────────────┤
│ OPFS (`mymemos-attachments/`) - per-origin, hidden          │
│   images/  → img_*.png                                      │
│   audio/   → voice_*.webm                                   │
│   (paths referenced from block attrs only)                  │
├─────────────────────────────────────────────────────────────┤
│ Settings tier (chrome.storage.local OR localStorage)        │
│   theme, lastView, customThemes, collapsedDirs,             │
│   productTour, demoWorkspaceSeeded                          │
├─────────────────────────────────────────────────────────────┤
│ Ephemeral (never persisted)                                 │
│   FlexSearch index, UI-only state, drag refs                │
│   voiceNote blocks with status "recording"                  │
└─────────────────────────────────────────────────────────────┘
```

**Rules:**

- Persist **ProseMirror/Tiptap block JSON** (`BlockDoc`) only. Never store rendered HTML or parallel markdown copies.
- On decode failure: fall back to `EMPTY_BLOCK_DOC`, `console.warn`, do not throw into UI.
- Schema changes require `DB_VERSION` bump and non-destructive upgrade path in `extension/src/storage/db.ts`.
- Search index rebuilds on demand - do not write FlexSearch to disk.
- **Attachments:** binary files in OPFS; block attrs hold `attachmentPath` + metadata. Run `sanitizeBlockDocForPersistence()` before save. Page delete must GC orphaned OPFS files (`collectOrphanedAttachmentPaths`).

### 3.2 Platform abstraction

`extension/src/lib/platform.ts`:

- `isExtensionContext()` → `typeof chrome?.runtime?.id === "string"`
- `isWebAppContext()` → negation

Branch settings I/O only - page data path is shared (IndexedDB). Web and extension data **do not sync** (different origins).

### 3.3 State management

Single Zustand facade: `extension/src/store/useStore.ts` (composes slices under `store/slices/`).

| Concern                                                | Owner                                          |
| ------------------------------------------------------ | ---------------------------------------------- |
| Pages, CRUD, workspace tree                            | `slices/pagesWorkspace.ts`                     |
| View routing (`dashboard` \| `page`)                   | pages slice + `lastView` setting               |
| Theme                                                  | `slices/themeUi.ts` + `data-theme` on `<html>` |
| Dialogs (serializable payloads)                        | `slices/dialogs.ts`                            |
| TipTap bridge (`pageEditor`, links, attachment delete) | `slices/editorBridge.ts`                       |
| Product tour coachmarks                                | `slices/onboarding.ts` + `onboarding/`         |
| Editor debounced saves                                 | `PageView` + `EDITOR_SAVE_DEBOUNCE_MS`         |

Prefer **selectors** (`selectSearchablePages`, etc.) for derived data. Do not duplicate tree logic in components. Do not store `onConfirm` closures in dialog state.

### 3.3.1 Attachments vs editor layering

- **`lib/attachments/`** - OPFS I/O, sanitize, recorder, waveform (no TipTap imports).
- **`editor/commands/`** - TipTap insert helpers (`insertImage`, `insertAudioFromFile`, `insertVoiceRecording`, `insertSelection`).
- **`editor/hooks/`** - fat node-view logic (voice record/playback, image chrome).
- **`lib/workspaceDrag.ts`** - sidebar workspace DnD session helpers.

### 3.3.2 Product tour + web-demo seed

- Coachmark tour (`extension/src/onboarding/`) auto-starts once when `SETTINGS_KEYS.productTour` is unset; Skip/Done persist `PRODUCT_TOUR_STATUS_DONE`. Replay via header **Tour** next to the theme control.
- Tour covers create page, **workspace drag-and-drop**, slash menu, image, and voice. Steps that need the editor navigate to a page first; create-page and workspace-drag prefer the dashboard. Spotlight anchors use `data-tour-target` (`PRODUCT_TOUR_TARGETS`).
- Fresh empty workspaces seed sample pages/folders (`maybeSeedDemoWorkspace`) and set `SETTINGS_KEYS.demoWorkspaceSeeded` so later empties do not reseed. This applies to both the web demo and a fresh extension install.

### 3.4 Workspace model

- `Page.kind`: `"page"` | `"directory"`
- `Page.section`: workspace lives under `WORKSPACE_SECTION` (`"Pages"`)
- `parent_id`: `""` (root) - check with `len(parent_id) === 0`
- Drag-and-drop uses `WORKSPACE_DRAG_MIME`; validate moves via `workspaceTree.ts` helpers
- Favorites / Recent are sidebar **views**, not stored sections. The `favorite` flag is persisted on `Page`; Recent is derived from timestamps.
- `tags` and `archived` exist on the schema for forward-compat / filtering - no product UI today (see §2.5).

### 3.5 Editor pipeline

```
Paste / typing / image drop → Tiptap input rules → ProseMirror doc
                ↓
         markdown paste + image paste/drop layers
                ↓
         sanitizeBlockDocForPersistence → debounced save → encode → IndexedDB
         (attachment binaries → OPFS; paths in block attrs)
```

Markdown paste: `extension/src/editor/markdownPaste.ts` + tests in `tests/extension/editor/markdownPaste.test.ts`. Task lists (`- [x]`) must not be swallowed by bullet-list rules. Image paste/drop: `imagePasteDrop.ts` (priority above markdown paste).

### 3.6 Landing SEO & AI discoverability

```
┌─────────────────────────────────────────────────────────────┐
│ Content source: src/lib/ai-content.json (FAQ, llms summary) │
├─────────────────────────────────────────────────────────────┤
│ Runtime helpers: src/lib/seo.ts, landingFaqContent.ts     │
├─────────────────────────────────────────────────────────────┤
│ Static output (gitignored): public/robots.txt, sitemap.xml, │
│   llms.txt ← scripts/generate-sitemap.mjs                   │
├─────────────────────────────────────────────────────────────┤
│ Dynamic route: src/routes/llms[.]txt.ts → GET /llms.txt     │
└─────────────────────────────────────────────────────────────┘
```

**Rules:**

- Set `VITE_SITE_URL` at deploy time (no trailing slash; prefer `https://www…`). Drives canonical URLs, JSON-LD, FAQ demo links, and generated static SEO files.
- Apex / HTTP hosts must **301/308** to that origin (`vercel.json` or host-equivalent). GSC "Page with redirect" on apex is expected - only the canonical host should be indexed.
- FAQ link segments in `ai-content.json` use `"path": "/demo/"` - never hardcode production domains in JSON.
- Keep `scripts/generate-sitemap.mjs` and `buildLlmsTxt()` in `seo.ts` structurally aligned.
- `/demo/` stays `Disallow` in `robots.txt` (`SEO_ROBOTS_DISALLOW_PATHS`).

**Verify:**

```bash
npm run generate:seo
npm run test -- tests/landing/lib/seo.test.ts
npm run dev:web
curl -s http://localhost:8080/llms.txt | head
```

### 3.7 Themes & dark mode (extension UI)

The extension / web demo ships **multiple themes** (including dark presets like `dark`, `midnight`, `dracula`, `forest`, `ocean`). Theme is applied via `data-theme` on `<html>` and CSS variables in `extension/src/styles/index.css` (`--ko-bg`, `--ko-surface`, `--ko-surface-2`, `--ko-border`, `--ko-text`, `--ko-text-muted`, `--ko-accent`, …).

**Rules for any new or restyled UI:**

- Prefer **`--ko-*` tokens only** for colors, borders, and backgrounds. Do not hardcode `#fff` / `#000` / `white` / `black` (or light-only mixes like `color-mix(..., white 96%)`) in extension CSS/components — they break dark themes.
- Soft fills: mix against `var(--ko-bg)`, `var(--ko-surface)`, or `var(--ko-surface-2)`, not literal white.
- Text: `var(--ko-text)` / `var(--ko-text-muted)` only.
- When adding a component or CSS block, **verify at least one light theme and one dark theme** (e.g. Light + Dark or Midnight) in `npm run dev` / `npm run dev:web`.
- Custom themes reuse the same `--ko-*` variables (`slices/themeUi.ts`); token-based styles stay compatible automatically.

### 3.8 Constants contract (shared module)

Agents **must** read `.cursor/rules/constants-policy.mdc` before changing copy, labels, errors, or tunables.

| Role                 | Path                                                                   |
| -------------------- | ---------------------------------------------------------------------- |
| **Canonical source** | `shared/constants.ts`                                                  |
| Landing re-export    | `src/lib/constants.ts` (`export * from "@shared/constants"`)           |
| Extension re-export  | `extension/src/lib/constants.ts` (`export * from "@shared/constants"`) |

**Rules:**

- Edit **`shared/constants.ts` only**. Do not add constants to the re-export files. Do not add `strings.ts`, `messages.ts`, `copy.ts`, or inline user-facing literals in components.
- App code imports via `@/lib/constants` (preferred) or `@shared/constants`.
- Exceptions: FAQ/llms body → `src/lib/ai-content.json`; SEO builders → `src/lib/seo.ts`; build-only knobs → Vite/manifest config; pure CSS tokens → stylesheets; theme type shapes → `shared/themeTypes.ts`.

```typescript
// ✅
import { DEFAULT_FOLDER_TITLE, MICROPHONE_DENIED_MESSAGE } from "@/lib/constants";

// ❌
title: "Untitled folder";
throw new Error("Microphone access was denied…");
```

---

## 4. Decision trees

### 4.1 "Where do I put this constant?"

```
Is it FAQ / llms.txt / AI crawler body copy?
  YES → src/lib/ai-content.json (+ resolve links in landingFaqContent.ts)
 NO ↓
Is it SEO meta, JSON-LD, robots, or sitemap builders?
  YES → src/lib/seo.ts (+ scripts/generate-sitemap.mjs for static files)
 NO ↓
Is it a product constant (copy, label, error, path, MIME, debounce, theme label, marketing string)?
  YES → shared/constants.ts  (import via @/lib/constants)
 NO ↓
Is it build-time only (Vite, manifest)?
  YES → colocate in config with a short comment
 NO ↓
STOP - do not hardcode in a component.
```

### 4.2 "Which dev server do I run?"

```
Editing extension UI (Sidebar, Editor, Store)?
  → npm run dev
  → Load unpacked from extension/dist/
  → NEVER npm run build:extension during active dev (kills HMR)

Editing landing page?
  → npm run dev:web  (http://localhost:8080)
  → Includes /demo/ via webAppDevPlugin.ts

Editing web demo only (no landing)?
  → npm run dev:app
```

### 4.3 "Do I need a test?"

```
Touches storage encode/decode, workspace move rules, markdown paste,
text helpers, or store invariants?
  YES → add/update mirrored test under tests/ (see mapping below)
  NO ↓
Pure UI tweak with no logic change?
  → manual verification sufficient unless user requests tests
```

**Mapping:** `extension/src/<path>/<file>.ts` → `tests/extension/<path>/<file>.test.ts`; landing `src/<path>/<file>.ts` → `tests/landing/<path>/<file>.test.ts`. See [`tests/README.md`](tests/README.md).

---

## 5. Code style contract

### 5.1 Documentation

- **JSDoc** on exported constants, functions, interfaces, and non-obvious fields.
- File-level comments only for non-obvious architecture - prefer self-explanatory code.

### 5.2 Naming

| Artifact         | Convention                                               |
| ---------------- | -------------------------------------------------------- |
| **Filenames**    | See below (also `.cursor/rules/file-naming.mdc`)         |
| React components | PascalCase, default export                               |
| Hooks            | `use` prefix                                             |
| Store actions    | camelCase verbs                                          |
| DB fields        | snake_case (`parent_id`, `created_at`, `doc_c`)          |
| CSS              | `ko-` prefix, `--ko-*` variables                         |
| Tests            | `tests/<surface>/…/<file>.test.ts` mirroring source path |

**Filename conventions (source of truth for agents):**

| Kind                     | Convention        | Example                               |
| ------------------------ | ----------------- | ------------------------------------- |
| React component modules  | PascalCase        | `Sidebar.tsx`                         |
| Hooks                    | camelCase + `use` | `useStore.ts`                         |
| Non-component TS modules | camelCase         | `workspaceTree.ts`, `errorCapture.ts` |
| CLI scripts (`.mjs`)     | kebab-case        | `generate-sitemap.mjs`                |

Exceptions: generated/framework routes (`__root.tsx`, `routeTree.gen.ts`), package-mirrored `.d.ts` names.

### 5.3 Error handling

| Layer                   | Pattern                                              |
| ----------------------- | ---------------------------------------------------- |
| Storage decode          | Safe fallback + warn                                 |
| User actions (download) | Inline error state, no `alert()`                     |
| SSR                     | `errorCapture.ts` + `ErrorComponent` in `__root.tsx` |
| Missing entities        | Inline empty states                                  |

### 5.4 Formatting & lint

- ESLint 9 flat config + Prettier
- 2-space indent, LF, UTF-8 (`.editorconfig`)
- Unused vars: prefix `_`

---

## 6. AI operating model (FDE-relevant)

This repo is maintained with **high-context, verification-driven AI pair programming**. Expected agent behaviors:

### 6.1 Read before write

Agents must inspect adjacent implementations before proposing new abstractions. Prefer extending `useStore` actions, existing editor extensions, and `constants.ts` entries over new parallel systems.

### 6.2 Prove it works

- Run commands, don't suggest them passively.
- For UI changes: describe what to click AND verify in dev server when possible.
- For regressions: reproduce → hypothesize → fix → re-run failing check.

### 6.3 Keep packages separate (even on the same React major)

Both landing and extension use **React 19**, but they remain separate apps:

| Surface          | React | Styling / build                       |
| ---------------- | ----- | ------------------------------------- |
| Extension + demo | 19    | Tailwind 3, Vite 5 + CRXJS / web Vite |
| Landing          | 19    | Tailwind 4, TanStack Start / Vite 7   |

Do not share React components across packages without an explicit build boundary - different CSS systems and entrypoints, not React major mismatch.

### 6.4 Security & privacy posture

- Local-first: no cloud sync, no telemetry beyond optional web analytics on the landing site (if enabled)
- Never commit secrets, `.env`, or API keys.
- Report security issues per `SECURITY.md` - do not open public issues for vulnerabilities.

### 6.5 Git discipline

- **Do not commit** unless the user explicitly asks.
- **Do not force-push** `main`.
- PRs via `gh` per user rules; include test plan.
- Local hooks: **pre-push** auto-fixes format/lint when possible, then runs `npm run check`. If the tree was rewritten, commit and push again. Full builds: `npm run ci` / GitHub Actions. Skip with `SKIP_PRE_PUSH_CI=1` only when necessary. Refresh the landing download ZIP with `npm run package:extension` when shipping extension UI changes.

---

## 7. Domain glossary

| Term                 | Meaning                                                                      |
| -------------------- | ---------------------------------------------------------------------------- |
| **BlockDoc**         | ProseMirror-compatible JSON document tree                                    |
| **Page**             | Core entity: title, metadata, `doc`, tree position                           |
| **Directory**        | Folder node (`kind: "directory"`) in workspace tree                          |
| **Workspace**        | User-organized tree under Pages section                                      |
| **Dashboard**        | Home view: recent pages, quick create                                        |
| **Search palette**   | ⌘K fuzzy search over titles and body text (tags field indexed but no tag UI) |
| **OPFS attachments** | Images and voice/audio binaries under `mymemos-attachments/`                 |
| **New Tab override** | `chrome_url_overrides.newtab` → extension UI                                 |
| **Scroll runway**    | Landing sticky-scroll section driving launch video                           |
| **Web demo**         | Static SPA at `/demo/` for try-before-install                                |
| **llms.txt**         | Machine-readable product summary for AI crawlers                             |
| **GEO**              | Generative Engine Optimization - structured FAQ/schema for AI search         |

---

## 8. Related files

| File                                                                       | Purpose                                           |
| -------------------------------------------------------------------------- | ------------------------------------------------- |
| [`.cursor/SKILLS.md`](.cursor/SKILLS.md)                                   | Task → skill/workflow routing                     |
| [`.cursor/rules/`](.cursor/rules/)                                         | Scoped Cursor rules (`.mdc`)                      |
| [`.cursor/rules/constants-policy.mdc`](.cursor/rules/constants-policy.mdc) | Single-source constants policy (always apply)     |
| [`shared/constants.ts`](shared/constants.ts)                               | Canonical product constants (landing + extension) |
| [`tests/README.md`](tests/README.md)                                       | Source ↔ test path mapping contract               |
| [`CONTRIBUTING.md`](CONTRIBUTING.md)                                       | Human contributor guide                           |
| [`extension/README.md`](extension/README.md)                               | Extension architecture deep-dive                  |
| [`src/routes/README.md`](src/routes/README.md)                             | TanStack Router conventions                       |
| [`src/lib/seo.ts`](src/lib/seo.ts)                                         | Meta tags, JSON-LD, robots/sitemap/llms builders  |
| [`scripts/generate-sitemap.mjs`](scripts/generate-sitemap.mjs)             | Static SEO file generation                        |

---

## 9. Quick command reference

```bash
# Fast gate (also pre-push)
npm run check

# Full local CI (run before PR) - check + builds
npm run ci

# Extension development
npm run dev
npm run dev:check --prefix extension

# Landing + embedded demo
npm run dev:web

# Regenerate gitignored SEO files in public/
npm run generate:seo

# Landing SEO unit tests
npm run test -- tests/landing/lib/seo.test.ts

# Package extension for download button (manual when shipping UI)
npm run package:extension

# Watch tests
npm run test:watch
```

---

_Last structured for AI consumption. When human docs and this file diverge, trust generated artifacts and `package.json` scripts as runtime truth, then update this file._

---
> Source: [aryancodes-tech/my-memos](https://github.com/aryancodes-tech/my-memos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
