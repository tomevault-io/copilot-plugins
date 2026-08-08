## autoharness

> Greenfield, Apache-2.0, macOS-only Rust workspace. Read `PLAN.md` before

# AutoHarness — Agent Guide

Greenfield, Apache-2.0, macOS-only Rust workspace. Read `PLAN.md` before
changing contracts. Phase work ends in working software with a focused
commit per phase.

## Conventions

- Rust edition 2024 (workspace `resolver = "3"`), stable toolchain.
- Library crates under `crates/`, binaries under `bins/`.
- Keep dependencies minimal; the policy list is in the root `Cargo.toml`
  workspace table. Confirm a crate is already depended upon before using it.
- No `unsafe` outside the daemon's `getpeereid` peer-UID check,
  `engines::process` (pre_exec setrlimit, killpg), and the menu-bar's single
  fixed AppKit target/action selector. Each boundary is documented beside the
  block; no selector or executable input may come from a user or model.
- Never panic on bad input: illegal state transitions, malformed frames, and
  unknown RPCs return structured errors.
- One commit per phase; do not mix phases in a commit.

## Crate map

| Crate | Path | Owns |
|-------|------|------|
| `autoharness-core` | `crates/core` | Domain types (EngineKind, RunState, NodeState, RouteDecision, GraphProposal, Budget, Artifact, Checkpoint, MemoryFact, PolicyVersion); run/node state machines as `Result`-returning functions; `router` (task profiling, route selection, budgets) and `detector` (fingerprints, loop/stall triggers, recovery ladder) — both pure |
| `autoharness-protocol` | `crates/protocol` | JSON-RPC 2.0 envelopes (`Request`/`Response`/`Event`), 4-byte length-delimited framing, method constants, `DedupCache` |
| `autoharness-store` | `crates/store` | SQLite (rusqlite, bundled, WAL), migrations table, append-only event ledger, checkpoints/snapshots, replay, engine-session persistence |
| `autoharness-daemon` | `crates/daemon` | Unix socket server (0600, `getpeereid` UID check), token auth, RPC dispatch, `events.subscribe` replay+live, run lifecycle, `runner.rs` (direct + bounded-loop run task, steering, detector/recovery, finalize, restart reconciliation), `worktree.rs` (daemon-managed git worktrees), `sandbox/` (Seatbelt runner, profiles, proxy, canaries, policy) |
| `autoharness-engines` | `crates/engines` | `EngineAdapter` contract, normalized `EngineEvent` model, `EngineDiagnostics`, FakeEngine, Codex (app-server v2) and Claude (stream-json) adapters, contract suite, `EngineRegistry` |
| `autoharness-ui-gpui` | `crates/ui-gpui` | GPUI desktop shell. See the module map below. |
| `autoharness` | `bins/autoharness` | Desktop binary |
| `autoharnessd` | `bins/autoharnessd` | Daemon binary |
| `autoharness-updater` | `bins/updater` | Authenticated update request, independent bundle verification, active-run refusal, atomic swap, rollback, and relaunch |

## Shell module map (`crates/ui-gpui/src`)

| Module | Owns |
|--------|------|
| `lib.rs` | The `Shell` root view, layout, key routing, the utility overlays, and the in-app attention banner |
| `client.rs` | Daemon socket client, `UiState`, command encoding, response and event folding |
| `theme.rs` | Design tokens: type scale, radii, spacing, colours, fills, status vocabulary, motion |
| `layout.rs` | Pane widths and heights, seams, minimum sizes |
| `toolbar.rs` `sidebar.rs` `coordinator.rs` `workbench.rs` `inspector.rs` | The panes of the cockpit |
| `overview.rs` `palette.rs` `switcher.rs` `navigation.rs` | Navigation surfaces and typed navigation actions |
| `history.rs` | Provider history rows, grouping, and adoption |
| `settings.rs` | Persisted-setting rows and their typed update commands |
| `usage.rs` | Ledger-derived usage rows. Never prices anything |
| `worktrees.rs` | The worktree panel: filters, dry run, and explicit confirmation |
| `attention.rs` | The pure attention reducer: triggers, replay suppression, unseen state |
| `notify.rs` | Bounded main-thread Notification Center bridge, click-to-run actions, fixed opt-in sound, and test fake |
| `menubar.rs` | Pure unseen-priority rollup plus the native macOS status item |
| `update.rs` | Compile-time feed pinning, bounded fetch and private staging, signed-bundle verification, and helper handoff |
| `query_editor.rs` `fuzzy.rs` `ansi.rs` `components.rs` | Text editing, ranking, terminal colour, shared elements |
| `preview.rs` | The deterministic cockpit fixture used by `examples/shell_preview` |

