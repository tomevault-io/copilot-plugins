## yes-master

> This is the active YES Master repo. Do not treat old handoff files or prior

# YES Master Agent Instructions

This is the active YES Master repo. Do not treat old handoff files or prior
phase plans as active spec. Use the current code plus the docs listed below.

> **AGENTS.md and CLAUDE.md are kept byte-identical — edit both together.**
> Live work queue + owner decisions: `docs/OPEN_THREADS_AND_DECISIONS.md`.
> Shipped history: `docs/CHANGELOG.md`. Idea backlog: `docs/IDEAS_BACKLOG.md`.
> Retired handoffs/reviews/plans: `docs/archive/`.

## Required Reading

1. `docs/PRODUCT.md`
2. `docs/APP_BEHAVIOR.md`
3. `docs/ARCHITECTURE.md`
4. `docs/TESTING.md`
5. `docs/RELEASE_STABILIZATION.md`
6. `docs/plans/2026-06-30-launch-plan.md` (GTM / pricing strategy)
7. `docs/plans/2026-07-24-001-feat-public-beta-quality-plan.md` — the active
   forward queue (chunks C1–C5 + a chunk status ledger; resume from the ledger)
8. `docs/plans/beta-go-no-go.md` — the live release gate + exact-commit
   evidence ledger
9. `docs/OWNER_INPUT_QUEUE.md` — questions waiting on the owner. Read it before
   concluding a decision was never made, and add to it instead of guessing.
10. `docs/CAPABILITY_EVIDENCE_MATRIX.md` — every public claim bound to its
    evidence source

### Which beta document is authoritative

Three documents describe the beta and they do **not** rank equally. Stated here
so no future session has to guess:

- `docs/plans/2026-07-07-beta-execution-plan.md` is **executed history.** Its
  slices shipped. Read it for *why* decisions D1–D16 were made — it is still the
  source of those decisions — but do not treat it as an open work queue. It is
  superseded as a forward queue by the 2026-07-24 quality plan.
- `docs/plans/2026-07-24-001-feat-public-beta-quality-plan.md` is the **active
  forward queue.**
- `docs/plans/beta-go-no-go.md` is the **live release gate.** The quality plan
  feeds it rather than replacing it; nothing ships until its blocking boxes are
  checked against real evidence.

## Non-Negotiables

- Local-first desktop app — Mac and Windows ship first and are the primary
  stabilization target (Linux deferred). The same engine also powers CI-tested
  native iPhone/Android bridges and a public web landing page. The landing page
  is a **supported product surface, in scope for launch/agent work** (decided
  2026-07-07). Mobile stays parked until the owner judges it ready; desktop
  ships first. See `docs/PRODUCT.md` "Distribution & Business Model".
- Track Master stabilization comes before new feature expansion.
- Real-time or near-real-time audition must stay responsive.
- Original/Mastered switching must preserve playhead.
- Volume Match is optional, off by default, and must not change export level.
- Exports never overwrite source files or prior renders by default.
- Export warnings are advisory unless the export is technically invalid.
- Users may overcook their own track, but the app must show clear metering,
  warnings, and review states.
- The Adaptive Compressor MVP is built but **gated OFF by default** (owner
  calibration pending). Do not enable it or change its `TBD-CALIBRATION`
  constants without an owner listening signoff.
- Private audio and rendered private masters never belong in git, unless they're
  test files used for data/research — ask the user if they're needed, since he
  moves across a lot of machines.

## Current Jump-Fix Queue

The previous five queue items all shipped — see "Implemented Stabilization
Slices" in `docs/RELEASE_STABILIZATION.md`. The full live queue and owner
decisions live in `docs/OPEN_THREADS_AND_DECISIONS.md`.

**Most recent DSP change:** the 8 character presets were re-voiced to the
"85% lean" (commit `659bea5`; `custom` untouched). Windows byte-identity
snapshots were regenerated and the Windows build is installed. The macOS
snapshot SHAs were recorded on Apple Silicon in `cc03d56`, and the follow-up
deep-analysis per-OS fixture bits landed in `88853dc`, so the Mac SHA issue is
closed; do not treat snapshot regeneration as open. Listen before any further
preset retuning.

Owner-gated listening signoffs, **deferred to Wave 10** (per
`docs/RELEASE_STABILIZATION.md`) — not the immediate queue:

1. Manual Listening Gate — normal / already-mastered / long-source sweeps and
   a clean-vs-warning export comparison, by ear (owner signoff).
2. Reference Retune listening — aggregate runner completed 2026-05-28;
   listening notes pending. Oomph is the least-matched preset; listen before
   changing it. Re-run after the 85% lean.
