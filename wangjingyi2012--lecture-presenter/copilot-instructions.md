## lecture-presenter

> This file provides guidance to AI coding agents (Codex, Claude Code, Kimi Code, etc.) when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding agents (Codex, Claude Code, Kimi Code, etc.) when working with code in this repository.

## Overview

Lecture Presenter（演讲宝）is a Tauri 2 desktop app for presenting course materials: PDF, video, Markdown, source code, and **PPTE (PPT-EXTRA)** — HTML files that simulate PowerPoint slides. All app code lives in `lecture-app/`.

The desktop app is the open-source half of a two-repo product split:

- **This repo (desktop, open source)** = editor + presenter shell. It is the landing container for content generated elsewhere.
- **Lecture Web / LectureAI (separate closed-source repo)** = generation backend (FastAPI). Only operational notes about it live here, in `memory/2026-07-24-lecture-web-backend.md`; its code is NOT in this repo. The empty `backend/app/{api,models,schemas,services}/` directories at this repo's root are unused placeholders — do not put code there.

The desktop app talks to the cloud server (`https://design.hz-study-system.com`, configurable via `app-config.json`) for auth/membership, update checks, notifications, and login-only analytics.

## Commands

All commands run from `lecture-app/`:

```bash
npm install            # install deps (also requires Rust stable + Tauri 2 system deps)
npm run dev            # tauri dev — launches the desktop app with live frontend
npm run build          # tauri build — bundles to src-tauri/target/release/bundle/
npm run build:release  # obfuscated production build (scripts/build-obfuscated.js)

# Frontend logic tests (plain node, no framework — each script loads a src/js file in a vm sandbox):
npm run test:ppte            # scripts/test-ppt-extra-viewer.js (slide URL/platform logic)
npm run test:annotator       # scripts/test-ppte-annotator.js (annotation overlay)
npm run test:ppte-shared     # scripts/test-ppte-shared-groups.js (shared page groups)
npm run test:resource-center # scripts/test-resource-center.js (resource center logic)
npm run test:captions        # scripts/test-live-caption.js (caption transcript logic)
npm run test:course-manager  # scripts/test-course-manager.js (course grouping/switcher logic)
node scripts/test-auth.js    # auth UI logic (no npm script alias)

cargo check            # run inside src-tauri/ to type-check Rust
cargo test             # run inside src-tauri/ — unit tests in lib.rs plus tests/info_plist.rs
```

There is no bundler, framework, or lint step: the frontend is plain HTML/CSS/JS loaded directly from `lecture-app/src/` (`frontendDist: ../src`). A quick sanity check for frontend edits is `node --check src/js/<file>.js`. `npm run build:release` copies `src/` to `dist/`, obfuscates non-vendor JS, temporarily patches `tauri.conf.json`, then restores it.

**Packaging gotcha (macOS Apple Silicon):** the default Rust toolchain may be `stable-x86_64-apple-darwin` (installed under Rosetta), so a plain `npm run build` produces an x86_64 unsigned bundle. Build with `npx tauri build --target aarch64-apple-darwin` (do NOT use `npm run build -- -- --target`, the double `--` is unreliable); output lands in `target/aarch64-apple-darwin/release/bundle/`. The result may carry a broken linker signature — re-sign with `codesign --force --deep --sign -` and verify with `codesign --verify --deep --strict` before replacing `/Applications`.

## Architecture

### Frontend (lecture-app/src/)

- `index.html` — single page containing ALL UI: sidebar, content area, and every modal (PDF/video/code/HTML/PPTE viewers, course creator, settings, auth, resource center). Modals are `.hidden`-toggled divs.
- `audience.html` — separate Tauri window shown to the audience in speaker mode; receives `slide-change` events and emits `audience-navigate` events back.
- `js/*.js` — plain scripts, each defining one global object (e.g. `PptExtraViewer`, `CourseLoader`, `Content`). No modules/imports; load order is set by `<script>` tags in `index.html`. `app.js` installs global error logging (`window.errorLogs`) and calls each component's `init()` on DOMContentLoaded.
- Frontend ↔ backend via `window.__TAURI__.core.invoke(...)` and `window.__TAURI__.event` — guarded by `if (window.__TAURI__)` so browser dev mode degrades to `fetch`.
- `vendor/` holds third-party browser libs (pdf.js, marked, highlight.js, pptxgenjs bundle) loaded via `<script>`/`<link>`; these are not obfuscated.

