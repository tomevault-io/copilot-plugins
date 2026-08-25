## gpuix

> **Read [README.md](./README.md) first** to understand what GPUIX is, the architecture, mutation API, event flow, supported elements/events/styles, and the test renderer.

# AGENTS.md - GPUIX Codebase Guide

**Read [README.md](./README.md) first** to understand what GPUIX is, the architecture, mutation API, event flow, supported elements/events/styles, and the test renderer.

Not **remorses**? Do not open a pull request. Open an issue. See [External contributors](#external-contributors).

## Project Goal

GPUIX enables building **native GPU-accelerated desktop applications** using **React and TypeScript**, powered by [GPUI](https://github.com/zed-industries/zed/tree/main/crates/gpui) (Zed's rendering framework).

Instead of Electron/web rendering, your React components render directly to the GPU via Metal/Vulkan.

```
React (TypeScript)  →  napi-rs  →  GPUI (Rust)  →  GPU
     Your code         Bridge      Native render    Metal/Vulkan
```

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  JavaScript / TypeScript                                        │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Your React App                                          │   │
│  │                                                          │   │
│  │  function App() {                                        │   │
│  │    const [count, setCount] = useState(0)                 │   │
│  │    return (                                              │   │
│  │      <div style={{ display: 'flex', bg: '#1e1e2e' }}>    │   │
│  │        <div onClick={() => setCount(c => c + 1)}>+</div> │   │
│  │      </div>                                              │   │
│  │    )                                                     │   │
│  │  }                                                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ↓                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  @gpuix/react (packages/react)                           │   │
│  │                                                          │   │
│  │  - React Reconciler (react-reconciler)                   │   │
│  │  - Builds element tree from React components             │   │
│  │  - Serializes to JSON ElementDesc                        │   │
│  │  - Manages event handler registry                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ↓ JSON                             │
└─────────────────────────────────────────────────────────────────┘
                               ↓ napi-rs FFI
┌─────────────────────────────────────────────────────────────────┐
│  Rust / Native                                                  │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  @gpuix/native (packages/native)                         │   │
│  │                                                          │   │
│  │  - GpuixRenderer: receives JSON, triggers re-render      │   │
│  │  - build_element(): ElementDesc → GPUI elements          │   │
│  │  - apply_styles(): StyleDesc → GPUI style methods        │   │
│  │  - Event handlers → ThreadsafeFunction callbacks to JS   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ↓                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  GPUI (from zed)                                         │   │
│  │                                                          │   │
│  │  - Immediate-mode UI framework                           │   │
│  │  - Flexbox layout via Taffy                              │   │
│  │  - GPU rendering via Metal (macOS) / Vulkan (Linux)      │   │
│  │  - Native window management                              │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## Key Insight: Immediate Mode Alignment

GPUI is **immediate-mode** - it rebuilds the entire UI tree every frame. This actually aligns perfectly with React's model:

| Traditional DOM Renderer | GPUIX |
|--------------------------|-------|
| `appendChild(node)` | Rebuild tree each render |
| `node.style.color = x` | Send full tree description |
| Mutation-based | Description-based |

We don't fight GPUI's architecture - we embrace it by sending a complete element description on every React render.

## Package Structure

```
gpuix/
├── packages/
│   ├── native/                 # Rust napi-rs bindings
│   │   ├── src/
│   │   │   ├── lib.rs          # Module exports
│   │   │   ├── renderer.rs     # GpuixRenderer, GpuixView, build_element()
│   │   │   ├── element_tree.rs # ElementDesc, EventPayload types
│   │   │   ├── style.rs        # StyleDesc, color parsing
│   │   │   ├── theme.rs        # Comet palette, oklch helpers, JS overrides
│   │   │   ├── text/           # Selection: state, paint registry, TextRuns
│   │   │   ├── syntax/         # Tree-sitter highlighting + bounded cache
│   │   │   ├── markdown/       # pulldown-cmark parser + gpui renderer
│   │   │   ├── diff/           # Unified-patch parser + row flattening
│   │   │   └── custom_elements/# input, img, svg, anchored, code, diff, markdown
│   │   ├── examples/
│   │   │   └── hello.rs        # Pure GPUI test (no JS)
│   │   ├── Cargo.toml
│   │   └── build.rs
│   │
│   └── react/                  # React reconciler
│       ├── src/
│       │   ├── index.ts        # Public exports
│       │   ├── reconciler/
│       │   │   ├── host-config.ts  # React reconciler implementation
│       │   │   ├── reconciler.ts   # ReactReconciler instance
│       │   │   └── renderer.ts     # createRoot(), event bridge
│       │   ├── hooks/
│       │   │   ├── use-gpuix.ts    # Context access
│       │   │   └── use-window-size.ts
│       │   └── types/
│       │       └── host.ts     # TypeScript types
│       └── package.json
│
├── examples/
│   ├── package.json            # Workspace package for examples
│   └── counter.tsx             # Example React app
│
└── AGENTS.md                   # This file
```

## Text rendering: one funnel, no exceptions

Every string GPUIX paints goes through `crate::text`:

- `selectable_text(..)` for content — registers into the per-frame selection
  registry and installs the window mouse and key listeners
- `chrome_text(..)` for line numbers, language tags and file headers — painted
  and logged for tests, but never part of a selection

**Never call `div().child(some_string)` in a new element.** Doing so makes the
text invisible to selection AND to `getPaintedText()`, so it cannot be tested
except by screenshot.

The registry is rebuilt during **paint**, not during build, because paint order
is the only place document order is guaranteed: a `list()` decides at paint time
which rows exist. `selection_frame_reset()` must stay the first child of the
root, or stale entries from the previous frame leak into the next drag.

## Layout numbers live in `Theme::metrics`, not in Rust constants

Row heights, gutter widths, paddings, text sizes and the heading scale are all
fields on `crate::theme::Metrics`, reachable from JS as `theme.metrics`.

**Do not add a new `const` for anything that decides layout.** Put it on
`Metrics`, give it a default, add it to `MetricsOverride`, `hash_into`, and the
`GpuixMetrics` TypeScript interface. The whole point is that a design tweak is a
React re-render, not a native rebuild.

Two things stay constant, because they are paint geometry and cannot move a
glyph: the table hairline, and the inline-code wash overhang.

`<diff>` derives its virtualized height model from the metrics without
measuring, so `DiffElement` re-runs `reset_with_uniform_height` whenever
`Metrics::hash_into` changes. Forget that and the scrollbar drifts from the
content.

## Iterating on the Rust side

There is no hot reload and there cannot be: `require()` of a `.node` calls
`process.dlopen`, Node has no unload, and the event loop, GPU device, window and
selection registry all live in thread-locals of the loaded library.

Use `bun run dev` (see `scripts/dev.ts`). It watches `packages/native/src`,
rebuilds, and re-renders the screenshot tests. **A Rust edit reaches fresh PNGs
in about 4 seconds.** Prefer screenshot mode over `--app`: PNGs in
`packages/react/screenshots/` can be read by an agent, a live window cannot.

**Never ship or start the app on a debug native build.** `bun run build:debug`
and `cargo build` without `--release` produce an unoptimized `.node`. GPUI
paint is then many times slower, and that looks like an app bug. Always use
`bun run build` in `packages/native` (release). Use `build:debug` only when
the user asks, or when a debug-only tool (lldb, sanitizers) cannot run on
release. After any debug build, rebuild release before starting `chat.tsx`
or judging frame time.

## Virtualized React children re-enter through `cx.processor`

`<virtual-list>` does not build its retained children during `GpuixView::render`.
Its `gpui::list()` callback uses `cx.processor` to re-enter the `GpuixView`
entity after the root render has returned, creates a fresh `BuildCtx`, and builds
only the rows GPUI requests. Never capture the root render's tree guard or
`BuildCtx` in that callback.

`<diff>` still owns its parsed Rust data because one native diff node is much
cheaper than retaining one React node per line.

## Nested scrolling is not supported

Never put a scroll container inside another scroll container. That includes
`overflow: "scroll"`, `<virtual-list>`, and `<diff>` (`gpui::list()` always
takes the wheel). GPUI delivers the same wheel event to both hitboxes. The
inner list steals the gesture. Nested scroll looks broken and there is no
GPUI API to turn list scroll off.

Keep **one** scroll parent. Long inner content must grow with that parent, or
collapse behind an expandable (file header, first N lines, Show more). `<diff>`
defaults to flow layout. Pass `scroll` plus a bounded height only for a
dedicated viewer. Do not give `<diff>` a bounded height inside a parent
scroller just so it can virtualize.

`overflow-x: scroll` is allowed inside a vertical scroller. GPUI remaps a
vertical wheel onto overflow-x unless `restrict_scroll_to_axis()` is set.
Every `overflow_x_scroll()` in native code must call that, or the parent
scroller jumps sideways when the pointer is over `<code>` or a markdown table.

## Scroll cost

A wheel event calls `cx.notify` on the one `GpuixView`. That rebuilds the
tree. `gpui::list()` then re-renders every **visible** item. Cached heights
only skip overdraw items that are off screen.

```
wheel  ►  notify GpuixView  ►  render()  ►  Taffy on visible rows  ►  paint
```

If scroll is smooth on empty padding and slow or stuck on text, a filled
child is stealing the wheel. `occlude()` is **BlockMouse**. It stops the
hit test. The parent list never sees the event. In-flow fills must use
`block_mouse_except_scroll()`. Keep `occlude()` for absolute/fixed overlays
and `pointerEvents: "auto"`.

The chat "jank" over code and tables was the Y-to-X remap above, not the
tick loop. After that fix, remaining cost is Taffy on fat visible rows.
`<code>` is one flex row per line. Safe-mdx is ~100 host nodes. Flatten
paint before changing the frame loop.

Keep `<virtual-list>` `overdraw` modest. 820px on a short chat kept almost
every row live. Profile with `debugFrameOverlay: 'full'`. The overlay is
draw time, not FPS. `8.3 MS` is about 120 Hz.

A long `{rows.map(...)}` is slow **at start**. `createInstance` runs in the
render phase. Use `VirtualList` with `itemCount` and `renderItem` so React
only mounts the visible window. The host `<virtual-list>` children API still
retains every child. After mount, scroll cost is visible Taffy only.

Keep chrome state out of the component that maps the list. `memo(Transcript)`
so a sidebar click or composer keystroke does not remap every row. A 5k-row
chat paid 250ms per click before that. Profile that path with
`INTERACT=1 bun profile-chat-scroll.tsx`. Do not treat a fast wheel flush as
proof that chrome updates are cheap.

## Profiling and optimizing

Load the **profano** skill first. Fetch its README. Do not guess CLI flags.

Separate **first mount**, **scroll**, and **chrome setState**. They are
different paths.

```
first mount
  React maps every child
    ►  createInstance / setStyle / setCustomProp  (queued)
    ►  one applyBatch JSON
    ►  Rust RetainedTree
    ►  first paint (list builds visible rows only)

scroll
  wheel  ►  notify GpuixView  ►  render()  ►  Taffy on visible rows  ►  paint

chrome setState
  sidebar click / composer key
    ►  parent re-render
    ►  {rows.map(...)} again unless memo(list)
    ►  same JS cost as mount if you forget
```

### JS / mount

Write a short script that mounts through `createTestRoot()` and exits. Profile
that, not the live window. The tick loop will drown the mount.

```ts
import React from 'react'
import { createTestRoot } from '@gpuix/react'
import { ChatApp } from './chat'

const root = createTestRoot()
const start = performance.now()
root.render(<ChatApp turnCount={10_000} />)
console.log(`mount ${(performance.now() - start).toFixed(1)}ms`)
```

```bash
cd examples
MOUNT_ONLY=1 bun --cpu-prof --cpu-prof-dir=../tmp/cpu-profiles profile-chat-scroll.tsx
INTERACT=1 bun profile-chat-scroll.tsx
npx profano ../tmp/cpu-profiles/CPU.*.cpuprofile -n 30
npx profano ../tmp/cpu-profiles/CPU.*.cpuprofile --sort total -n 20
```

Read **self** first. That is where the CPU sat. **Total** is the caller chain.

The 10k chat mount was 850ms. profano said:

| Function | Self | What it was |
|---|---|---|
| `applyBatch` | 626ms | Rust parsing the mutation JSON |
| `FiberNode` | 31ms | React |
| `stringify` | 26ms | `JSON.stringify(queue)` |

React was not the problem. The batch **stringified every style and theme**, then
stringified the queue, then Rust parsed each escaped string again.

Queue **raw objects**. Opcode `setCustomPropValue` carries a raw JSON value.
`setCustomProp` still means a JSON **string** (legacy). A raw `"top"` or
`"true"` on `setCustomProp` is parsed as JSON and throws. That is why the
composer Selects died after the first applyBatch change: `<anchored side="top">`
never committed.

```ts
queue.push(['setStyle', id, styleObject])
queue.push(['setCustomPropValue', id, 'side', 'top'])
```

After a JS reconciler change, **build `@gpuix/react`**. `examples/` and
`bun --hot chat.tsx` load `packages/react/dist`, not `src`. packages/react
vitest uses `src`. You will think the fix works in one suite and fail in the
app.

```bash
cd packages/react && bun run build
```

### Scroll / paint

Turn on `debugFrameOverlay: 'full'`. The number is **draw time**, not FPS.
`8.3 MS` is about 120 Hz.

The chat wheel jank was **not** the tick loop. GPUI remaps a vertical wheel
onto `overflow-x`. `<code>` and markdown tables stole the gesture. Fix is
`restrict_scroll_to_axis()` on every `overflow_x_scroll()`.

Keep `overdraw` modest. 820px on a short list keeps almost every row live.

Do not flatten the frame loop to hide fat rows. Flatten the rows
(`<markdown>` / `<code>` / `<diff>` as one native node).

### Native

For Rust time, `sample` the bun/node pid, or `samply`. GPUI also has
`ZED_MEASUREMENTS=1`. That is Zed's frame log, not our overlay.

A `.node` cannot unload. After a native rebuild, restart the app. `bun --hot`
only remounts React.

## Overlays and icons

Menus, tooltips, and dialogs go through **`SelectContent` / `ComboboxContent` /
`<anchored deferred>`**. Never overflow a `position: "absolute"` card out of the
composer into a `<virtual-list>`. The list paints after the composer, so the
list shows through the menu and clicks hit the text behind it.

Do not paint `#00000000` over a blurred window. A transparent GPUI quad punches
through Metal to the desktop. Omit the fill, or use the parent color. Overlay
rows need a **solid** fill too, not a transparent idle state.

A filled in-flow `div` uses **BlockMouseExceptScroll**. Clicks and hovers stop.
The parent scroller still gets the wheel. `position: "absolute"` / `"fixed"`
or `pointerEvents: "auto"` uses **BlockMouse** and steals the wheel too.
`pointerEvents: "none"` opts out.

Text **selection** still uses window mouse events and text bounds, not hitboxes.
A drag on a menu over markdown can still start a selection. Do not skip
selection tests to hide that.

If `<svg>` icons are blank in vitest, `src` is probably a `data:image/svg+xml`
URL from `import … with { type: 'file' }`. Native decodes that URL. Do not write
a temp-file workaround. Prefer `fill="#000"` / `stroke="#000"` plus
`style.color`. `currentColor` in the file is not `style.color`.

macOS traffic-light clearance is **86px**. The test renderer does not draw
traffic lights, so that gap looks empty in PNGs.

## Ported code

`text/`, `syntax/`, `markdown/`, `diff/`, `theme.rs`, `custom_elements/code.rs`,
`custom_elements/diff.rs`, and the caret blink sections of
`custom_elements/input.rs` are ported from [Comet](https://github.com/zeronsh/comet)
(MIT). Each file names its original in
its header, and `THIRD_PARTY_NOTICES.md` has the full table. When fixing a bug in
one of them, read the Comet original first: it usually documents why the code is
shaped that way.

## Auto-generated files (do NOT edit manually)

The following files in `packages/native/` are auto-generated by napi-rs during `bun run build`. Never edit them by hand — they are regenerated from the Rust `#[napi]` annotations every build:

- `packages/native/index.d.ts` — TypeScript type declarations
- `packages/native/index.js` — Node.js loader/binding glue
- `packages/native/*.node` — compiled native binary

To update the TypeScript API surface, edit the Rust source files in `packages/native/src/` (add/modify `#[napi]` structs, methods, functions), then run `bun run build` in `packages/native` to regenerate.

## Changesets

**Always** add a `.changeset/*.md` file after a user-facing fix or feature. Do this before you consider the work done. Never skip it. Never edit CHANGELOG.md. Never bump `package.json` version by hand.

Load the `changesets` skill for format and rules. If the change fixes a GitHub issue or should close a PR, put `Fixes #N` / `Closes #N` on its own line. changepub copies those onto the release commit.

## Publishing

**Never publish from a local machine.** CI is the only release path.

`.github/workflows/ci.yml` builds `@gpuix/native` for every napi target (macOS arm64/x64, Linux x64/arm64, Windows x64/arm64), uploads the `.node` artifacts, then the `publish` job downloads them, runs `napi create-npm-dirs` + `napi artifacts`, and publishes `@gpuix/native` and `@gpuix/react`.

Each build job also compiles `examples/chat.tsx` with `bun build --compile` against that target's `.node`, and uploads `example-chat-<target>`. On `main`, the publish job attaches those binaries to the `@gpuix/react@x.y.z` GitHub release.

Publish order is required. `@gpuix/react` depends on `@gpuix/native` (`workspace:^`). If React publishes first, an install in that window cannot resolve native.

1. `napi pre-publish` publishes the per-platform packages (`darwin-arm64`, `linux-x64-gnu`, …)
2. `npm publish` publishes `@gpuix/native`
3. `npm publish` publishes `@gpuix/react`

A local `npm publish` / `bun publish` would ship only the host binary and break every other platform. `prepublishOnly` exits if `CI` is unset.

To release: bump versions via changesets, push to `main`. The publish job skips versions already on npm.

## Communication Flow

### Render Flow (JS → Rust)

```
1. React state changes
         ↓
2. React reconciler builds Instance tree
         ↓
3. instanceToElementDesc() converts to JSON-serializable format:
   {
     type: "div",
     id: "btn-1", 
     style: { display: "flex", backgroundColor: "#ff0000" },
     events: ["click", "mouseEnter"],
     children: [...]
   }
         ↓
4. renderer.render(JSON.stringify(tree))
         ↓
5. Rust parses JSON into ElementDesc structs
         ↓
6. build_element() recursively builds GPUI elements:
   div().id("btn-1").flex().bg(rgba(0xff0000ff)).on_click(...)
         ↓
7. GPUI renders to GPU
```

### Event Flow (Rust → JS)

```
1. User clicks element with id="btn-1"
         ↓
2. GPUI fires click event on element
         ↓
3. Rust closure calls emit_event("btn-1", "click", position)
         ↓
4. ThreadsafeFunction calls into JS with EventPayload
         ↓
5. JS event registry looks up handler:
   eventHandlers.get("btn-1")?.click?.(event)
         ↓
6. React handler runs: onClick={() => setCount(c => c + 1)}
         ↓
7. State update triggers re-render → back to Render Flow
```

## Key Types

### ElementDesc (Rust ↔ JS)

```rust
pub struct ElementDesc {
    pub element_type: String,      // "div", "text", "img"
    pub id: Option<String>,        // For event handling
    pub style: Option<StyleDesc>,  // CSS-like styles
    pub content: Option<String>,   // Text content
    pub events: Option<Vec<String>>, // ["click", "mouseEnter"]
    pub children: Option<Vec<ElementDesc>>,
}
```

### StyleDesc (CSS-like properties)

```rust
pub struct StyleDesc {
    // Flexbox
    pub display: Option<String>,        // "flex"
    pub flex_direction: Option<String>, // "row", "column"
    pub align_items: Option<String>,    // "center", "start", "end"
    pub justify_content: Option<String>,
    pub gap: Option<f64>,
    
    // Sizing
    pub width: Option<DimensionValue>,
    pub height: Option<DimensionValue>,
    
    // Spacing
    pub padding: Option<f64>,
    pub margin: Option<f64>,
    
    // Colors (parsed centrally in src/color.rs with csscolorparser 0.8.3;
    // parser-version changes require running both absolute and relative matrices)
    pub background_color: Option<String>,
    pub color: Option<String>,
    
    // Border
    pub border_radius: Option<f64>,
    pub border_width: Option<f64>,
    pub border_color: Option<String>,
}
```

### EventPayload (Rust → JS)

```rust
pub struct EventPayload {
    pub element_id: String,
    pub event_type: String,  // "click", "mouseEnter", etc.
    pub x: Option<f64>,
    pub y: Option<f64>,
    pub key: Option<String>,
    pub modifiers: Option<EventModifiers>,
}
```

## Building

### Standalone Build

The `zed/` submodule tracks the `gpui-macos-embedded` branch of `remorses/zed`. Cargo uses path
dependencies from that submodule so the native addon and native platforms always
compile from the same source:

- macOS uses `MacPlatform::new_embedded()` and pumps AppKit on Node's main thread
- Windows and Linux run `gpui_platform::application().run()` on a dedicated UI thread
- `gpui_macos` is a direct macOS dependency for production and the GPU-backed test renderer
- `core-text = 21.0.0`, `core-graphics = 0.24.0` for macOS

These avoid the core-graphics 0.24 vs 0.25 conflict between `core-text` and Zed's `font-kit` fork.

### Rust toolchain

`rust-toolchain.toml` pins the same channel as `zed/rust-toolchain.toml`. When the
submodule moves, update ours to match or GPUI may not compile.

### Metal toolchain (macOS)

`gpui_apple` compiles `shaders.metal` in its build script. Xcode 26 no longer ships the
Metal compiler by default, so a fresh machine fails with
`cannot execute tool 'metal' due to missing Metal Toolchain`. Install it once:

```bash
xcodebuild -downloadComponent MetalToolchain
```

### Bumping the gpui revision

1. Merge upstream Zed into the `gpui-macos-embedded` branch in `remorses/zed`.
2. Resolve any embedded `gpui_macos` conflicts in a new commit; do not rewrite history.
3. Fast-forward the `zed/` submodule to the updated `gpui-macos-embedded` branch.
4. Match `rust-toolchain.toml` to `zed/rust-toolchain.toml`.
5. Run `cargo check --all-targets`, `bun run build`, and the test suites.

### PRs to Zed

A "PR to Zed" means **upstream** [`zed-industries/zed`](https://github.com/zed-industries/zed)
`main`. Never open that PR from this checkout. Never point it at `remorses/zed`.

Do **not** branch, commit review markers, or reset `zed/` inside this checkout.
That submodule is what GPUIX builds against. A dirty or switched `zed/` breaks
the native addon and the test renderer.

```bash
# from gpuixlocal/zed. leaves this submodule on its current commit
git remote add upstream https://github.com/zed-industries/zed.git  # once
git fetch upstream
git worktree add /Users/morse/Documents/GitHub/zed-<branch-name> -b <branch-name> upstream/main
```

Commit only in that worktree. Do not add comments to Zed source. Push the branch
to `remorses/zed`, then open the PR with `--repo zed-industries/zed --base main`.
After merge, cherry-pick onto `gpui-macos-embedded` and fast-forward the submodule
here. Never run `git reset` in `zed/` to "undo" PR work.

### PRs to GPUIX

When you open a PR with `gh pr create` against **this repo** (`remorses/gpuix`),
the body must name the **harness**, **agent**, and **model** that wrote the
change. Then put **every user prompt** from the session in a collapsed
`<details>` block. Reviewers use that to judge prompt quality and how much
the agent invented.

Do this for `gh pr create` and for later `gh pr edit` if the first body missed
it. Do not add this block to Zed PRs.

```md
**Harness:** OpenCode / Kimaki
**Agent:** build
**Model:** xai/grok-4.6

<details>
<summary>User prompts</summary>

1. first user message, verbatim

2. second user message, verbatim

</details>
```

- **Harness:** the product that ran the agent. Examples: OpenCode, Kimaki,
  Claude Code, Cursor, Codex.
- **Agent:** the named agent if the harness has one (`build`, `plan`, `opus`).
  Write `none` if there is no named agent.
- **Model:** the exact model id from the session (`xai/grok-4.6`,
  `anthropic/claude-opus-4.6`). Do not guess a shorter marketing name.
- **User prompts:** every user message that drove the PR, in order, verbatim.
  Skip system reminders, tool output, and your own replies. If a prompt is
  huge, keep the full text inside the details block; do not summarize it.

## Current Status

Keep this list in sync with the README **Status** section. User-facing APIs
belong in README. This list is only the remaining engineering work.

### Completed

- [x] React reconciler with mutation-based protocol
- [x] napi-rs FFI bindings and RetainedTree
- [x] Style mapping, including native `hover` / `active`
- [x] Mouse, keyboard, focus, scroll, and click-outside events
- [x] `commitMutations()` stores the view entity and calls `cx.notify()`
- [x] GPU-backed test renderer
- [x] Native `<input>` and `<textarea>`
- [x] `<img>` (local raster/SVG) and `<svg>` (tintable monochrome icons)
- [x] `<virtual-list>`
- [x] `<code>`, `<diff>`, `<markdown>` with Tree-sitter
- [x] Cross-element text selection
- [x] Headless Select, Combobox, Tooltip
- [x] `setWindowTitle`
- [x] Window chrome (`titlebarTransparent`, `windowBackground`, traffic-light position)
- [x] Last window close quits the process
- [x] Debug frame overlay (`setDebugFrameOverlay`)

### TODO

#### High Priority

- [ ] **Background highlighting** - move Tree-sitter off the frame thread once
      there is a way to request a repaint from a background task

#### Medium Priority

- [ ] **Canvas** - custom drawing element (`<canvas>` is typed, not implemented)

#### Low Priority

- [ ] **Window controls** - resize, minimize (title already works)
- [ ] **Multiple windows** - Support multiple GPUI windows
- [x] **JS remount** - `render()` plus `bun --hot` remounts the React tree on the same window
- [ ] **React Refresh** - keep `useState` across saves. Needs Bun to run the Fast Refresh transform during `bun --hot`
- [ ] **Native hot reload** - cannot unload a `.node`. `bun run dev` rebuilds and restarts
- [ ] **DevTools** - React DevTools integration
- [ ] **Animations** - Interpolated style transitions

## Testing

### Unit Tests

```bash
# Rust unit tests (selection, syntax, diff parser, markdown parser, theme)
cd packages/native && cargo test --lib

# React reconciler + GPU-backed test renderer
cd packages/react && bun run test

# Example app tests
cd examples && bun run test

# Chat draw / chrome regression (same suite, file filter)
cd examples && bun run test chat.perf.test.tsx

# macOS CPU clamp. E-cores, not Chrome 6x. Do not set in CI.
THROTTLE=utility bun run test chat.perf.test.tsx
THROTTLE=utility bun profile-chat-scroll.tsx
THROTTLE=utility bun --hot chat.tsx
```

`examples/chat.perf.test.tsx` is the automated profile. It uses `createTestRoot()`,
not the live window. Assert **p95 draw / flush ms**, not a per-frame FPS floor.

`THROTTLE` re-execs under `taskpolicy -c`. `utility` is an M1/M2 Air CPU proxy.
`background` is harsher, closer to a 2019 Intel Mac. GPU and RAM stay on this
machine. `taskpolicy -c` only works at launch. The vitest config wraps the main
process so workers inherit the clamp. A throttled run **logs** numbers and
skips the default budgets. Those budgets are for an unclamped M-series CPU.

Use `bun run test`, not `bun test`. The suites are vitest, so `bun test` picks the
wrong runner and fails on the `vitest` imports.

### Asserting on native elements

`getAllText()` reads the retained tree, so it only sees `<text>` nodes. `<code>`,
`<diff>` and `<markdown>` paint inside gpui and are invisible to it. Use
`renderer.getPaintedText()` (every string painted last frame, in paint order) and
`renderer.dragSelect(x1, y1, x2, y2)` instead.

`dragSelect` exists because selection listeners are registered during **paint**:
calling `simulateMouseDown` / `Move` / `Up` by hand without a flush between each
step silently selects nothing.

Screenshots go to `packages/react/screenshots/` (gitignored), not `/tmp`, so they
can be inspected after a run.

### Integration Test

```bash
cd examples && bun --hot chat.tsx
```

Use tuistory for the long-running process. Do not use `tsx` or raw `tmux`.

### Drive the live window

**Do not use `usecomputer`, `screencapture`, or desktop clicks.** GPUIX has a
Playwright-like automation API. Full docs are in the README **Automation**
section.

Mark targets with `testId`. Then either:

- `connectTest(renderer)` against `createTestRoot()` in vitest
- `launch({ command, args })` against a child process. The app serves commands
  on stdin only when stdin is a **pipe**

```ts
import { launch } from '@gpuix/react/automation'

const app = await launch({ command: 'bun', args: ['chat.tsx'], cwd: 'examples' })
await app.getByTestId('sidebar-collapse').waitFor({ timeoutMs: 30_000 })
await app.screenshot({ path: 'tmp/chat.png' })

const startedAt = await app.clock.pause()
await app.getByTestId('sidebar-collapse').click()
await app.captureFrames('tmp/sidebar', [startedAt, startedAt + 100, startedAt + 200])
await app.clock.resume()
await app.close()
```

`click()` hits the last painted bounds. `clock.pause` / `set` / `fastForward`
freeze native motion. `captureFrames` writes one PNG per timestamp. That is how
you record a sidebar open/close, not a screen recorder.

## Related Projects

- [GPUI](https://github.com/zed-industries/zed/tree/main/crates/gpui) - Zed's GPU UI framework
- [opentui](https://github.com/anomalyco/opentui) - Terminal UI with React (reconciler reference)
- [create-gpui-app](https://github.com/zed-industries/create-gpui-app) - Official GPUI starter template
- [react-reconciler](https://github.com/facebook/react/tree/main/packages/react-reconciler) - React's custom renderer API

## External contributors

This section is for anyone who is not [remorses](https://github.com/remorses) (Tommy).

**Do not open a pull request.** Open a GitHub issue. Describe the bug or the idea. Wait.

Open a PR only after remorses says it is OK on that issue. Unsolicited PRs will be closed.

If remorses says OK, follow the rest of this file and these rules.

**How to work**

1. For Rust changes, work in `zed/crates/gpuix` (easier to build)
2. Copy changes to `packages/native/src/` when ready
3. TypeScript changes can be made directly in `packages/react/`

**Do not**

- Edit auto-generated files: `packages/native/index.d.ts`, `packages/native/index.js`, `packages/native/*.node`. Change the Rust `#[napi]` source and run `bun run build` in `packages/native`
- Edit `CHANGELOG.md` or bump `package.json` version by hand
- Publish from a local machine. CI is the only release path
- Branch, commit, or reset the `zed/` submodule in this checkout. Do not open a Zed PR from here
- Ship or start the app on a debug native build. Use `bun run build` in `packages/native` (release)
- Use `bun test`. The suites are Vitest. Use `bun run test`

**Must**

- Add a `.changeset/*.md` file for every user-facing fix or feature. Put `Fixes #N` on its own line when the work closes an issue
- Run the package test scripts: `packages/react` then build `@gpuix/react`, then `examples`
- Keep one scroll parent. Nested scrolling is not supported
- Send every painted string through `crate::text`. Never `div().child(some_string)`
- Put layout numbers on `Theme::metrics`, not new Rust constants

If an agent writes the change, the PR body must include **harness**, **agent**, **model**, and every user prompt. See **PRs to GPUIX**.


## Examples using same tech as ours. To unblock on issues and compare to our code

For example usage of projects depending on gpui in rust: opensrc https://github.com/zed-industries/create-gpui-app

For examples of NAPI rs native packages: https://github.com/napi-rs/package-template and https://github.com/Brooooooklyn/Image

For reading gpui source code: https://github.com/zed-industries/sed inside crates/gpui

For examples of a custom React renderer: https://github.com/anomalyco/opentui inside packages/react

---
> Source: [remorses/gpuix](https://github.com/remorses/gpuix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
