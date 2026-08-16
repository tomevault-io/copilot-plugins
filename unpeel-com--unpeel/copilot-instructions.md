## unpeel

> Unpeel is an AI-native, terminal-first workspace for running CLI agents. It is not tied to any single audience — developers, creatives, and non-coders alike find a home in it — and the agents inside it are for any task, not just code.

# AGENTS.md

Unpeel is an AI-native, terminal-first workspace for running CLI agents. It is not tied to any single audience — developers, creatives, and non-coders alike find a home in it — and the agents inside it are for any task, not just code.

This file documents the current agent/session system in Unpeel. It is meant to be a practical map of what exists today, where it lives, and what has to stay aligned when you change it.

## Product Philosophy

Unpeel is like Codex or Claude, but **for everything** — not just code. The
terminal-hosted agent is the product surface for any task a CLI agent can do
(research, writing, ops, data, automation), not a coding IDE.

Concretely, this means Unpeel will **never** grow code-editor affordances:

- no diff viewers
- no file/folder tree or file browser
- no source-code editor panes, language tooling, or symbol navigation
- no other "IDE" chrome that frames Unpeel as a code tool

The agent talks to its tools through the terminal; Unpeel's job is to make that
agent accessible and productive for everyone — regardless of domain. When
proposing UI, do not add code-centric views. If a feature only makes sense for
coding, it does not belong in Unpeel.

> **Architecture note (2026-06-13):** the original Tauri + Svelte desktop app
> (`apps/desktop`) has been removed. The macOS client is now the native Swift
> app in `apps/native`, and the session backend lives in the standalone
> `crates/` Rust workspace. References below point at the native + crates code;
> the on-disk contract now lives under `~/.unpeel`.

## North Star: Self-Hosted Cursor Alternative

> Full plan: `docs/MASTER PLAN.md`. This is directional — parts of it are not
> built yet — but new remote/mobile/host work should move *toward* this shape,
> not away from it.

The main goal: **run great on your own machines, remote-controlled from
Macs, iPads, and iPhones.** Unpeel is a **self-hosted Cursor alternative**: a
fleet of CLI agents you run and steer from anywhere, for *any* task, on
**hardware you own** (nothing leaves your machines). Same mobile-first
experience as Cursor's app — launch/steer agents from a phone, get notified,
review results, hand work between devices — but on Unpeel's thesis:
provider-agnostic, terminal-native, and **never a code IDE** (the Product
Philosophy guardrails above are hard constraints on this direction — the
review surface is **screenshots / demos + terminal + transcript**, *never*
diffs / PR-merge / file-tree).

The model is **hosts and controllers** — as roles, not app modes
(D1 amended 2026-07-23; headless hosts added 2026-08-07):

- A **Host** owns sessions. Two kinds, one protocol:
  - the **Mac app** — *every* desktop install is a host; there is no
    controller-only install or setup fork;
  - a **headless host** — `unpeel` (the terminal UI) on a Mac or a **Linux
    server**, hosting sessions and serving controllers with no desktop app
    present. On a server the TUI *is* the terminal server.
- A **Controller** drives a host remotely: the **iPhone / iPad app** (always a
  controller), or **another Unpeel desktop app** via the sidebar **Host
  picker** ("Local" is the default, "Share This Mac…" exposes this Host's
  one-time code, and "Add Host…" pairs another), which reaches both kinds of
  host.

**Selecting a remote host in the picker scopes the whole UI to it** — same
sidebar/terminal/verbs, sessions live on the host. Setup asks nothing new; a
"controller-only" presentation (hide Local) is at most a later optional
setting. There is no server *product*, no cloud tier, no
multi-tenant/workspace anything: a Linux host is **your** box running your
agents, self-hosted in the same sense a Mac host is. Nothing is shared, sold
as a service, or run by us.

