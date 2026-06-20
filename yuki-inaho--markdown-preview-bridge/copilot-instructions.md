## markdown-preview-bridge

> Preview and inspect local Markdown files through a fast ByteMD + Vite browser bridge controlled by Playwright. Use when the user wants an Overleaf-like workflow for Markdown: load a local .md file, render Mermaid/math/tables/code in Chrome, ask the user for visual feedback, inspect rendered DOM, map rendered blocks back to likely Markdown source lines, or iteratively edit Markdown with Codex/Claude assistance.


# Markdown Preview Bridge

Use this skill to provide an Overleaf-like loop for local Markdown:

```text
local .md -> ByteMD/Vite browser preview -> Playwright DOM inspection -> Codex edits local .md -> reload/recheck
```

## Core Design

- Keep the local Markdown file as the source of truth.
- Treat this as a trusted local development bridge, not a shared web service or collaborative editor.
- Use a local Vite dev server with ByteMD `Editor` + `Viewer`.
- Keep the left pane as source editing only and the right pane as rendered review. ByteMD `Editor` must run with `mode="tab"` to avoid rendering a duplicate split preview inside the editor pane.
- Keep page-level scrolling disabled. The editor pane and rendered pane should own their scrollbars; otherwise the user sees duplicated browser/pane scrollbars.
- Enable simple scroll-ratio sync between the source editor and rendered pane by default. It is a navigation aid, not exact source mapping.
- Use official ByteMD plugins for GFM, Mermaid, and math.
- Expose a lightweight DOM bridge at `window.__mdPreviewBridge` so Playwright does not need to snapshot very large DOMs.
- Set the browser title to `<markdown-file-name> - Markdown Preview Bridge` so agents can identify and focus the right tab/window while keeping a stable search suffix.
- Show only the Markdown filename in the header. Put the full absolute path in a hover/focus overlay below the filename; long full paths are too noisy in normal review mode.
- Prefer bridge diagnostics over full `playwright-cli snapshot` for large documents.

## Prerequisites

- `uv` for reproducible execution of the bundled Python launcher and validation scripts.
- Python 3.10 or newer for the bundled launcher scripts. `.python-version` pins the local dev/test baseline.
- Node.js and npm for the Vite preview app. The launcher installs app dependencies into a cache directory when needed.
- `playwright-cli` and a local Chrome/Chromium for browser operation.
- `xdotool` only when headed browser window geometry/focus must be normalized on Linux/X11. Page and DOM inspection should still use Playwright.

## Trust Boundary

- Bind the preview server to `127.0.0.1` unless there is a deliberate reason to expose it.
- Use `--root` to restrict which Markdown files the bridge may read or write. If omitted, root defaults to the input file's parent directory.
- Only Markdown-like extensions are allowed by default: `.md`, `.markdown`, `.mdown`, `.mkd`.
- Local Markdown images are served through `/api/asset` only when their real path stays under `--root` and their extension is in the image asset allowlist.
- Read-only mode is the default. Use `--allow-write` only when browser-side save is needed.
- Do not treat this as a safe renderer for untrusted Markdown. Raw HTML, Mermaid, KaTeX, and plugin behavior should be considered trusted-local-input concerns.

## Quick Start

Start a preview server for a local Markdown file:

```bash
uv sync --frozen
uv run --frozen scripts/preview.py \
  --file /absolute/path/to/document.md \
  --root /absolute/path/to/repo-or-doc-root \
  --port 8777
```

Run commands from the skill root, or resolve `scripts/preview.py` relative to this `SKILL.md`. If `just` is installed, the root `justfile` exposes the common `uv run` wrappers.

Markdown hot reload is enabled by default. Disable it only when file watching is undesirable:

```bash
uv run --frozen scripts/preview.py \
  --file /absolute/path/to/document.md \
  --port 8777 \
  --no-watch
```

Enable browser-side saving only when needed:

```bash
uv run --frozen scripts/preview.py \
  --file /absolute/path/to/document.md \
  --root /absolute/path/to/repo-or-doc-root \
  --port 8777 \
  --allow-write
```

Relative Markdown image links are resolved from the current Markdown file's
directory and served through `/api/asset` if they stay under `--root`. Use
`--asset-ext` to change the allowed local image extensions.

Open the preview with Playwright:

```bash
playwright-cli -s=md-preview open http://127.0.0.1:8777/ --browser chrome
```

For a visible browser:

```bash
playwright-cli -s=md-preview-visible open http://127.0.0.1:8777/ --browser chrome --headed
```