Key frontend modules:

- `settings.js` — theme, font size, view switching (course view / PPTE management / PPTE editor). It was split up; satellite modules are `ai-settings.js` (AI provider config), `gitee-settings.js` (Gitee backup UI), `dev-settings.js` (developer settings, template center, updater config), `caption-settings.js` (live-caption credentials), `ppte-recent.js`, `ppte-create.js`, and `ppte-shared-groups.js` (`Object.assign`ed into `Settings`).
- `course-manager.js` — the course switcher panel in the titlebar (`#course-switcher` button + `#course-panel` dropdown): search, course groups (collapsible, persisted in `app-config.json` as `groups[]` + per-course `group` id; courses without a group render under 未分组), click-to-switch, inline rename/delete/move-to-group, drag-and-drop reorder across groups, and the create/import/PPTE entry buttons. Pure data helpers (`_groupedCourses`, `_assignGroup`, `_deleteGroup`, `_moveCourseBefore`, …) are covered by `npm run test:course-manager`.
- `ppte-editor.js` — the PPTE editor (largest frontend file, ~100KB). Includes save-conflict protection: file stats (`stat_files`) are captured at load and re-checked before save, and only dirty slides are written, so external edits (e.g. by Claude Code) are never silently overwritten. Integrates live reload via folder watching.
- `auth.js` / `auth.css` — login/membership against the cloud server; token persisted in localStorage; admin gating (`Auth.isAdmin()`) controls the local PPTE agent entry.
- `live-caption.js` / `caption-settings.js` / `live-caption.css` — microphone capture and streaming captions via Alibaba Cloud Fun-ASR; the websocket lives in Rust (`caption_*` commands), frontend renders final/partial transcript lines.
- `local-ppte-agent.js` — admin-only bridge to a private local PPTE generation agent (job start/poll/read via `local_ppte_agent_*` commands).
- `updater.js`, `notification-center.js`, `tracker.js` — update checks, server notifications, and login-only fire-and-forget analytics. All endpoints default to `design.hz-study-system.com` and are overridden from `app-config.json`.

### Backend (lecture-app/src-tauri/src/lib.rs)

Single ~5400-line file with ~60 `#[tauri::command]` handlers. Groups:

- File/course I/O: `read_text_file`, `write_text_file`, `read_file_bytes`, `stat_files`, `read/save_course_config`, `read/save_app_config`, pickers (`pick_files`, `pick_folder`, ...), `import_course`, `resolve_asset_path`.
- PPTE: `create_ppt_extra_folder`, `save_ppt_extra`, `import_ppte_resources`, `list_ppte_resources`, shared groups (`ppte_shared_group_inspect/snapshot/hash`), resource center copy (`ppte_copy_slides`), Gitee backup (`gitee_token_set/status/clear`, `ppte_git_info/init/sync`), live refresh (`watch_ppte_folder`/`unwatch_ppte_folder`, built on the `notify` crate).
- AI calls: `call_ai`, `call_ai_stream`, `test_ai_config` (DeepSeek/MiniMax/LectureAI; `reqwest` streaming).
- Live captions: `caption_start/stop/audio_chunk/test`, `caption_token_set/status/clear` — websocket to Fun-ASR via `tokio-tungstenite`; the token is stored in the OS keychain (`keyring` crate), as is the Gitee token.
- Cloud/auth/misc: `auth_api_request`, `check_update`, `fetch_notifications`, `local_ppte_agent_*`, audience window management (`open_audience_window`, `emit_slide_change`, `close_audience_window`), `open_external`, `run_in_terminal`, `detect_python`/`detect_terminal`, template listing/export.

