## vvvv-mcp

> MCP (Model Context Protocol) tooling for **vvvv gamma** (visual programming on .NET, .vl files).

# AGENTS.md — vvvv-mcp

MCP (Model Context Protocol) tooling for **vvvv gamma** (visual programming on .NET, .vl files).
Goal: a professional AI patching assistant that builds whole connected subgraphs in one call,
verified against the live vvvv instance.

> Terminology: "vvvv gamma" = current product. "VL" = its visual language. "vvvv beta" = the
> old product — never apply its concepts.

## Architecture (two processes)

1. **`src/VvvvMcp` + `src/VvvvMcp.Core`** — the MCP server (stdio, .NET 8 global tool).
   Used by MCP clients (Kilo/VS Code, Claude, Cursor, Open WebUI). Reads/writes .vl files
   directly (XML), talks to the bridge over HTTP when vvvv runs.
2. **`VL.MCP.HDE`** — vvvv editor extension (HDE package) hosting the bridge:
   `HttpListener` REST on `:7123` + MCP-over-SSE on `/sse` + chat host (Open WebUI, `:7125`).
   Bridge starts automatically with the extension (always on, `:7123`); **Alt+C** toggles chat.

Key services (VvvvMcp.Core/Services):
- `PatchBuilderService` — `build_patch`: one-call subgraph builder (resolve → deps → pins →
  layout → links → save → reload → verify). THE primary write path.
- `SpecValidator` — extensible pre-validation rules that run BEFORE build_patch writes anything.
  Catches common mistakes (missing RootScene, type mismatches) and returns
  actionable fix suggestions. Add new `ISpecValidationRule` implementations as patterns are found.
- `NodeResolutionService` — live registry first (bridge `/api/nodes`), offline catalog fallback.
- `PatchWriterService` / `PatchReaderService` — .vl XML read/write.
- `SearchIndexService` — SQLite FTS5 (two-phase: AND then OR fallback). Schema version via
  `PRAGMA user_version` — bump `SchemaVersion` when FTS tables change.
- `BridgeClientService` — HTTP client for the bridge. Env override `VVVV_MCP_BRIDGE_PORT`.

Bridge (VL.MCP.HDE/src):
- `MCPBridgeServer.cs` — ProcessNode, REST routes, `IDisposable` (MUST release the listener
  on dispose or the port stays hostage across recompiles).
- `McpSseServer.cs` — MCP JSON-RPC 2.0 server (Streamable HTTP POST `/mcp` + legacy SSE
  `/mcp/sse`). Sends `BridgeInstructions.Text` in the `initialize` response — this is the
  Tier-0 knowledge that bridge-connected LLMs (Open WebUI/kimi) need to produce working patches.
  Keep `BridgeInstructions.cs` in sync with `ServerInstructions.cs` (the stdio equivalent).
- `LiveNodeCatalog.cs` — live node snapshot from `NodeFactoryRegistry.Factories` (.NET nodes)
  + `LatestCompilation.DocumentsAndPackages → DefinedSymbols` (VL-defined nodes). Reflection
  over VL.Lang (no compile-time ref). Auto-rebuilds when the factory set changes.
- `BridgeState.cs` — documents/errors/packages via reflection. Errors carry
  `DocumentId`/`ElementId` (== .vl XML Id attributes) from `VL.Lang.Message.Location`.
- `McpChatHost.cs` — Open WebUI lifecycle: **named mutex `Global\vvvv-mcp-chat-start`**
  serializes startups across Alt+C presses AND HDE reloads; adopt healthy running instance
  (MCP-registration failure is non-fatal); never cancel/kill on disable (the "Open Chat"
  pin is a one-frame bang); server dies only on Dispose (vvvv exit). `/chat` 302-redirects
  to OWUI when up, else serves the placeholder page (polls same-origin `/api/chat/status`,
  reloads on ready — never navigate cross-origin from CEF client-side). Chat host sets
  `WEBUI_AUTH=False`/`ENABLE_SIGNUP=False` (else OWUI shows an admin-setup prompt).
  Loading `/chat` (a chat window opening) sets `_chatWanted` → auto-starts OWUI even
  without Alt+C (handles vvvv-restart-with-chat-open). OWUI console output: only error-ish
  lines forward to the vvvv console; all lines buffered → `GET /api/chat/log`.

## Ground-truth rules learned the hard way