For a visible browser with normalized geometry, prefer the bundled wrapper:

```bash
uv run --frozen scripts/open_visible.py \
  --url http://127.0.0.1:8777/ \
  --session md-preview-visible \
  --width 1600 \
  --height 1000
```

Override geometry per environment with flags or environment variables:

```bash
MD_PREVIEW_WINDOW_X=80 \
MD_PREVIEW_WINDOW_Y=40 \
MD_PREVIEW_WINDOW_WIDTH=1600 \
MD_PREVIEW_WINDOW_HEIGHT=1000 \
uv run --frozen scripts/open_visible.py
```

If using raw `playwright-cli`, normalize after launch:

```bash
window_id=$(xdotool search --onlyvisible --name 'Markdown Preview Bridge' | tail -n 1)
xdotool windowmove "$window_id" <x> <y> windowsize "$window_id" <width> <height>
```

## Switching Files

The preview page has a `File path` input. Use it for normal operation instead of restarting the server:

1. Paste an absolute Markdown path into `File path`.
2. Click `Open`.
3. Confirm the `sourcePath` with:

```bash
playwright-cli -s=md-preview-visible eval '() => window.__mdPreviewBridge.diagnostics().sourcePath'
```

The server accepts `path` query parameters for `/api/markdown` and `/api/save`, so `Reload` and `Save` apply to the currently opened file path.
The path must stay under the configured preview root and use an allowed Markdown extension.
Do not treat a successful `goto ?path=...` or a visible tab title as proof that the Markdown loaded. Always confirm `diagnostics().sourcePath`, `diagnostics().rootPath`, `diagnostics().status`, and `diagnostics().renderErrors` after switching files.
If `status` is `error`, `sourcePath` is empty, or `renderErrors` says the Markdown path is outside the preview root, the browser tab is open but the file is not loaded; restart or create a preview server with a `--root` that contains the target file.

You can also open a document directly with a URL query, which is useful for creating additional browser tabs:

```bash
http://127.0.0.1:8777/?path=/absolute/path/to/document.md
```

The app mirrors the current `sourcePath` back into the URL with `history.replaceState`, so the tab URL stays copyable and restorable.

## Multi-Tab Workflow

When comparing multiple Markdown files, do not restart the preview server and do not repeatedly run `playwright-cli open` against the same session. Reuse the existing browser and create tabs inside it.

Recommended workflow:

1. Keep one preview server running with `--root` set to the repo or document root. If a server is already listening on the default port, do not assume it is the right one; first check its `rootPath` with diagnostics.

```bash
playwright-cli -s=md-preview-visible eval '() => window.__mdPreviewBridge?.diagnostics?.()'
```

If the existing server root does not contain every target document, start a new server on another free port with the correct `--root` and open all tabs against that port:

```bash
uv run --frozen scripts/preview.py \
  --file /absolute/path/to/first.md \
  --root /absolute/path/to/repo-or-doc-root \
  --port 8780
```

2. Check existing tabs:

```bash
playwright-cli -s=md-preview-visible tab-list
```

3. Create a new tab using the same bridge app:

```bash
playwright-cli -s=md-preview-visible tab-new http://127.0.0.1:8777/
```

4. In the new tab, either use the `File path` input manually, or navigate directly with a `path` query:

```bash
playwright-cli -s=md-preview-visible goto 'http://127.0.0.1:8777/?path=/absolute/path/to/other.md'
```

5. Confirm the current tab:

```bash
playwright-cli -s=md-preview-visible eval '() => window.__mdPreviewBridge.diagnostics()'
```

6. Switch tabs explicitly when needed:

```bash
playwright-cli -s=md-preview-visible tab-select 0
playwright-cli -s=md-preview-visible tab-select 1
```

Notes:

- `playwright-cli open` may create or reuse a browser context in a way that feels like the visible window disappeared or the current page was replaced. Prefer `tab-new`, `tab-select`, `goto`, and the in-page `File path` input for a stable review workflow.
- If a window is still alive but not visible or not focused, use Playwright diagnostics first, then use desktop focus only for window concerns:

```bash
playwright-cli -s=md-preview-visible eval '() => ({ title: document.title, url: location.href })'
xdotool search --onlyvisible --name 'Markdown Preview Bridge'
```

- Keep one visible browser session for the review set. Use tabs for documents, not new browser sessions.
- If the target tab title is important for automation, wait for diagnostics after navigation because the title changes from `Markdown Preview Bridge` to `<filename> - Markdown Preview Bridge` after the Markdown loads.

## Hot Reload

