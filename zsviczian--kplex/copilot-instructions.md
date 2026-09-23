## kplex

> Develop **K-Plex (Knowledge Plex)** as a dedicated React application inside Obsidian while preserving the relationship semantics, ontology model and useful settings compatibility of classic ExcaliBrain.

# AGENTS.md

## Mission

Develop **K-Plex (Knowledge Plex)** as a dedicated React application inside Obsidian while preserving the relationship semantics, ontology model and useful settings compatibility of classic ExcaliBrain.

K-Plex is not an Excalidraw extension. Excalidraw and Dataview must not be runtime requirements.

The plugin ID is **`k-plex`** so K-Plex can coexist with legacy ExcaliBrain during migration.

## Build contract

Baseline Node.js: **22.22.2**.

```bash
npm i
npm run build
```

Use `npm run dev` for watch-mode development.

A change is not complete if the real repository does not build successfully.

Installable output must be written to `./dist/`:

- `dist/main.js`
- `dist/manifest.json`
- `dist/styles.css`

Do not claim a successful build from a stub-only/type-harness check. When Obsidian APIs are involved, validate against the actual installed `obsidian` type package.

## Obsidian API discipline

This project has already lost time to invented/assumed APIs. Do not guess Obsidian methods.

Rules:

1. Check the installed Obsidian type declarations and current official documentation before using an unfamiliar API.
2. Prefer public typed APIs.
3. Example: `MetadataCache.getFileCache(file)` is valid and `getAllTags(cache)` is exported; do **not** invent `metadataCache.getTags()`.
4. Use Obsidian `Modal` for centered modal dialogs.
5. Use `workspace.getLeaf("window")` for pop-out workflows when supported by the installed API.
6. Use the Obsidian declarative settings API for settings UI.
7. All plugin UI icons must be Lucide icons obtained through Obsidian `getIcon()` (or a thin React wrapper around it). Do not ship hand-coded icon SVGs or unrelated icon libraries.
8. Moment is host-provided by Obsidian. Do not runtime-import `moment` or call the `moment` export from `obsidian`; production code should use Obsidian's `window.moment` through narrow local typing. Tests may install a Moment test double on `window`.

## Non-negotiable compatibility rules

1. Preserve classic ExcaliBrain ontology semantics unless a deliberate migration/change is documented.
2. Explicit document-property relationships take precedence over inferred/body relationships.
3. Preserve parent / child / left-friend / right-friend / previous / next reconciliation behavior.
4. Preserve support for Markdown notes, attachments, folders, tags, URLs and virtual/unresolved nodes.
5. Preserve legacy style inheritance as closely as possible without relying on Excalidraw rendering.
6. Preserve/migrate legacy persisted settings instead of silently reinterpreting them.
7. Migrate old `hierarchy.friends` to `hierarchy.leftFriends`.
8. If legacy ExcaliBrain is installed and running, automatically import compatible settings the first time K-Plex runs in that vault. Keep a manual import path as well.
9. Folder and tag nodes may be central nodes, but drag-link creation/relinking involving folder/tag endpoints is disabled.
10. A Markdown document property overrides an equivalent relationship discovered in body text. This rule is important for deterministic relinking.

## Architecture boundaries

- `src/index/` owns graph construction, relationship semantics, search data, caches and compatibility.
- `src/ui/` owns React presentation and interaction.
- `src/settings.ts` owns settings schema/defaults, persistence compatibility and migration.
- `src/main.ts` owns Obsidian lifecycle, commands, workspace/window/leaf integration and rebuild scheduling.

Do not reimplement relationship classification inside React components. UI code should consume normalized index APIs.

Keep Obsidian-specific side effects behind clear boundaries. Presentational components should not reach deeply into workspace/vault APIs when plugin/index services can perform the operation.

## Performance is a product requirement

The plugin must remain responsive in vaults with 20,000+ files and 100,000+ graph/search entries.

### Startup

Obsidian emits large numbers of file events while initializing a vault. Never trigger a full rebuild for every startup `vault:create` or metadata event.

Required strategy:

- wait until layout is ready and metadata is sufficiently stable before the initial full graph build
- register/coalesce normal rebuild listeners only after startup initialization is under control
- mark the index dirty on changes and skip periodic refreshes when it is clean
- collapse event bursts into at most one rebuild/catch-up rebuild

### Persistent parsing cache

Markdown body parsing is expensive and should not repeat on every startup.

- persist parsed inline-field / external-link body metadata in vault-local storage
- key cache records by path + modification time (or an equivalent safe invalidation key)
- restore the cache before the first graph build
- write cache updates lazily/debounced

### Worker boundary

CPU-heavy Markdown-body text parsing may run in a Web Worker.

Do not attempt to use Obsidian APIs inside the worker. Vault/metadata access, graph mutation and Obsidian object handling remain on the main thread.

### Derived-data caching

Avoid repeated global work while rendering one scene.

Cache or precompute:

- relationship classifications / normalized neighbor views
- node titles
- title-script results
- sort keys
- gate counts where safe
- search-normalized strings

Compile user title scripts once, not per node.

Never call `titleFor()` repeatedly from an `Array.sort()` comparator. Compute titles/sort keys once and sort on the cached keys.

UI-only actions such as pan, zoom, hover, history updates and density changes must not trigger index rebuilds.

### Search

Search must feel immediate in large vaults.

- normalize search text ahead of time
- use exact > prefix > substring > fuzzy ranking
- fuzzy matching is ordered subsequence matching
- search aliases and paths
- reuse previous-query candidate sets for longer prefixes rather than rescanning the entire index on every keystroke
- do not globally sort all search entries at startup just to support query-time ranking

### Instrumentation

Temporary performance instrumentation may be added while diagnosing regressions, but it must be disabled or removed from normal production output once the issue is resolved.

If profiling is needed again, add it behind an explicit development/debug flag and emit copy-friendly string lines. Do not leave high-volume console logging enabled by default.

## Plex layout contract

K-Plex is not a force-directed graph. Spatial meaning is deterministic.

### Zones

- parents: top / center
- children: bottom / center
- friends + previous: left
- challengers + next: right
- siblings: separate right-side peripheral zone

Left and right primary zones should be symmetrical. Siblings are the intentional asymmetry.

The zones must not visually overlap one another in normal use.

Current target defaults:

- parent columns: **2**
- child columns: **5**
- parent max height: **300 px**
- friend/challenger max height: **350 px**
- child max height: **400 px**
- sibling max height: **250 px**
- density: **2**, configurable up to **4**
- max nodes per zone: **100**, configurable up to **300**

Parent columns must not exceed 2. Children may be configured up to 7 columns.

Friends/challengers and siblings may extend upward into otherwise unused parent-area space. They should not be artificially clipped by the parent's vertical boundary.

When their occupied strip fits, Friends/Previous and Challengers/Next are **bottom-aligned** within their shared-height lateral bands and grow upward. A sparse lateral list (including a single node) must stay near the lower edge of the lateral region instead of being centered high in the available band. Overflowing strips remain scrollable; if filtering reduces an overflowed lateral zone to a result set that fits, preserve the same bottom-aligned behavior.

Leave a small vertical gap between the bottom of lateral zones and the start of children; children should sit slightly lower than in the earlier layout.

Siblings:

- move slightly upward relative to the current layout
- render at `0.85` normal scale
- expanded-view descendants of sibling nodes inherit the same `0.85` multiplier

### Scroll zones

Each major zone has its own maximum height.

When the node list exceeds that height:

- make the zone internally scrollable
- show a funnel/filter control for first-level friend/challenger/sibling/parent/child scroll regions where applicable
- filtering must remove and repack nonmatching items; invisible placeholders that preserve whitespace are a bug
- the count above the funnel shows the total/current visible item count even when the text filter is closed

Expanded-descendant scrollers do not get filter controls.

### Density

Density/compactness affects:

- inter-node spacing
- label truncation / maximum displayed characters
- overall layout density

Density must **not** change node interior padding. Use the tight padding from the compact design at every density.


### Mobile / view-surface rules

