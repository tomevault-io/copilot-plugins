## whiteproxy

> Operating manual for Claude Code in this repo. Strict architecture,

# CLAUDE.md — White Proxy

Operating manual for Claude Code in this repo. Strict architecture,
loose implementation. Read once per session, then ship.

White proxy is a **pure-Go**, cross-platform (Linux, macOS, Windows)
proxy + scanner + DPI-desync suite for circumventing TLS-based
censorship. The previous Python+Go hybrid has been retired — there is
no `cores/`, `utils/`, `api/`, or `main.py` anymore. If your training
data remembers them, ignore it. The current source of truth is the
tree under `cmd/`, `internal/`, and `pkg/`.

## Project shape (orient quickly)

- `cmd/whiteproxy/` — the **single bundled binary**. Dispatches to one
  of the inner subcommands based on `os.Args[1]`. With no args, opens
  the TUI (auto-spawning the daemon if no socket answers).
  Subcommands:
    - `daemon`         — long-lived daemon (RPC over AF_UNIX)
    - `tui`            — interactive bubbletea TUI (default)
    - `ctl`            — kubectl-style client (status, down, config,
                        events, route, scan, sub, relay subcommands)
    - `scan`           — proxy/SOCKS5/HTTP discovery scanner
    - `whitescan`      — T14 white-IP TLS-admit scanner
    - `snicheck`       — SNI accessibility probe (CLI)
    - `snicheck-tui`   — SNI probe (TUI front-end)
    - `routeverify`    — route-verify daemon (long-running, stdin/stdout)
    - `relay`          — SNI-desync TCP relay (Linux only)
- `cmd/<other>/` — historical entry points still wired through the
  dispatcher in `cmd/whiteproxy/`. `whiteproxyd`, `whitep`, `relay`,
  `scan`, `whitescan`, `sni_check`, `sni_check_app`, `routeverify`
  are all packages exposing a `Run()` rather than `main()` — the
  bundled binary calls them by name. Don't add new `package main`
  entry points; extend the dispatcher table instead.
- `internal/daemon/` — daemon glue: RPC server, job orchestration,
  proxy/relay/scan/sub method handlers. Compiler-enforced no external
  import (Go enforces `internal/` visibility).
- `internal/tui/` — bubbletea TUI. Tabs are Scan / Monitor / Routing
  / Sub / Inspect / Settings (white mode) plus Control / Test / Relay
  (desync mode, Linux only).
- `pkg/` — importable libraries. The package boundary IS the API.
    - `paths`         — XDG-style per-user dirs (Config / Cache /
                        Data); `paths.Ensure()` mkdirs them.
    - `config`        — JSON config snapshot + reload.
    - `ipc`           — JSON-RPC 2.0 over AF_UNIX (Linux/macOS/
                        Windows10+); streaming responses.
    - `eventbus`      — typed, in-process pub/sub used inside the
                        daemon and to fan out to RPC subscribers.
    - `jobs`          — job lifecycle (state, events, cancel).
    - `pause`         — cooperative pause/resume `Gate` with
                        Begin/End/Inflight counters.
    - `asn`           — embedded Iran + Global ASN databases
                        (`assets/filtered_ipv4.csv` and
                        `assets/ipv4.csv.xz`). Sorted-prefix index
                        for O(log N + small) lookups; warmable.
    - `classify`      — verdict classifier (HasBrowserBlock,
                        HardReject patterns, etc.).
    - `route_engine`  — routing + scan pool + bans + cache.
    - `proxy`         — HTTP CONNECT + SOCKS5 listener; routes via
                        route_engine; MMDF MITM domain-fronting.
    - `mmdf`          — MITM domain-fronting (CA install +
                        per-profile config).
    - `desync`        — TLS-desync (Linux only via `//go:build
                        linux`). macOS/Windows builds compile it out.
    - `sub`           — sub-server: in-process HTTP UI (port 7081)
                        + autoload of latest scan results.
    - `scan/<X>`      — scanner pipelines, all in-process library
                        calls (NOT subprocess + JSONL anymore):
        - `common`    — shared probe/Config types.
        - `targets`   — target expansion (CIDR, file, paste, ASN
                        scopes, streaming).
        - `cache`     — alive/dead IP caches (byte-compatible
                        with the legacy on-disk JSON shape).
        - `white`     — T14 white-IP TLS-admit scanner.
        - `proxy`     — HTTP/SOCKS5 proxy discovery scanner.
        - `sni`       — SNI accessibility pipeline (resolvers,
                        sinkholes, profiles, fingerprints).
        - `route`     — route-verify probes.
        - `speed`     — concurrency / wave profiles.