By default, the server watches the active Markdown file and sends a Vite HMR event when it changes on disk. The page reloads the current file automatically when `Auto reload` is checked.

- Use `Auto reload` for the normal editor/browser feedback loop.
- Uncheck `Auto reload` before making large external edits if repeated refreshes become distracting.
- Use `--no-watch` when filesystem watching is unsupported or undesirable.
- Confirm state with:

```bash
playwright-cli -s=md-preview-visible eval '() => window.__mdPreviewBridge.diagnostics()'
```

## Playwright Inspection

Prefer direct bridge calls:

```bash
playwright-cli -s=md-preview eval '() => window.__mdPreviewBridge.diagnostics()'
playwright-cli -s=md-preview eval '() => window.__mdPreviewBridge.headings()'
playwright-cli -s=md-preview eval '() => window.__mdPreviewBridge.findText("検索文字列")'
```

Avoid full `snapshot` on large Markdown unless needed; it can timeout on long rendered documents.

## Marker Workflow

Use the rendered preview pane, not the source editor, when creating a marker.

1. Select text in `#rendered`.
2. Create a marker through the UI or:

```bash
playwright-cli -s=md-preview-visible eval '() => window.__mdPreviewBridge.addMarkerFromSelection()'
```

3. Inspect the current document markers:

```bash
playwright-cli -s=md-preview-visible eval '() => window.__mdPreviewBridge.markers()'
```

4. Inspect every marker stored in the browser origin, including markers created in another tab:

```bash
playwright-cli -s=md-preview-visible eval '() => window.__mdPreviewBridge.allMarkers()'
```

5. Remove one marker or all markers for the current document:

```bash
playwright-cli -s=md-preview-visible eval '() => window.__mdPreviewBridge.clearMarker("marker-id")'
playwright-cli -s=md-preview-visible eval '() => window.__mdPreviewBridge.clearMarkersForDocument()'
```

Notes:

- Markers are browser review state stored in `localStorage`; they do not edit the Markdown file itself.
- Stale markers are preserved instead of auto-deleted when their anchor no longer restores.
- `diagnostics()` reports `markerCount`, `allMarkerCount`, `staleMarkerCount`, and `paneRatio`.

## Resizable Panes

The source editor and rendered preview are separated by a draggable divider.

- The default split is source:preview = 2:3 so the rendered review has more room.
- Drag the center handle left or right to change the split.
- Use ArrowLeft/ArrowRight on the focused divider for keyboard adjustment; hold Shift for larger steps.
- Double-click the divider, or press Enter/Space while focused, to reset to the 2:3 review layout.
- The ratio is persisted per source path in `localStorage`.
- The panes keep a minimum width, so very small windows may clamp the split.

Playwright smoke for the current pane ratio:

```bash
playwright-cli -s=md-preview-visible eval '() => window.__mdPreviewBridge.diagnostics().paneRatio'
```

## User Feedback Loop

1. Open the local Markdown file in the preview.
2. Ask the user to inspect the visible browser if needed.
3. When the user says a rendered area is wrong, inspect with:

```bash
playwright-cli -s=md-preview eval '() => window.__mdPreviewBridge.getSelectionContext()'
```

4. Use the returned `sourcePath`, `sourceLine`, and selected text to edit the local `.md`.
5. Reload or wait for Vite HMR, then run diagnostics again.

When the user selects a rendered area in the browser, use `getSelectionContext()` first. It can read the browser selection and return the selected text, nearest annotated block, likely source line, and surrounding Markdown context. Treat the returned `sourceLine` as a navigation hint, not an exact source span.

## Bridge API

The page exposes:

- `diagnostics()`: source path, status, hot reload state, counts for headings/tables/code/math/mermaid, render errors, and text length.
- `diagnostics().readOnly`, `diagnostics().rootPath`, and `diagnostics().allowExt`: current file-access guardrails.
- `selectionDetails()`: raw selection text, math-aware plain text, and whether the current selection includes rendered math.
- `headings()`: rendered headings with approximate source lines.
- `images()`: rendered image records, including original source, `/api/asset` URL, resolved asset path, and natural size.
- `findText(query)`: rendered block matches and likely source lines.
- `getSelectionContext()`: current browser selection and nearest rendered block metadata.
- `gotoLine(line, options)`: jump the source editor to a logical Markdown line.
- `sourceForLine(line, radius)`: Markdown source context around a line.
- `getMarkdown()`: current Markdown source text.
- `reloadMarkdown()`: reload the local file from disk.