- Use only the public `Platform.isMobile` flag. Distinguish phone vs tablet using the shortest CSS-pixel screen dimension; do not rely on undocumented `Platform.isPhone` / `Platform.isTablet` members.
- Phone: the generic K-Plex open action routes to the right sidepanel; command palette should expose only **Open in side panel** among K-Plex surface-opening commands.
- Tablet: **Open graph** opens a normal K-Plex tab and **Open in side panel** remains available; pop-out is desktop-only.
- Touch activation must not depend on a synthesized browser click. A stationary one-finger pointer-up activates the node explicitly; movement owns pan/pinch; long-press owns context menus.
- K-Plex canvas touch gestures stop propagation/default handling so Obsidian Mobile edge/top swipe gestures do not steal pans. Scrollable internal zones remain native scroll surfaces.


### Index lifecycle and mobile memory rules

- Restore the persisted semantic graph snapshot before doing expensive Markdown parsing. IndexedDB generations are activated only after all records are written; cache data is never authoritative.
- Persist parsed-body metadata by path+mtime+parser-version in IndexedDB on every platform so an interrupted cold build can reuse completed parsing. Any IndexedDB store/index schema change requires a database-version bump; silently adding a store name without upgrading the DB leaves existing vault databases permanently unable to create it. On iOS keep only a tiny bounded hot in-memory parser cache; worker parsing remains disabled there to avoid structured-clone duplication of Markdown strings/payloads.
- For a large iOS cold start with no semantic snapshot, prewarm bodies into transactional IndexedDB batches **before** retaining the full semantic graph. Bound concurrent native reads, yield real paint/autorelease windows between I/O waves, keep the prewarm cancellable, then run the authoritative GraphBuilder from durable hits. Do not combine a ten-thousand-file body scan with a simultaneously growing full graph on iOS.
- Persist resolved neighbour maps alongside provenance so warm startup does not replay the full classifier. Keep relationship evidence declaration-compact: retain one original declaration and derive its inverse perspective on demand rather than storing duplicate forward/reverse evidence objects.
- Reject structurally stale or half-bound cached graphs instead of publishing them while a replacement is built; holding old+new full graphs simultaneously is especially dangerous on iOS.
- Perform one initial session index (or accept a fresh restored snapshot), then stop reactive rebuild work while no K-Plex view is open. Vault/metadata events accumulate as a dirty backlog until the next open. On iOS, cancelling the last open Plex must also cancel a first cold build; completed IndexedDB checkpoints remain reusable.
- Large build and resolver loops must cooperate with the host using **time-budgeted slices**, not tiny fixed record-count yields. `setTimeout(0)` should occur only after the current synchronous slice has consumed its budget; yielding every note can turn an otherwise fast iOS pass into tens of seconds of timer overhead. All long work must remain cancellable. Never clear the dirty backlog unless a snapshot was actually published.
- Per-file incremental edits must use the evidence store's path index (`declarationsTouching` / `removeDeclarationsTouching`) rather than scanning every evidence declaration. Full semantic snapshot persistence after small edits should be deferred/coalesced and cancelled when the last K-Plex view closes.
- During iOS pinch, prefer a temporarily simplified scene over GPU/WebView memory pressure: no expensive shadows/filters and relationship SVGs may be temporarily suppressed until the gesture ends.

### Companion sidecar

The sidecar is a **native adjacent Obsidian WorkspaceLeaf**, never a fake nested leaf inside React. “Sidecar” is a geometric/UI state of a pinned tab: when the pinned tab is adjacent to K-Plex the edge controls are visible; moving it away hides those controls without breaking the pin; moving it back restores them. On startup prefer a visible adjacent loaded document leaf over Obsidian's deferred/hidden “most recent” leaf. `_loaded` may be used only as an isolated optional compatibility hint (`FileView & { _loaded?: boolean }`), never as the sole criterion. Detach breaks synchronization but leaves the native tab open. Closing K-Plex must also release ownership without detaching the user's companion document. Sidecar Markdown mode is a persisted K-Plex default (Reading view vs Edit/source mode). Folding K-Plex in sidecar mode hides the complete K-Plex tab-group DOM container, keeps both workspace leaves alive, and mounts the recovery/unfold button on the surviving document tab group; all fold state is ephemeral and must be restored on leaf removal/plugin unload. Sidecar is unavailable when K-Plex itself is hosted in a sidepanel.