Two documents are normative for the shell:

- `docs/reference/COCKPIT.md` — the exact acceptance reference for the
  default window, with the image beside it.
- `docs/STATUS.md` — what is verified locally, what waits on a macOS
  permission, and what waits on signing and a hosted feed.

## Commands

```sh
cargo build --workspace   # build everything
cargo test --workspace    # run all tests (must pass before committing)
cargo run -p autoharnessd # run daemon
cargo run -p autoharness  # run desktop shell (Cmd-Q quits)

# The cockpit fixture: no daemon, no provider account, no client token.
cargo run -p autoharness-ui-gpui --example shell_preview
```

## Key invariants (from PLAN.md — do not break)

1. **The daemon persists every event to the store BEFORE broadcasting it.**
   See `AppState::emit` in `crates/daemon/src/lib.rs`.
2. Event sequences are monotonic; `events.subscribe` replays
   `seq > since_sequence` then streams live, without duplicates or gaps.
3. Illegal run/node transitions are `Err`, never a panic.
4. Every envelope carries `protocol_version` (currently 1).
5. Socket is mode 0600; peer UID must match the daemon's UID; the client
   token lives in the Keychain (0600 token file is the documented fallback).
6. **No external writes in V1**: no push, PR, merge, deploy, publish, email.
7. The daemon owns all state; closing the UI must not stop work.
8. Engine is chosen per run and never switched automatically. An
   unavailable engine emits `run.blocked` with diagnostics plus a
   `derived_run_offer` for a NEW run on the other engine.
9. Engine detection never panics and never spends tokens: version/help
   probes and credential-existence checks only. Provider secrets are never
   read.
10. Engine processes are spawned ONLY through
    `engines::process::spawn_json_lines_child`; daemon-owned worker commands
    ONLY through `daemon::sandbox::Sandbox::run_command` (Seatbelt profile +
    external-write policy), and daemon-owned git ONLY through
    `Sandbox::run_bookkeeping` (same policy, plus explicitly declared writable
    roots outside the worktree). Interactive terminal escape sequences are
    never parsed as the control protocol.
11. Provider session/thread IDs are persisted (`engine_sessions` table) and
    resumed when safe; a failed resume falls back to a fresh session.
12. **Fail closed on sandbox**: if sandbox-exec, the proxy, or any canary is
    unavailable, `run.start` is refused with structured diagnostics. There is
    no unsandboxed fallback. External-write-looking commands (git push,
    publish, PR/release, deploy, mutating HTTP) are rejected BEFORE execution
    and audited.
13. Seatbelt profile rules (verified live): reads are broad with explicit
    canonical-path denies (dyld needs broad reads; `/tmp`→`/private/tmp`
    symlinks break non-canonical rules); writes only to worktree/session
    dirs; network only to `localhost:<proxy-port>` (Seatbelt rejects
    `127.0.0.1` in network rules).
14. **Every editing run works in its own worktree and branch.** The user's
    checked-out branch and working tree are never mutated. Worktrees are
    reclaimed only when clean AND commit-free; anything else is preserved and
    surfaced (`run.worktree_preserved`).
15. **Stale work is never reported as success.** Runs left Running/Paused by a
    dead daemon are reconciled to Blocked during `Daemon::bind`, before the
    socket serves anyone (`direct::reconcile_after_restart`).

## Phase status tracker

- [x] **1. Foundation** — workspace scaffold, core state machines, versioned
  protocol, SQLite ledger, daemon lifecycle, socket auth, replay, minimal
  winit/vello/parley window.
- [x] **2. Engine bridge** — `autoharness-engines` with the adapter contract,
  normalized event model, FakeEngine + contract suite, Codex (app-server v2)
  and Claude (stream-json) adapters, detection, session persistence,
  `run.start/pause/resume/cancel` daemon wiring.
