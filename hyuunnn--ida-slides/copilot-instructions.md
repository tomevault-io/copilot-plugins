## ida-slides

> An IDA Pro plugin (IDA 9.2+, Python) that renders real **Marp** or

# CLAUDE.md — ida-slides

## What this project is

An IDA Pro plugin (IDA 9.2+, Python) that renders real **Marp** or
**Slidev** slide decks inside a dockable IDA tab, for presenting reverse
-engineering work live. The deck and the IDB are bridged both ways:
`@name` tokens in slides drive IDA (jumps, embedded pseudocode), and a
right-click action copies references from IDA back into deck syntax.

The core concept is non-negotiable: **the deck lives in an IDA docking
tab**, side by side with disassembly/pseudocode. Designs that move
rendering into an external browser window were considered and rejected.

## Implemented features

- `@name` / `@0xADDR` → clickable links that jump the disassembly view;
  `@name:N` opens the Hex-Rays pseudocode at line N.
- `@name[a:b]` / `[7]` / `[]` / `[a:b@N]` → embeds decompiled lines into
  the slide as a code block, read live from the IDB on every save;
  `@N` marks one line with `►`.
- Hover preview: mousing over any `@` link shows a decompiled excerpt
  tooltip without leaving the slide.
- Copy @reference: right-click in disasm/pseudocode/hex view copies the
  token for that spot (`@name`, `@name:line`, or a selection as
  `@name[lo:hi]`). Names the token grammar can't re-parse (ObjC
  selectors, demangled C++) fall back to `@0xADDR` so the token always
  works.
- Deck lint: every load/save resolves all `@` tokens against the open
  IDB; unresolved ones show as `⚠ N unresolved @ref(s)` in the toolbar
  (tooltip lists token + slide number, details in Output). It is a
  *resolution* checker, not a syntax checker.
- Live reload: a debounced QFileSystemWatcher survives atomic-rename
  saves (VS Code, marp CLI) by re-adding the path; gives up after ~5s if
  the file is truly gone.
- Engine detection per deck: `ida-slides-engine:` front-matter override →
  `marp:` key (value respected, `marp: false` ≠ marp) → Slidev-specific
  front-matter keys (if the slidev CLI exists) → default marp.
- Focus preservation: jumps never steal keyboard focus from the deck, so
  arrow keys keep driving slides (details below).

## Architecture

```
ida_slides_entry.py      plugin entry (env gate) → ida_slides.py
ida_slides.py            action/menu registration (Ctrl+Shift+M)
presenter_form.py        dockable PluginForm: toolbar, renderer via
                         deck_view dispatch, file watcher wiring, lint
deck_view.py             platform-neutral core: DeckViewBase (marp -w /
                         slidev pipelines, preprocessing, status),
                         injected USER_JS, engine detection, tool
                         discovery, dispatch_page_message, and the
                         create_renderer()/availability_error() dispatch
webkit_view.py           macOS renderer: native WKWebView via PyObjC —
                         implements DeckViewBase's _native_* hooks
webview2_view.py         Windows renderer: native WebView2 attached to
                         the container HWND — same hook surface
webview2_com.py          ctypes-only COM layer for WebView2 (no pip
                         deps); IIDs/vtable indices frozen from the SDK
win/WebView2Loader.dll   vendored x64 loader (Microsoft.Web.WebView2)
ida_links.py             @token grammar (TOKEN_RE / JS_TOKEN_RE),
                         resolution, jumps
deck_preprocess.py       embed expansion, hover-preview text, deck lint
marp_markdown.py         deck-structure single source: front-matter
                         boundary, fence tracking (iter_fenced), slide
                         splitting — engine detection, embed expansion
                         and lint all build on it
copy_ref.py              Copy @reference context-menu action
file_watcher.py          debounced, rename-surviving file watcher
```

