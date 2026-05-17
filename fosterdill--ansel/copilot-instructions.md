## ansel

> A desktop disk usage visualizer — think "Disk Inventory X with a cleanup workflow." Scans a volume, renders contents as a squarified treemap, sunburst diagram, or flat table — colored by file type or age — and lets you queue files for Trash/permanent deletion. Built with **Tauri v2** (Rust backend + React/TypeScript frontend). Cross-platform: macOS, Linux, Windows.

# CLAUDE.md

## Project: Ansel

A desktop disk usage visualizer — think "Disk Inventory X with a cleanup workflow." Scans a volume, renders contents as a squarified treemap, sunburst diagram, or flat table — colored by file type or age — and lets you queue files for Trash/permanent deletion. Built with **Tauri v2** (Rust backend + React/TypeScript frontend). Cross-platform: macOS, Linux, Windows.

## Git workflow (IMPORTANT)

**Direct push to `main` is blocked.** `main` has branch protection enabled:

- **Require a pull request before merging** — all changes must go through a feature branch → PR → merge
- **Require status checks to pass** — CodeQL must succeed before merge
- **Only squash merges** are allowed (default merge method is squash)
- **No force pushes, no deletions** on `main`
- **Admins included** — no bypass allowed

**How to land changes:**

```bash
git checkout -b <branch-name>
# make changes, commit
git push -u origin <branch-name>
# Create a PR on GitHub
# Self-approve the PR (review → approve)
# Wait for CodeQL to pass (~2 min)
# Squash & merge
```

**CRITICAL: Before pushing to any branch, verify the associated PR is not already merged.** Run `gh pr view <branch> --json state` or check the PR URL. If the PR is merged, create a new branch from `main` (after `git pull`) instead of pushing to the old one.

## Supply-chain / security

- **OpenSSF Scorecard** — runs automatically on every push to `main` (`.github/workflows/scorecard.yml`)
- **CodeQL SAST** — runs on push to `main`, PRs into `main`, and weekly (`.github/workflows/codeql.yml`)
- **Dependabot** — enabled for automated dependency updates (`.github/dependabot.yml`)
- **Build provenance** — release workflow attests build provenance via `actions/attest-build-provenance`
- **Security policy** — [`SECURITY.md`](SECURITY.md) with reporting instructions
- **All GitHub Actions pinned by commit hash** (not version tags)
- **Top-level `permissions: read-all`** on all workflows; write permissions only at job level where needed

## Build & Dev

```bash
pnpm install          # first time only
pnpm tauri dev        # hot-reload frontend, incremental Rust rebuilds
pnpm tauri build      # production build → src-tauri/target/release/bundle/
```

- Requires: **Rust stable**, **Node 20+**, **pnpm**
- Frontend-only dev (without Tauri): `pnpm dev` (Vite only, no Rust backend)
- First cold-compile of Rust backend: ~30–60s on Apple Silicon

## Architecture

### Two halves

| Layer | Path | Language | Role |
|-------|------|----------|------|
| Backend | `src-tauri/` | Rust | File scanning (rayon-parallel), tree mutation, Tauri IPC commands |
| Frontend | `src/` | TypeScript + React 18 | UI, treemap rendering, state management, IPC calls via `@tauri-apps/api` |

### Backend (`src-tauri/src/`)

- **`lib.rs`** — Tauri app setup, all `#[tauri::command]` handlers, shared `AppState` (holds the scan tree in an `Arc<RwLock<Option<Node>>>`). Handlers: `start_scan`, `cancel_scan`, `get_node`, `get_subtree`, `list_children`, `top_files`, `extension_breakdown`, `move_to_trash`, `delete_paths`, `delete_path`, `show_in_file_manager`, `list_volumes`, `get_volume`, `home_dir`, `search`. Volume listing includes `total_bytes`/`free_bytes` via `disk_space()` (statvfs on Unix, GetDiskFreeSpaceExW on Windows). `get_volume` re-queries disk space for a single path (used to refresh after scan). Uses `tauri-plugin-opener` for file manager reveal.
- **`scanner.rs`** — The scan engine. `scan()` recursively walks the filesystem using **rayon parallel iteration** over directory entries. Nodes accumulate `size` (on-disk blocks × 512 on Unix, not apparent size), `file_count`, `mtime`, and `atime`. Directory `mtime`/`atime` are propagated as max-of-children up the tree. Helper fns: `find()`, `top_files()`, `extension_breakdown()`, `is_protected_path()` (prevents deletion of system directories like `/System`, `/Library`, `C:\Windows`, etc.).
- **`main.rs`** — Just calls `lib::run()`.