- `data/`, `cyclic_archives/` — runtime state (created lazily). The
  daemon defaults to `paths.DataDir()` (e.g.
  `~/.local/share/whiteproxy`) when no `--proxy-scan-dir` /
  `--proxy-log-dir` is passed, so the binary is truly standalone.
- `research/` — Obsidian-style experiment vault. **Index-first**
  navigation: `research/CHECKLIST.md` is the Map-of-Content; every
  experiment row has a `[[wiki-link]]` + one-sentence conclusion.
  See §5 below.

---

## 1. The Sandbox (Hard Rules)

These are non-negotiable. Breaking one is a regression even if tests
pass.

1. **Daemon owns long-lived state.** The proxy listener, route
   engine, MMDF CA, MMDF registry, scan job manager, relay process,
   sub server, and config snapshot all live in the daemon. The TUI
   and `ctl` CLI are thin clients — they hold zero authoritative
   state and **must** round-trip every action through the daemon
   over AF_UNIX RPC.
2. **All IPC is JSON-RPC 2.0 over AF_UNIX.** Linux/macOS native;
   Windows 10 1803+ AF_UNIX. No named pipes, no `go-winio`.
   Streaming responses are supported (`events.subscribe`,
   `scan.tail` patterns). When you add a method, register it in
   `internal/daemon/<area>_methods.go` and consume it from the
   client side in `internal/tui/` or `cmd/whitep/`.
3. **Package boundaries are the API.** `internal/` is enforced
   private by the Go toolchain. `pkg/<x>` is the only stable surface
   for cross-package use; don't reach into another package's
   internals.
4. **Scanners are libraries, not subprocesses.** The daemon imports
   `pkg/scan/*` directly. There is no stdin/JSONL/stderr wire
   protocol anymore — that was the Python-bridge contract and it's
   gone. New scanners go in `pkg/scan/<name>/` and ship as
   in-process pipelines. `cmd/scan*` binaries stay as thin wrappers
   for ad-hoc shell use.
5. **Single binary stays single.** `cmd/whiteproxy/main.go`
   dispatches to every entry point. New CLI surface = a new entry in
   the `subcommands` table, not a new `main()` package. Other
   `cmd/<x>/` trees are packages exposing `Run()`.
6. **Embed, don't ship beside.** The SNI default config, both ASN
   databases (Iran filtered + global xz-compressed), and the
   sub-server HTML template are baked in via `go:embed`. No part of
   the binary may require a sibling `data/`, `assets/`, or config
   file in the working directory. When adding new static assets, put
   them under the package that owns them and embed.
7. **Cancellation is sacred.** Every long-running goroutine must
   honour its `context.Context` and stop child subprocesses (relay,
   masscan, nmap) on cancel. RPC handlers receive a `ctx` — pass it
   through.
8. **No telemetry. No phone-home.** This tool runs in hostile-network
   contexts. Don't add crash reporters, version pings, or anything
   that leaks targets/IPs off-host.
9. **Subprocess hygiene.** Use `exec.CommandContext`; `cmd.Cancel`
   should always tear down children. On the auto-spawn path for the
   daemon (see `cmd/whiteproxy/main.go`) the child is intentionally
   detached (Setsid on Unix, `DETACHED_PROCESS`+`HideWindow` on
   Windows) so it survives the TUI exiting.
10. **DPI desync cross-platform: Linux + Windows live, Darwin
    runtime-gated.** `pkg/desync/` builds on Linux (`AF_PACKET` +
    raw `IPPROTO_TCP`), Windows (WinDivert kernel driver, requires
    Administrator; bundled in `pkg/desync/windivert/`), and Darwin
    (BPF + BSD raw socket), with `stub_other.go` returning
    `ErrNotSupported` everywhere else. On Darwin the build compiles
    but `Start`/`Stop`/`SelfTest` return `ErrNotSupported` until the
    Network Extension entitlement work + tcpdump-vs-offload behaviour
    is validated end-to-end. Removing the Darwin runtime gate
    requires (a) a code-signing / entitlement shipping plan and
    (b) an echidna-class macOS test vantage. See
    `research/dev/DESYNC.md` and `research/dev/DECISIONS.md` D6.