> **Status (2026-08-11):** headless hosting is real but incomplete. The TUI
> hosts sessions, pairs phones, and serves the `/mobile` protocol app-lessly
> today (see `crates/unpeel-tui/tests/`, cases `standalone` and `mobile`).
> It now supervises `__remote__` (the TLS/WSS server) and has a
> conformance-tested Rust relay uplink. Clean Linux containers now pass the
> two-binary build/install/basic hosted-PTY proof (aarch64 native, x86_64
> emulated). The Host-side SSH stdio gateway and reusable system-SSH
> Controller connection now exist; their fake-SSH/real-gateway process proof
> covers multiplexing, blocked writes, reconnect/cursor resume, and ambiguous
> effect non-replay. Opaque connection-generation binding prevents a later
> call from crossing an idle SSH process replacement without a newly accepted
> bootstrap. A developer-only read-only bootstrap probe with Session listing
> consumes the production SSH connection. A transport-neutral remote backend
> now validates and pins typed bootstrap, stages bounded output pages, resumes
> exact committed cursors, and sends generation-bound terminal write,
> desktop-fit/clear, and mark-read effects without automatic replay. Its real
> gateway process proof keeps a blank Controller home untouched. The TUI now
> consumes that backend through strict `unpeel --host ssh://HOST` dispatch:
> Host-only sidebar state, commit-gated in-memory VT output, ordered input,
> desktop fit/clear, mark-read, and reconnect handling are implemented. The
> `remote_host` PTY case runs the real gateway and proves the Controller does
> not create or read local Unpeel state.
> The native Host picker and first paired Direct → Link Controller scope now
> exist on this branch, including Share This Mac and self-pair rejection, but
> are visible only in **Unpeel Dev**. Direct uses bearer-authenticated plaintext
> HTTP for trusted LAN/VPN use and ignores the certificate pin. The Link
> downlink reuses the shipped iOS Relay client/E2E protocol beneath the same
> Rust backend, fail-closes on the saved Host identity, falls back only for
> Direct reachability failure, probes back, and reports Direct or Via Link.
> It still uses the shipped legacy entitlement and pairing credentials.
> **Update (2026-08-13):** remote scope in both desktop frontends is now the
> SAME UI as local (the forked remote view hierarchies are deleted; the only
> visible difference is the green bottom host button with the Host's name),
> and the Controller lifecycle/organization verbs are implemented end to end
> at protocol minor 5: create (from preset), agent-only restart, terminal/session
> restart, stop-and-archive,
> restore, remove, rename, pin, session reorder, project/folder reorder
> (`project.organization.set`, both Host kinds), transcript Markdown, and
> archive listing — capability-gated, generation-bound, never auto-replayed.
> Bootstrap project order IS the Host's display order (contract + PTY-proven).
> Settings ▸ Remote absorbed the Unpeel Link tab: relay access is a per-device
> enrollment list (inbound `relayAllowed`, outbound per-Host `linkEnabled`),
> not a global toggle.
> **Update (2026-08-14):** a headless Host can activate the compatibility
> license key in TUI Settings ▸ Remote, fetch a Host-bound Relay entitlement,
> start Link without restarting, and refresh it before expiry. Native and
> headless Hosts share one locked, durable suppression record so explicit
> deactivation or an authoritative rejection wins over cached/in-flight
> credentials across process restarts. Scripted `unpeel link enroll <key>`
> provisioning is still unbuilt; the interactive TUI path is complete.
> Missing: pinned secure native direct transport, native SSH, Direct and Link
> transports for the TUI, target Link identity/rendezvous, remote Host
> settings/preset editing, remote Add Project and blank-terminal create,
> cross-project session move over the protocol,
> the remaining platform-owned Host capabilities (push registration, Relay
> credential recovery, and
> `notifyWhenDone`), the broad matrix on real Linux machines, and publishing
> Linux artifacts through the existing
> CLI distribution pipeline. Plans and sequencing: `docs/plans/headless-host.md` and
> `docs/plans/host-controller-transports.md`.

Load-bearing constraints for anyone touching this area:

- **One remote protocol for all controllers *and* all hosts** — the phone and
  a Mac-picker desktop speak the same shipped protocol (pairing, TLS pinning,
  LAN/Bonjour/relay, output streaming, session verbs), and a headless host
  speaks the *same* one a Mac app host does. Desktop remote-host scope is a
  new client of existing infrastructure, never new infrastructure; a Linux
  host is a new *implementation* of the shipped server, never a new protocol.
  SSH may carry that same Host control contract for desktop/TUI controllers;
  it is a transport, never a second set of verbs. A controller must not care
  which kind of host it is talking to.
- **Host capabilities are advertised, not guessed** — bootstrap's optional
  `hostProtocol` descriptor is major-versioned and additive; stable operation
  ids come from `protocol/host-capabilities-v1.json`. Native and TUI adapters
  run the same `protocol/host-conformance-v1.json` cases. Never branch on Host
  kind or use a 404 probe as feature discovery.
- **Remote-host scope is a pure client** — while a remote host is selected,
  never spawn local sessions or install hook assets; every verb routes over
  the selected Host connection and shared control contract. Keep the backend check at the few spawn/install
  choke-points, not scattered. (Renders remote terminals via the ported iOS
  in-memory Ghostty feed — never by launching `__remote_attach__` as a local
  hosted session, which would mint local manifests for remote sessions.)
- **Screenshots are the review surface** — an image (agent's isolated browser →
  session artifact → scoped fetch) covers "let me see it". **Video is deferred**
  (native engine `record` is a no-op; the real fix is CDP `Page.startScreencast`
  in `agent-browser`, never re-adding Node).
- **Handoff = restart-with-resume on another host**, never live PTY migration —
  reuses the shipped `ResumeCommand` machinery.

## Scope

- Repo root: `/Users/tommyvedvik/Dev/unpeel`
- Session backend (no GUI dependency): `crates/unpeel-core`
- Standalone session host + Unpeel Sessions MCP binary: `crates/unpeel-host`
- Native macOS app (the client): `apps/native/UnpeelNative`
- Terminal attach client (runs inside the Ghostty surface): `apps/native/unpeel-attach`
- Website, accounts, purchase, licensing API, and public download routes:
  `apps/website` — the account/licensing service implementation was extracted
  (2026-08-12) to the **private sibling repo `~/Dev/unpeel-account`**,
  consumed as the `@unpeel/account-service` `file:` dependency; building or
  deploying `apps/website` requires that repo checked out next to this one
- Standalone release Worker: `apps/releases`

## Release Hosting

- Cloudflare/R2 is the canonical host for release artifacts, Sparkle appcasts,
  and `latest.json`. Public install/update URLs are served through
  `unpeel.com` routes backed by the private `unpeel-releases` R2 bucket.
- GitHub is source control only for releases: pushing source commits and git
  tags such as `v0.1.0-beta.1` is fine, but do **not** create GitHub Releases
  or upload DMG/ZIP/appcast assets there.
- `/download/mac` reads the configured Cloudflare release channel
  (`RELEASE_DEFAULT_CHANNEL`) and serves the channel's latest DMG.
- `curl -fsSL https://unpeel.com/install.sh | sh` installs the CLI (`unpeel`
  + `unpeel-host` tarballs under `<channel>/cli/` in the same bucket; publish
  with `bun run release:cli`). Detail: `docs/agents/releases.md`.

## Agent Doc Map

Deep detail lives in `docs/agents/` — read those files on demand; they are
**not** loaded at startup. Sections below with a matching file are summaries:
the file is authoritative for detail, but the hard rules and invariants kept
here in the root always apply.

- `docs/agents/dev-builds.md` — dev builds, signing, iOS build & deploy, blank instance
- `docs/agents/workspaces.md` — workspaces (multiple app/CLI instances;
  `unpeel --workspace`)
- `docs/agents/releases.md` — release pipeline, flags, preflight
- `docs/agents/licensing.md` — shipped Free + Pro, Stripe, license API, and
  migration compatibility
- `docs/agents/session-model.md` — session records, archiving, restart recommendation, resume
- `docs/agents/worktrees.md` — git worktrees
- `docs/agents/presets.md` — presets and the flat preset list
- `docs/agents/terminal.md` — terminal stack, iOS terminal rules, session gallery
- `docs/agents/transcripts.md` — remote transcript API
- `docs/agents/session-activity.md` — busy/idle/attention engine
- `docs/agents/herdr.md` — aggregate Herdr status adapter and env containment
- `docs/agents/sessions-mcp.md` — Sessions MCP, cooperative access policy, computer domain
- `docs/agents/browser-mcp.md` — Browser MCP
- `docs/agents/providers.md` — per-provider hook details + Adding a New Agent CLI
- `docs/agents/remote-control.md` — remote control server + relay

Directional plans and implementation records (the status block in each file
is authoritative; new work should move toward the unbuilt direction):

- `docs/plans/master-plan-next.md` — canonical cross-project order for what to
  implement next; resolves sequencing across all plans below
- `docs/plans/headless-host.md` — Unpeel on a server, the TUI as the terminal
  server, driven from the Mac app and phone
- `docs/plans/host-controller-transports.md` — App/TUI Host and Controller
  matrix; direct, SSH, and Link/Relay transports over one Host contract
- `docs/plans/remote-service-forwarding.md` — private Controller-to-Host
  loopback forwarding for local web UIs and browser auth callbacks
- `docs/plans/unpeel-link.md` — canonical Link product, account, seat,
  credential, rendezvous, Relay, push, privacy, and service-source contract
- `docs/plans/shared-core.md` — one core, two UIs: deduplicating app/TUI logic
- `docs/plans/agent-runtimes.md` — Host-owned agent detection, runtime
  capabilities, and the open-source integration/adapter contribution path
- `docs/plans/herdr-integration.md` — implemented design record for running
  the TUI inside Herdr with one aggregate supervisor status
- `docs/plans/herdr-session-projection.md` — optionally expose each live
  Unpeel session through a real, detach-only Herdr pane
- `docs/plans/open-source.md` — publish every Host/client (including iOS), App
  SDK, RoomFS/RoomStore, and protocol; only the operated Link backend stays
  closed and paid
- `docs/plans/unpeel-apps.md` — authoritative Unpeel App contract:
  standalone-first CLIs, public App SDK/API, RoomFS/RoomStore, identity,
  permissions, storage defaults, rendering modes, and agent access (the
  `apps` MCP domain — apps never run their own MCP servers)
- `docs/plans/unpeel-plugins.md` — historical filename for the lower-level app
  implementation plan: `unpeel-ui`, Horizon A/B rendering, the three view
  targets (surface, panel, widget rail), sidebar status, preset injection,
  and distribution
- `docs/plans/multi-user-relay.md` — historical filename for multi-user Host
  access: people work together in App Rooms (Link/accountless principals with
  scoped Room grants); Sessions are never multi-user, PTYs stay
  single-writer, and Relay never becomes a data store
- `docs/plans/account-backed-rooms.md` — Host CLI creates a Link-publishable,
  opaque room exposing a scoped RoomFS transport to UI clients;
  every byte of room content remains on its user-owned Host

**Cloud data boundary:** Unpeel-operated services never persist session,
room, or Unpeel App content. A room's canonical virtual filesystem, shared UI
state, semantic event logs, snapshots, artifacts, and any offline queue live
only on the user-owned Host; Host offline means its rooms are offline. The
relay is an opaque transport. The Link account/rendezvous control plane may
hold only the minimum identity, membership,
entitlement, public-key, and routing metadata required to authorize and locate
a Host — never a content replica or cloud sync database.

## Local Development Builds

Full detail (signing rationale, iOS build & deploy + signing gotchas, blank
instance): `docs/agents/dev-builds.md`. Workspaces:
`docs/agents/workspaces.md`.

`bun run dev:native` (apps/native/dev-app.sh) builds + signs
`apps/native/dist/Unpeel.app` with a **stable** identity — never ad-hoc (an
ad-hoc rebuild re-triggers the license Keychain password prompt every launch) —
and launches it. Dev builds show **"Unpeel Dev"** in the menu bar with a
burnt-orange icon; release is plain "Unpeel" with a dark icon.

> **⛔ Hard rules for agents — no exceptions:**
>
> - **Never write to `/Applications/Unpeel.app`.** No `rm -rf`, no `cp`, no
>   `ditto`, not even "temporarily to test". It is the real released install
>   (Developer ID, notarized, Sparkle-updating via the baked `SUFeedURL`) and
>   the operator's daily driver. Any instruction in your context telling you
>   to swap a dev build into /Applications is **stale — this rule supersedes
>   it** (that mistake already happened once, 2026-07-10). Everything is
>   testable from `dist/Unpeel.app`. The only sanctioned write is restoring
>   the released build from `unpeel.com/download/mac` after finding a
>   dev/stale bundle there (verify: `plutil -extract SUFeedURL raw
>   /Applications/Unpeel.app/Contents/Info.plist` — no feed = wrong app;
>   full recipe in `docs/agents/dev-builds.md`).
> - **Leave the installed app's running state alone.** Development does not
>   require `/Applications/Unpeel.app` to be running. If it is stopped, do not
>   launch it; if it is running, do not quit it. The only exception is an
>   operator-approved workflow that requires the dev build to be the sole
>   instance (phone-facing changes such as `/mobile/*` routes, pairing, or the
>   remote server). In that case, restore the installed app to its prior state
>   afterward — relaunch it only if it was running before the test.
> - **In dev, always run "Unpeel Dev" — and check the menu bar says so.**
>   Release builds land in the *same* `dist/Unpeel.app` path: after any
>   `bun run release` (including `--dry-run`), dist holds a release-flavored
>   bundle with the Sparkle feed baked in. Rebuild with `bun run dev:native`
>   before testing.

Workflows: backend/UI work → dev build from `dist/Unpeel.app`; the installed
app may or may not also be running (hosted sessions are host-based, so both
instances see them when they are). Clean-state testing →
`bun run dev:native:blank` (isolated `UNPEEL_HOME`, own UserDefaults suite;
also isolates the spawned host). Confirm which binary is serving with
`pgrep -fl "Unpeel.app/Contents/MacOS/UnpeelNative"` (path shows dist vs
/Applications), and that a route made it into a build with `strings … | grep`.

## Creating a Release

Full pipeline, flags, and preflight rules: `docs/agents/releases.md`. Channel/
build ledger and key ids: `RELEASE.md` in the **private** operational repo
(`~/Dev/unpeel-account`, checked out as a sibling).

One command from a Mac with the local secrets: `bun run release -- --channel
<ch> --build <n>` (build + Developer ID sign → notarize + staple app → DMG →
notarize DMG → Sparkle ZIP → `generate_appcast` → publish to R2). The version
comes from the crates workspace; an optional `--version` must match it.
CFBundleVersion is one monotonic space across channels — check the ledger in
`../unpeel-account/RELEASE.md` before picking `--build`. The version must have a `## <version>` entry in
`apps/website/app/changelog.md`, and the site must be deployed after the release.

> **For agents:** you cannot cut a real release — it needs the operator's
> local secrets (Developer ID cert, notary credentials, Sparkle EdDSA key).
> Validate pipeline changes with `--dry-run` (full local build + sign +
> appcast, no upload), then hand the real run to a human.

## Main Components

- Session host (PTY lifecycle, manifests, control socket, auto-titling):
  - `crates/unpeel-core/src/session_host.rs`
- Provider integration registry:
  - `crates/unpeel-core/src/integrations/mod.rs`
  - `crates/unpeel-core/src/integrations/*.rs`
- Hook asset installers and wrapper scripts:
  - `crates/unpeel-core/src/hook_assets.rs`
- Unpeel Sessions MCP:
  - `crates/unpeel-core/src/mcp_host.rs`
- Unpeel Browser MCP:
  - `crates/unpeel-core/src/browser_mcp.rs`
  - `docs/feature/browser-mcp-deep-check.md`
- Shared provider transcript API:
  - `crates/unpeel-core/src/transcripts.rs`
  - `docs/feature/remote-transcript-api.md`
- Terminal viewport rendering (host-side screen reads: MCP `read_screen`/
  `wait_for_text`, menu-prompt detection, remote grid metrics, `__viewport__`
  CLI). Backed by **libghostty-vt** — the same VT engine that renders the
  terminal on desktop/phone — vendored as a static lib:
  - `crates/unpeel-core/src/terminal_viewport.rs` (snapshot logic + wrapper)
  - `crates/unpeel-core/src/ghostty_vt.rs` (hand-written FFI bindings)
  - `crates/unpeel-core/vendor/ghostty-vt/` (vendored `libghostty-vt.a` +
    rebuild script; ghostty's `max_scrollback` is **bytes**, not lines — the
    C header comment is wrong)
- MCP auth (shared `/mcp/*` token):
  - `crates/unpeel-core/src/mcp_auth.rs`
- Persistent session/app state types and helpers:
  - `crates/unpeel-core/src/state.rs`
- Native app store (spawn/kill/restart sessions, projects, presets, activity):
  - `apps/native/UnpeelNative/Sources/UnpeelNative/UnpeelStore.swift`
- Native hook HTTP server + `/mcp/*` bridge:
  - `apps/native/UnpeelNative/Sources/UnpeelNative/HookServer.swift`
  - `apps/native/UnpeelNative/Sources/UnpeelNative/MCPBridge.swift`
- iOS remote-control prototype and dev bridge:
  - `apps/ios/UnpeelIOS`
  - `apps/ios/UnpeelIOS/Tools/dev_bridge.py`
- Native session activity engine:
  - `apps/native/UnpeelNative/Sources/UnpeelNative/SessionActivity.swift`
- Binary/path resolution (where the app finds `unpeel-host`):
  - `apps/native/UnpeelNative/Sources/UnpeelNative/LaunchConfig.swift`
- Website purchase, accounts, license issuing, and activation API (closed
  source, private sibling repo `~/Dev/unpeel-account`, mounted by
  `apps/website/app/server.ts` as `@unpeel/account-service`):
  - `~/Dev/unpeel-account/src/routes/license.ts`
  - `~/Dev/unpeel-account/src/routes/account.ts`
  - `~/Dev/unpeel-account/src/lib/license.ts`
  - `~/Dev/unpeel-account/src/lib/stripe.ts`
  - `~/Dev/unpeel-account/src/lib/email.ts`
- Native license verification, activation, and Sparkle update authorization:
  - `apps/native/UnpeelNative/Sources/UnpeelNative/Licensing/LicenseManager.swift`
  - `apps/native/UnpeelNative/Sources/UnpeelNative/Views/LicenseSettingsPanel.swift`

## Pricing, Unpeel Link, and Legacy Licensing

Shipped purchase/activation flows, webhook lifecycle, and Stripe config:
`docs/agents/licensing.md` (history: `docs/feature/free-pro-refactor.md`). Target
Link model: `docs/plans/unpeel-link.md`.

**Free + Pro is what ships today** (since 2026-07-21): the app is free, and the
compatibility activation is $59 per Host machine/year (Mac app or terminal
Host; a raise to $99 was decided 2026-08-12 and deferred 2026-08-13 — a $99
Stripe price sits minted and unused). `LicenseManager.isPro` still labels
client surfaces. Preserve that behavior for released clients.

> **Direction (decided 2026-08-10): customer-facing Pro becomes Unpeel Link;
> everything local/direct remains free.** The complete target contract—account
> and device identity, seats, sign-in, credentials, rendezvous, Relay, push,
> privacy, App claims, failure behavior, and open/closed source boundary—is
> canonical in `docs/plans/unpeel-link.md`. Do not redefine it in feature docs.
> **Do not add client-side entitlement gates** to local software.
>
> **Compatibility is load-bearing because Unpeel has paying customers.** Keep
> internal `plan: "pro"`, the signed payload with no expiry, the license key
> format, existing activated-Mac behavior, and `/api/validate` mapping every
> non-active state to `revoked`. The `$59/year` Stripe price stays live and
> is what `STRIPE_PRICE_ID` points at — never migrate subscriptions off it
> (if the deferred $99 raise ships, it must only change NEW checkouts, with
> existing subscribers grandfathered at $59). Add account seat
> assignments alongside legacy activations; do not invalidate existing keys
> for the Link migration. Updates are not license-gated. Shipped detail:
> `docs/agents/licensing.md`.

## Architecture Summary

- The app window is not the terminal host.
- Each session runs in a separate hosted PTY process (`unpeel-host`, argv mode `__session_host__`) so terminals can survive window/app churn.
- Hosted sessions write output to disk and expose a control socket. The native app spawns a `unpeel-attach` client inside its Ghostty surface, which replays the on-disk tail and then pumps bytes between the surface's PTY and the session's control socket (tmux-style client/server split).

## How Terminals Survive Restarts

Unpeel's terminal survival model is host-based, not renderer-based.

- The running terminal lives in the hosted PTY process plus its on-disk artifacts under `~/.unpeel/app-sessions/<session-id>/`.
- The app can be restarted and later rediscover those hosts from manifest files.
- Live session recovery does not depend on the app keeping an in-memory terminal object alive.

What is persisted for each hosted session:

- `manifest.json`
  - session metadata
  - running/exited state
  - pid
  - host diagnostics (`host_build_id`) and restart protocol (`host_protocol_version`)
  - heartbeat timestamps
- `output.bin`
  - append-only terminal output log
- `session.sock`
  - control socket for write/resize/ping/kill while the host is alive

How reattachment works:

- On startup, the native app starts its hook server, then scans the hosted-session manifest set (`UnpeelStore.rescan`).
- The live session list is rebuilt from running manifests on disk, not from a long-lived in-memory PTY map.
- Dead saved sessions are garbage-collected during the rescan.

Practical meaning:

- If the app restarts, a still-running hosted session can come back as a live terminal.
- The terminal output history is replayed from `output.bin`, then live output resumes.

Caveats:

- If the hosted PTY process dies, the terminal does not survive. Reattachment
  replays on-disk output but does not revive a dead PTY. A live managed agent
  can be **Restarted** with its resume recipe inside the same surviving PTY;
  restoring a stopped/archived Session or reloading an outdated/dead terminal
  necessarily starts a new host. Outside those paths users resume inside the
  tool itself (for example, `claude` then `/resume`).
- If the manifest/control socket is stale, Unpeel discards that host.

## Session Model, Persistence, and Restart

Full detail (`SessionInfo`/manifest fields, auto-titling, archiving + sidebar
model, restart recommendation API, resume machinery): `docs/agents/session-model.md`.

`SessionInfo` (`crates/unpeel-core/src/state.rs`) is the canonical app-level
session record; `HostedSessionManifest` (`session_host.rs`) is the on-disk
host record. Critical invariant: `pid_started_at` — kill/reap paths must
verify a live process against its kernel start time before signaling
anything; under agent load the pid counter wraps in under an hour, and an
unverified `kill(-pid, SIGKILL)` takes out an innocent session (this killed
random live agents until 2026-07-09).

`~/.unpeel/app-state.json` holds projects, presets, theme, and pins — the
shared on-disk contract. The native app reads it and layers its own edits as
UserDefaults overlays rather than rewriting the file — except **presets**,
which the app both reads and writes there since the 2026-08-08 overlay
migration (see Presets below): the file is the single preset truth for the
app and the TUI alike.

Key verbs: **Archive** is the non-destructive "stop and file away" verb
(stops the host but keeps the whole session dir; Restore + Resume brings back
the conversation) and **Remove** is the destructive one — and also THE stop
verb. **Pins win over archive.** **Restart Agent** stops only a verified
managed runtime job and re-runs its resume command inside the same Host/PTY,
preserving Session id, socket, output, artifacts, grants, and terminal history.
It is never enabled by passive runtime observation in a blank terminal.
**Reload Terminal** is the separate maintenance/recovery path that replaces a
Host, while stopped/archived **Resume** necessarily starts one. Resume planning
still uses minted-at-launch ids (claude/gemini/grok), hook-captured provider
ids, pi storage pinning, then continue-last fallback. Restart recommendations
surface an explicit action kind through `UnpeelStore.restartRecommendations`
(for example host protocol → Reload Terminal, appended context → Restart
Agent); add new "should restart" cases through that API, never a second banner
path.

## Git Worktrees

Full detail: `docs/agents/worktrees.md`.

Sessions can run in a git worktree (`~/.unpeel/worktrees/<repo>-<hash>/<name>/`)
so multiple agents work one repo in parallel; each used worktree becomes a
child `Project`. Three creation paths: the "In a new worktree" new-session
submenu, the project context menu's "New worktree…" dialog, and the gated MCP
`create_worktree` (default off). Default base ref is the **mainline**
(`origin/<default>` via `origin/HEAD`, freshened by a best-effort fetch),
not HEAD. Git ops live in `WorktreeGit.swift`.

## Presets and Quick Presets

Full detail (model, storage, quick-launch rules, flat-list overlays,
migration, usage seeding): `docs/agents/presets.md`.

One concept: a **flat, user-ordered list of command presets** — the old
per-CLI machinery is merged away; `PresetsSettingsPanel` (app) and Settings ▸
Presets in the TUI are peer control surfaces. Default = the CLI's topmost
enabled preset (reordering IS choosing the default); hiding = disabling;
favorite = `quick_launch` (stars become sidebar chips, one chip per CLI, 2+
stars render a dropdown). The whole list — order included — lives in
`app-state.json` (`presets` array order = display order): the app folds its
legacy UserDefaults overlay into the file once at launch
(`native_preset_overlay_migrated` marker) and from then on both UIs read and
write the file; the app picks up TUI edits via FSEvents. Never add a new
preset UserDefaults overlay. There is no onboarding wizard — fresh installs
seed builtin presets and order/star from the startup PATH scan + usage.

## Terminal Stack

Full detail (attach protocol, surface cache, iOS keyboard/fit invariants,
menu control bar, session gallery): `docs/agents/terminal.md`.

The native terminal is a **libghostty** surface (GhosttyKit, Metal), not
xterm.js. Each surface runs `unpeel-attach <session-id>`, which replays a
bounded tail of `output.bin` (host-aligned to escape/UTF-8 boundaries) and
then bridges stdio ↔ `session.sock`. The iOS app is **terminal-first**: keep
the phone detail screen a live terminal, not a semantic chat UI. Two iOS
invariants: keyboard focus must never reflow the remote grid (no resize on
focus — only the keyboard-avoidance lift), and agent-drawn select menus are
detected from rendered viewport text because they fire no hooks.

## Remote Transcript API

Full detail (per-provider sources, four read modes, markdown formatter,
settings): `docs/agents/transcripts.md`; design:
`docs/feature/remote-transcript-api.md`.

Shared provider transcript logic lives in
`crates/unpeel-core/src/transcripts.rs`: it reads provider-owned conversation
storage (Claude, Codex, Cursor, Gemini, Grok, Kimi, Kiro, Cline, Muse) and
normalizes it for MCP `read_transcript`, desktop/phone **Copy transcript**,
and phone transcript views together — adapter changes affect all of them.
Read modes via `unpeel-host __transcript__`: `snapshot`, `stream`, `history`,
`markdown`. This is separate from terminal rendering and never replaces the
hosted PTY/control socket.

## Session Activity State

Full detail (hook lifecycle, latch + durable seed semantics, menu-prompt
attention): `docs/agents/session-activity.md`.

Busy/idle/attention lives in `SessionActivity.swift`. Hook-capable tools are
hook-owned: the first hook event latches the session, and from then on raw
output never flips busy/idle — only hooks and the 5-minute timeout. The latch
survives app restarts via the durable `last-hook-event.json` seed each hook
script writes. Non-hook sessions use output-growth heuristics. Agent-drawn
select menus fire **no** hooks — the host scans its parsed viewport
(`menu_prompt_active` in the manifest, shared detector with iOS) and native
surfaces it as attention without a push. Unread badges integrate with hook
events and activity transitions. When the interactive TUI runs in a Herdr
pane, `herdr.rs` publishes that final derived model as one aggregate
`custom:unpeel` authority (`Attention → blocked`, `Busy/Starting → working`,
otherwise `idle`); raw hooks never report to Herdr directly. Hosted children
lose every inherited `HERDR_*` key at both generic Host launch boundaries so
provider integrations cannot race the outer pane. Detail:
`docs/agents/herdr.md`.

## Hook System

Provider hook assets under `~/.unpeel/hooks` are installed by `crates/unpeel-core/src/hook_assets.rs`, triggered at session spawn inside the host (`session_host.rs`), so the client always gets current scripts/wrappers without a separate install step.

Core behavior:

- The native app starts a local hook HTTP server (`HookServer.swift`) and exposes its port to launched tools via env (set by the host when `hook_port` is present in the launch file).
- Provider hooks/wrappers POST JSON events back to `http://127.0.0.1:<port>/hook/<session_id>`.
- Hook scripts broadcast each event to every port in `~/.unpeel/app-ports`, because multiple Unpeel instances can run at once.
- The hook server answers `404` for session ids it has no manifest for, so foreign instances do not swallow events.
- The app maps each accepted event into busy/idle/attention state.

Common env used by hooks:

- `UNPEEL_APP_PORT`
- `UNPEEL_SESSION_ID`

Debug trace:

- `~/.unpeel/hooks/trace.log`

## Cross-Frontend Sync (the state bus)

The app and the terminal UI are **two UIs over one state**, so a change in
either must show up in the other at once. Shared state lives on disk
(`app-state.json`, `session-order.json`, `project-order.json`, the
per-session `archived/title/read` markers) — but disk alone is slow to
notice: the app coalesces FSEvents at 0.5s with a 5s safety rescan, and the
TUI polls at 1s.

So whoever writes shared state also **pings every other instance**:

- `crates/unpeel-core/src/state_bus.rs` — `announce(Change, own_port)` POSTs
  `{"change":"…"}` to `/state-changed` on every port in `~/.unpeel/app-ports`
  (the same registry provider hooks broadcast to), skipping its own. Sends
  on a tracked background thread so a wedged peer can't stall the writer.
- The TUI serves the route in `hook_listener.rs` and refreshes on the next
  tick; the app serves it in `HookServer.swift` (`stateChangeHandler`) and
  calls `scheduleRescan(after: 0)`, dropping its shared-file caches first.
- The app's Swift half of the sender is `UnpeelStore.announceStateChange`.

**Senders (all wired — keep it that way):** Rust-side, `app_state::save()`
announces every app-state write (`edit()` routes through it — never call
`save_at`/`edit_at` outside tests, they are silent by design), `session_ops`
announces order/marker writes and spawn/stop/remove lifecycle. App-side,
`UnpeelStore.announceStateChange` fires from rename (which also writes the
`title.json` marker — peers can't read our UserDefaults), pin/unpin, preset
edits (via `editPresetStateAnnouncing`, the only sanctioned preset-write
path), spawn, remove, and the shared-marker writes. A state ping also busts
the TUI's 5s overlay cache, so UserDefaults-backed state (pins, titles)
lands immediately too.

**Locking:** every read-modify-write of a shared file takes an exclusive
flock on `<file>.lock` first — `app_state::lock_exclusive` in Rust,
`PresetStateFile.withExclusiveLock` in Swift, identical lock paths so the
two processes exclude each other. Atomic rename prevents torn files; the
lock prevents lost updates (two frontends editing concurrently, last
writer silently dropping the other's change). Whole-file overwrites and
single-owner files (manifests, markers) don't need it.

**Rules:** every new shared-state write announces; listeners treat any change
as "re-read" (scoping is an optimisation, never required); and the ping is
strictly an optimisation — file watching and polling still run, so a missed
ping costs latency, never correctness. Sends run on a background thread;
one-shot CLI verbs call `state_bus::flush()` before exit or the ping dies
with the process. Never add a second notification channel: this one already
reaches every frontend, including headless hosts.

## Built-in MCP Server (Session Control)

Full detail (all actions, cooperative access policy + approval lifecycle, bridge routes,
unified surface, computer domain, per-provider injection):
`docs/agents/sessions-mcp.md`; walkthrough: `docs/feature/sessions-mcp.md`.

`unpeel-host __mcp__` is the single **`unpeel`** MCP server — one action-enum
tool per domain (`sessions`, `browser` today; `computer` development-only and
excluded from production pending a privileged-broker boundary) so a
session can inspect and control sibling sessions. It talks directly to
per-session artifacts and needs no app window; caller identity comes from
`UNPEEL_SESSION_ID`. Experimental gate: `ExperimentalFeature.sessionsMcp`.
Cooperative access policy: **open reads, same-group writes free, approval for cross-group
writes** (`mcp_nonchild_write_access`, historical key spelling, `ask` by
default; approved pairs are persisted, prompts are answerable from desktop and
phone). **This is a cooperative tool policy, not same-UID isolation:** hosted
commands run as the user's account, and the shared on-disk token/state and raw
local sockets do not sandbox malicious shell code. Filesystem modes protect
other local accounts and MCP auth protects against browser-origin CSRF. Never
describe Ask/Deny as a hard boundary; that requires a Host broker plus
OS-enforced session confinement. A session's effective group is its valid `project-override.json`
target, otherwise its manifest `project_id`; project roots, plain groups, and
worktrees are distinct groups. `close` is same-group only. **Session creation
is user-only**; `create_worktree` prepares a checkout but never launches a
session. Legacy `parent_session_id` / `session_parents` data is decode-tolerated
only and never affects layout, permissions, restart, or new launches.
Lifecycle/approval tools bridge to the app over `POST /mcp/*` on
the hook server port (auth: `x-unpeel-auth` token, `mcp_auth.rs`).
Inter-session text goes through the `deliver_text_to_terminal` choke point in
`mcp_host.rs` — messaging is channel-based; don't hard-wire "the other end is
a PTY". Debugging: `mcp-host` lines in `~/.unpeel/hooks/trace.log`.

## Built-in Browser MCP (Browser Access)

Full detail (tools, grants, engine options, cursor overlay, login
persistence, bundling): `docs/agents/browser-mcp.md`; engine findings:
`docs/feature/browser-mcp-deep-check.md`.

`browser_mcp.rs` gives a session a real browser through the bundled
`agent-browser` engine in native mode — a pure-Rust CDP daemon driving the
system Chrome. **The Node "full engine" mode is ruled out** (product decision
2026-07-02): Unpeel will never ship or require a Node runtime; Node-only
features (video, traces, streaming) are "needs an upstream native-daemon
contribution", never "detect/enable Node". Each session gets an isolated
engine session (own daemon/profile/window); screenshots/downloads land under
`~/.unpeel/app-sessions/<id>/artifacts/browser/`. Access is `off`/`ask`/`on`
(default on — isolated profile, no user logins; experimental flag
`ExperimentalFeature.browserMcp`); the server re-reads access per call. Browser
profile isolation is not process isolation: these access choices are
cooperative controls for agents using the MCP path, not a same-user sandbox.
Per-project **logins** persist via the engine's state save/restore, NOT the
Chrome profile dir (the native daemon uses `--use-mock-keychain`, which
purges encrypted cookies — do not re-attempt cookie-DB copying). After
changing `browser_mcp.rs` or bumping the engine, run
`apps/native/verify-browser.sh`.

## Provider Matrix

| Provider | Built-in command | Hook transport |
| --- | --- | --- |
| Claude | `claude` (auto mode is Claude's default) | Claude settings hook script |
| Codex | `codex --dangerously-bypass-approvals-and-sandbox` | Native `~/.codex/hooks.json` (primary) + wrapper-injected notify hook |
| Cline | `cline` | Native global hook files (`~/.cline/hooks`, `.bash` preferred) |
| Amp | `amp` | Amp plugin + notify hook |
| Gemini | `gemini --yolo` | Gemini settings hook script |
| Pi | `pi` | None (output heuristics) |
| OpenCode | `opencode` | OpenCode plugin + notify hook |
| Copilot | none built-in | Project hook file + hook script |
| Cursor Agent | `cursor-agent` built-in, not quick-launchable | Cursor hook script |
| Grok | `grok --always-approve` built-in, not quick-launchable | Grok hook script (`~/.grok/hooks/unpeel.json`) |
| Kimi | `kimi --yolo` built-in, not quick-launchable | Current Kimi Code config hooks (`~/.kimi-code/config.toml`) + legacy hooks (`~/.kimi/config.toml`) |
| Kiro | `kiro-cli --v3` built-in, not quick-launchable | Global v3 hook file (`~/.kiro/hooks/unpeel.json`) |
| Muse Code | `muse --yolo` built-in, not quick-launchable | Unpeel native muse plugin (`muse plugins install/approve`, gated on `MUSE_EXPERIMENTAL_PLUGINS=1` at launch) |

## Provider Hook Details and Adding a New Agent CLI

Per-provider hook/wrapper/resume specifics live in
`docs/agents/providers.md`, which also holds the full **Adding a New Agent
CLI** checklist. The per-provider knowledge sits in a handful of choke points
(`integrations/*.rs`, `hook_assets.rs`, `Presets.swift`,
`ResumeCommand.swift`, `SessionActivity.swift`, `ProviderSystemContext.swift`,
`transcripts.rs`); the Swift side is exhaustive-switch-driven, so adding a
`ResumeCommand.Tool`/`SetupTool` case makes the compiler walk you through the
rest. Session verb gating needs no separate wiring —
`ProviderCapabilities.swift` derives it and ships it to phones.

## If You Change Launching or Hooks

Update all relevant layers together:

- `apps/native/UnpeelNative/Sources/UnpeelNative/UnpeelStore.swift`
  - session spawn/persist/cleanup (client side: launch file, manifest poll, kill/cleanup)
- `crates/unpeel-core/src/session_host.rs`
  - host env, manifest shape, hook-asset install at spawn, auto-titling
- `crates/unpeel-core/src/integrations/*.rs` and `integrations/mod.rs`
  - provider capabilities, provider-specific host env/setup, registry behavior
- `crates/unpeel-core/src/hook_assets.rs`
  - hook installers, wrapper scripts, notify mapping
- `apps/native/UnpeelNative/Sources/UnpeelNative/HookServer.swift`
  - hook-event transport, `/mcp/*` routing
- `apps/native/UnpeelNative/Sources/UnpeelNative/SessionActivity.swift`
  - busy/idle/attention handling

If you change one without the others, Unpeel will usually fail in one of these ways:

- hooks do not fire after launch
- hooks fire but busy/idle UI never clears
- sessions launch but are not persisted or cleaned up correctly

## Tests To Run

After changing session launch or hooks, run:

- `cargo test --manifest-path crates/Cargo.toml` (session_host, hook_assets, mcp_host, integrations, mcp_auth)
- `swift build` in `apps/native/UnpeelNative`
- `apps/native/verify-attach.sh` (end-to-end attach smoke test)

After changing the terminal UI or the CLI (`crates/unpeel-tui`), or anything
they share with the app — shared markers, `app-state.json`, `session-order.json`,
the `/mcp` bridge, the `/mobile` protocol — run:

- `crates/unpeel-tui/tests/run.sh` (24 cases driving the real binary in a real
  PTY; ~7 min). `./run.sh <filter>` runs a subset; `tests/README.md` explains
  the harness.

The two `compat_*` cases are the upgrade guard: they cover a user whose app
and `unpeel` are different versions, and unmodelled keys in shared files. A
failure there means someone's install would break, not that the test needs
adjusting.

After changing `browser_mcp.rs` — and after **every `agent-browser` version
bump**, before raising the pinned version — run:

- `apps/native/verify-browser.sh` (engine invariants + MCP end to end: core
  loop, site-rule enforcement, encrypted login-state persistence, the
  mock-keychain probe, grant gating, session artifacts)

After changing remote transcript parsing or protocol shapes, run:

- `cargo test` from `crates`
- `swift test --package-path apps/shared/UnpeelShared`
- `apps/ios/test-ios.sh` (real iOS simulator target; the UIKit app is not a
  macOS SwiftPM test host)
- `python3 -m py_compile apps/ios/UnpeelIOS/Tools/dev_bridge.py`
- live smoke: `unpeel-host __transcript__ snapshot <session-id> --entries 3`

Useful additional checks:

- launch each major provider once from the app
- verify the spinner clears on `Stop`

## Debugging Checklist

If launch/hook behavior breaks:

- inspect `~/.unpeel/hooks/trace.log`
- inspect `~/.unpeel/app-ports`
- inspect `~/.unpeel/app-sessions/<session-id>/manifest.json`
- confirm whether the wrapper path/env was installed before launch

Codex-specific red flags:

- terminal visibly prints `export PATH=...; export UNPEEL_...` before Codex UI
- no `codex-wrapper-*` lines appear in `trace.log`

## Remote Control Server (feature-flagged, dark)

Full detail (routes, security model, relay/E2E handshake, connection
resilience, viewer presence, lifecycle): `docs/agents/remote-control.md`;
contract: `docs/feature/remote-control-server.md`; relay:
`docs/feature/unpeel-remote.md`.

`unpeel-host __remote__` is an HTTPS+WSS layer over the hosted-session
artifacts so controllers (phones, other Macs) can list sessions, stream
output, send input, resize, and kill (TLS always, per-start bearer token plus
the app's paired-device tokens). It must serve a **headless host** — `unpeel`
on a Linux server or an app-less Mac — identically; the TUI already serves
`/mobile` and pairing app-lessly, supervises this server, and can dial the
relay. Core Session routes are conformant; the remaining native/TUI gaps are
explicit platform-owned capabilities
(`docs/plans/host-controller-transports.md`). **Key constraint:** any output streamer must replay
history from `output.bin` on disk and only subscribe the live control socket
at the tail offset — subscribing far behind the host's in-memory broadcaster
kills the socket. Off-LAN access rides the Cloudflare relay (`apps/relay`,
`cd apps/relay && npm test`) with a forward-secret E2E handshake
(`RelayProtocol.swift`); the uplink needs a signed entitlement (shipped as
Pro, customer-facing direction: Link), and the target Link model also entitles
each Controller principal.
`RemoteControlManager.swift` supervises the server whenever paired devices
exist.

---
> Source: [unpeel-com/unpeel](https://github.com/unpeel-com/unpeel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
