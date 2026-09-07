## jani

> Jani is a native macOS operator console for a fleet of local git repos — sync,

# Jani — CLAUDE.md

Jani is a native macOS operator console for a fleet of local git repos — sync,
supervise (dev processes/stacks), and map (static dependency index). Personal
project. Tauri 2 (Rust) + React 19. Dark-only. No AI inside the app.

## Source of truth

- **Spec (behavior):** `docs/specs/jani-v3.md` — read before any work. All scope
  lines and policies live there; if a request conflicts with it, raise the
  conflict instead of silently deviating.
- **Mockup (look):** `docs/research/jani-v3-mockup.html` — open in a browser.
  Where prose and mockup disagree: mockup wins for look, spec wins for behavior.
- **Status:** v3 is DONE (v3.4.0 shipped 2026-07-22; milestones M0–M4 complete).
  Work now comes from `docs/BACKLOG.md` and spec §13 — features only on
  demonstrated need. Interpretive calls settled during the build: `docs/decisions.md`.

## Hard rules (from spec §2 — do not relitigate)

- Never `git push`, stash, merge, or rebase. Never touch a dirty working tree.
- Nothing runs on app startup; no timers except the opt-in background fetch.
- Jani only owns processes it started. No adoption of external processes.
- No new npm dependencies without explicit approval. Rust deps only as spec §3 lists.
- **User commands** (tasks, services, requires checks, fixes — §4.2) run via
  `$SHELL -lc` with the repo as cwd. Internal git is the opposite: spec §3
  says shell out to the `git` CLI directly, *not* through `$SHELL`.

## Architecture invariant

`core/` (jani-core lib crate) holds ALL logic; the Tauri layer stays thin enough
that a future CLI binary could wrap jani-core without changes. If a Tauri command
contains logic beyond translation, it's in the wrong layer.

## Workflow (working agreement — spec §12, as practiced)

John holds three checkpoints personally: plan approval before any code, the
feel-check, and the merge call. Never merge unprompted.

- **Branch model:** topic branches only — no `dev` branch, no PRs. Merge to
  `main` on John's call. `main` = exactly what's installed and running
  (`/Applications/jani.app`); after a merge, rebuild + reinstall so that stays true.
- TDD (test first, always); small commits, conventional messages. Gate:
  `pnpm check` (cargo test + vitest + tsc) green at every commit. **Verify the
  gate by exit code (`pnpm check && echo GREEN`), never by grepping output** —
  a grepped "ok" once hid a red vitest run.
- Remote: github.com/parkingsoju/jani via SSH (v2 history on branch `v2`).
  Never force-push — John runs those himself.
- **Never switch branches or merge while `pnpm tauri dev` is running.** The
  watcher rebuilds and restarts the backend mid-session, killing in-flight IPC
  (surfaces as spurious red failures). Stop the dev app first, or warn John
  the app will restart under him.

## UI grammar (spec §8.5 is binding; short form)

Quiet console: health is darkness, problems glow. One status silhouette
(lozenge-in-socket); breathing = live process. Bordered button = acts now
(beacon = mutates, ghost = reads); quiet text + `▾` = picker. Status hues never
on buttons. Labels opening more UI end with `…`. Disabled controls always show
their reason. Plain-language copy.

---
> Source: [parkingsoju/jani](https://github.com/parkingsoju/jani) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
