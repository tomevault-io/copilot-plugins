## memodismiss

> Guidance for AI coding agents (Claude Code, Codex, etc.) working on MemoDismiss.

# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex, etc.) working on MemoDismiss.

## What this project is

A macOS menu bar utility (Swift + Cocoa + AX APIs) that auto-dismisses the "Airmail Pro" paywall popup inside the Memo app. Event-driven, no polling, zero CPU when idle.

## Source layout

```
Sources/MemoDismiss/
  main.swift         # NSApplication bootstrap
  AppDelegate.swift  # Status item, menu, Launch-at-Login, AX permission prompt
  MemoWatcher.swift  # Core logic: NSWorkspace + AXObserver, dismiss on detect
Resources/Info.plist # Bundle metadata, LSUIElement=true (no Dock icon)
Makefile             # Universal (arm64 + x86_64) build + ad-hoc codesign + install
```

Keep the surface area small. Three Swift files, one plist. Don't introduce Xcode projects, SwiftPM manifests, or third-party dependencies without a strong reason.

## Build / install / run

```bash
make             # builds build/MemoDismiss.app (universal, ad-hoc signed)
make install     # builds + copies to /Applications/MemoDismiss.app
make uninstall   # removes /Applications/MemoDismiss.app and the LaunchAgent plist
make clean       # wipes build/
```

Run locally during development:

```bash
pkill -f MemoDismiss 2>/dev/null     # stop any old instance first
make
open build/MemoDismiss.app
```

## ⚠️ Accessibility permission resets on every rebuild — read this before testing

**TL;DR: after every `make`, you must re-grant Accessibility permission, or the AX APIs silently return nothing and the app looks broken.**

### Why

The Makefile uses ad-hoc signing (`codesign --sign -`). There is no stable Team ID, so macOS TCC identifies the app by `(bundle id, CDHash)`. The CDHash is a digest of the binary — it changes on every compile, even for a one-line edit. TCC treats the new build as a different app and revokes trust.

Symptoms of this gotcha (all mean "permission lost", not a code bug):

- Menu bar reads `Memo: not running` even though Memo is clearly open.
- Dismiss count never increments when the Airmail Pro popup appears.
- **System Settings → Privacy & Security → Accessibility** shows two `MemoDismiss` rows, or one row whose checkbox does nothing.
- `AXObserverAddNotification` returns success but callbacks never fire.

### Dev self-test workflow after any rebuild

The fastest path is the `dev` make target — it bundles every reproducible step into one command:

```bash
make dev
```

What it does:

1. `pkill -f MemoDismiss` — stop any old instance.
2. `tccutil reset Accessibility com.github.MemoDismiss` — wipe the stale Accessibility entry so the new CDHash gets a clean authorization (the row disappears from the list entirely).
3. `make build` — rebuild the universal binary and ad-hoc sign it.
4. `open build/MemoDismiss.app` — launch; the app's first AX call surfaces the system permission prompt and adds itself to the Accessibility list.
5. `open "x-apple.systempreferences:..."` — jump straight to **Privacy & Security → Accessibility**.

**Manual step that cannot be automated:** flip the **MemoDismiss** toggle to ON. macOS blocks every tool (including this Makefile, AppleScript, and direct TCC.db writes) from toggling Accessibility switches — that's a SIP-enforced rule, not a missing API.

After toggling on, verify: menu bar status flips to `Memo: running (watching)` within a second of Memo being open. Trigger the paywall (or wait for it) and confirm the dismiss counter increments.

If you need to run against the installed copy instead of `build/`, swap step 4 for `make install && open /Applications/MemoDismiss.app`.

### When reproducing the wake/unlock scenario specifically

The original bug fixed in `MemoWatcher.systemDidResume` requires a real sleep/lock cycle to verify — you can't fake it reliably. Minimum repro:

1. Complete the self-test workflow above so permissions are known-good.
2. Make sure Memo is running.
3. Lock the screen (`Ctrl+⌘+Q`) and leave it locked **at least 15 minutes** (App Nap + AX event coalescing only kick in after sustained idle; short locks won't reproduce it).
4. Unlock. The popup should be killed before you can focus on it. If not, capture `log stream --predicate 'process == "MemoDismiss"'` output and inspect whether the wake observers fired.

## Architecture notes

- `MemoWatcher` owns all AX state. `AppDelegate` is a thin UI shell — don't push AX logic into it.
- AX calls must run on the main thread. The `AXObserver` run loop source is added to `CFRunLoopGetCurrent()` from `start()`, which is called on the main thread from `applicationDidFinishLaunching`.
- Window detection is by title (`"Airmail Pro"`). If Memo renames the paywall, this is the first place to look.
- Dismiss path: move window off-screen (prevents visual flash) → find `AXCloseButton` subrole via depth-limited tree walk → `AXUIElementPerformAction(kAXPressAction)`.
- Retry cadences exist in two places and serve different races:
  - `attach()` retries on launch: `[1, 3, 5, 8, 12, 15]` seconds — covers Memo's AX tree not being ready at process start.
  - `systemDidResume()` retries on wake/unlock: `[0.3, 1, 2, 4, 8]` seconds — covers Memo creating the popup asynchronously after the system wakes.
  Don't merge or "simplify" these — they target different failure modes.
- System events that trigger a full re-scan: `didWakeNotification`, `screensDidWakeNotification`, `sessionDidBecomeActiveNotification`, and the distributed notification `com.apple.screenIsUnlocked`. All four are needed because macOS fires different subsets depending on whether the Mac slept, only the display slept, or only the session was locked.

## Change guidelines

- Prefer minimal diffs. This codebase is ~200 lines; preserve that.
- No new dependencies, no package managers, no Xcode project. `swiftc` + `make` only.
- Comments are rare and only where the *why* is non-obvious (AX races, TCC quirks, wake handling). Don't narrate what the code does.
- When adding a new AX-triggered codepath, also think about the wake/unlock case — events fired while the process is throttled can be dropped, so any new state machine needs a resume-time re-scan hook.
- After non-trivial changes, run the full dev self-test workflow above before claiming the fix works. Type-checking and "it builds" prove nothing about AX behavior.

## Known non-goals

- Paid Apple Developer signing / notarization. Accepted cost: users re-authorize on every release. Documented in `README.md` under "Updating to a new version".
- Sandboxing. AX APIs require a non-sandboxed process.
- Supporting apps other than Memo. Bundle id `com.nebula.memo` and window title `"Airmail Pro"` are hardcoded intentionally.

---
> Source: [eleven-net-cn/Memodismiss](https://github.com/eleven-net-cn/Memodismiss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
