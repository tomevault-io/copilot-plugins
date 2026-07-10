## vean

> > **vean studio** — Video Editor, Agent Native · vean.studio

# vean — the agent-native video editing core

> **vean studio** — Video Editor, Agent Native · vean.studio

vean is the **typed document, edit algebra, and diagnostics layer** for video
editing on top of MLT. Think of it as a *language server for video*: a
frame-exact, rationally-timed model of a timeline that an agent (or a UI, or a
human) mutates through a closed set of operations, each of which reports its
consequences **before a single frame renders**.

vean is **not only an app.** Its core is the headless editing engine the app
sits on: files in, files out, no network, no secrets. Product surfaces (the
Commander CLI, LSP, MCP, and the future local Tauri Mac app) share local
coordination state in a gitignored `.vean/vean.db` SQLite database. The render is
delegated to `melt` (MLT/FFmpeg) as a separate process; the Mac app may bundle
pinned renderer sidecars, while the source/CLI/Homebrew artifact treats them as
system dependencies. vean owns the part nobody else does: a *typed, validated,
diagnosable, agent-authorable* representation of an edit.

This file is the **resolver** — it doesn't hold the knowledge, it routes to it.
It is the single canonical brain; `CLAUDE.md` is a one-line `@AGENTS.md` shim so
Claude and Codex read the same bytes.

## What it is / isn't (read first)

- **IS:** a typed IR for an MLT timeline (multi-track, audio, keyframes,
  filters, transitions); a deterministic serializer + parser (IR ⇄ `.mlt` XML);
  a pure edit algebra (`op(state) → {state', consequences, inverse}`); a
  diagnostics engine (the LSP); a render/inspect driver that shells out to
  `melt` and `ffmpeg`; a local product-state substrate for projects, setup
  choices, jobs, and future UI/agent coordination.
- **ISN'T:** a renderer (MLT/FFmpeg do that), a motion-graphics engine (Remotion
  does that), or a UI implementation inside the core (the local Mac app does
  that through the action runtime).

## The seam: depend / mine / build

The whole strategy in one table. We **fork the engine** (depend on MLT, which we
already drive), **mine Shotcut's source as the spec** (its edit algebra and dial
schemas are the answer key — we lift the semantics, drop the Qt), and **build**
the typed/diagnosable layer nobody else has.