11. **Iran-path probes ONLY via wlo1; the default route is a VPN and
    it's the WRONG path.** `jibril` has two routes: default = VPN
    (wrong); `wlo1` = residential Iran wifi (right, `loc=IR`). The
    day-to-day vantage is the **`iran-netns`** namespace
    (`research/dev/iran-netns/`, user-aliased as `ir-run`): a veth
    + NAT + policy route that pins the netns to wlo1, leaving the
    VPN connected on the host. Inside, probes don't need
    `-bind-iface` or `cap_net_raw`. Fallback if the netns is
    unavailable: `-bind-iface wlo1` on the bare host (root, or
    `setcap cap_net_raw+ep`). Preflight: `hcaptcha.com` (or
    `-preflight-ip <CF IP>`); `1.1.1.1` is Iran-blocked.

    Claude CAN run SHORT smoke probes from this shell — but only
    via `ir-run` so they go through wlo1. "Short" = a single
    targeted check, <60 s wall, ≤~50 probes; the kind you'd run
    to confirm a single hypothesis (does this IP land at that
    colo? does this SNI handshake?). Sweeps, multi-IP
    ground-truth, full /24 or ASN scans → hand off to the user,
    who runs the same binary with `ir-run`. Default loop: write
    the probe → user `ir-run`s the long one → analyse together.

    `gh` / `git push` / other non-research utilities can use the
    default VPN route — they don't care about path correctness.
    `echidna` (192.168.0.123) is deprecated; iran-netns on jibril
    replaces it. Keep echidna only as a wlo1 fallback. Even a 2-IP
    smoke test on the bare shell (VPN path) corrupts the tuning
    that follows — so EVERY probe goes through `ir-run`, no
    exceptions.

---

## 2. Engineering Autonomy (The Creative Brief)

When given a general feature request, **you are the Lead Engineer**.
You own the technical solution end-to-end. If a path is ambiguous,
pick the most performant and idiomatic option for *this* codebase
rather than asking for permission. The user prefers a working v1
they can redirect over a perfect plan they have to read.

**Proposing vs. doing.**

- **Trivial / localized change** (bugfix, helper, single-file edit):
  do it. No plan.
- **Non-breaking new feature** (new scanner, new RPC method, new TUI
  panel, new CLI subcommand): post a 3-bullet architecture sketch —
  *where it lives, what it talks to, what it returns* — then ship
  the code in the same turn. Don't wait for a "yes."
- **Breaking change** (touches the RPC wire shape, on-disk file
  shapes in `data/`, the dispatcher table, or removes a public
  `pkg/<x>` symbol): stop. Surface the breakage and the migration
  cost first. Wait for explicit approval.

**Default playbook for a new networked feature:**

1. Drop the library code under `pkg/<area>/<name>/` as an in-process
   pipeline. Mirror the shape of an existing pipeline (the
   `pkg/scan/sni` package is a good template for state-heavy
   pipelines; `pkg/scan/white` for stateless ones).
2. Register a daemon RPC handler in
   `internal/daemon/<area>_methods.go`; emit events via `EventBus`
   if streaming is needed.
3. Wire the TUI client in `internal/tui/<tab>.go` to call the new
   RPC and render the result.
4. Stop. Tests come when asked.

---

## 3. Ambiguity Protocol

When a prompt is too general, optimize for, in this order:

1. **Maximum throughput, minimum latency.** Bound concurrency
   explicitly (`pkg/scan/speed` has the canonical profile shape);
   avoid unbounded `go ...`; prefer worker pools.
2. **Zero bloat.** No new dependencies. No new abstraction layers
   "in case." No config flags for one caller.