Two custom URI scheme protocols:

- `slide://` — serves slide files preserving real path slashes (the built-in asset protocol encodes `/` as `%2F`, breaking relative URLs in slide HTML). HTML responses get `src-tauri/src/ppte-slide-bridge.js` appended: slide frames are cross-origin to the app window on macOS, so this in-frame script forwards navigation keys/shortcuts and editable-focus state to the parent via `postMessage` (`slide-navigate` / `slide-shortcut` / `slide-edit-focus`). Clicks are never forwarded as navigation — slides own their click-driven animations; page navigation is keyboard-only.
- `media://` — serves video with HTTP Range support.

`Info.plist` enables `NSAllowsArbitraryLoadsInWebContent` so WKWebView can load external HTTP content — guarded by `src-tauri/tests/info_plist.rs`.

### PPTE slide loading is platform-split (critical, easy to regress)

- **macOS WebKit**: iframe loads `slide://localhost/<abs-path>` directly.
- **Windows WebView2**: cannot reliably load `slide://` in iframes; instead the HTML is read via `read_text_file`, a `<base href="http://slide.localhost/...">` is injected (`_injectBaseHref` replaces any existing `<base>`), and assigned via `srcdoc`.

Platform detection is `_usesCustomProtocolHost()` in `ppt-extra-viewer.js`. `npm run test:ppte` covers both paths plus `<base>` replacement and Chinese-path URL encoding — run it after touching slide-loading logic. See `memory/2026-06-10-ppte-animation-blank.md` for the debugging history.

### PPTE viewer (js/ppt-extra-viewer.js)

`PptExtraViewer` handles three display states inside `#ppt-extra-modal`:

1. **Normal**: TOC sidebar + slide iframe.
2. **Play mode** (`F`): `playing-mode` class fullscreens the iframe.
3. **Speaker mode** (`S`): hides normal view, shows `#speaker-view` (current/next slide previews, Markdown notes with edit/preview toggle, timer), and opens the separate audience window via Tauri command. Notes are stored per-slide as `.note` files next to the slide HTML.