- [x] **3. Sandbox** — Seatbelt runner behind `SandboxBackend`, deny-default
  profiles with canonical sensitive-path denies, sanitized env (fake HOME/
  TMPDIR, controlled PATH, no inherited tokens), process groups + rlimits,
  codex `workspace-write` / claude `acceptEdits`+`--disallowedTools`,
  read-only audited proxy, external-write policy, fail-closed startup
  canaries, `sandbox.check` RPC.
- [x] **4. Direct chat** — daemon-managed worktree + branch per run, direct-run
  task streaming normalized engine events to the ledger, `chat.send` steering
  queued to turn boundaries and `chat.interrupt` cutting the turn first,
  sandboxed verification command + local commit on the run branch, cancel
  cleanup that preserves dirty worktrees, restart reconciliation, `engine.list`,
  and a UI that talks to the daemon (replay-then-live, projects/chat/event
  panes, slash commands). `engine.list` also returns token-free model
  capabilities; provider/model/reasoning are persisted per run and passed to
  planning, resume, direct turns, and graph workers.
- [x] **5. Router and loops** — deterministic task profiling and route
  selection (`core::router`) with per-shape budgets, `run.routed` on the
  ledger, bounded loops that feed a failing check back as the next turn's
  evidence, loop/stall detection with the six PLAN triggers plus
  false-positive fixtures (`core::detector`), and the recovery ladder
  (nudge, replan, checkpoint restart, blocked).
- [x] **6. Graphs and swarms** — `core::graph` compiles a model proposal into a
  runnable plan or refuses it with every reason (cycles, non-convergence,
  excessive nodes/depth/width, scope collisions, unverified editing branches,
  budget overflow); `AwaitingApproval` + `run.approve` gate execution; the
  scheduler runs dependency waves concurrently with a worktree and branch per
  node, merges verified branches in an integration staging worktree, and skips
  dependents of a failed node.
- [x] **7. Review surfaces** — GPU-rendered plan graph (wave columns, exact
  dependency flow, selectable objective/scope/check detail, per-node live colour), scrollable review tail over the
  ledger, verification/commit/detector cards, `/approve` for a plan awaiting
  review, `/open` to hand a run's worktree to the user's own tools, captured
  ANSI check output, and classified/tinted unified diff hunks. Raw interactive
  PTY ownership is intentionally excluded — see the Phase 7 notes.
- [x] **8. Memory and evolution** — evidence-backed project memory over SQLite
  FTS5 (a verified fact without evidence is refused; superseded facts never
  resurface), `memory.list`/`memory.forget`, append-only policy versions with
  exclusive promotion and exact rollback, and `core::policy` guarding the
  capability boundary a candidate may never cross.
- [x] **9. Release hardening** — database integrity reporting + startup check,
  full JSON export, irreversible per-project purge, one-command
  `app.diagnostics` bundle, a fail-closed signed-update path with atomic
  rollback, a universal sign/notarize/ZIP/DMG script, and semantic keyboard +
  VoiceOver accessibility. The structured-console PTY exclusion is documented
  in the Phase 9 notes.
- [x] **10. Cockpit** — the exact black three-pane shell, structured run
  projection, real navigation surfaces, and automatic read-only provider
  history. `docs/reference/COCKPIT.md` is the acceptance reference.
- [x] **11. Product state** — persisted settings and ledger-derived usage, the
  authoritative worktree index with fail-closed reclaim, bounded artifact and
  check evidence, the attention reducer with replay suppression, and the
  update verification pipeline. What is verified locally and what waits on a
  macOS permission or on signing is recorded in `docs/STATUS.md`.

## Notes carried into later phases

- Event `sequence` is globally monotonic; `run_sequence` is per-run. The
  subscribe cursor uses the global sequence.
- Token storage: release builds use the Keychain (`dev.autoharness.app` /
  `daemon-client-token`); debug builds use the 0600 `client-token` file
  because ad-hoc-signed rebuilds trigger blocking Keychain consent dialogs.
- `cargo run -p autoharness-ui-gpui --example shell_preview` opens the
  cockpit against an in-memory fixture — no daemon, account, or token.
- Notification delivery is complete in code; production acceptance still
  requires the packaged app and the user's macOS permission. Update
  installation still requires real signing and feed inputs.
- Codex adapter pins `approvalPolicy: "never"` and `sandbox:
  "workspace-write"`; any approval server request that still arrives is
  declined and surfaced as `engine.question`. Provider tool approvals are not
  forwarded because the engine control process is outside the daemon's
  Seatbelt boundary; graph approval remains a separate typed AutoHarness gate.