- **`build_patch` auto-layout** (`ComputeLayout` in `PatchBuilderService.cs`): bottom-up Sugiyama-
  style layout. Sources (depth=0) at top, sinks (depth=max) at bottom. Y rows spaced by nodeHeight
  + 20px gap (≈39px top-to-top for standard nodes). X positions determined by processing deepest
  layers first; each node's preferred center X = average of `(successor.left + 5 + 20*pinIndex)`
  over placed successors. Nodes within a layer sorted by preferred X then packed left-to-right
  without overlap. StartX=43, StartY=40. ColGap=20. Pads get type-specific 4-value bounds
  (Float32/Int=35×15, Vector3=85×15, Vector2=65×15, RGBA=65×15, Boolean=35×35, String=150×20).
- **Live catalog symbol sources** (`ExtractFromCompilation`): iterates `DocumentsAndPackages` which
  has two fundamentally different source types:
  - `DocSymbols` (open documents) → VL source, always current, contains `PatchedXxx` symbols
  - `CompiledSymbols` (binary packages like VL.CoreLib.dll) → Roslyn analysis of the binary,
    may contain stale symbols removed from the .vl source but still in old binary metadata
  - `CompiledSymbols.DefinedSymbols` groups contain `ImportedConcreteTypeSymbol` items. Each
    such type has `OriginalSymbolSource.IsForeign`:
    - **`IsForeign = false`**: VL-compiled type (from VL.CoreLib.vl source) → operations are valid
      patchable nodes (e.g. `Split (BezierSegment)` on BezierSegment record)
    - **`IsForeign = true`**: C# type from a foreign assembly (e.g. Matrix from Stride.Core.Mathematics)
      → operations are C# method wrappers that may be stale/removed (e.g. `RotationX`, `RotationY/Z`)
  - **Fix**: `ExtractSymbolAndMembers` skips `OperationDefinitions` when `OriginalSymbolSource.IsForeign=true`.
    This prevents stale C# method nodes from appearing in the patchable catalog.
  - C# methods from foreign types are NOT in the node browser — they were C# method node WRAPPERS
    that were defined in VL.CoreLib.vl at some point but removed. The binary retains them for
    backward compat; the live vvvv session reports "Not found" when they're used in patches.
  - **Discovery path**: the `IsForeign` flag is a DIRECT property on `ImportedConcreteTypeSymbol`
    accessible via `sym.GetType().GetProperty("IsForeign").GetValue(sym)`. `OriginalSymbolSource`
    exists as a property but returns null at runtime — use `IsForeign` directly on the type symbol.
    Verified live: `Matrix` (Stride C# struct) has `IsForeign=true`; `BezierSegment` (VL record)
    has `IsForeign=false`. Fixing Block 2 in `ExtractSymbolAndMembers` reduced catalog from ~18,764
    to ~16,258 nodes, removing 2,506 stale C# method nodes.
- Node XML `Bounds` height: **19** for single-line nodes; **26** for two-line nodes (record/class
  method operations that show a type-name subtitle below the op name — detected by `IsState=true`
  on any input or output pin).
- Node XML `Bounds` width = `max(pin_width, text_width + 12)` where:
  - `pin_width = 5 + 20*(n−1)`, `n = max(1, max(effIn, effOut))` — counts non-hidden,
    non-state, non-optional-unlinked input and output pins
  - `text_width` = Lucida Sans Unicode font measurement (7px in vvvv canvas units → 11px in
    GDI+ pixel units). Strip variant suffixes like " (Successive)" before measuring.
  - For two-line nodes also include `subtitle_text_width + 12` in the max, measuring the last
    segment of the category (e.g. "BezierSegment") in the 5.5px/8px-GDI+ subtitle font.
  - Calibrated against real patches: LFO=45 ✓, +=25 ✓, Cons=39 ✓, RGBA(Join)=65 ✓,
    SwitchOutput=77 (~84 due to Skia vs lookup-table discrepancy), Split(ADSRSettings)=225 ✓.
  - `NodeSizer.cs` implements this with a hardcoded per-character width table (measured from
    Lucida Sans Unicode at 96 DPI) — no GDI+/Skia runtime dependency, DPI-scaling-safe.
- vvvv serializes ALL pins; hidden ones get `IsHidden="true"`: `Node Context`, **state
  outputs** (hide unless operating on the instance), optional-unlinked pins. Pin-group base
  pins (`Child`) are hidden in symbol data but their INSTANCES are visible — never hide them.
- Pin groups serialize as `Child`, `Child 2`, … (build_patch auto-indexes on repeat links).
- `LastDependency` (current) supersedes `LastSymbolSource` (legacy). Value = the .vl file
  actually defining the node (e.g. `VL.Stride.Runtime.vl`).
- NugetDependency elements are children of `Document`, conventionally AFTER `</Patch>`.
- Compile errors only exist for documents LOADED in the session — verify requires opening.
- External file edits do NOT refresh the vvvv UI — use bridge `/api/reload`
  (`Document.ReloadAsync`).
- Packages load lazily: a pack's nodes appear in the live registry only after a document
  referencing it is in the solution.
- **Live pin edit** (`set_value_live`): target the NODE's elementId + pin NAME, use
  `DevEnvHost.CurrentSolution` (NOT `SessionNodes.CurrentSolution` — active-canvas-scoped),
  `ReplaceDescendent` (close the generic) + `MakeCurrent(CommitToValue | UpdateUIAndRuntime)`.
  `AffectCompilation` does NOT commit pin values.
- **Document reload (build_patch)**: `Document.ReloadAsync(bool)` returns `Task<Document>` — a NEW
  immutable Document. The result must be captured and passed to `LivePinWriter.CommitDocument(newDoc)`,
  which does `DevEnvHost.CurrentSolution → ReplaceDescendent(newDoc) → MakeCurrent(Default, canvas)`.
  Without this step the new Document is never wired into the live solution and the editor shows the old
  content until close+reopen. `SolutionUpdateKind.Default` (0xFF, `VL.Model` namespace, `VL.Core` assembly)
  must be used — `UpdateUIAndRuntime` (0x11) skips `AffectCompilation` (0x08), so vvvv detects
  model≠compiled mismatch on focus and reverts the canvas. The earlier "vanishing" bug was caused by
  calling CommitDocument with the OLD doc AND using `UpdateUIAndRuntime`; the fix is `Task<Document>.Result`
  + `SolutionUpdateKind.Default`.
- **New patches not on disk**: vvvv creates new sketches in memory with a pre-determined file path but
  does NOT write them to disk until the user presses Ctrl+S. `build_patch` detects this (file not found),
  calls `SaveDocumentByPath` (bridge REST path; synchronous via `PostToUIThreadAndWait`), polls up to 2 s
  for the file to appear, then loads it. This preserves the document ID so `ReloadAsync` matches. If the
  bridge is down, falls back to `CreateDocument()` with a fresh ID — the editor cannot match it; tell
  the user to save (Ctrl+S) first.
- **Angle convention**: ALL rotation nodes in vvvv use **cycles (0..1 = full 360° turn)**.
  Both `3D.Transform` category (`Rotate`, `Rotation (Successive)`, `TransformSRT`) and `3D.Matrix`
  category (`RotationX/Y/Z`, `Rotation (Vector3)`) use cycles. NEVER convert to radians.
  LFO.Phase outputs 0..1 cycles — wire directly to any rotation input.
  Preferred idiom for continuous rotation: `Rotation (Successive) [3D.Transform]` with `Angular Delta`
  in cycles (e.g. `"0.003, 0, 0"` ≈ 1 rpm around X; `"0, 0, 0.003"` for Z). No LFO needed.
- **Pin default suppression**: `build_patch` automatically skips writing `DefaultValue` when the
  provided value equals the VL type-system default (Float32=0, Boolean=false, Integer32=0,
  Vector2=(0,0), Vector3=(0,0,0)). Setting these explicitly causes vvvv to render the pin enlarged
  ("modified" indicator) even though nothing changed. Only non-default values are written.
- **build_patch error recovery**: When a `build_patch` call results in compile errors or the wrong
  graph, do NOT create a new file. Instead: (1) use `remove_node` to delete the wrong nodes from the
  same file (it automatically removes all connected links), (2) re-run `build_patch` on the same file
  with only the corrected nodes and links. Staying in the same file preserves the document ID so
  `ReloadAsync` picks up the changes. `remove_node` accepts the node `id` returned in build_patch's
  result or from `read_patch`.
- **Duplicate link guard**: `build_patch` automatically removes any pre-existing link to an input pin
  before creating a new one. This prevents stale connections when re-patching an existing canvas and
  mirrors vvvv's own UI behaviour (connecting to a pin that already has a link replaces it).
- **Multi-component value format**: Vector3, RGBA, Vector2, Vector4, and Matrix values must use
  `"x, y, z"` (comma + space) as separator. `"1,1,1"` (no space) may parse incorrectly in some
  IOBox contexts; `"111"` (no commas) is parsed as a scalar and causes visual corruption.
- **IOBox type rules**: A Vector3 IOBox outputs `Vector3`. It CANNOT connect to individual `Float32`
  component inputs (X, Y, Z pins of `Vector (Join)`). Use separate `Float32` pads for components.
  Same applies to RGBA → float and any typed IOBox → individual component. This is a type mismatch.
- **Node-level defaults differ from type defaults**: `build_patch`'s `IsTypeDefault` only suppresses
  the VL type-system zero defaults (Float32=0, Vector3=0,0,0, etc.). Node-level defaults like
  Box.Size=(1,1,1) or Box.Tessellation=1 are NOT suppressed — don't pass them in `values{}` since
  they also enlarge the pin unnecessarily. Only set values that differ from the node's natural default.
- **Stride GPU<T> pins**: `ColorMaterial.Color` and `PBRMaterial.Color` accept `GPU<RGBA>`.
  `RGBA (Join)` output (`RGBA`) connects directly to these pins — `ColorIn [Stride.Materials.Inputs]`
  is NOT needed. `ColorIn` is only required when bridging from a reactive channel or IOBox at runtime.
  Never place ColorIn without an RGBA source connected to its Value pin.
- **PBR vs Color material / lighting**: `PBRMaterial` is physically based and renders completely
  BLACK without a light in the scene. `ColorMaterial` is flat/unlit and always shows the color
  regardless of lighting. Prefer `ColorMaterial` for simple colored scenes (no light needed).
  When using `PBRMaterial`, ALWAYS include `DirectionalLight [Stride.Lights]` in the RootScene.
- **State Output vs Output**: Many Stride/vvvv process nodes have two outputs:
  - `Output: <MaterialType>` — the usable VALUE (e.g. `Material`) for connecting downstream (e.g. `Box.Material`)
  - `State Output: <NodeType>` (hidden by default) — the process INSTANCE for direct manipulation
  Always use the `.Output` pin. `State Output` is only for operating on the instance directly.
- **Error reporting**: `build_patch` now reports ALL severity levels (Error + Warning). Type-mismatch
  errors ("entity is not sceneinstance") may arrive as non-Error severity from vvvv. If the
  verification shows messages, treat them as problems even if compileErrors=0.
- **Error doc attribution**: `FilterDocErrors` now falls back to ALL errors when none match our
  document — vvvv sometimes attributes link type errors to the library document (VL.Stride.Runtime)
  rather than the patch containing the bad connection.

## Build & test

```powershell
dotnet build src/VvvvMcp.sln                  # MCP server
dotnet build VL.MCP.HDE/src/VL.MCP.Bridge.csproj   # bridge (also hot-recompiled by vvvv when editable)
dotnet run --project src/VvvvMcp.Tests        # smoke tests incl. live build_patch benchmark
```

vvvv side: start vvvv with `--package-repositories "X:/_dev/vvvv-mcp/" --editable-packages VL.MCP.HDE`.
If the bridge was loaded as binary (lib/net8.0 dll), a vvvv restart is needed after
`dotnet build` of the bridge.

## Knowledge & data pipelines

- `knowledge/*.md` — auto-discovered by `KnowledgeService` (top-level only).
  Manually maintained: vl-quickref, vl-patterns, vl-building-blocks, vl-common-graphs,
  vl-project-architecture, vvvv-internals-advanced (registered in `scripts/build-knowledge.ps1`).
- `scripts/build-knowledge.ps1` — regenerates gray-book-*.md from the submodule.
- `scripts/describe-graybook-images.ps1` — PREFERRED image pipeline: local Ollama vision model
  (default qwen3-vl:8b) describes gray book images → `knowledge/gray-book-image-text.md`.
  Incremental + resumable + abort-safe; `-Model`, `-OllamaUrl`, `-TimeoutSec`, `-MaxImages` params.
  (supersedes `scripts/ocr-graybook-images.ps1` — Windows OCR quality is too poor for screenshots).
- `scripts/scrape-forum.ps1` — Discourse scrape → vl-forum-solutions.md / vl-forum-snippets.md
  (must run in Windows PowerShell 5.1-compatible syntax — no `?.`).
- `scripts/index-help-patches.ps1` — help-patch index from `packs-community/` (LOCAL ONLY,
  not git-tracked, never redistribute the packs).
- `VVVVNodeAnalyzer/` — offline catalog builder (vvvv_nodes_mcp.json). Known limits: misses
  VL-defined nodes and C# nodes in package DLLs; live registry is the better source.
  The catalog node types `Method`, `Getter`, `Setter` are C# class member nodes — only usable
  when writing custom C# node bodies (not for general patching). `search_nodes` filters these
  by default (`includeMethodNodes=false`). Node visibility (`advanced`/`experimental`/`obsolete`/
  `internal`) is inferred from category path segments or parenthesized version qualifiers like
  `"Cast (Advanced)"` / `"2D.Obsolete.GridPoints"` — the `Aspects` field is not in the JSON yet.

## Node catalog — visibility and tool usage

- **4-level visibility** (matches vvvv node browser):
  - `default` — hi-level nodes, shown by default — what `search_nodes_live` returns normally.
  - `advanced` — low-level; use `visibility='advanced'` or `visibility='all'`.
  - `experimental` — unstable/future; same flag.
  - `obsolete` — deprecated; same flag.
  - `internal` — completely hidden, never returned even with `visibility='all'`.
- **Efficient patching workflow**: use `search_nodes_live` with `includePins=true` to get
  pin details in one call (avoids follow-up `get_node_details_live` calls). When you already
  know which nodes you need, use `batch_lookup_nodes_live` to get all pins in one request.
- **C# method nodes**: the live catalog includes accessor nodes (IsAccessor=true, visibility
  shown in result). These are synthesized property/field getters/setters — demoted in search
  scoring. The offline catalog `search_nodes` hides `Method`/`Getter`/`Setter` type nodes by
  default; pass `includeMethodNodes=true` when writing custom C# node bodies.