Line mapping is approximate. It uses block text matching against the Markdown source and is good enough for review navigation, not a compiler-grade SyncTeX equivalent.

## Non-Goals

- This is not a collaborative editor.
- This is not a WYSIWYG authoring environment.
- This is not a StackEdit/HedgeDoc replacement.
- Current source mapping is block-navigation assistance, not exact compiler-grade source mapping.
- Current scroll sync is a navigation aid, not a guarantee that source and rendered pixels correspond exactly.
- Marker positions are approximate and rely on rendered block metadata, not an exact compiler-grade source map.
- The browser review state stays local to the current origin; it is not synced to disk or shared across machines.

## Validation Checklist

After opening a visible preview, verify the workflow with bridge diagnostics instead of judging only by screenshots:

```bash
playwright-cli -s=md-preview-visible eval '() => window.__mdPreviewBridge.diagnostics()'
```

Check these points before using the preview for review/editing:

- `pageTitle` contains the opened Markdown filename and keeps the `Markdown Preview Bridge` suffix.
- The header shows the Markdown filename, not the full path; the full path is visible from the filename hover/focus overlay and diagnostics.
- `sourcePath` points to the intended local Markdown file.
- `rootPath`, `allowExt`, and `readOnly` match the intended trust boundary.
- `status` starts with `loaded:` for the intended file. A page with `status: error` is not usable even if the tab URL contains the expected `path` parameter.
- `renderErrors` is empty or contains only known non-blocking renderer warnings.
- `autoReload` and `hotReloadAvailable` match the intended mode.
- Tables have visible `th`/`td` borders when table readability matters.
- Code blocks are visually distinct from paragraphs and inline code.
- The page body has no top-level scrollbar; scrolling should happen in the source pane and rendered pane.
- Scroll sync is checked by comparing source/rendered scroll ratios, not raw pixel offsets.

## Tips

- Use explicit absolute Markdown paths for reproducibility.
- When reusing an existing preview server, verify `diagnostics().rootPath` before opening repo-local files. A server started for another root can accept the URL but reject the Markdown with "outside preview root".
- If the default port is already occupied by a server with the wrong root, leave it alone and start a second bridge on a free port, for example `8780`; then retarget every browser tab to the new port.
- A tab title of `Markdown Preview Bridge` or a URL containing `?path=...` is not enough. The usable check is `diagnostics().status` beginning with `loaded:`, `sourcePath` matching the intended file, and `renderErrors` being empty.
- For Japanese paths, the server URL-encodes path headers; do not put raw non-ASCII paths in custom HTTP headers.
- For very large Markdown files, diagnostics are more reliable than `snapshot`.
- Avoid opening very large Markdown files in a visible browser unless needed. Documents in the hundreds-of-KB range can be heavy because the browser still needs to maintain source editing state, rendered DOM, plugin output, and selection metadata.
- If the preview feels slow, close old heavy sessions first:

```bash
playwright-cli -s=md-preview-visible close
playwright-cli -s=md-preview close
```

- Prefer opening a smaller target document, then use the `File path` input for switching.
- For two or more documents, use one visible browser session and multiple tabs. Prefer `tab-new` and `tab-select` over repeated `open`.
- If a huge source file is unavoidable, use bridge diagnostics first and only make the browser visible when user visual review is required.
- If Mermaid/math is not rendered, inspect `diagnostics().renderErrors` and browser console output.
- KaTeX warnings such as "LaTeX-incompatible input" are not always fatal. They can happen when ordinary text contains `$...`-like fragments or symbols that the math plugin attempts to parse.
- If the user wants to interact manually, use `--headed` and keep the browser open.
- If rendered tables have no visible grid lines, inspect computed styles for `table`, `th`, and `td`. Some Markdown renderers do not provide GitHub-style table borders by default, so the app CSS should set `border-collapse` and cell borders explicitly.
- If code blocks look like ordinary text, style `pre`, `pre code`, and inline `code` explicitly. Some Markdown renderer defaults are too subtle for browser-based visual review.
- For user range selection, ask the user to select text in the right rendered pane, then run:

```bash
playwright-cli -s=md-preview-visible eval '() => window.__mdPreviewBridge.getSelectionContext()'
```

- If the selection spans multiple rendered blocks, `sourceLine` may point to the nearest/anchor block rather than the whole selected range. In that case, use `text` plus `sourceContext` to infer the exact Markdown region.
- Tables often normalize into dense selected text. If a selected table is hard to map, use the heading/block context and then inspect the surrounding Markdown with `sourceForLine(line, radius)`.