| | What | Why |
|---|---|---|
| **Depend on** (reuse as-is) | MLT + `melt` + FFmpeg; Remotion's renderer + `@remotion/player` | Solved problems — engine semantics, codecs, the React-in-Chrome render. Never reimplement. Driven at arm's length (see Hard boundaries #1). |
| **Mine** (source as spec, don't run) | Shotcut `src/commands/` (edit algebra + undo mechanics); `src/qml/filters/*/meta.qml` (dial schemas); the nested-tractor-dissolve + blanks-as-gaps XML shapes | Debugged knowledge, but Qt-coupled. Lift the semantics, drop the framework. |
| **Build** (own, greenfield) | the typed IR; serialize + parse; the keyframe model; the dials schema (from `melt -query` + overrides); the edit algebra as pure ops; the diagnostics LSP; the action registry; the Commander CLI; the agent bridge + skills; the local Mac app | This is the project. The LSP especially is net-new — no NLE exposes consequences. |

## The layer model

Four layers. The agent-vs-UI ownership split falls out of them cleanly.

1. **The core (this repo's heart, headless, no UI).** The typed video *document*
   and everything that reasons over it: IR, serialize+parse, keyframe model,
   dials schema, edit algebra, diagnostics, the `melt`/ffmpeg driver. The video
   equivalent of a language server + the engine driver. ~Agent-buildable,
   because it's well-specified (the spec is Shotcut's source).
2. **The action runtime.** A typed registry defines product behaviors once, then
   projects them to Commander CLI, MCP tools, narrow LSP code actions, and Tauri
   invoke commands. This is where project/media ergonomics, policy, permissions,
   output envelopes, and job semantics stay consistent across surfaces.
3. **The agent bridge.** Two coordinated surfaces over the same core:
   **`vean-lsp`** for ambient feedback (push diagnostics, code actions,
   definitions/references/hover over the timeline document) and **MCP/CLI tools**
   for domain actions (`apply-op`, `preview-op`, `undo`, `render`, `still`).
   `diagnose` exists for CI/debug/manual inspection, but it is **not** the normal
   agent loop. Agents should see adverse effects the same way they do in code:
   edit/change happens, diagnostics are pushed without a separate "remember to
   call diagnose" step.
4. **The local product layer.** The local Mac app is the primary GUI: project
   list, media catalog, timeline drawn from the IR, render/still preview, and
   agent-orchestration UI (sessions, diffs, git worktrees for exploration). The
   website is only for download/docs, not a web editor.

## Local state contract

The timeline core stays file-in/file-out. Product coordination state lives in
`.vean/vean.db`, a repo-local SQLite database ignored by git. It is owned by
`src/state/`, modeled with Drizzle, and migrated by committed SQL under
`drizzle/`.

- **Store in `.vean/vean.db`:** project metadata, setup choices, preferences,
  render/agent jobs, job leases, media probe caches, future UI session metadata.
- **Do not store in `.vean/vean.db`:** canonical timeline placement, committed
  `.mlt` XML, assets, rendered deliverables, secrets, or anything required to
  reproduce a project from git plus media files.
- **Concurrency rule:** use WAL + `busy_timeout`; keep writes short; never hold a
  transaction while rendering, probing media, or running agents. Job claiming is
  a tiny lease transaction (`queued` → `running` with `locked_by` /
  `locked_until`); long work happens after the transaction and finishes with a
  short status update.

CLI is the canonical command surface for this state. LSP remains ambient and
independent. MCP and the future app are adapters backed by the same action
runtime, not the source of truth.

## Action runtime contract

Define a product behavior once in `src/actions/`, then expose it through the
surfaces that make sense. `src/ops` remains the pure edit algebra; actions wrap
real workflows such as `timeline.applyOp`, `render.still`, `project.init`,
`media.scan`, and `jobs.claim`.

- **Commander CLI is first-class and complete.** No hand-rolled CLI parsing.
  Every public action is available either as an ergonomic Commander command or
  through the escape hatch: `vean action list`, `vean action describe <id>`,
  `vean action run <id> --input-json ...`.
- **MCP is generated from the registry where possible.** MCP annotations are
  hints projected from vean's native effect metadata, not the authorization
  source of truth.
- **LSP stays narrow.** It owns ambient diagnostics, navigation, and
  deterministic code actions only. Structural edits needing choices belong to
  CLI/MCP/app actions.
- **Tauri calls actions.** The Mac app may add richer presentation and
  interaction, but every mutating button/menu routes through a registered action
  rather than duplicating domain behavior.
- **Effect metadata is load-bearing.** Actions declare scopes, mutated
  resources, idempotency, destructive/open-world status, dry-run support,
  reversibility, approval level, audit level, and job behavior. Policy is
  enforced before execution and projected to CLI confirmations, MCP hints, and
  Tauri capabilities.

`DESIGN-MOVE3.md` is the detailed contract for action/runtime/media routing.
When adding a product action, update the registry, add/confirm CLI and MCP
projection, add a stable JSON test, and keep the app path as an adapter over the
same action id.

## Media routing contract

Media ergonomics are first-party, but inference-heavy media intelligence is not
first-party until the product/install/privacy shape is decided.

- **First-party now:** media roots, route aliases, lightweight scans,
  path/kind/size/mtime catalog rows, CLI/MCP list/find, and touched-path output.
- **Route addresses:** prefer aliases such as `media:raw`, `media:proxy`,
  `timeline:main`, `renders:review`, `stills:latest`, and
  `transcripts:source` over repeated absolute paths once a project is selected.
- **State boundary:** `.vean/vean.db` is cache/coordination state only. Media
  files, `.mlt` timelines, renders, transcripts, and other deliverables remain
  files on disk.
- **Deferred action families:** transcription, semantic labels, embeddings,
  scene/object/face inference, waveform analysis, proxy generation, and
  filesystem watching. Add them later as registry actions backed by short job
  leases, not as bespoke scripts or app-only flows.

The payoff of the split: **human gestures and agent actions become the same
operations** — both call the edit algebra, both get undo, and both update the
same document that `vean-lsp` watches. That unification is only possible because
we did not fork Shotcut's Qt-welded gestures/undo. Build the editing logic once.

## Agent feedback contract

This is load-bearing. Do not regress it into manual lint/tool polling.

- **Diagnostics engine is shared core.** `src/diagnostics/` owns domain rules
  once: timeline validity, keyframe bounds, transition overlap, asset refs,
  dial ranges, A/V hazards, and related locations. LSP, MCP, CLI, tests, and the
  future UI all call this engine; none reimplement the rules.
- **LSP is ambient truth.** `vean-lsp` publishes the current complete diagnostic
  set for each open/project document after document changes. Like TypeScript,
  Pyright, and rust-analyzer, it does not make agents ask for `diagnose` after
  every edit. An empty diagnostic set clears prior diagnostics.
- **MCP tools are domain actions, not the diagnostics source of truth.** Mutating
  tools return `consequences`, `inverse`, touched URIs, and optional `alerts`
  only when the mutation introduced new blocking errors. They do **not** return a
  standing health snapshot or dump the whole diagnostic set on every call; that is
  noisy and belongs to the ambient LSP stream or explicit debug/CI commands.
- **`diagnose` is a debug/CI verb.** It is allowed for gates, tests, one-off
  inspection, and non-LSP clients. It must not be the required step that makes a
  Claude Code edit loop safe.
- **Code actions are fixes.** Diagnostics that have deterministic repairs should
  expose LSP code actions and/or MCP safe-edit actions: shorten a transition,
  relink/remove a dangling asset, remove an orphaned filter, or make a ripple
  choice explicit.

Because the entire edit state is **text** (IR/XML + Remotion `.tsx` + asset
refs), `git worktrees` give parallel exploration for free — branch a project,
let an agent try a tighter cut, render + diff + preview, keep or discard.

## The Remotion seam

**MLT XML is the spine / single source of truth for placement. Remotion is a
producer.** They are two runtimes with no shared frame buffer; you cannot put
them on one *engine* track, but you can put them on one *editor* track:

- **Export:** pre-render each composition to an alpha clip (ProRes 4444,
  `yuva444p10le`, scoped with `frameRange`) and drop it onto an upper MLT track.
- **Live preview:** embed `@remotion/player` slaved to the editor's master
  playhead, composited over the footage `<video>` in the browser.
- **Accept two compositing paths** (live preview ≠ bit-exact export). That's the
  real cost, and it's manageable.

Do **not** rebuild Remotion. Its value is the React-in-Chrome render; what it
lacks (a data document) is exactly the thing we build on the MLT side.

## Load-bearing invariants (don't violate these)

- **Frame-exact rational time everywhere.** fps is `[num, den]` (29.97 is
  `30000/1001`, never `29.97`); positions/in/out/length are **integer frames**.
  A float fps anywhere makes the diagnostics subtly, permanently wrong.
- **Identity = stable producer UUIDs, not indices.** Indices are ephemeral;
  refer to a clip by stable id across a session. (This is how Shotcut's undo
  works, and it's why undo can be "store the inverse op".)
- **Keyframes live inside property strings** — `"0=100;50~=0"` (`|`=hold,
  `~`=smooth, etc.). A property is animated iff its string contains `=`. The
  typed keyframe model round-trips these byte-faithfully.
- **Determinism.** Same IR → byte-identical XML. Golden tests guard it.

## Status: scaffolded, building in phases

This is **infrastructure stood up early**, building behind verification gates —
see [ROADMAP.md](ROADMAP.md). Nothing stacks on an unverified phase. The Move-0
seed is studio's `src/mlt` toolkit, ported here with the `@/brand` coupling
stripped (colors become plain hex/named — vean is standalone).

## Conventions

- **Work on `main`.** Solo, fast-moving repo — commit directly to `main`.
- **Bun** runtime, `type: module`, ESM, `.ts`, `@/*` → `src/*`.
- **Zod** schemas on the IR and every op input — typed, validated before `melt`
  runs.
- **Biome** lint+format. **Vitest** for pure units — serializer, parser,
  op-inverse, diagnostics, keyframe round-trip are all **golden-tested**. Frame
  rendering is verified by an actual `melt` render + still-frame compare, never
  in vitest.
- **No coupling to any in-house brand or app.** vean is a standalone OSS
  project; it must never import from `carlo`, `studio`, etc. This is a feature,
  not a constraint — it's why the repo is public.

## Hard boundaries

1. **Never link GPL code.** We drive `melt` (GPLv2) as a **separate process**
   via the public `.mlt` file format + CLI — arm's-length per the FSF GPL FAQ —
   so vean stays AGPL-by-choice, not GPL-by-force (see [LICENSING.md](LICENSING.md)).
   Never statically or dynamically link `libmlt` or `libavcodec` into vean.
2. **Never link GPL/media libraries; bundle sidecars only in the app.** The
   source/CLI/Homebrew artifact is pure TypeScript and treats `melt`/`ffmpeg` as
   system deps. A signed Mac app may bundle pinned `melt`, MLT modules/profiles,
   `ffmpeg`, and `ffprobe` as subprocess sidecars with license/provenance
   manifests. Never statically or dynamically link `libmlt` or `libavcodec`.
3. **Core stateless, product state local.** `src/ir`, `src/ops`,
   `src/diagnostics`, and `src/driver` remain deterministic/file-based. Shared
   product state is allowed only in gitignored `.vean/vean.db` via `src/state/`.
   No network calls or secrets.
4. **License discipline.** AGPL-3.0; contributions only under the CLA (see
   [CONTRIBUTING.md](CONTRIBUTING.md)). Never merge un-CLA'd code — it forecloses
   dual-licensing, which is the monetization escape hatch.

## The resolver (capability × work axis)

Capabilities (the *how*) live canonically in `.agents/skills/<skill>/`; host
compatibility shims may point there from `.claude/skills/<skill>/`. Work (the
*what*) lives in `src/`. Shallow today — skills get written *from real
experience* as each Move lands, not guessed ahead. Until a skill exists, the
method is in [ROADMAP.md](ROADMAP.md). Codex agents should follow this resolver
and read the repo-local skill file directly when named; do not depend on
Claude-only skill discovery. Claude Code can load this checkout as a plugin with
`claude --plugin-dir /Users/tejas/Github/vean`; `.lsp.json`, `.mcp.json`, and
`skills/` are the Claude plugin-facing shims, and `.claude/launch.json` wires
the Claude Desktop preview pane to this checkout's live viewer (via
`scripts/preview-launch.ts`, port 5176/`PORT` — see the `view` skill).

### Parallelization / thread model

Distributed work means **fresh agent threads** the lead agent spins up, scopes,
and synthesizes — not a reason to keep one context overloaded. Use them readily
when an independent context window improves the work: research, codebase
exploration, adversarial review, test-plan critique, UI/design critique, and
post-implementation audit. The lead thread remains the PM: it owns the goal,
sequencing, integration, verification, and final answer.

For implementation, split only along disjoint write scopes. Worker threads get a
bounded ownership area, are told other sessions may be active, and report changed
paths plus verification evidence. The PM thread integrates, re-runs the gates, and
resolves conflicts. For research/review, prefer multiple fresh-context reviewers
over one long self-confirming pass when the question is broad or correctness is
load-bearing.

Parallelization does **not** weaken the completion bar: every thread follows the
parallel session safety rules above, never reverts unrelated work, and never uses
"another thread can do it later" as a deferral mechanism.

| When you want to… | Load skill | Work in |
|---|---|---|
| extend the timeline model (tracks, audio, keyframes, filters) | _(none yet — see ROADMAP Move 0)_ | `src/ir/` |
| serialize/parse `.mlt` | _(none yet — see ROADMAP Move 0)_ | `src/ir/`, `tests/`, `corpus/` |
| add/verify an edit operation | _(none yet — see ROADMAP Move 1)_ | `src/ops/`, `tests/` |
| add a diagnostic / lint | _(none yet — see ROADMAP Move 1)_ | `src/diagnostics/` |
| wire the agent bridge / a skill | _(none yet — see DESIGN-MOVE2/GATE-MOVE2; monitor active builds with `BUILD-MONITOR.md`)_ | `src/lsp/`, `src/bridge/`, `.agents/skills/` |
| set up or verify a fresh clone / host integration | `setup` (`.agents/skills/setup/SKILL.md`) | system deps, `bun install`, `.lsp.json`, `.mcp.json`, skill shims, CLI PATH registration, `bun run doctor` |
| initialize repo-local product state | `setup` (`.agents/skills/setup/SKILL.md`) | `.vean/vean.db`, Drizzle migrations, `src/state/`, `drizzle/` |
| edit a timeline as an agent (apply an op, fix a diagnostic, tighten a cut) | `editing` (`.agents/skills/editing/SKILL.md`) | a `.mlt` doc via the bridge tools (`apply-op`/`preview-op`/`undo`/`render`/`still`) |
| parallelize research, review, implementation, or verification | _(none — PM thread delegates directly)_ | fresh agent threads; disjoint code scopes; PM integrates + verifies |
| add or expose a product action | _(none yet — see DESIGN-MOVE3)_ | `src/actions/`, CLI/MCP/LSP/Tauri adapters, tests |
| improve project/media ergonomics | _(none yet — see DESIGN-MOVE3)_ | `src/actions/`, `src/state/`, Commander CLI, media catalog migrations |
| build the local Mac app | _(none yet — Move 4)_ | `app/` (Tauri), action IPC, bundled renderer sidecars |
| prove a material UI change works — drive the running app, screenshot/record a clip, verify the effect visually + structurally | `drive` (`.agents/skills/drive/SKILL.md`) | `bun run drive up` + `agent-browser` against the loopback viewer (live HMR by default — your current `viewer/` code; `--prod` serves the dist snapshot the shipped app renders); clips to `$TMPDIR`, referenced back in chat |
| bring up the app so the HUMAN can click around this worktree's latest code (the sanctioned foreground-window path) | `view` (`.agents/skills/view/SKILL.md`) | `bun src/cli.ts open [project] --dev` (native Tauri dev window, live HMR of this checkout) or `--view browser` (instant, identical UI); launch backgrounded, report branch+project, step back — don't drive it (that's `drive`) |

### Keeping the resolver healthy

When a pattern repeats or a correction lands, **promote it** — write or update a
skill in `.agents/skills/`, then fix the row above. Keep host-specific paths
(`.claude/skills/`, future `.codex/` shims, etc.) as pointers to that canonical
repo-local skill, not divergent copies. The bar for a skill is judgment/method;
pure mechanics stay scripts. Write skills from what actually happened in a Move,
not a hypothesis.

## Layout

```
src/
  ir/          ← the typed document: IR types, serialize, parse, keyframes (Move 0)
  ops/         ← the edit algebra: pure op(state) → {state', consequences, inverse} (Move 1)
  diagnostics/ ← the LSP: static checks, resolve-value-at-frame, find-references (Move 1)
  actions/     ← typed product actions + metadata projected to CLI/MCP/LSP/Tauri (Move 3)
  bridge/      ← the agent surface: CLI / MCP verbs (Move 2)
  state/       ← repo-local product state: SQLite/Drizzle, projects, jobs, setup choices
  driver/      ← the melt/ffmpeg subprocess driver + inspect (render, still, contact)
  index.ts     ← barrel
tests/         ← vitest: golden round-trips, op-inverse invariants, diagnostics fixtures
corpus/        ← real .mlt files for round-trip + render-faithfulness gates (Move 0)
drizzle/       ← committed local-state SQL migrations for `.vean/vean.db`
.agents/skills/← CAPABILITY axis: methods, versioned (written as Moves land)
.claude/skills/← compatibility shims for hosts that discover Claude-style skills
skills/        ← Claude Code plugin skill shims (symlink back to .agents where shared)
.lsp.json      ← Claude Code plugin LSP registration for .mlt / vean-lsp
.mcp.json      ← Claude Code plugin MCP registration for vean-mcp
app/           ← the local Tauri Mac app (Move 4, TBD)
ROADMAP.md     ← the phases + their verification gates (the plan of record)
BUILD-MONITOR.md ← 15-minute checkpoint/review protocol for active agent builds
LICENSING.md   ← why AGPL-3.0 + CLA, and the no-linking nuance
```

## Quick reference

Mostly **planned** — implemented per Move. See [ROADMAP.md](ROADMAP.md).

| Task | Command | Lands in |
|---|---|---|
| Install | `bun install` | — |
| Test / typecheck / lint | `bun run test` · `bun run typecheck` · `bun run lint` | now |
| Round-trip a `.mlt` (parse→IR→serialize) | `bun run roundtrip <file.mlt>` | Move 0 |
| Render-faithfulness gate over the corpus | `bun run verify:corpus` | Move 0 |
| Discover commands/actions/ops | `vean discover --json` · `vean discover <query> --kind op --json` | Move 3 |
| Discover timeline ops | `vean timeline ops list --json` · `vean timeline ops describe <op-or-alias> --json` | Move 3 |
| Select/current timeline | `vean timeline use <path-or-alias> --json` · `vean timeline current --json` | Move 3 |
| Apply/preview an edit op | `vean timeline apply-op <op-or-alias> --args-json '<json>' --json` · `vean timeline preview-op <op-or-alias> --args-json '<json>' --json` | Move 3 |
| Diagnose a timeline | `vean timeline diagnose [file] --json` | Move 3 |
| Resolve a param's value at a frame | `vean timeline resolve-value-at-frame <frame> --target-json '<json>' --json` | Move 3 |
| Render headless / inspect a frame | `vean render video <file> --out <path>` · `vean render still <file> <frame> --out <path>` | Move 3 |
| Verify local setup / host integration | `bun run doctor` | Move 2 |
| Register the CLI on PATH | `bun run setup:cli` then `vean doctor --surface cli-lsp` | Move 2 |
| Initialize local state | `bun run state:init` · `bun run project:init` | now |
| Inspect local jobs | `vean jobs list` | now |
| Inspect/run action registry | `vean action list` · `vean action run <id> --input-json '{}'` | Move 3 |
| Add/scan/find media | `vean media root add <path>` · `vean media scan` · `vean media find <query>` | Move 3 |
| Resolve a route alias | `vean route resolve media:raw` | Move 3 |
| Verify local app scaffold | `bun run app:doctor` · `bun run app:doctor -- --native` | Move 4 seed |
| Drive the app to prove a UI change (headless) | `bun run drive up` (live HMR; `--prod` for the dist snapshot) → `agent-browser --session "$(bun scripts/drive.ts name)" open "$(bun run drive url)"` → `bun run drive down` | now |
| Bring up the app so the human can click around (`/view`) | `bun src/cli.ts open [project] --dev` (native dev window, this worktree's live viewer) · `bun src/cli.ts open [project] --view browser` (instant browser tab) | now |

## System deps (not bun packages)

`mlt` (provides `melt`) and `ffmpeg`. Mac: `brew install mlt ffmpeg`. Linux:
`apt install melt ffmpeg`. Native Tauri app builds also need Rust/Cargo
(`brew install rust` on macOS). Optional, for the Remotion producer: a Node/Bun
Remotion install (peer, user-provided). Optional, for the `drive` skill (proving
UI changes in a real browser): `agent-browser` (`brew install agent-browser`) — a
Playwright/Chromium automation CLI; the harness degrades to nothing without it.

---
> Source: [Tshah-95/vean](https://github.com/Tshah-95/vean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
