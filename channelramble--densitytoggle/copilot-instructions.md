## densitytoggle

> Context file for Claude Code. Read this before changing anything.

# Density Toggle - Wear OS display density control

Context file for Claude Code. Read this before changing anything.

## Goal

A Galaxy Watch Ultra (480x480 round, Wear OS / One UI Watch) runs sideloaded
**phone** APKs - YouTube, Uber, Bitwarden, Proton - whose layouts are built for
a large rectangular screen. At stock density their UI overflows the round
display and controls fall outside the visible circle.

The app lowers system display density so those layouts fit, and puts it back
afterwards. Auto-first since v5.9:

- **Auto** - an accessibility service that shrinks when a chosen app comes to
  the foreground and restores when it leaves. Each chosen app can have its
  own density (default from the main screen, per-app override in the picker)
  and its own full-size-typing setting (per-app chip in the picker; no global
  toggle anymore).
- **"Full-size typing" was removed in v5.15** after empirical retesting: the
  current One UI Watch keyboard works fine at low densities (its
  numbers/symbols gesture functions at 140), so lifting density while the
  keyboard was up no longer bought anything and cost a chip + a state
  machine. If keyboard breakage at low dpi ever returns, the git history has
  the whole implementation (kbPause/kbPaused/kbLiftFor + KB_SAFE_DPI=190) -
  re-test on the device before resurrecting it.
- **Reach** (v5.12, optional, off by default): corner controls of phone
  layouts are physically outside the round panel at every density - overscan
  was removed in Android 11 and One UI Watch's WM coerces freeform back to
  fullscreen (both verified on-device), so proxy-clicking is the ONLY
  workaround. The service shows a TYPE_ACCESSIBILITY_OVERLAY chip while a
  watched app is shrunk; the panel lists clickable nodes whose centre lies
  outside the inscribed circle and fires ACTION_CLICK on the chosen one.
  Overlay sizes are PHYSICAL px, never dp - dp shrinks with the density
  override, exactly when the chip is needed. Useless for unreadable trees
  (YouTube, invariant 10).