### Runtime section outline

Central-note heading expansion remains outside the persistent graph index. Nested headings form a transient foldable outline tree. Section nodes are compact theme-aware outline cards; structural outline edges use a distinct **solid orthogonal folder-tree** geometry: vertical spine from the parent's lower-left structural port, horizontal L-branch into the child's left-center port. Semantic relationships still use normal K-Plex gates. A Markdown central node always exposes the small lower-left fold handle so sections can be unfolded directly; non-Markdown centers must not render that handle. Folding hides descendants and projects their relationships to the nearest visible folded ancestor while preserving the original hidden section as explainability provenance. Fold/unfold must preserve the exact camera and suppress transient ResizeObserver auto-fit long enough for the complete section reflow to settle.

## Expanded view contract

Expanded mode shows children of visible first-level nodes.

- child-of-node display is approximately 50% normal node size
- smaller font
- approximately 70% opacity
- maximum 3 columns × 2 visible rows per first-level node
- overflow is scrollable
- no filter UI in the small expanded-child scroller
- do not reserve descendant space for a first-level node with no children
- when multiple nodes share a row, row height equals the maximum descendant-space requirement among nodes in that row
- sibling-node expanded descendants inherit sibling's 0.85 scale multiplier

## Gates and connectors

Every visible node has four gates.

Gate semantics:

- hollow = no connected relationships
- filled = relationships exist, even if related nodes are hidden/filtered
- displayed count = relationships represented under the active filter/visibility rules

Connectors must originate/terminate at the relevant gates, never at node centers.

Connector styles:

- `straight`
- `curved` (user-facing wording; do not call it Bézier in settings)

Curved connectors should be broad/flatter rather than excessively bowed.

Relationship labels should sit over a small break in the connector line. Suppress generic labels such as parent, child, friend, challenger and file-tree; show meaningful custom ontology labels.

Optional arrowheads indicate link direction and must preserve legacy direction/reversal semantics.

## Hover behavior

Do not cause the graph to flash as the pointer crosses dense scenes.

- relationship/node/gate highlight delay target: **750 ms**
- hover preview only while Ctrl (Windows/Linux) or Cmd (macOS) is held
- modifier-assisted preview should appear immediately
- normal hover without Ctrl/Cmd must not trigger Obsidian page preview

Use context-appropriate cursors: nodes/links that can be activated use pointer-style feedback; empty canvas uses panning/grab feedback.

## Navigation and state

- clicking a node activates/navigates it
- folders and tags are navigable and may be central
- graph navigation can be synchronized to an active or pinned Obsidian leaf, or decoupled
- if an active file leaf exists at K-Plex startup, use it as center
- otherwise restore the last displayed node
- persist Past nodes/history across sessions
- add command palette action to open K-Plex in a pop-out window
- pinned nodes are persistent bookmarks/quick-access entries displayed beneath the main toolbar

## Search behavior

Search dropdown must support full keyboard interaction:

- Up/Down moves selection
- Enter activates selection
- Escape closes
- clicking the Plex/outside the dropdown closes
- opening a new query after a previous search must be immediate

Use a wide dropdown and do not allow long paths/titles to make results unreadable.

## Panning and zoom

- wheel scrolling zooms without requiring another mouse button
- dragging empty Plex space may pan with left, middle/wheel, or right mouse buttons
- context menus must not accidentally fire during right-drag panning
- zoom is hard-capped at 300%; do not expose a separate max-zoom setting

## Drag-connect and relinking

### Add-relationship composer