## Findings From Initial Use

- ByteMD `Editor` defaults to an auto/split mode on wide screens. If the app also renders a separate `Viewer`, the user sees source + preview on the left and another preview on the right. Use `mode="tab"` for the editor pane so the default view is source-only.
- Page-level scrollbars plus pane-level scrollbars are confusing. Set the app shell to `height: 100vh` and `overflow: hidden`; then synchronize the editor pane and rendered pane scroll positions.
- Some React wrappers around ByteMD insert an extra anonymous `div` between the pane and `.bytemd`. If the editor still expands vertically, constrain that wrapper as well as `.bytemd`, `.bytemd-body`, `.CodeMirror`, and `.CodeMirror-scroll`.
- Scroll sync should be verified by comparing scroll ratios, not raw `scrollTop`, because source and rendered documents have different heights.
- Browser range selection from the rendered pane is readable through `window.getSelection()` and `getSelectionContext()`.
- Multi-block selections can return a nearest-block `sourceLine` rather than an exact source span. This is acceptable for navigation but not exact source mapping.
- `playwright-cli open` does not currently expose a direct `--window-size` option. Use the bundled `open_visible.py` wrapper or `xdotool` after headed launch to normalize the visible window.
- Prefer Playwright APIs for page/tab-level control. Use `xdotool` only for desktop-window concerns that the current CLI wrapper cannot express, such as focusing an existing visible Chrome window or enforcing OS window geometry.
- Running both a large headless preview and a large visible preview wastes CPU/memory. Keep one active preview session per document review unless comparing renderers.
- Repeated `playwright-cli open` calls can make it look like the browser closed or the current page was replaced. The safer multi-document workflow is: keep the server, keep the session, create a tab, then set `File path` or `goto ?path=...`.
- Reusing a visible browser session does not guarantee the backing server is rooted at the current repository. A common failure mode is: tabs are created with correct-looking `?path=` URLs, but diagnostics report `status: error`, empty `sourcePath`, empty `rootPath`, and a render error such as `Markdown path is outside preview root`. The fix is to start a new preview server with the correct `--root` on an unused port, navigate the existing tabs to that port, and re-run diagnostics for each tab.
- When the launcher runs as a long-lived foreground process, the agent environment may show a background terminal/process. Keep it running while the user is reviewing the browser, and close it only when the user is done or asks to stop the preview.
- The first implementation served `/api/markdown` after Vite fallback, causing HTML to be returned instead of Markdown. The API must be registered as a Vite plugin before the fallback middleware.
- Vite 7 requires newer Node versions than some LTS environments provide; this skill pins Vite 5.x for broader compatibility.
- The line mapping is intentionally approximate. For exact source mapping in a future stage, replace text matching with `markdown-it` token maps or unified/remark AST `position` metadata.
- The local server can read Markdown files under the configured preview root. It can write only when started with `--allow-write`. Keep it bound to `127.0.0.1` and treat it as a local development tool, not a shared service.

## StackEdit-Inspired Next Steps

This skill intentionally starts as a lightweight browser bridge, not a full StackEdit replacement. If higher-fidelity preview/edit sync is required, prioritize these upgrades:

- Keep the file-access guardrails strict: root-limited paths, Markdown extension allowlist, and read-only review mode.
- Add a small browser smoke test fixture set before large renderer changes.
- Replace text-search source mapping with renderer-level source positions. Use `markdown-it` token `map` or unified/remark AST `position` data and emit `data-source-line-start` / `data-source-line-end` on rendered blocks.
- Replace scroll-ratio sync with section-based sync. Build a section map from source line ranges and rendered block offsets, then synchronize within the active section.
- Use a source-aware renderer for the right pane. ByteMD can remain as the left editor, while the right preview can be rendered by markdown-it/unified with explicit source metadata.
- Wrap tables in a scroll container instead of making `<table>` itself `display: block` when renderer control is available.
- Add syntax highlighting with Prism, Shiki, or highlight.js; include language labels, fallback behavior, and optional copy controls.
- Collect renderer errors structurally: `window.onerror`, `unhandledrejection`, Mermaid failures, KaTeX parse warnings/errors, broken images, and broken anchors should appear in `diagnostics()`.
- Add render option toggles for expensive or ambiguous plugins such as Math and Mermaid, especially for large conversation logs.
- Add an outline UI from heading metadata and temporary selection highlights to improve human/agent review handoff.

---
> Source: [yuki-inaho/markdown-preview-bridge](https://github.com/yuki-inaho/markdown-preview-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