- Claude adapter: `--permission-mode acceptEdits` + `--disallowedTools
  WebFetch,WebSearch,NotebookEdit`. Bash and other prompting tools are
  deliberately declined: approving them inside the unsandboxed provider
  control process would bypass the daemon's external-write policy. Daemon-owned
  checks and Git operations still run through Seatbelt.
- Pause is a soft pause (interrupt current turn) on real adapters.
- Normalized `Usage` excludes cost (provider-specific, not comparable).
- `AUTOHARNESS_LIVE_TESTS=1` gates the live direct-run adapter test; CI must
  never set it because real turns spend provider tokens.
- Worker-command spawn for Phase 4 verification/build/integration:
  `Sandbox::run_command(worktree, dirs, program, args, cwd)` — it enforces
  the external-write policy, generates the profile, and applies the
  sanitized worker env (proxy vars injected). `Sandbox::probe` skips the
  policy check (canaries only).
- Proxy audits land in the ledger as `sandbox.proxy.request` /
  `sandbox.proxy.denied` (run-less events).

### Reusing the user's saved Codex/Claude login

The user logs in once with their own CLI; AutoHarness must never ask again.
All of this was established by live probing, not documentation — re-verify
before changing any of it (`cargo test -p autoharness-engines --test
auth_passthrough`, which is differential and passes on a logged-out machine).

- **Codex** needs `CODEX_HOME` pointed at the real `~/.codex`. Necessary and
  sufficient; without it `codex login status` says "Not logged in".
- **Claude** keeps its OAuth credentials in the **macOS Keychain**, not on
  disk (`~/.claude/.credentials.json` does not exist on a normal install).
  Reaching them needs BOTH:
  1. `USER`/`LOGNAME` in the environment — `env_clear` without them yields
     `loggedIn: false` even with the real HOME;
  2. `$HOME/Library/Keychains`, because macOS resolves the default keychain
     search list through HOME. A relocated HOME hides the login keychain, so
     `SessionDirs::create_for_engine` symlinks that one directory in.