The create-relationship path uses one React modal and the shared `FuzzySearchInput`; do not reintroduce a second note picker or a separate “create note” modal. Both suggesters are closed until the user actually types. Selecting an existing note is a two-step flow: retain the selection, allow ontology edits, then commit with the fixed-width Link action. New-note buttons stay disabled unless the filename is valid and globally unused; Markdown is the default action and Ctrl/Cmd+Enter is its shortcut. Command-palette actions for Parent/Child/Friend/Challenger are gated by a running K-Plex view and are intended to be user-hotkeyable. New Markdown/Excalidraw files must resolve their parent through the public `app.fileManager.getNewFileParent(sourcePath, newFilePath?)` API using the current K-Plex center as `sourcePath`. Ontology input is fuzzy-searchable, remembers one default per gate role, and a newly typed ontology field becomes a real hierarchy item in settings before the relationship is written.

Connector unlinking is provenance-safe. Direct deletion is allowed only when one frontmatter ontology declaration is the sole editable source of the visible pair; Obsidian `resolvedLinks` entries whose source positions fall inside that same YAML property block (including indented list items below the property key) are mirrored cache views, not additional user declarations. Obsidian may omit source positions for YAML links in `CachedMetadata.links`; in that case, verify that the relevant property block itself resolves to the same target before treating the generic resolved-link evidence as a mirror. Any body link, link in another property, inline ontology, or other competing evidence must fall back to the explanation dialog. Explanation rows for Markdown-backed evidence should provide line navigation in a **new Markdown tab** via ephemeral state rather than mutating the user's existing document leaf or persisted scroll state.

### Extension-resolution compatibility

Some upgraded development checkouts may still contain both `src/ui/NewRelatedNoteModal.ts` and the older `.tsx` path. The `.ts` module is canonical; `esbuild.config.mjs` deliberately resolves `.ts` before `.tsx` so the production bundle matches TypeScript. Keep the two files behaviorally synchronized until the legacy `.tsx` copy can be removed from a full-repository distribution.

### Create relationship from gate

Dragging from a gate and releasing on empty Plex opens an Obsidian `Modal`.

The modal provides:

- Markdown target selection
- ontology/document-property dropdown appropriate to the relationship direction
- check/confirm and X/cancel Lucide buttons via `getIcon()`

Only targets not already connected in the relevant way are selectable.

Default relation by gate:

- top → parent
- bottom → child
- left → friend
- right → challenger

Dropping onto a specific target gate must also influence the proposed relationship direction.

If origin is Markdown, write YAML relationship on origin.

If origin is non-Markdown, target must be Markdown and write the inverse relationship on target.

While drag-connecting, only nodes already connected through the origin gate should be visually de-emphasized as invalid/redundant; other valid targets remain available.

Disable drag-linking to/from folder and tag nodes.

### Move an existing direct neighbor

Dragging a node directly connected to the center across top/bottom/left/right regions may propose changing its relationship class.

When rewriting relationship data, prefer adding/updating document properties because document-property relationships override duplicate body relationships.

## Styling

A `Note type` frontmatter/document property may drive a note's primary node style. Settings must allow defining styles per note type.

Continue supporting legacy tag-specific style compatibility where feasible.

## Settings UX

Use grouped declarative settings pages such as:

- Graph
- Ontology
- Compatibility
- Appearance

The root K-Plex settings page begins with a compact centered row:

`Buy me a coffee | Read Sketch Your Mind | Join SYM Community`

with links:

- https://ko-fi.com/zsolt
- https://community.sketch-your-mind.com/sym
- https://community.sketch-your-mind.com

Do not place these links inside the Graph page and do not add explanatory marketing copy around them.

Use sliders where a bounded numeric range is meaningful (zone heights, gate radius, etc.) and show the current value next to the slider.

## Naming

Use **K-Plex**, not ExcaliBrain, in user-facing UI and docs except when explicitly discussing compatibility/migration.

Use **nodes**, not "thoughts", in user-facing terminology. Legacy internal names can be migrated gradually, but new UI strings should say nodes.


## Graph predicates and lenses

K-Plex graph filtering is a **presentation-layer operation over the currently materialized Plex**, not a whole-vault graph query engine. Keep the persistent semantic index focused on data required to build and explain the Plex.