- **Pin defaults**: `defaultValue` on pins is usually empty — VL only stores explicit
  overrides; type-system defaults (Float32=0, Boolean=false) are not in the metadata.
- **Batch node lookup** (`POST /api/nodes/batch-lookup`): accepts `[{name, category?}]` array,
  returns `[{name, result:{found, matchCount, nodes, suggestions}}]` for each input.
  Always uses `includeHidden=true` — batch/explicit lookup finds nodes at any visibility tier.
- **Exact lookup always finds any visibility tier**: `get_node_details_live` defaults to
  `includeHidden=true`. `NodeResolutionService.ResolveAsync` also passes `includeHidden=true`.
  When a user names a specific node, it must be found regardless of `advanced`/`experimental` etc.
- **Advanced/experimental nodes require explicit visibility in search**: `search_nodes_live`
  defaults to `visibility='default'` (hi-level only). Nodes like `Cast` (advanced) or
  `Create (BezierKnot - Advanced)` need `visibility='advanced'` or `visibility='all'` in search,
  OR use `get_node_details_live`/`batch_lookup_nodes_live` which always search all tiers.
  **Zero-result fallback**: when a search with default visibility returns 0 results, the bridge
  automatically re-searches across all tiers and returns those results (with `visibility` field
  showing the tier). So `search_nodes_live("Cast")` will return the Cast node even without
  `visibility='advanced'`.