3. **Idiomatic with the existing code.** Match patterns already in
   the repo:
    - context-propagating handlers, not background `go func()`
    - `paths.DataDir()` / `paths.ConfigDir()` for filesystem state,
      never `cwd`-relative paths
    - `eventbus.Publish` for cross-component notifications
    - `pause.Gate` for cooperative pause/resume in long loops
    - `jobs.Job` for any user-visible long-running operation

If you're still ambiguous after the above, pick the smaller, more
reversible option and ship it.

---

## 4. Operational Efficiency (Token Savers)

- Be concise. No apologies, no preambles, no recap of what the user
  just said.
- Show diffs and final code, not narration. Let the code speak.
- Don't restate the file you just edited — the diff is visible.
- One-sentence end-of-turn summary, max two. What changed, what's
  next.
- No emojis. No multi-paragraph docstrings. One-line comments only
  when the *why* is non-obvious.
- Don't read a file twice in one session unless it changed. Don't
  re-grep what you already grepped.
- Parallelize independent tool calls in a single message.
- Skip `git status` / `git diff` chatter unless committing.
- **Always commit your changes.** Standing user instruction: when a
  logical unit of work is complete, commit it — clear message,
  related edits grouped into one commit. Never hand back a dirty
  working tree. Commit directly to the working branch (this repo
  keeps a linear `main`); don't open branches or PRs unless asked.

---

## 5. The Research Vault (`research/`)

`research/` is an Obsidian-style knowledge base. Navigate it *as a
vault* — index first, pointers second. This is the token-saver:
never raw-read folders or scroll the big docs end-to-end to find a
fact.

**Reading — start with the top-level navigation docs, in this order.**

The cross-cutting docs at the root of `research/` are the fastest
path to knowing what is true:

1. **`research/CHECKLIST.md`** — the Map of Content. Index tables
   for the **E** (regime-pivot), **R** (router-design), and **S**
   (scanner) series. Each row has a `[[wiki-link]]` + one-sentence
   conclusion + status flag (`[x]` / `[x?]` / `[?]` / `[ ]` /
   `[-]`). Open this **first**.
2. **`research/NETWORK_STATE.md`** — what works today on the Iran line.
   Rewritten in place each time the regime visibly shifts; loadable
   in 60 seconds. Open this **second** to know whether the regime
   has moved since your last session.
3. **`research/INVARIANTS.md`** — the architectural facts that
   survive regime changes (TTL preserved, (IP,SNI) match key,
   RST-injection mechanism, CGNAT shape, etc.). Pull these instead
   of re-reading the E-series.
4. **`research/PRIMITIVES.md`** — operational primitive catalog
   (bare-IP, `bad_csum + cover-SNI`, `badcsum_ttl6`, ECH wire shape,
   per-fleet cover SNI rule). Each entry has `last_validated:`,
   `depends_on:`, `status:` so you know what to trust.
5. **`research/PRIOR_REGIME.md`** — frozen Internet Pro snapshot
   (2026-04-14 → 2026-05-2X). Compare current measurements against
   this baseline.
6. **`research/dev/DECISIONS.md`** — ADR log. Why the code is shaped
   this way. Read before re-litigating any sandbox-rule decision.
7. **`research/dev/DESYNC.md`** + **`research/dev/MMDF.md`** —
   non-obvious internals for `pkg/desync` and `pkg/mmdf` (mechanism,
   per-platform backends, profile + CA flow). The code is
   authoritative; these explain the *why* and the cross-platform
   surprises.

`research/REGIME_PIVOT.md` is the **long-form historical
threat-model derivation** (Internet Pro era). Useful when you need
source citations or the section-by-section reasoning; do not read
it for current operations — the top of the file points back to
NETWORK_STATE.

**Conventions.**

- Three namespaces, never conflated: **`E<n>`** = regime-pivot
  censorship experiments (folders `e<n>_topic/`); **`R<n>`** =
  router-design experiments; **`S<n>`** = scanner-tooling
  experiments. `E2` ≠ `S2` ≠ `R2`.
- Per-experiment findings live as `experiment_findings.md` or
  `RESULTS.md` **inside** the experiment folder. The CHECKLIST is
  pure index — don't inline result transcripts into it.
- `FINDINGS_part1-4_*.md` are the atomically-split scanner-research
  monolith; `FINDINGS.md.bak` is a **frozen archive** — don't read,
  edit, or link it.