- `src/lens/GraphPredicate.ts` owns the declarative predicate AST, dependency discovery and safe evaluator. Do not use `eval`, `Function`, DataviewJS or any other runtime code execution for filters/lenses.
- The supported predicate contexts are deliberately distinct: `node`, `edge`, `evidence`, `note`, `file` and `this` (the current center thought). Do not collapse evidence provenance into resolved-edge fields.
- `note.<property>` must be resolved lazily from Obsidian `MetadataCache`. Do not copy arbitrary frontmatter values into `GraphPage`, `GraphState`, snapshots or IndexedDB just to support lenses.
- K-Plex-native graph properties should be read from the existing graph model. File metadata should come from `TFile`/cached graph fields. Relationship provenance should come from `RelationEvidence`.
- Predicate dependency discovery must remain explicit. Metadata-dependent predicate refreshes are UI/presentation refreshes and must not call `rebuildIndex()` or otherwise couple lens evaluation to semantic graph reconstruction.
- The current Keyword / Tag / Note type controls are a compatibility UI over the generic predicate engine. New filtering behavior should compile to the same predicate representation rather than adding another ad-hoc matcher in React.
- Named lenses and styling rules should reuse this same selector engine. A future text syntax should be declarative and Bases-inspired, parsed by K-Plex into the AST; do not execute user-authored scripts.
- Lens evaluation is scoped to the visible/current Plex (normally tens to a few hundred candidate nodes). Do not add whole-vault scans or arbitrary-depth traversal as a side effect of lens evaluation.

## Code-scanner compatibility

Treat the Obsidian code scanner as a release gate, not as post-release cleanup. In particular:

- Do not assign static styles with `element.style.foo = ...`; prefer semantic CSS classes, `setCssStyles`, or `setCssProps`. Dynamic per-frame transforms may remain direct only when they are genuinely runtime values and the scanner accepts them.
- Use Obsidian DOM helpers (`createEl`, `createDiv`, `createSpan`, etc.) rather than `document.createElement` / `ownerDocument.createElement`.
- Do not use `!important` or broad `:has(...)` selectors in plugin CSS. Prefer explicit state classes and normal selector specificity.
- Command names shown in the Command Palette must not repeat the plugin name; Obsidian already displays the owning plugin.
- Keep strict TypeScript boundaries: avoid unnecessary assertions, `any`-typed member access/calls/arguments, and unsafe `JSON.parse`/IndexedDB assignments. Narrow `unknown` with type guards instead.
- Never reject a Promise with a raw unknown value; normalize it to an `Error`.
- Do not leave empty catch blocks, unused catch parameters, unused imports/locals, or stale instrumentation counters. A best-effort catch should either return explicitly or contain a meaningful compatibility/cleanup action.
- Avoid regex constructs that trigger scanner lint (unnecessary escapes or literal control-character ranges). Prefer small string/code-point helpers when validation is clearer than a regex.
- Before packaging a release, re-check for dead imports/variables and for scanner regressions that were fixed in earlier rounds; do not reintroduce them while adding adjacent features.

## Testing expectations

Before returning a patch:

1. run `npm test` when indexing, ontology, provenance, folders, tags, URLs, Date properties or relationship classification changed
2. build against the actual repository and Obsidian typings
3. test startup with an existing K-Plex data file
4. test a large-vault path if the change affects indexing/search/rendering
5. verify Markdown, folder, tag, attachment, URL and virtual-node navigation as relevant
6. verify linked/unlinked leaf behavior for navigation changes
7. verify no new high-volume console logging
8. package only requested modified/new files when the user asks for a patch ZIP

### Relationship explanation navigation and sidecars
- Provenance navigation from **Explain relationship** should reuse the sidecar belonging to the K-Plex view that opened the explanation when that sidecar is currently available. Do not pick an arbitrary global sidecar or unrelated recent tab.
- Opening provenance in a pinned sidecar is a temporary inspection action; suppress the corresponding sidecar-to-Plex follow event so the graph center does not unexpectedly change.
- Keep the provenance location ephemeral (`setViewState(..., { line })`) and force Markdown source for `.excalidraw.md` evidence.

---
> Source: [zsviczian/kplex](https://github.com/zsviczian/kplex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