Navigation events flow from slide iframes via `postMessage({type:'slide-navigate'})` to the parent (`_installFrameNavigation` injects key/click listeners into each frame's document), and from the audience window via the Tauri `audience-navigate` event.

### Annotation overlay (js/ppte-annotator.js)

`PpteAnnotator.create({container, ...})` mounts a transparent canvas + toolbar over a container without touching slide HTML or the platform-split loading. Annotations (pen/highlighter/text/eraser) are memory-only and per-page (`setPage`), discarded on `reset()`. Two instances exist: one over `#ppt-extra-container` (toggled by the header button or `P`), one in `audience.html` (corner floating button) — they are independent, with no cross-window sync by design. When inactive the canvas is `pointer-events:none` so keyboard-only navigation still works.

Annotator styles live in `css/ppte-annotator.css` and must be loaded via `<link>`, not runtime `<style>` injection: the production CSP blocks inline styles on pages that already have them. `npm run test:annotator` covers the overlay logic.

### PPTX export (js/ppte-ppt-exporter.js)

`PptePptExporter.export(viewer)` converts the loaded PPTE HTML slides into an editable `.pptx` via the `pptxgenjs` dependency (`LAYOUT_WIDE`, 13.33×7.5in / 1920×1080 render basis). It reads the live DOM of each slide iframe, so it depends on slides being loaded through `PptExtraViewer`. This is the only feature using `pptxgenjs`; there is no test script for it.

### Page reuse: shared groups (js/ppte-shared-groups.js) and resource center (js/resource-center.js)

Two reuse mechanisms, both merged into / cooperating with the PPTE editor:

- **Shared groups** (`PpteSharedGroups`, `Object.assign`ed into `Settings`): a source PPTE marks contiguous slides as a shared source group (`sharedGroups[]` in its manifest, schemaVersion 2 with stable `deckId`/`slide.id`); other PPTEs insert it as a linked snapshot (`linkedGroups[]`, files under `.ppte-links/<groupId>/snapshots/<contentHash>/`). Linked pages are read-only and manually synced via content hashes. Backend commands: `ppte_shared_group_inspect/snapshot`, `ppte_shared_snapshot_hash`.
- **Resource center** (`ResourceCenter`): global page browser over all known PPTEs (imported courses' `course.json` — both the `r['ppt-extra']` field shape and the standalone `"ppt-extra": [{title, dir}]` key — plus `recentPpte`). Three-pane UI (deck list / multi-select pages / live iframe preview reusing `PptExtraViewer._assetUrl`/`_injectBaseHref`/`_usesCustomProtocolHost`). Two entries: titlebar button (full-screen modal, inserts by rewriting the target manifest directly) and the editor toolbar 资源中心 button (drawer, splices into `pb.slides`). Default action is **copy-insert** via the `ppte_copy_slides` backend command: selected slides + `.note` + shared resources land in the target's `.ppte-copies/<copyId>/` isolation directory and become ordinary editable pages (no `linkedFrom`). Link-insert (shared groups) is offered as the secondary action. `npm run test:resource-center` covers source discovery/parsing; Rust `copy_slides_*` tests cover the copy command.

### PPTE data safety (anti-overwrite, live refresh, Gitee backup)

A core product rule: **PPTE data must never be lost** (see `docs/plan-live-refresh-and-settings-refactor.md`). Three layers:

1. **Anti-overwrite**: the editor captures file stats at load, re-checks before save, refuses to silently clobber external changes, and writes only dirty slides.
2. **Live refresh**: `watch_ppte_folder` (Rust `notify`) pushes external changes to the player/editor; the player also has a manual refresh fallback.
3. **Gitee backup**: token in the OS keychain (never in `.git/config`), create private repo / git init / one-click manual backup; existing git repos are reused; SSH origins are rewritten to HTTPS.

### Course data model

A course is a folder with `course.json` (schema in `COURSE_FORMAT.md`): `weeks[]` → `resources` (slides/videos/readings/assignments/sourceCode/ppt-extra). A PPTE resource is a directory containing `manifest.json` (`{title, slides:[{file, title}]}`) plus one HTML file per slide. App config (imported course list, optional course `groups[]` with per-course `group` ids, settings, API keys) is stored in the OS app-data directory; `lecture-app/app-config.json` is the bundled default (cloud endpoints, theme, font size). `src-tauri/PPT-Template/` is bundled as a resource (slide templates).

## Testing strategy

- Frontend: framework-free node scripts in `lecture-app/scripts/test-*.js` — they read a `src/js` file, run it in a `vm` sandbox with a stubbed DOM, and assert behavior. When you change a file covered by one of these scripts, run the matching script; when you add a comparable feature module, add a matching test script.
- Rust: unit tests inline in `lib.rs` (e.g. snapshot/copy exclusion rules) plus `src-tauri/tests/info_plist.rs`. Run `cargo test` after backend changes.
- Manual smoke: some things (dual-window speaker mode, real WebKit/WebView2 iframe behavior, caption audio) can only be verified via `npm run dev` on a real machine — say so when you could not verify them.

## Conventions

- UI strings are Chinese; code comments are English.
- Repo docs: `COURSE_FORMAT.md` (course/PPTE format spec), `PUBLISHING.md` (release process: tag `v*` triggers GitHub Actions builds), `docs/` (design/implementation plans), `memory/` (debugging and feature reports — append new reports there, named `YYYY-MM-DD-topic.md`).
- This is the public GitHub repo; never commit personal courses, API keys, tokens, or `.db` files. Cloud tokens (Gitee, caption) belong in the OS keychain via the backend commands, not in config files.

---
> Source: [wangjingyi2012/lecture-presenter](https://github.com/wangjingyi2012/lecture-presenter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