- `CLAUDE_CONFIG_DIR` is deliberately NOT set: it relocates the expected
  `.claude.json` to `$CLAUDE_CONFIG_DIR/.claude.json`, which does not exist on
  a normal install, and it does NOT restore the login. `~/.claude.json` is
  copied into the engine session home instead (a copy — a run must never
  mutate the user's config).
- **`SessionDirs::create` (workers) stays bare; `create_for_engine` (engine
  control processes) adds the seeded config and the keychain link.** The
  `cannot_read_login_keychain` canary proves daemon-owned worker commands are
  still denied. Granting the control process this reach adds no authority — it
  is not Seatbelt-wrapped by design and could always open the path directly;
  per-item Keychain ACLs still apply.
- Detection asks the CLI itself (`codex login status`, `claude auth status`)
  **through `engine_control_env_for`**, so `engine.list` can never claim an
  auth state the session will not have. Both are token-free. Two quirks, both
  verified, both easy to reintroduce: `claude auth status` exits **1** while
  reporting a logged-in account, and `codex login status` prints its verdict to
  **stderr** — so exit status is ignored and both streams are read
  (`probe::combined_output`). Auth output carries the account email/org; only
  the bool may escape.
- The live end-to-end run is `crates/daemon/tests/live_direct_run.rs`, gated on
  `AUTOHARNESS_LIVE_TESTS=1` (it spends tokens). It drives the real daemon over
  the real socket with the real CLI. Verified green on both engines: objective
  to worktree to edit to sandboxed check to local commit, with the user's
  checkout unmoved.
- `CONTROLLED_PATH` omits `~/.local/bin`, where `claude` installs. Adapters use
  `engine_control_env_for(dirs, binary)`, which prepends the directory the
  binary was actually found in — that directory only.

### Phase 9 notes

- `integrity_report()` checks SQLite's own verdict AND the ledger invariants
  this product relies on (no orphan events, no undefined run states). It
  **reports, never silently repairs**: an append-only audit log that rewrites
  itself is not an audit log. The daemon runs it at startup and logs loudly,
  but does NOT refuse to start — refusing would also block the `app.export`
  that rescues the data.
- `app.purge_project` requires the caller to pass the project's own path back
  as `confirm_path`. Irreversible by design; a privacy control that left
  residue would not be one. `vacuum()` is separate because rewriting the whole
  file should be the user's explicit choice.
- `app.diagnostics` is deliberately paste-safe: paths and verdicts only, and a
  test asserts the client token never appears in it.
- `core::version` exists because string comparison would break at the ninth
  minor release — lexically `"0.10.0" < "0.9.0"`. Pre-releases are never
  offered to a stable user, and an unreadable feed is `Unknown` rather than
  "up to date", so a parsing change cannot silently hide every future release.
- `scripts/package.sh` builds a universal (arm64 + x86_64) `.app` with the
  daemon and independently signed updater beside the app binary — exactly
  where the UI looks for them — plus a ZIP and DMG. Signing and notarization
  are **opt-in via `SIGN_IDENTITY` and `NOTARY_PROFILE`, and never faked**:
  without them the script says plainly that Gatekeeper will refuse the output
  on another Mac. A pinned feed is compiled in only when
  `RELEASE_TEAM_ID` and `RELEASE_FEED_URL` are both supplied. The
  hardened-runtime entitlements allow JIT/unsigned memory for the Node-based
  provider CLIs; the app itself is granted no broad network entitlement.
- The app performs update I/O through fixed system binaries with fixed
  arguments, not a general daemon HTTP client. `/usr/bin/curl` may fetch only
  the compile-time `.json` URL without redirects, and the asset must be a
  same-origin HTTPS ZIP. Private staging, SHA-256 verification, symlink and
  archive-shape rejection, bundle ID/version/Team ID checks, codesign and
  Gatekeeper assessment all precede helper launch. The helper independently
  rechecks identity and active-run state, atomically swaps the bundle, and
  rolls back on swap or relaunch failure.

#### Intentional architecture exclusion

- **Raw interactive PTY ownership.** Neither structured provider emits a cell
  grid, and daemon-owned checks are captured rather than interactive. The app
  renders their ANSI output and real unified diffs; inventing a terminal input
  channel would add authority without a producer or V1 workflow that needs it.

Production acceptance of the update path requires inputs that are deliberately
not committed: the owner's Developer ID identity and Team ID, an Apple
notarization profile, and the hosted pinned feed and ZIP. A local unsigned
build therefore reports updates unavailable; it must never pretend that an
unverified artifact is installable.

### The UI runs on GPUI

`crates/ui-gpui` replaced a hand-rolled vello + parley shell. That shell drew
every row, clipped every string with a `chars * size * 0.55` estimate, and
tracked its own scroll offsets — which bought custom rendering nothing needed
and cost clicking, hovering, scrolling, and focus. Those estimates were the
direct cause of a tint bleeding past a panel edge and a status chip colliding
with the title it annotated.

- **GPUI is a git dependency on Zed**, pinned to the revision diri uses. It is
  not on crates.io. Bumping the rev is a deliberate act, not a `cargo update`.
- **Building it needs the Metal toolchain**: `xcodebuild -downloadComponent
  MetalToolchain` (688 MB, one time). Without it `gpui_macos`'s build script
  fails on shader compilation, which reads as a confusing cargo error.
- Scrollable containers are stateful elements: `div().id(...)` before
  `.overflow_y_scroll()`, or it does not compile.
- Rust 2024 makes `impl Trait` capture every in-scope lifetime, so a view built
  from a borrowed `MutexGuard` keeps borrowing it. View helpers return
  `impl IntoElement + use<>` to opt out.
- `client.rs` and `theme.rs` carried across unchanged. The UI is a socket
  client; nothing in `core`, `protocol`, `store`, `engines`, or `daemon` knows
  what draws the pixels, which is what made the swap contained.

### Switching engines mid-conversation

A provider session belongs to its provider: a Codex thread id means nothing to
Claude. `run.start` therefore refuses to resume across engines — that check is
correct and must stay — but refusing alone left the new engine starting blind,
which is what made switching models look like the app forgot the conversation.

`handoff::brief` composes what actually happened from the ledger and prepends
it to the new engine's first turn, emitting `run.handoff` so it is auditable
and replayable.

- **From the ledger, not from the outgoing model.** Free (no model call, no
  latency), deterministic (a replay reconstructs exactly what the second engine
  was told), and available even though the previous engine's process is
  usually gone. Asking a model to summarize its own work would read better and
  be precisely the claim a handoff should not take on trust.
- The brief ends by telling the new engine to **verify what it depends on**:
  the notes are what was claimed and what the checks reported, not a guarantee.
- Bounded per section (`MAX_ITEMS`, `MAX_LINE`), because it is prepended to a
  real prompt. A test drives 200 chatty turns to prove it stays small.
- Same-engine follow-ups get NO brief: they resume the real session, and a
  summary would be strictly worse than the context the model already holds.

### Why there is no terminal emulator

diri's `diri-term` is not one either: it depends only on `diri-proto` and
`gpui` and draws a pre-computed cell grid. The PTY, the escape parser, and the
screen model live in their Swift daemon (`HeadlessScreen.swift`), which ships
`GridUpdate` deltas to the client.

AutoHarness has nothing to feed such a renderer. Its engines speak JSON lines
over stdio, and the only thing that produces terminal output is a daemon-owned
check command whose stdout and stderr the sandbox captures as strings. Porting
a grid renderer would build a viewer for data this product never emits.

What was ported instead is the part that applies: the colour model, in
`src/ansi.rs`. It parses SGR from captured output into styled spans — no
cursor, no scroll region, no alternate buffer, because nothing here sends one.
Non-SGR sequences are consumed and dropped so they cannot surface as mojibake,
background colours are parsed but NOT applied (a build log painting over the
themed pane looks broken), and malformed input degrades to plain text. Build
output is untrusted input like anything else a model produced.

`run.check` carries 16 KiB tails, and the review pane shows a failing check's
real output rather than a one-line verdict.

### The command palette

`Cmd-K` opens it. `crates/ui-gpui/src/palette.rs` ports diri's model:

- **Invisible keywords, penalized.** A run matches its project, engine and
  state — none of which its row prints — but a title hit always outranks a
  keyword hit, so typing what you can see wins. Tests assert both directions.
- **Curated order is the tiebreak**, so an empty query renders the list a human
  arranged and equal scores never shuffle between frames.
- The matcher is a ~40-line subsequence scorer, not a dependency: consecutive
  and word-start matches score higher, earlier matches score slightly higher.
  It ranks tens of rows, not thousands.
- The palette owns the keyboard while open, which is why it is a mode on
  `Shell` rather than another pane.

### The design system

`crates/ui/src/theme.rs` is a port of diri's design system
(<https://github.com/cristicretu/diri>, Apache-2.0), attributed in `NOTICE`.
Change tokens there, never inline in `lib.rs`.

Two rules the tokens encode, worth keeping:

- **Text tone is alpha over ONE foreground**, not a palette of greys. A
  secondary label is the primary colour at 60% (70% on a sidebar, which sits
  over denser material and loses perceived contrast otherwise). There is never
  a "which grey was that" question.
- **Status colour is semantic, and "needs you" outranks everything.** Amber for
  a run wanting input, red when the ask carries risk, green for freshly
  finished work, and anything settled fades into the neutral scale. Settled
  states draw as an outline rather than a filled dot, so the eye lands on what
  is live or waiting. A test asserts no other state can collide with "needs
  you".
- The type scale is three sizes (11/13/15) separated by WEIGHT. A test asserts
  it stays three; adding a fourth size is how a dense tool becomes unreadable.

New ledger event: `run.diff` carries the run's patch (bounded to 256 KiB, cut
on a line boundary so the UI never parses half a hunk). The UI parses it into
classified lines and renders file headers, hunks, line numbers, and tinted
additions/removals — review without leaving the app.

`cargo run -p autoharness-ui-gpui --example shell_preview` opens the
deterministic populated cockpit — a live graph, both engines, and review
evidence. Capture the real window with `screencapture` and LOOK at a change
instead of guessing. **Keep UI strings ASCII** unless a render proves the
glyph exists — status marks are drawn shapes, not text.

### Phase 7 notes

- The graph is laid out by **execution wave**, one column per wave, because
  waves ARE the schedule: nodes drawn side by side genuinely run side by side.
  Node colour is the live state, so the picture answers "what is happening"
  without reading the tail.
- The whole graph is rebuilt from ledger replay (`run.awaiting_approval` then
  `node.started`/`node.finished`), so a relaunched UI shows the same plan and
  the same progress. Nothing is held only in memory.
- The review tail is effectively virtualized already: `draw_lines` lays out
  only the lines that fit, so `PageUp`/`PageDown` just move a window over a
  history that costs nothing to hold. `/follow` returns to live.
- `/open` shells out to `/usr/bin/open` on a local path. It is read-only from
  the product's side and is NOT an external write — it hands the user their
  own worktree.
- **Intentionally excluded:** raw interactive PTY ownership. A real PTY needs
  a pseudo-terminal, escape parser, mutable cell grid, and raw user input;
  AutoHarness engines instead speak structured JSON and daemon checks return
  captured output. The applicable review experience does ship: ANSI-styled
  check tails plus line-numbered, classified and tinted unified diff hunks.

### Phase 8 notes

- **A verified fact must carry evidence.** `add_memory_fact` refuses a
  verified fact with an empty evidence list, and `verify_memory_fact` refuses
  to promote one. Unevidenced facts are storable as proposals but never
  listed as verified and never injected.
- **Superseding is materialized, not derived.** Inserting a fact with
  `supersedes` DELETES the old row. A derived "not superseded by anything"
  query looks equivalent until you forget the newer fact — then the stale one
  resurrects and the project is described by what it used to be. Full history
  stays in the event ledger.
- FTS5 has its own query syntax, so user text goes through
  `sanitize_fts_query`, which quotes each word as a literal term. A stray
  quote or a bare `NEAR` would otherwise be a syntax error or an unintended
  query.
- `core::policy::validate_candidate` is an allow-list AND a deny-list. The
  exact allow-list is checked FIRST: `approved_graph_templates` contains the
  substring "approve", so a marker check running first would refuse a field
  PLAN.md explicitly permits. Ceilings may be lowered, never raised — a policy
  that could raise its own limits could spend without bound.
- **Promotion re-validates at promotion time**, not only when a candidate was
  stored, so a candidate cannot move a capability boundary just because time
  passed. Promotion is exclusive (exactly one row), which makes rollback
  simply "promote the older version" — exact, not an undo.
- New ledger events: `policy.promoted`, `policy.rejected`, `memory.forgotten`.

### Phase 6 notes

- **A model proposes; Rust decides.** `graph::compile` returns ALL violations,
  not the first — a user judging a plan deserves the whole picture — and an
  invalid plan falls back to a bounded loop rather than running.
- "Connected to a final outcome" is enforced as **exactly one sink**. In a
  finite DAG every node reaches some sink, so the naive orphan check can never
  fire; the rule that bites is that the plan must converge. A plan ending in
  several places has branches whose work nobody consumes.
- `Role::parse` is lenient about the model's vocabulary but defaults unknown
  roles to `Editor`, the MOST constrained kind — never the least.
- Scope collision uses a conservative glob overlap: a false collision costs a
  bounded loop, a missed one corrupts a merge. Integration nodes are exempt
  from colliding with the editors they merge, by design.
- Waves are genuinely concurrent (`tokio::spawn`), and the compiler has already
  capped wave width at the budget's worker limit. A sequential "join" here
  would silently make a swarm pointless.
- Every node gets its own worktree AND branch (`ah/node-<id>`), including
  read-only nodes — sharing the base checkout with a live worker is exactly the
  collision worktrees exist to prevent. Integration merges its dependencies'
  branches into its staging worktree BEFORE the engine sees it; a merge
  conflict is left in place for the model to resolve, not treated as an error.
- A node whose dependency did not succeed is `Skipped`, never run: its inputs
  do not exist and running it would produce confident nonsense.
- New ledger events: `run.awaiting_approval`, `run.approved`, `graph.rejected`,
  `graph.wave_started`, `graph.finished`, `node.started`, `node.finished`,
  `node.check`, `node.conflict`.
- Graph versions are append-only (`store::save_graph`): a replan adds a
  version, it never rewrites the plan the user approved.
- Still open: the structured planning CALL that produces a `GraphProposal`.
  `run.start` passes `None`, so graphs are reachable via `run.approve` with a
  stored plan (as the tests do) but no engine proposes one yet.

### Phase 5 notes

- **Routing bias is one-directional: when in doubt, run something simpler.** A
  low-confidence or unsupported plan falls back to `BoundedLoop`; nothing ever
  escalates to a swarm on uncertainty. `route()` is pure — the daemon gathers
  `TaskFacts` and decides nothing itself.
- "Atomic" requires no multi-step marker AND no parallel language. "Add a
  docstring to each function across the adapters" is twelve words but dozens of
  edit sites, and treating it as one direct edit was a real bug caught by the
  router tests.
- A direct run REQUIRES a check command; without one the router picks a loop.
  A direct run has no second attempt, so its correctness rests entirely on that
  command.
- **The detector's hard problem is not catching loops, it is not catching
  productive work.** TDD repeats the same failing command on purpose, research
  reads twenty files without editing, a long build emits nothing. Every trigger
  is gated on an explicit absence of `Progress`, and there are false-positive
  fixtures for all four cases — do not add a trigger without one.
- `normalize()` collapses digits to `#`, so `file1`/`file2` share a
  fingerprint. That is intended (two runs of the same work must compare equal)
  but it surprises tests: use non-numeric names when you want distinct actions.
- The recovery ladder is spent once per run. Progress clears the loop evidence
  but does NOT rewind the ladder — a run that already needed a nudge and got
  stuck again goes to replan.
- Budgets are hard ceilings, not heuristics: crossing one blocks the run
  instead of escalating the ladder. Blocked is not failure — the worktree is
  preserved and the exact evidence is on the ledger.
- New ledger events: `run.routed`, `run.detector`, `run.loop_iteration`,
  `run.checkpoint_restart`. `run.blocked` now also carries detector evidence.
- The daemon's approval gate for swarms/DAGs exists but is unreachable until
  Phase 6 supplies a planning call — `route()` is currently passed `None`.

### Phase 4 notes

- Worktrees live at `<data_dir>/worktrees/<run_id>` on branch `ah/run-<id8>`.
  `worktree::create` fails (and the run is blocked) if the path already exists,
  so a Blocked run cannot be restarted in place yet — adopting an existing
  worktree belongs with Phase 5's recovery ladder.
- Direct-run ledger order for a successful run: `run.created`,
  `run.worktree_created`, `run.started`, engine events, `run.check` (only with
  a `check_command`), `run.commit`, optional `run.worktree_removed`, and
  `run.succeeded` LAST. Cancel ends with `run.cancelled` last. Tests assert
  this order — extend it deliberately.
- Steering never interrupts a turn. `chat.send` queues; the run task drains the
  queue at each `engine.completed` and finalizes only when it is empty.
  `chat.interrupt` is the deliberate mid-turn path.
- Failure paths always preserve the worktree, even an empty one — fail-safe by
  choice. Only cancel and a no-op success reclaim. Garbage-collecting worktrees
  and `ah/run-*` branches from long-dead failed runs is Phase 9 (repair tools).
- `FakeEngine::gated` holds a turn's events until the test releases the gate —
  the only deterministic way to simulate an engine editing the worktree.
  `sandbox::ready_for_tests(data_dir)` gives daemon tests the REAL Seatbelt
  backend without the canary/proxy startup cost.
- Daemon tests build real temp git repos and run real `sandbox-exec`; test
  fixtures may shell out to git directly, product code may not.
- The UI's input line is slash-command driven: `/add <path>`, `/projects`,
  `/engines`, `/p <n>`, `/engine codex|claude`, `/check <cmd>`, `/new`,
  `/pause`, `/resume`, `/cancel`, `/i <text>`; anything else starts a run or
  steers the live one. Panes truncate rather than wrap (they are log tails).
- **The UI starts the daemon itself** when the socket is absent
  (`connect_starting_daemon_if_needed`). If the socket EXISTS but refuses a
  connection it waits instead of spawning — a second daemon rebinds the socket
  and would steal it from the live one. `autoharnessd` is looked for beside the
  running binary, then one directory up (dev builds run from `examples/` and
  `deps/`), then PATH.
- The UI calls `engine.list` on connect, shows each engine's state in the
  projects pane with the CLI's own sign-in command (`codex login`,
  `claude auth login`), preselects an engine that is actually ready, and
  refuses to start a run on one that is not — with the fix in the status line
  instead of a daemon rejection.
- Run status uses diri's vocabulary (`client::run_status_glyph`): working /
  needs you / done, so a glance answers "does this want me?".
- `cargo run -p autoharness-ui-gpui --example client_smoke` verifies handshake,
  replay, and `project.list` against a running daemon without a GPU window. It
  deliberately never starts a run (that would spawn a real CLI and spend
  tokens).

---
> Source: [codejunkie99/autoharness](https://github.com/codejunkie99/autoharness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