Render pipeline (.md): deck.md → `deck_preprocess.expand_embeds`
(decompiles `@name[a:b]` tokens) → hidden `.name.ida-slides.md` → marp
CLI (QProcess) → `.name.ida-slides.html` → native webview (WKWebView on
macOS, WebView2 on Windows). Slidev decks run a local dev server instead
and rely on Vite HMR. `USER_JS` (injected per platform: WKUserScript /
AddScriptToExecuteOnDocumentCreated) linkifies `@tokens` in the rendered
DOM and posts click/preview messages to Python (WKScriptMessageHandler /
chrome.webview.postMessage → WebMessageReceived); both bridges land in
`deck_view.dispatch_page_message`.

## Design decisions & tradeoffs

- **Native OS webview over QtWebEngine.** IDA's bundled PySide6 has no
  QtWebEngine on any platform, and pip Qt wheels can ABI-clash with
  IDA's bundled Qt — so the plugin never touches the Qt web stack. macOS
  uses the system WKWebView (PyObjC); Windows uses the system WebView2
  runtime driven over raw COM with stdlib ctypes (`webview2_com.py`, no
  pip deps; Windows support added 2026-07 by owner request, revising the
  earlier macOS-only call). The split lives in `deck_view.DeckViewBase`
  (all pipeline logic) + per-platform `_native_*` hooks. There are NO
  fallback renderers: without the platform webview or the deck's engine
  CLI (marp/slidev), decks simply don't render (a warning / status
  message says why).