Key types:
- `Node` — full tree node (children inline), not sent over IPC (children skipped in serialization)
- `NodeSummary` — lightweight node sent to frontend (has `child_count` but no `children`)
- `SubtreeNode` — a subtree for treemap rendering, with `children` included
- `AppState` — `root: Arc<RwLock<Option<Node>>>` + `cancel: Mutex<Option<Arc<AtomicBool>>>`
- Uses **rayon global thread pool** (configured in `setup()` to `num_cpus - 2`, min 2 threads)

### Frontend (`src/`)

- **`App.tsx`** — Layout shell: three-column layout (sidebar, center, rightbar), toolbar controls. Manages side effects: Tauri event listeners, keyboard shortcuts, welcome→main cross-fade, sidebar resize. **State lives in the zustand store, not here.**
- **`lib/store.ts`** — **Zustand store** containing all application state and actions. State is organized into logical groups: scan lifecycle, navigation (focused path/subtree), selection (selectedNodes, markedNodes with synchronized `Set<string>` paths), UI (sidebar width, expanded paths, modals). Actions include scan/cancel, fetch focused node+subtree, select/mark/mark-folder/select-folder, trash/delete batch operations with tree mutation via `applyBatchResult`. Exports utility functions: `parentPath`, `legendCategories`. Uses a module-level `fetchSeq` counter to abort stale async fetches.
- **`lib/api.ts`** — Typed wrappers around `invoke()` for every Tauri command. Defines shared types: `NodeSummary`, `SubtreeNode`, `Volume`, `DeleteBatchResult`, etc.
- **`lib/treemap.ts`** — Squarified treemap layout algorithm (`squarify()`), takes `TreemapItem<T>[]` + bounding `Rect`, returns `LaidOut<T>[]` with computed rects.
- **`lib/color.ts`** — Color utilities: FNV-1a hash, HSL jitter for adjacent-files-in-same-category distinction, age heatmap (blue→red gradient), age bucket labels, `computeFileTimeRanges()` for unified treemap/sunburst age/access color scaling.
- **`lib/fileTypes.ts`** — `CATEGORIES` map (14 categories with hex colors), `EXT_TO_CATEGORY` mapping ~120 extensions to categories, `categoryForNode()` / `colorForNode()` helpers.
- **`lib/format.ts`** — `formatBytes()` (platform-aware: decimal on macOS to match Finder, binary on other platforms to match Windows Explorer/Linux conventions) and `formatCount()`.

### Components (`src/components/`)

| Component | Purpose |
|-----------|---------|
| `Treemap.tsx` | Canvas-rendered squarified treemap. Handles click/⌘-click/⌥-click, renders rects with jittered colors. Category pills and access info in tooltips |
| `Sunburst.tsx` | Canvas-rendered concentric-ring sunburst. Polar layout with polar hit-testing. Click center to zoom out, double-click segment to zoom in. Three color modes (type/age/access); folder mode excluded since hierarchy is already spatial |
| `SearchBar.tsx` | Search input above the tree sidebar. Walks the in-memory scan tree for case-insensitive name matches, sorted by size. Results dropdown with keyboard nav (arrows/enter/escape) |
| `TreeView.tsx` | Lazy-loaded file tree sidebar. Auto-expands to reveal selections |
| `LargestFiles.tsx` | Flat sortable table of biggest files in current folder |
| `InfoPanel.tsx` | Inspector: shows selected item details, per-selection actions (Reveal in File Manager, Mark for cleanup). Trash/delete only via cleanup queue |
| `Breadcrumbs.tsx` | Clickable path breadcrumbs for zooming out |
| `CleanupQueue.tsx` | Modal listing all marked-for-cleanup items with Trash All / Delete All actions |
| `DeleteConfirm.tsx` | Two-step confirmation for permanent deletion |

