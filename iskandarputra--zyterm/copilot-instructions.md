## zyterm

> This is a **map**, not a rulebook. It tells you what zyterm is, the facts you cannot change, the

# CLAUDE.md — orientation for AI assistants and new developers

This is a **map**, not a rulebook. It tells you what zyterm is, the facts you cannot change, the
one rule that protects the codebase, and where to find everything else. The binding details live
in [`docs/`](docs/) — this file points at them.

## What zyterm is

zyterm is a single-binary Linux serial-terminal emulator written in C (~12.5K LOC, version
**1.5.0**). It opens a serial port (or a `tcp://` / `telnet://` transport), renders RX in a small
TUI with a HUD, and supports framing/CRC decoders, XMODEM/YMODEM/ZMODEM transfers, autobaud,
auto-reconnect with port discovery, logging (text/json/raw with rotation), scrollback with regex
search and bookmarks, profiles, event hooks, detach/attach sessions, an HTTP/SSE/WS bridge with
Prometheus metrics, and an embedding API. Several historically advertised features are **dead,
stubbed, or broken** — see "The one rule" and [`docs/tracking/KNOWN_ISSUES.md`](docs/tracking/KNOWN_ISSUES.md)
before you describe any feature as working.

## Non-negotiable facts

These are structural. Do not break them, and do not write docs that contradict them.

- **Linux target.** zyterm targets Linux (termios2/`BOTHER` custom baud, inotify, epoll-era APIs).
  See [`docs/decisions/0001-linux-first-single-binary.md`](docs/decisions/0001-linux-first-single-binary.md).
- **Single binary, libc-only runtime.** Ships as `./zyterm` plus `build/zyterm_embed.a`. The only
  runtime dependency is libc. X11 clipboard is loaded at runtime via `dlopen("libxcb.so.1")` — it
  is **not** a build dependency. (Release pins `strtol`/`strtoul` to a base glibc symbol via
  `.symver` in `src/zt_internal.h`; skipped under sanitizers.)
- **Plain Makefile.** A hand-written Makefile auto-discovers `src/**/*.c`. No CMake, no autotools.
  `make` (release -O3) · `make debug` · `make test` · `make modules`.
- **One `zt_ctx` + a poll loop.** All per-process state lives in a single `zt_ctx`
  (`src/zt_ctx.h`); the runtime is a `poll(2)` event loop. An **optional** `--threaded` reader
  (`src/loop/rx_thread.c`) owns only a private `dup()` of the serial fd and a 1 MiB lock-free SPSC
  ring — everything else is main-thread-only.
- **Enforced module dependency chain.** Each `include/zyterm/internal/<m>.h` includes exactly the
  one beneath it, so the layering is compiler-enforced:
  `core ← serial ← log ← proto ← render ← tui ← net ← ext ← loop`. `main.c` is the only translation
  unit allowed into `loop/`; the umbrella `src/zt_internal.h` includes only `loop.h`. Verify in
  [`docs/invariants/INVARIANTS.md`](docs/invariants/INVARIANTS.md) §8.
- **Version source of truth.** `ZT_VERSION` is `"1.5.0"` at `src/zt_ctx.h:71`. Release tagging
  checks `tag == ZT_VERSION`; never hard-code the version elsewhere.

## The one rule

**Before you touch a subsystem, read the matching section of
[`docs/invariants/INVARIANTS.md`](docs/invariants/INVARIANTS.md) (`INVARIANTS §N`).** Those nine
sections are contracts, not suggestions. Source comments may cite a section by name (e.g. "must
stay async-signal-safe — see Signals invariant"); treat that citation as binding. If a change would
break an invariant, the change is wrong — or you need a new ADR in
[`docs/decisions/`](docs/decisions/) to supersede the old decision first.

## Where to find things

| You want… | Go to |
|---|---|
| What zyterm is / how to run it | [`README.md`](README.md) |
| Build, test, change workflow | [`CONTRIBUTING.md`](CONTRIBUTING.md) |
| Module map + dependency chain | [`docs/reference/ARCHITECTURE.md`](docs/reference/ARCHITECTURE.md) |
| `zt_ctx`, poll loop, `--threaded` reader, signals | [`docs/reference/INTERNALS.md`](docs/reference/INTERNALS.md) |
| The rules you must not break | [`docs/invariants/INVARIANTS.md`](docs/invariants/INVARIANTS.md) |
| Why a choice was made | [`docs/decisions/`](docs/decisions/) |
| Every CLI flag | [`docs/reference/CLI.md`](docs/reference/CLI.md) |
| Every keybinding | [`docs/reference/KEYBINDINGS.md`](docs/reference/KEYBINDINGS.md) |
| The embedding API (7 symbols) | [`docs/reference/EMBEDDING.md`](docs/reference/EMBEDDING.md) |
| Open defects (`ZT-001 … ZT-028`) | [`docs/tracking/KNOWN_ISSUES.md`](docs/tracking/KNOWN_ISSUES.md) |
| Non-defect work in flight | [`docs/tracking/STATUS.md`](docs/tracking/STATUS.md) |
| Release mechanics | [`docs/ops/RELEASE.md`](docs/ops/RELEASE.md) |
| User how-to | [`docs/guide/`](docs/guide/) |
| Security posture | [`SECURITY.md`](SECURITY.md) |
| The docs map itself | [`docs/README.md`](docs/README.md) |

## Managing `docs/`

The docs tree is organized **by kind, not topic** (see [`docs/README.md`](docs/README.md)). Keep
it that way:

- **Route, don't dump.** New writing belongs in an existing kind: bug → `tracking/KNOWN_ISSUES.md`;
  non-defect work → `tracking/STATUS.md`; why → `decisions/`; rule → `invariants/`; how-it-works →
  `reference/`; user how-to → `guide/`; release → `ops/`; future work → `plans/`. Do not create a
  new top-level `.md` for something that is one of these in disguise.
- **Current truth only.** Never document a dead/stubbed/broken feature as working. If it doesn't
  work, it belongs in `tracking/KNOWN_ISSUES.md` or `plans/` with an accurate status. Every code
  claim must be checkable in `src/` — cite `file.c:line` when it helps.
- **Sync in the same change.** When code changes behavior, update the affected `reference/` doc,
  invariant, or board *in the same commit*. Docs that drift from `src/` are the bug we are
  avoiding.
- **Prune as work completes.** When a STATUS item ships, move its truth into `reference/` and drop
  the board row. When a defect is fixed, close its `ZT-NNN` row. Re-stamp boards with
  `_Last updated:_`. ADRs are the exception — they are append-only; supersede, never delete.

---
> Source: [iskandarputra/zyterm](https://github.com/iskandarputra/zyterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