- **The macOS webview is embedded through Qt, never poked into winId().**
  `QWindow.fromWinId(<WKWebView NSView>)` + `QWidget.createWindowContainer`
  — so Qt owns the native window's geometry and compositing. The obvious
  alternative (grab `container.winId()`'s NSView and `addSubview_` the
  WKWebView into it) *renders* fine but never reaches the screen inside
  IDA's dock: the pane shows stale pixels of whatever was behind it
  (black boxes, other widgets' text) while a WKWebView snapshot is
  pixel-perfect. Nothing programmatic reconnects it — frame nudges, Qt
  resizes, splitter and main-window resizes, delayed attach all fail;
  only a manual dock drag does, once, permanently. Don't "simplify" this
  back to addSubview_ (2026-07).
- **PyObjC crash safety (documented in webkit_view.py header).** A
  Python exception escaping a PyObjC delegate aborts IDA, and PyObjC
  cannot call WebKit completion-handler *blocks* at all. Therefore no
  delegate method that receives a block is ever implemented — click
  routing uses WKUserScript + postMessage instead of navigation
  delegates. Keep it that way.
- **WebView2 COM safety (documented in webview2_com.py header).** vtable
  slot indices and IIDs are hardcoded from the official SDK header —
  they are frozen ABI; never guess or reorder them. ctypes COM callbacks
  must never raise (each is wrapped), and IDA work is deferred out of
  callback frames via `QTimer.singleShot(0)` exactly like the ObjC rule.
  The attach is a two-step async chain (environment → controller); the
  environment is cached process-wide, the controller/webview are
  per-view. `npm` launcher shims (`marp.cmd`) are bypassed in favor of
  `node <real .js entry>` (`_spawn_spec`) so killing the QProcess reaps
  the actual renderer instead of orphaning a `marp -w` under cmd.exe.
  NavigationCompleted must stay SUCCESS-ONLY on its way to
  on_load_finished (the view filters on IsSuccess, matching WKWebView's
  didFinishNavigation): an aborted navigation reported as finished
  consumes `_pending_hash` and snaps the deck off the current slide.
  ComCallback handlers follow standard COM in-param refcounting — every
  create/call site must dispose() the construction reference right after
  the API call (see the REFCOUNT MODEL note), or handlers leak in _LIVE
  and pin closed views.
- **Save-time batch rendering, not incremental.** Every save re-runs
  the preprocess; marp runs as a persistent `-w` watcher (one per deck,
  stopped on cleanup / file switch / engine switch) that re-renders when
  the prepared md is rewritten. The view reloads when marp logs its
  render-complete line (`[ INFO ] … => <out>`) on stderr — NOT on output
  mtime as a trigger, which can be seen mid-write. A save landing
  mid-render makes the first `=>` line describe the pre-save render, so
  `_output_is_current` (output vs prepared-input mtime) discards it and
  the follow-up render's line loads instead. While an `[ ERROR ]` is
  latched, the fast path is disabled and the input force-rewritten, so a
  same-content save re-surfaces the error instead of presenting the
  pre-error html as success. A 15s timeout only shows a "taking longer" heads-up; a dead
  watcher is caught by `_on_marp_exit`. Tradeoff accepted for
  pixel-perfect themes; costs are blunted by the 200ms debounce,
  Hex-Rays' internal cfunc cache, slice-only tag_remove in
  `decompile_lines`, an output-diff guard, a single deck read per load
  (shared by detect_engine and _prepare_md), and a per-pass name cache
  in the lint. The diff guard sits
  intentionally AFTER `expand_embeds`: a same-content save is the
  documented gesture for refreshing embeds after an IDB rename, so
  identical input must not skip expansion. Status label policy: error
  messages only — no per-save "rendering…" text (the label popping in
  reflows the pane on every save). The 15s "taking longer" notice is the
  one allowed non-error message, since it fires only on an abnormal stall.
- **Focus invariants.** Verified mechanics, easy to regress:
  - `jumpto(ea, -1, 0)` (no UIJMP_ACTIVATE) repositions without taking
    focus but does NOT raise a buried tab; `activate_widget(w, False)`
    raises without focus.
  - `open_pseudocode(..., OPF_REUSE)` steals focus even when reusing a
    view — capture `get_current_widget()` before, restore synchronously
    after.
  - The `_position` caret-retry loop must use `activate_widget(ct, False)`.
- **Never hold a TWidget/vdui across a QTimer delay.** Stale SWIG
  pointers can hard-crash IDA (not catchable in Python). Re-resolve via
  `find_widget(title)` + `get_widget_vdui()` and compare `cfunc.entry_ea`
  before touching the viewer (see `_jump_to_pseudocode_line`).
- **@token linkify lives in one place** — `deck_view.USER_JS` (shared by
  both platform webviews; a `post()` shim picks chrome.webview vs
  webkit.messageHandlers at runtime, and startup is gated on DOM
  readiness because WebView2 injects at document-created). The grammar
  is single-sourced from `ida_links._NAME_PATTERN` via `JS_TOKEN_RE`.
  (The former Python `linkify_html` and QtWebEngine `LINKIFY_JS` copies
  were deleted with the fallback renderers.)
  Behavioral quirk: the lint (`unresolved_refs`) trims trailing dots only
  while unresolvable; USER_JS trims unconditionally (no IDB access at
  render time).
- **Markdown parsing is a pragmatic subset, aligned with marp where it
  matters:** fences follow the CommonMark closing-length rule; a `---`
  directly under text is a setext H2, not a slide break. `split_slides`
  drives the lint's slide numbers, so divergence from marp shows up as
  off-by-one lint numbers.
- **Removed features (owner decisions — do not re-add):**
  - `@!` presenter-follow (auto-jump when a slide becomes visible):
    removed with its toolbar toggle. Legacy `@!name` tokens render as
    dead text and are invisible to the lint — known and accepted.
  - Landing flash (tinting the pseudocode line a `:N` jump arrived at).
  - Fallback renderers (`renderers.py`: built-in QTextBrowser viewer and
    the thin QtWebEngine .html view), removed 2026-07 along with
    `linkify_html`/`LINKIFY_JS`/`make_href`/`name_from_url`. marp/slidev
    via WKWebView is the only supported path — no degraded rendering.
- **Accepted behaviors (not bugs):** unmapped raw-hex refs
  (`@0xDEADBEEF`) render as live-looking links and silently fail on
  click — the lint warning is considered sufficient. IDA's native
  Close/Float/Fullscreen dock tooltips stay; only the unresolved-refs
  warning label carries a plugin tooltip. Saving/Reload never switches a
  running slidev deck back to marp (the slidev save path skips
  detect_engine on purpose — full loads would defeat Vite HMR): the
  engine is chosen when work on a deck starts, and Open… is the
  re-route path (owner decision, 2026-07).

## Working on the code

- The repo lives in the IDA plugins dir (macOS: symlink at
  `~/.idapro/plugins/ida-slides`; Windows: directly at
  `%APPDATA%\Hex-Rays\IDA Pro\plugins\ida-slides`); a running IDA loads
  this working tree directly.
- Both renderers are testable OUTSIDE IDA. macOS:
  `python3 tests/test_webkit_standalone.py` (needs 3.10+, PySide6, pyobjc;
  E2E: attach, marp watcher, linkify, save + rapid-save cycles, the
  JS→Python click bridge via a monkeypatched do_jump, cleanup — no IDB;
  no COM-style Part 1 on purpose: PyObjC crash safety is a structural
  rule, not a probeable surface). Windows:
  `python tests\test_webview2_standalone.py` (part 1: COM layer in a bare
  Win32 window; part 2: DeckWebView2View + real marp pipeline under Qt).
  Run it BEFORE touching vtable/COM code — a wrong slot index fails there
  cleanly instead of crashing IDA. It uses private temp user-data folders
  on purpose: a WebView2 user-data folder is owned by one host process's
  browser instance, and a second process attaching to it gets
  ERROR_INVALID_STATE (0x8007139f) — or, observed empirically, the
  controller callback simply never arrives. The view therefore runs an
  8s watchdog per attach attempt and retries once with a per-pid folder
  (swept on later attaches); attempts carry a generation counter so a
  stalled attempt completing late is dropped instead of racing the
  retry. Never pip-install/upgrade PySide6 in IDA's Python to test —
  IDA loads it.
- A live IDA is usually reachable via the `ida-pro-mcp` MCP server
  (`py_eval`) — verify UI/focus/crash behavior empirically there rather
  than reasoning about it. Reload flow: `importlib.reload` the changed
  modules, then reload `presenter_form` and reopen the form. Gotchas:
  - Reload does not delete removed attributes; injected-JS changes need
    the form reopened (the user script is baked in at webview creation).
  - Close and re-Show the form in SEPARATE py_eval calls — widget close
    is async and a same-caption collision makes `Show()` fail silently.
  - Too many reload/reopen cycles can orphan the dock tab (unclosable
    ghost); only an IDA restart clears it.
  - IDA's dock chain has no QDockWidget — never walk Qt parents calling
    `close()` without stopping before QMainWindow.
- `py_eval` uses exec-with-dict scoping: module-level names are invisible
  inside nested `def`/`lambda` — bind them via default arguments.
- Focus can be tested with IDA in the background via
  `mainwindow.focusWidget()`; `get_current_widget()` returns None when
  the app is inactive.
- Regression tests live in `tests/test_in_ida.py` (run inside IDA — see
  README "Tests"). Pure-logic checks always run; DB-dependent ones pick a
  function from the open IDB. Add a case here when fixing a logic bug.
- Owner tests UI changes himself and reports back; commit per feature
  batch (English, imperative), push only when asked.

## Outstanding cleanups

- The Windows stack was statically audited from the mac side and
  reconciled against the Windows-side review the same day — see
  `win/VERIFICATION-NOTES.md`. As of 2026-07-12 **all its findings are
  closed** (fixed and live-verified, or refuted by live test); do not
  re-fix items the notes mark resolved. Still live-only: high-DPI and
  the NTFS-junction install check.
- High-DPI on Windows is unverified: every WebView2 check so far ran on a
  100%-scale display (2026-07). On a 125%/150% monitor confirm the deck
  is crisp and fills the pane; if not, the fix is RasterizationScale /
  bounds handling in webview2_view. (marp/slidev flows, the watchdog
  retry, and the COM layer are all verified — see tests/
  test_webview2_standalone.py.)

---
> Source: [hyuunnn/ida-slides](https://github.com/hyuunnn/ida-slides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