Components subscribe to the zustand store directly (no prop drilling). `Breadcrumbs` and `DeleteConfirm` are the exceptions — they receive props as controlled presentational components.

### Data flow

1. User picks a volume on the WelcomeScreen → `startScan` invoke → Rust `scanner::scan()` walks filesystem with rayon → progress emitted via Tauri events (`scan_progress`) on a background thread at ~8 Hz → result stored in `AppState.root` (in-memory tree)
2. Frontend transitions from welcome screen to main UI with cross-fade animation
3. `get_subtree` fetches a slice of the tree (max-depth + `minSizeFraction` filtered, default `1/500_000` of total scan size) for the currently focused path; `get_node` fetches the focused node itself (two parallel calls, sequenced via `fetchSeq`)
4. Frontend runs `squarify()` on the subtree to compute treemap rectangles (or the sunburst's recursive polar layout), drawn to `<canvas>` at device-pixel-ratio. Leaf rects indexed in a 48×48 spatial grid for O(1) treemap hit-testing; sunburst uses polar coordinate hit-testing.
5. User interactions: clicks on treemap rects, sunburst segments, tree items, table rows call `handleSelect` / `handleActivate` / `handleMark` on the zustand store. Shift+click / right-click on folder rects zooms in. Drag-to-select creates a marquee selection. Sunburst center-click zooms out to parent directory.
6. Cleanup operations (`move_to_trash`, `delete_paths`, `delete_path`) mutate the in-memory tree via `remove_from_tree()` and return `bytes_removed`/`files_removed` so the frontend can update all totals and refresh the current view

## Conventions

### Commit messages (REQUIRED)

All commits must follow [Conventional Commits](https://www.conventionalcommits.org/):

```
type(scope): description
```

**Types:** `feat`, `fix`, `perf`, `refactor`, `style`, `docs`, `chore`, `build`, `ci`, `test`, `revert`, `security`

**Scopes (optional):** `treemap`, `sunburst`, `scanner`, `store`, `ui`, `release`, `deps`, etc.

**Examples:**
```
fix: use correct folder size when Shift+Alt+click marking for cleanup
feat(treemap): add drag-to-zoom on folder rects
docs: update contributing guide with commit format
chore(release): bump version to 0.5.0
build(deps): bump windows-sys
```

The changelog is auto-generated from conventional commits via `git-cliff`. Non-conventional commits are silently dropped from the changelog. **Always use conventional commit format.**

### TypeScript
- **Strict mode** enabled (`strict`, `noUnusedLocals`, `noUnusedParameters`)
- React functional components with hooks, no class components
- State lives in the zustand store (`lib/store.ts`); components subscribe directly via selectors
- Use `useCallback`/`useMemo` for derived data and stable callbacks
- Multi-select via `Set<string>` of paths
- `fetchSeq` pattern for aborting stale async fetches (module-level counter in the store file)

### Rust
- **Edition 2021**
- Serialization: `serde` with `Serialize` derive; fields not sent to frontend use `#[serde(skip_serializing)]`
- Concurrency: `Arc<RwLock<>>` for shared state, `Arc<AtomicBool>` for cancellation, `rayon` for parallel scans, `tauri::async_runtime::spawn_blocking` for CPU-bound work
- Error handling: commands return `Result<T, String>` (String errors)
- Platform-specific code gated with `#[cfg(target_os = "...")]`
- On-disk size via `stat.st_blocks * 512` (Unix), `GetCompressedFileSizeW` (Windows)

### Styling
- Plain CSS in `App.css` (no CSS-in-JS, no Tailwind)
- Class names use BEM-like conventions (e.g., `.toolbar-left`, `.treemap-wrap`, `.view-tab.active`)

## Key behaviors

- **⌘-click** (Ctrl-click on Windows/Linux): toggle item in multi-selection
- **⌥-click** (Alt-click): toggle item in cleanup queue
- **Shift+⌘-click** (Shift+Ctrl-click): select all items in a folder group
- **Shift+⌥-click** (Shift+Alt-click): mark all items in a folder group
- **Double-click folder**: zoom into that folder (sets it as focused path) — works in treemap, sunburst, and tree
- **Shift+click on a folder rect**: zoom into that folder (treemap only)
- **Right-click on a folder rect**: zoom into that folder (treemap only)
- **Right-click on a folder segment**: zoom into that folder (sunburst only)
- **Click center circle (sunburst)**: zoom out to parent directory
- **Cmd+↑** (Ctrl+↑): zoom out to parent directory
- **Shift+↑/↓**: adjust folder grouping depth
- **Hold Shift**: highlight the current folder group under cursor (treemap dimming overlay)
- **Drag-to-select**: marquee-select multiple rects on the treemap
- **Breadcrumbs**: click any ancestor to zoom back out
- **Escape**: clear selection / close modal
- **Sidebar resize**: drag the gutter between sidebar and content
- **Folder depth**: number input (1–99) controls how many directory levels are separate nodes in the treemap
- **Color modes**: by type (14 categories), by age (cool→warm heatmap), by access (cool→warm heatmap), by folder (hash-based palette; treemap-only — switching to sunburst from folder mode auto-flips to type)
- **On-disk size, not apparent size**: files sized by allocated blocks, so sparse files show their real footprint
- **System directory protection**: `is_protected_path()` prevents deletion of system dirs (macOS: `/System`, `/Library`, `/private`; Linux: `/etc`, `/boot`, `/usr`, `/bin`; Windows: `C:\Windows`, `C:\Program Files`)
- **Platform-aware unit formatting**: `formatBytes()` uses decimal (base-1000) on macOS to match Finder and binary (base-1024) on Windows/Linux to match Explorer/Linux conventions
- **macOS firmlinks**: `/System/Volumes/Data` is skipped to avoid double-counting the data volume
- **Linux virtual fs**: `/proc`, `/sys`, `/dev`, `/run` are skipped
- **macOS TCC**: parts of `~/Library`, `/private/var`, etc. may return 0 bytes without Full Disk Access
- **Volume detection**: macOS `/` + `/Volumes/*`, Linux `/` + `/mnt/*` + `/media/*` + `/run/media/*/*`, Windows A–Z drives
- **Cleanup deduplication**: `dedupe_targets()` removes child paths whose parent is already in the batch
- **Tree mutation**: `remove_from_tree()` walks up the tree updating ancestor sizes after deletions
- **Canvas rendering**: DPR-aware (HiDPI/Retina sharp). Dimming overlay cached per frame to avoid re-compositing. Idle-callback defers layout computation. ResizeObserver debounced at 300ms.
- **Fade transitions**: treemap and sunburst fade out on navigation, then fade back in after the new layout is drawn. Welcome screen cross-fades to main UI.

## Doc maintenance (AUTOMATIC)

At the end of every task that changes code, behavior, or architecture:
- Update CLAUDE.md if any documented behavior, file list, or convention changed
- Update README.md if user-facing features, install steps, or screenshots changed
- Update CONTRIBUTING.md, SECURITY.md, CODE_OF_CONDUCT.md, or other docs if their scope is affected
- **Do NOT update CHANGELOG.md** — it is auto-generated from conventional commits via `git-cliff`
- Only touch docs that are actually affected — don't touch everything

## Adding features

- New Tauri command → add fn in `lib.rs` with `#[tauri::command]`, register in `generate_handler![]`, add typed wrapper in `src/lib/api.ts`
- New frontend component → create in `src/components/`, import in `App.tsx`
- New store action → add to the zustand store in `src/lib/store.ts`
- New file category → add to `CATEGORIES` and `EXT_TO_CATEGORY` in `src/lib/fileTypes.ts`
- New platform-specific behavior → use `#[cfg(target_os = "...")]` in Rust, or check `navigator.userAgent` / `navigator.platform` in TS

---
> Source: [fosterdill/ansel](https://github.com/fosterdill/ansel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
