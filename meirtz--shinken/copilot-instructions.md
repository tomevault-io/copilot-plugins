## shinken

> Shinken is an **AI-native, cross-platform sandbox runtime + control plane + control panel for

# CLAUDE.md — guide for AI coding sessions in this repo

Shinken is an **AI-native, cross-platform sandbox runtime + control plane + control panel for
computer-use agents** — the runtime that benchmarks and harnesses plug into. See [README.md](README.md) and
[`docs/`](docs/README.md). The design corpus is complete; the **implementation is a proven
Linux/X11 vertical slice** — handshake/auth, pointer+keyboard actions, pixel observation
(screenshot + real-time screencast + bandwidth levers + focused-window capture),
Docker disk-tier checkpoint/fork/resume + the privileged-only **CRIU memory tier**
(live process+memory forks) + `run_eval_forked`, a local capability-gateway shim,
the **guest structured-observation engine v1 (Linux/AT-SPI: `observe` with stable element ids,
tree-text diff, settle; guest-side `element_ref` actions + `invoke_action`/`set_value`)**,
and a Python SDK, all under live CI, plus a **macOS engine v1** (native CoreGraphics/CGEvent
backend in `shinkend`; capture+input built, local-only proof — no mac CI, AX designed-only;
[docs/engineering/macos-engine.md](docs/engineering/macos-engine.md)). **Production permission
enforcement, `.skn` recording/playback, the control plane, and the rest of cross-platform
(Windows, Wayland, macOS AX) are designed-only and not yet built**. The a11y-coverage spike (#2)
has been **measured (E5) — verdict: hybrid per-window structured + pixel fallback, so D3's
structured-default stays Provisional** (canvas is a measured zero with a change-blind diff,
Electron is measured on both CDP and forced AT-SPI; games/native-GL still unmeasured; the
in-guest CDP backend and UIA/AX tiers remain unbuilt).
**[`docs/engineering/status.md`](docs/engineering/status.md) is the authoritative built-vs-designed map — read it before
trusting present-tense claims in the vision docs. This file's status summary must track
status.md; reconcile both when either changes.**

## ⛔ The one hard rule: this is a PUBLIC open-source project

This is a **public, vendor-neutral OSS project** (regardless of where it sits in a local
checkout). Anything committed is world-readable.

- **NEVER** put confidential or company-internal material in tracked files (`docs/`, `notes/`,
  `README`, code). No internal platform names, no internal links, nothing marked confidential.
- Internal/private design references must stay out of tracked files. Do not link to private working
  areas or use them as public documentation sources.
- **Public** vendor product facts (e.g. NVENC, NICE DCV, vGPU/MIG, GPU-TEE) ARE fine in docs when
  cited from public sources — the project stays vendor-neutral and runs on any cloud.
- Do not run internal-only tooling (e.g. company intranet search) for this project.

## Layout

| Path | Tracked? | What |
|------|----------|------|
| `docs/` | ✅ | Authoritative docs: vision, PRD, architecture, OSWorld teardown, landscape, ADRs (D1–D15), roadmap, glossary, isolation & capability note, economics, Phase-0 plan, ACI spec, operation layer |
| `notes/` | ✅ | 9 working notes: per-domain deep dives, open questions, sources |
| `README.md`, `LICENSE` (Apache-2.0) | ✅ | front matter |
| `schema/` | ✅ | ACI JSON Schema (`aci.schema.json`) |
| `shinkend/` | ✅ | Rust Guest Runtime (`shinkend`) |
| `sdk/python/` | ✅ | Python SDK and CLI (incl. `shinken/integrations/` — swerex/uni-agent, CUA-Gym, Agentix, ProRL-Agent-Server, NeMo-Gym interop adapters; `shinken/backends/` operation-layer backends — drive the ACI over third-party computer-control systems, e.g. trycua/cua) |
| `images/linux/` | ✅ | Local Linux Sandbox image |
| `examples/` | ✅ | Runnable interop examples (`gym_rollout.py`, `cua_gym_shinken.py`, `agentix_shinken.py`, `uniagent_shinken.py` — scripted, no model API) |
| `benchmarks/` | ✅ | Rerunnable local benchmark suites (incl. `bench_client_scale.py` N=3096 client plane) + tracked raw results (`results/*.json`, one-off WAN CSVs in `results/remote/`); ALL figures land in `docs/assets/bench/` (regenerate: `replot.py` + `plot_remote.py`); methodology in `docs/engineering/benchmarks.md`, headline report in `docs/benchmarks/README.md`; plus the agent-quality STUDY harness (`bench_agent_quality.py` — codec tier × task success over fork-identical episodes, `docs/engineering/agent-quality-study.md`; not in `run_all.sh`, needs a model endpoint) |
| `references/` | 🚫 git-ignored | 13 cloned prior-art repos studied for design (OSWorld, cua, codex, anthropic-quickstarts, neko, OpenAdapt, e2b-desktop, UI-TARS-desktop, OmniParser; + 2026-06: uni-agent, CUA-Gym, Agentix, ProRL-Agent-Server); provenance + re-clone in `references/README.md` (tracked) |

## Conventions

- **The public design canon is `docs/design/tech-decisions.md`** — decisions are numbered **D1–D15**.
  When changing a design decision, update the relevant ADR and reconcile sibling docs to the same
  D-number.
- Naming (use consistently): **Shinken** (platform), **Sandbox** / **Session**, **Guest Runtime**
  (`shinkend`), **ACI** (Agent-Computer Interface), **Operator**, **Control Plane** / **Control
  Panel**, **Substrate/Provider**, **Capability** / **Capability Manager**, **`.skn`** (replay
  bundle), the control/event/media **planes**. See [docs/design/glossary.md](docs/design/glossary.md).
- Docs are **self-contained**: cite external sources by URL and sibling docs by relative path; do
  not cite private working filenames.
- Mark unverified vendor numbers `(vendor-published, unverified)`.

## Status & next steps

**Built & proven (Linux/X11):** M0 transport/auth + M1 act-and-observe are done — the **22-verb
ACI surface** (pointer+keyboard incl. `drag`/`mouse_down`/`mouse_up`, act-returns-observation via
the per-action `observe` argument, the `observe`/`invoke_action`/`set_value` structured family,
the G2+G3 desktop verbs — `clipboard_get`/`clipboard_set` via native X11 selections (no xclip),
`launch_app`, `activate_window` (EWMH + WM-less fallback) — `list_windows`, and the **typed
in-guest `exec` channel** (G1) — argv default + `shell` opt-in, buffered typed result or streamed
`exec_output`/`exec_exit`, process-group-killed timeouts, gateway-audited with argv/shell in the
decision event; the swerex/CUA-Gym/ProRL integrations prefer it over `docker exec` when
advertised, `pty` reserved as the designed follow-up),
screenshot, real-time screencast (server-push) with idle-suppression + downscale,
focused-window/`window:<id>` capture, **binary WS media frames** (negotiated; wire −25%,
client-plane ceiling ~2.1×), and **XDamage event-driven capture** (idle ticks capture nothing —
~0 guest CPU; damaged ticks fetch only the damaged region), all with live Xvfb/Docker CI smokes.
String-form **XML tool calls** parse as first-class action input (`shinken.dialect.parse_actions`,
`format="auto"|"xml"|"dialect"`). **Docker disk-tier
checkpoint/fork/resume** (`docker commit`, #209) behind the provider interface, the **CRIU
memory tier** (S4c, opt-in `CriuDockerProvider`, `snapshot_kind="process"`, privileged-only:
`criu dump --leave-running` + commit, restore = live process+memory replica in ~0.4 s,
in-heap-marker-verified — a latency/state-fidelity tier, not an isolation posture), plus
`eval.run_eval_forked` (golden→fork-N→score, #231) and the **fork-native gym adapter**
(`shinken.gym`: trainer-facing `make/reset/step/evaluate` with reset()=fork, pool parallel
reset, HF-datasets exporter, MultiTurnDataloader-shaped iterator), are built; a **local
capability-gateway shim** (`sdk/python/src/shinken/gateway.py` + tests) is built. **Push-based boot readiness (S9)** is
built: the guest-side `ready` query + lazy/self-healing X11 connect in `shinkend` + a 15 ms
single-connection SDK readiness loop took `provider.create()` from ~7.7 s to ~0.19 s p50, and the
opt-in **warm-pool fork graft** (pre-booted containers + `docker diff` delta) serves
restore/fork without the boot (files-only, same tier as `docker commit`) — see
[docs/engineering/benchmarks.md](docs/engineering/benchmarks.md) §1/§9. The Python SDK
(sync facade + reader/demux, plus a pipelined `step()` — k actions + a fused observation in
~1 RTT, S11-measured) ships too. The **operation-layer backend contract (D15,
`shinken.backends`) is built**: `cua` / `mcp-computer` (codex-style AX MCP) / `browser-runtime`
(CDP) / `e2b` adapters + `RoutedSession` CU↔BU composition with `source` provenance — honest
capability negotiation, fixture-tested with protocol-faithful in-memory peers + env-gated
live smokes against the real drivers (`tests/test_backends_live.py`,
`SHINKEN_{BROWSER,E2B,CUA,MCP}_LIVE=1` — browser proven locally against real headless
Chrome; e2b/cua/mcp written but unrun; none in CI). `.skn` recording is **not** built (removed/deferred, #216/#217). Full built-vs-designed
map: **[docs/engineering/status.md](docs/engineering/status.md)**.

The immediate work (per the recalibrated priorities):
1. **#56 hardening is DONE** — the schema alignment (screencast verbs, `scope`/`fps`/`max_long_edge`,
   `stream`/`seq`, `hello.token`), the frame-queue bounds, the vacuous-test fixes, the **error
   taxonomy** (`sandbox_died` with exit/signal detail, typed per-action status), the
   **screencast reconnect semantics** (`resume_stream`: stream identity + seq continuity), the
   **trajectory-level `exit_reason`** (documented precedence, `shinken/runtime/trajectory.py`),
   and **subprocess scorer isolation** (T-5, `shinken/scorer_proc.py`) are all built — see
   [docs/engineering/v0.0.1-plan.md](docs/engineering/v0.0.1-plan.md) §6.
2. **a11y-coverage spike (#2) — MEASURED (E5), verdict in**: strong Qt/AT-SPI (0.87),
   Chromium-family controls via CDP (1.00 of labeled controls; 0.23 of all nodes — browser *and*
   Electron; Electron also hits 0.32 over forced AT-SPI), weak GTK, absent terminals, **canvas
   measured at zero** (5 drawn controls → 2 inert AX nodes; a real click changes pixels but the
   tree diff reports nothing); tree-diff ~2 KiB vs ~77 KiB screenshot. **D3 stays Provisional —
   hybrid, not structured-by-default.** The **guest observation engine v1 is now built for
   Linux/AT-SPI** (live-smoked: zenity id-stability, diff-after-typing, element click by id,
   invoke_action); remaining: games/native-GL coverage, the in-guest CDP backend, UIA/AX tiers;
   the screenshot baseline still carries v0.0.1 usability.
3. **Designed-only, not started:** the rest of the **operation layer** (D13,
   `docs/design/operation-layer.md`: `set_text_selection`/`scroll_element`, app/window scoping
   beyond `list_windows`, the in-guest CDP backend, the Browser Runtime); the Capability Manager
   (production enforcement beyond the local gateway
   shim), `.skn` recording/playback, the sub-ms CoW fork fast tier
   (the CRIU **memory tier is now BUILT** — `CriuDockerProvider`, productized from the positive
   `spikes/criu-memory-tier/` spike; only the CoW/microVM fast tier remains designed),
   control plane + concurrency, dual-channel WebRTC/NVENC, and the rest of cross-platform —
   **Windows** + **Wayland** + macOS AX/ScreenCaptureKit (the **macOS engine v1**
   capture+input slice IS built, local-only: `shinkend --backend macos`,
   [docs/engineering/macos-engine.md](docs/engineering/macos-engine.md)).
4. **CoW-fork density** and **dual-channel WebRTC latency** remain Phase-1 boundary spikes (D1/D4).

The **2026-06 recalibration change inventory** (positioning / architecture / functionality / contract /
testing / docs — what changed, why, status, and the still-open list) is
[docs/engineering/recalibration-2026-06.md](docs/engineering/recalibration-2026-06.md).

Open questions and risks: [notes/open-questions.md](notes/open-questions.md).

---
> Source: [Meirtz/Shinken](https://github.com/Meirtz/Shinken) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