- **Live search uses word-boundary matching**: `StartsAtWordBoundary` in `LiveNodeCatalog`
  prevents acronym false-positives (e.g. "LFO" no longer matches "Se**lfo**rParentDefinition").
  Scores only when the term starts at index 0, or after ' '/'.'/'('/')' /'_'/'-' in the text.
  Multi-term queries (e.g. "Create BezierKnot") penalise partial matches proportionally.
- **`build_patch` node resolution uses `includeHidden=true`**: Cast, Create (BezierKnot), and other
  advanced/experimental nodes resolve correctly in `build_patch` without pre-search. Use
  `skipUnresolvable:true` to place all resolvable nodes even when some names are unknown —
  unresolved ones appear in `skippedNodes` in the response (not a hard failure).
- **`"Name (TypeHint)"` convention**: `build_patch` and `get_node_details_live` resolve
  `"Split (BezierSegment)"` → `Split` in `Math.BezierSegment`, `"Create (BezierKnot)"` → `Create`
  in `Math.BezierKnot - Advanced`. The parenthetical is a category-hint, not a variant name.

## Licensing

Dual license (see LICENSE.md): PolyForm-Noncommercial for non-commercial use, paid commercial
licenses via polar.sh. Keep attribution/CC BY-SA for tebjan-derived knowledge files
(THIRD-PARTY-NOTICES.md). Never bundle VL.* assemblies or the packs-community folder.

---
> Source: [digitalwannabe/vvvv-mcp](https://github.com/digitalwannabe/vvvv-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