- `research/archive/` holds frozen older docs; the readme in each
  subfolder explains what's there and why it's archived.
- Every note carries YAML frontmatter (`tags`, `status`); read the
  frontmatter to judge relevance before opening the body.
- Follow `[[wiki-links]]` to pinpoint a note — cheaper than
  re-reading the big docs front to back.

**Writing — extend the vault, don't pile new files beside it.**

- New experiment results go **inside that experiment's folder** as
  `experiment_findings.md` (or append to existing `RESULTS.md`) —
  not a new top-level file.
- A genuinely new standalone note gets YAML frontmatter (`tags:`,
  `status:`) and lives under `research/`. New experiments take the
  next free `E<n>` / `R<n>` / `S<n>` — never reuse a number.
- Cross-link with `[[wiki-links]]` that point at real directories
  or notes. Never invent a target; if you mention an experiment,
  link it.
- After finishing an experiment, **update the CHECKLIST MOC**: add
  its row, wiki-link, status flag, and one-sentence conclusion.
  Then update `NETWORK_STATE.md` if the result changed what we
  believe works today, `INVARIANTS.md` if it crossed two
  independent E-rows and is regime-stable, and `PRIMITIVES.md` if
  it shifted an operational primitive's status.
- Never recreate a monolith. No new giant `FINDINGS`-style dumps.
- Design decisions on the codebase go in `research/dev/DECISIONS.md`
  as a new `D<n>` entry (append-only). Component-internals docs
  (like `DESYNC.md` / `MMDF.md`) only for components whose
  *mechanism* crosses network ↔ code; everything else stays in
  package-level `doc.go` comments next to the code.

---

## Quick reference

- **Build the unified binary:** `make` (drops `build/whiteproxy`)
- **Cross-build:** `make whiteproxy-linux` / `-darwin` / `-windows`,
  or `make whiteproxy-all-os`
- **Run the TUI (auto-spawns daemon if needed):**
  `./build/whiteproxy`
- **Run the daemon alone:** `./build/whiteproxy daemon`
  (defaults to `--proxy-listen=127.0.0.1:7080`; pass
  `--proxy-listen=""` to disable the listener)
- **kubectl-style CLI:** `./build/whiteproxy ctl <cmd>`
  (status, down, events, scan.*, route.*, sub.*, relay.*, config.*)
- **Subcommand list:** `./build/whiteproxy help`
- **Tests / vet / fmt / tidy:** `make wp-test` / `wp-vet` /
  `wp-fmt` / `wp-tidy`
- **Per-user dirs:**
    - Config: `paths.ConfigDir()` (Linux: `~/.config/whiteproxy`)
    - Cache:  `paths.CacheDir()` (Linux: `~/.cache/whiteproxy`) —
              AF_UNIX socket + pidfile live here
    - Data:   `paths.DataDir()` (Linux: `~/.local/share/whiteproxy`)
              — scan results, caches, parked-IP files, logs
- **Network research vantages (rule 11):** `jibril` is the host. Its
  default route is a **VPN** → wrong path, never use. Its `wlo1`
  interface is the **residential Iran wifi** → the right path
  (`loc=IR`). Day-to-day: run probes inside the **`iran-netns`**
  namespace (`research/dev/iran-netns/`), aliased by the user as
  `ir-run <cmd>` — no `-bind-iface` needed, no `setcap`, no DNS
  leak, VPN stays up on the host. Fallback: `-bind-iface wlo1`
  with `setcap cap_net_raw+ep` on the bare host. Preflight dials
  `hcaptcha.com` (or pass `-preflight-ip <CF IP>`); `1.1.1.1` is
  Iran-blocked. `echidna` (192.168.0.123) is **deprecated** —
  jibril+wlo1 replaces it end-to-end; keep echidna only as a
  wlo1 fallback.
- **Research vault:** `research/` is an Obsidian vault — see §5.
  `research/CHECKLIST.md` is the Map-of-Content index (every `E<n>`
  / `S<n>` experiment → one-line conclusion + `[[wiki-link]]`);
  `research/REGIME_PIVOT.md` holds the threat model and full specs.
  Index-first navigation, always.

---
> Source: [sahroush/WhiteProxy](https://github.com/sahroush/WhiteProxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