3. Already-mastered matrix listening signoff (runner evidence complete).

The 2026-06-16 final repo-wide review implementation queue has also shipped.
See `docs/archive/plans/2026-06-16-final-review-implementation-plan.md` for the commit
ledger and remaining owner-gated/parked items; do not treat that plan as an
open queue.

Album channel-count parity shipped 2026-06-16 (mixed mono/stereo resolution plus
above-stereo fold-down to stereo delivery; see the "Wave 9 Mechanical
Reconciliation" section in `docs/RELEASE_STABILIZATION.md`); it is no longer a
deferred slice.

The refactor backlog (former item 5) was executed in full on 2026-06-09 —
see the execution record at the end of
`docs/archive/reviews/2026-06-10-consolidated-refactor-backlog.md`. Only P2's
one-pole/soft-knee hoist (owner-deferred) and P4 tauri-specta stay parked.

## Verification

Use the fast lane for normal changes:

```powershell
npm test
npm run build
npm run build:windows
cd src-tauri
cargo fmt --check
cargo clippy --target-dir target\codex-rc --all-targets -- -D warnings
cargo test --lib --target-dir target\codex-rc
cargo test --target-dir target\codex-rc
```

The `target\codex-rc` target-dir is a **local convention** (avoids clobbering
other build dirs); CI uses the default target and runs plain `cargo fmt --check`,
`cargo clippy --all-targets -- -D warnings`, and `cargo test`. CI runs Windows,
macOS, and Android lanes on every push, and the **macOS lane runs `cargo test`**
— so the preset byte-identity snapshots gate there. `npm run verify:fast` /
`verify:rust` / `verify:iphone` /
`verify:android` wrap these lanes. If you touch anything that renders — the
landing page or the desktop UI — run the self-contained headless web lane:

```powershell
npm run verify:headless
```

It builds once, starts and tears down its own preview server, and runs the
landing + `/app` scenario suites in Playwright's **bundled Chromium** (pinned by
the lockfile — not the Chrome channel). Install the browser once per machine
with `npx playwright install --with-deps chromium`; a missing browser **fails**
the lane rather than skipping it. `npm run verify:landing` still runs the
landing assertions alone against an already-running server. Details and the
scenario list: `docs/TESTING.md` "Headless Web Lane".

The landing page **is** in agent scope — decided
2026-07-07 and reaffirmed by D16 on 2026-07-20; the Non-Negotiables above and
`docs/PRODUCT.md` "Public Surface" are the current word. (The "open owner
decision" phrasing that used to sit here was stale.)

When you touch shared crate types or `#[tauri::command]` signatures, also build
AND test the iPhone native bridge — it re-uses `yes_master_lib` but none of the
lanes above build it, so a struct-field or signature change drifts there silently
(it broke the bridge build once already). The bridge now carries real adaptive
logic (it resolves the source profile + confidence like desktop), so `cargo check`
alone is not enough — run its tests too:

```powershell
cd apps/iphone-native/rust
cargo check --all-targets
cargo test
```

The Android bridge (`apps/android-native/rust`) re-uses the same shared crate
AND the iPhone facade crate as an rlib. When you touch shared crate types,
the facade, or the android crate, run its lane too (host tests + arm64
cross-check; needs the toolchain from docs/ANDROID_NATIVE_SPEC.md "Build
prerequisites" — JDK 17 on JAVA_HOME, SDK+NDK r27.2 via local.properties or
ANDROID_HOME, rust android targets, cargo-ndk 4.1.2 or newer):

```powershell
cd apps/android-native/rust
cargo test
cargo ndk -t arm64-v8a --platform 29 check
```

`--platform 29` matches minSdk; cargo-ndk's default API 21 sysroot predates
libaaudio, so the audition link fails without it.

Use the slow fixture lane before DSP/export merges:

```powershell
cd src-tauri
$env:AMS_RUN_REAL_FIXTURE = "1"
cargo test --target-dir target\codex-rc
Remove-Item Env:\AMS_RUN_REAL_FIXTURE
```

## Working Style

- Prefer current code reality over historical prose.
- Keep changes scoped and testable.
- If a finding is objective, add a mechanical test.
- If a finding is taste/listening-dependent, capture the listening note before
  changing preset calibration.
- Do not call a slice complete because the UI resembles the goal; verify the
  behavior.
- Commit in very small chunks.
- When decisions have been made that contradict `docs/PRODUCT.md`, ask the user
  if you should update it as well as other documentation (e.g. the README).

---
> Source: [DanielKinsner/yes-master](https://github.com/DanielKinsner/yes-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