- The manual shrink workflow was removed in v5.9. The only manual control is
  a **Restore to stock** pill that appears while the screen is actually
  shrunk - the on-watch escape hatch. Do not remove it: without it a stuck
  shrink needs adb. (v6.0 briefly made it always-visible as "Reset to
  default"; reverted - it caused layout churn.)
- **Rotation is NOT an in-app feature** (v5.10 tried, v5.10.1 removed it -
  learn from this): shrinking density makes phone apps use tablet layouts,
  which honour SENSOR orientation, so apps rotate with the wrist. The
  `accelerometer_rotation` setting cannot stop sensor-orientation requests,
  so an in-app toggle built on it LOOKS like it works but doesn't. The mode
  that works - `cmd window fixed-to-user-rotation enabled` - is gated by a
  signature permission the app can never hold, is adb-only, and is persisted
  by the system across reboots. It lives in install.sh (`--lock-rotation`)
  and INSTALL.md. Do not re-add a rotation toggle to the app.

**Hard requirement from the user: the watch must not depend on a phone or
computer during normal use.** A one-time `adb` install plus permission grant is
acceptable (the grant persists across reboots). Anything needing a per-boot
command from another device is not.

## Current status

| Area | State |
|---|---|
| Builds and dexes | **Working** on macOS with build-tools 35.0.0 |
| APK installs on watch | Not yet confirmed |
| Density actually changes from the app | **Not yet confirmed** |
| Auto mode / accessibility service | **Not yet confirmed** |
| `adb shell wm density 240` on this watch | **Confirmed working** |

The last item matters: the underlying mechanism is known good on this exact
device. What is unproven is whether the *app* can reach it.

**First thing to check on device:** open the app and read the small grey line
at the bottom of the main screen. It reports which route worked or the exact
exception. Do not guess from behaviour - that line is the diagnosis.

| Grey line | Meaning |
|---|---|
| `via binder/direct` | working, plain reflection |
| `via binder/exempt` | working, needed the hidden-API exemption |
| `via exec wm` | working via the `wm` command |
| `SecurityException: Must hold permission...` | grant missing - re-run `install.sh` |
| `NoSuchMethodException` | hidden-API filtering blocked reflection; next step is a proper bypass (LSPosed HiddenApiBypass) |
| `accessibility: ...` | self-enabling the service was refused; use the settings fallback |

## How it works

`WindowManagerService.setForcedDisplayDensityForUser()` is guarded by exactly
one thing:

```java
if (checkCallingOrSelfPermission(WRITE_SECURE_SETTINGS) != GRANTED)
    throw new SecurityException(...);
```

`adb shell wm density` works because the **shell uid holds that permission** -
not because the caller is a shell. Any app holding it may make the same call.
So the app is granted `WRITE_SECURE_SETTINGS` once via `pm grant` and then
makes the binder call directly through reflection.

`IWindowManager` is hidden API, hence:

- `targetSdkVersion` is pinned to **27** on purpose. Hidden members marked
  `max-target-o` stay reachable for apps targeting API 27 or lower. **Raising
  targetSdk will probably break density changing.**
- `Density.run()` tries, in order: plain reflection -> reflection after a
  `VMRuntime.setHiddenApiExemptions` bypass -> `Runtime.exec("wm density N")`.
  Each failure is recorded in `Density.lastError` and surfaced in the UI.

The same `WRITE_SECURE_SETTINGS` grant also lets the app switch its own
accessibility service on by writing `enabled_accessibility_services`, so the
user never has to open watch settings. That setting *is* observed live by
AccessibilityManagerService (unlike `display_density_forced`, see below).

### How auto mode decides (v5)

Events only *schedule* an evaluation; the decision reads `getWindows()`:

- active application window is a watched app -> shrink to **that app's** dpi
  (`Prefs.dpiFor`), re-applying when switching between two watched apps with
  different values;
- keyboard (`TYPE_INPUT_METHOD` window) above a watched app while the
  "Full-size typing" option is on -> lift the shrink for the typing session
  (`Prefs.kbPaused` state; `autoPkg`/`appliedDpi` keep describing it) and
  re-shrink when the keyboard hides. Reason: very low dpi makes the 480px
  panel report phone/tablet dp-widths (140 dpi ~ 549dp!) and the watch
  keyboard switches to layouts whose gestures (swipe-up numbers) break;
- a watched app still has a window up but a transient surface (shade,
  keyboard - `Prefs.isIgnored`) took focus -> hold;
- no watched app on screen -> restore.

This exists because the event-only version could not see "user went back to
the watch face": the only events were from sysui, which was ignored as
transient, so the shrink was never undone. The window list needs
`canRetrieveWindowContent="true"` + `flagRetrieveInteractiveWindows` in
`accessibility_service_config.xml` - remove either and `getWindows()` comes
back empty (the service then degrades to the old event heuristic, restore-on-
watchface-return bug included).

Two *launch shortcuts* fire before the debounced scan, so a watched app never
lays out at stock density and then rescales mid-launch (that transition
visibly mangles apps that handle density config changes themselves):

1. `typeViewClicked` events whose label text matches a watched app's name
   (outside that app and outside this app - the picker lists those labels)
   shrink before the app process starts.
2. The first `typeWindowStateChanged` carrying a watched package (the system
   splash window uses the target's package name) shrinks immediately.

Both are guesses, so `holdRestoreUntil` (2.5 s) stops the scan from restoring
until the app's real window has had a chance to appear; a confirmed sighting
clears the grace, a wrong guess gets restored right after it expires. Do not
remove the grace: without it the scan sees "watch face still active" right
after a preemptive shrink and undoes it before the app opens.

## Files

```
src/com/dt/toggle/
  Density.java             reflection into IWindowManager + fallback chain
  Prefs.java               shared config; per-app dpi; accessibility self-enable
  Ui.java                  hand-rolled Material 3 styling (pills, cards, steppers)
  MainActivity.java        main watch screen
  AppPickerActivity.java   app list with per-app dpi steppers
  WatchService.java        AccessibilityService driving auto mode (window-list based)
res/xml/accessibility_service_config.xml
res/values/strings.xml
AndroidManifest.xml
build-macos.sh   build-termux.sh   install.sh
```

UI is entirely programmatic (no layout XML, no support libraries - they need
Gradle). All Material 3 look-and-feel lives in `Ui.java`. Keep steppers as
compact `[-] value [+]` rows: the v4 bug "cannot increase auto size" was five
default-width Buttons in one row being clipped outside the round screen.

No Gradle. Plain `javac` -> `d8` -> `aapt` -> `apksigner`. `android.jar` is
bundled for machines without an SDK platform; the script prefers a real
platform jar when one is installed.

## Invariants - breaking these has already cost hours

1. **Do not raise `targetSdkVersion` above 27.** See above.
2. **Do not add anonymous inner classes, lambdas, or `private` nested classes
   whose constructor is called from the enclosing class.** All three make javac
   emit unnamed `$N` classes. R8 8.2.2 (build-tools 34.0.0) crashes on them,
   and the source is written to avoid them - listeners are implemented on the
   activity itself and dispatched via `View.setTag()`. Check with:
   `ls build/classes/com/dt/toggle/ | grep -E '\$[0-9]'` (empty is good).
3. **Do not use build-tools 34.0.0.** Its d8 crashes on *any* nested class,
   named or not. `build-macos.sh` skips it automatically.
4. **Do not write `display_density_forced` and expect it to do anything.**
   It is read at boot / user switch only. This wasted a whole iteration.
5. **The accessibility service must only undo shrinks it applied itself**
   (`Prefs.autoShrunk`), or it will fight the manual button.
6. **Secure apps (FLAG_SECURE) are invisible to the window scan.** Bitwarden -
   and every password manager - set FLAG_SECURE, so `getWindows()` +
   `getRoot().getPackageName()` returns null for their windows: the scan sees
   `active=null` even while the app is genuinely in front. Detecting the
   foreground app from the window list ALONE therefore fails for these apps
   and the service restored density out from under them (the great v5.2-5.7
   Bitwarden bounce). Fix: `TYPE_WINDOW_STATE_CHANGED` event package names are
   NOT restricted, so when the scan sees an app window it cannot identify
   (`active==null && anyAppWindow`), fall back to `lastEventPkg`. Do not
   "simplify" this back to scan-only. Diagnose with
   `adb logcat -s DensityToggle` - the service logs every scan and decision.
7. **When the active window is inferred from the event package (secure app,
   `fromEvent`), schedule a re-check** (~1.5s). A single blind scan the instant
   you leave a secure app reads `active=null` and the event fallback re-holds
   the shrink; on the idle watch face nothing else fires, so it strands at the
   shrunk size. The re-check lets a settled scan read the watch face (sysui)
   and restore. This was the "rarely doesn't reset on the watch face" bug.
8. **A watched app's window TEARDOWN fires the same `TYPE_WINDOW_STATE_CHANGED`
   as its launch.** The splash launch-shortcut must only arm the launch grace
   when `applyFor()` actually changed density (it returns boolean for this).
   Arming it on the no-op apply as you LEAVE the app deferred the restore per
   stray event - indefinitely under a live media session (YouTube playing to a
   BT headset). This was the "doesn't resize back after YouTube" bug.
9. **`Prefs.isTransientSurface()` (media-output picker,
   `*wearable.media.sessions*`) holds the shrink even when the watched app's
   window is hidden from the list.** The picker is fullscreen and replaces the
   app window; treating it as "another app" restored density mid-video. Keep
   the list narrow and NEVER add home/sysui packages to it - seeing the watch
   face must keep meaning "restore" (invariants 6/7).
10. **UsageStats is the PRIMARY foreground truth (v5.11); events+scan are the
    fallback.** Two platform facts forced this: (a) One UI Watch fires NO
    accessibility event when an already-cached task is brought back to the
    foreground (warm resume = silence), and (b) window ROOTS are unreadable
    for surface-heavy apps - YouTube's root is null even while it is
    definitely foreground. `usageForeground()` reads MOVE_TO_FOREGROUND /
    MOVE_TO_BACKGROUND from UsageStatsManager (appop
    `android:get_usage_stats`, granted by install.sh); "" means home (the
    watch face is a wallpaper service and never RESUMEs - "last app paused"
    IS home). The Heartbeat (3s screen-on / 30s idle) drives scans between
    events. Tested 20/20 on SM-L705U across warm launches, direct app-to-app
    switches at four densities, rapid cycling, playback exit, force-stop.
    Rejected on the way: window-title matching (stale windows keep titles ->
    false "watched app visible" -> restore misses).

## Approaches already tried and rejected

| Attempt | Outcome |
|---|---|
| `Settings.Secure` write of `display_density_forced` | No runtime effect. Nothing reads it live; some builds read forced density from a root-only file. |
| Shell-uid helper started via `app_process` from a phone over adb | Worked in principle but needed restarting after every watch reboot from another device - violates the hard requirement. |
| Shizuku as the privilege source | Same per-boot restart problem, plus its phone-sized UI is painful on a round screen. |
| Patching target APKs to force their own density | Needs per-activity `attachBaseContext` overrides (LSPatch territory) and re-signing breaks Play Integrity, so Google apps refuse login. |
| `-g:none` to dodge the d8 crash | No effect. The crash was toolchain-version-specific, not debug-info-related. |

## Build and install

```bash
bash build-macos.sh                     # or: BT_VERSION=35.0.0 bash build-macos.sh
bash install.sh                         # or: bash install.sh <adb-serial>
```

`install.sh` installs, grants the permission, and greps `dumpsys` to confirm
`granted=true`. It fails loudly if the grant did not land.

Watch connection is over wireless debugging and **the port changes whenever the
watch sleeps** - re-read it from the watch immediately before `adb connect`.

## Escape hatch

If density is set so low the UI cannot be tapped:

```bash
adb shell wm density reset
```

## Ideas not yet explored

- If reflection is blocked, vendor in LSPosed's `HiddenApiBypass` (needs Maven,
  so it would require a real Gradle build or a vendored jar).
- A Wear OS Tile for one-swipe toggling without opening the app.

Done in v5: per-app density values; restore after reboot (the service
re-evaluates in `onServiceConnected`, which also runs at boot).

---
> Source: [channelramble/DensityToggle](https://github.com/channelramble/DensityToggle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
