## xsms

> Self-hosted Android app that bridges SMS to a Telegram bot. Phone runs everything; no server. Telegram is the UI.

# xsms

Self-hosted Android app that bridges SMS to a Telegram bot. Phone runs everything; no server. Telegram is the UI.

## Stack
- Kotlin 2.0.21, Jetpack Compose, Material 3
- AGP 8.7.3, Gradle 8.10.2, JDK 21 (Android Studio bundled JBR)
- compileSdk 36, minSdk 24, targetSdk 36
- `androidx.navigation:navigation-compose` for nav, `androidx.work:work-runtime-ktx` for the keep-alive watchdog

## Build & install

```bash
JAVA_HOME=/opt/android-studio/jbr ANDROID_HOME=/home/mmd/Android/Sdk ./gradlew assembleDebug
adb install -r app/build/outputs/apk/debug/app-debug.apk
adb shell am start -n dev.xsms/.MainActivity
```

System JDK is Java 26 (too new for Gradle 8.10) — must use the bundled JBR.

## Layout

```
app/src/main/kotlin/dev/xsms/
├── MainActivity.kt              entry, edge-to-edge, theme plumbing
├── XsmsApp.kt                   Application; auto-starts KeepAlive if preference set
├── data/
│   ├── AppState.kt              singleton in-memory state (bots, granted steps, theme mode)
│   ├── BotConfig.kt             telegram bot entity (id, nickname, token, allowed user ids)
│   ├── AntiKillStep.kt          5 keep-alive steps + OEM-aware rationale text
│   └── CorePermission.kt        runtime permission groups (SMS, CallLog)
├── keepalive/                   foreground service + boot receiver + watchdog + OEM intents
│   ├── KeepAlive.kt             public API (start/stop)
│   ├── KeepAliveService.kt      specialUse foreground service, stopWithTask=false
│   ├── BootReceiver.kt          restart on BOOT_COMPLETED / package replaced
│   ├── WatchdogWorker.kt        15-min periodic; restarts service if dead
│   ├── KeepAlivePreferences.kt  SharedPreferences flag (enabled, wake_lock)
│   └── OemHelper.kt             per-OEM Settings deep-links; Build.MANUFACTURER detection
├── permissions/
│   └── PermissionSync.kt        unified ON_RESUME re-check for anti-kill auto-detectables + core perms
├── navigation/
│   └── Navigation.kt            NavHost with 4 routes: main, add_bot, anti_kill, permissions
└── ui/
    ├── theme/{Color,Type,Theme}.kt   M3 + true-black (#000000) for OLED dark
    ├── components/{AntiKillBanner,PermissionsBanner,BotCard,EmptyState,ThemeSwitcher}.kt
    └── screens/{Main,AddBot,AntiKill,Permissions}Screen.kt
```

## State

All in-memory in `AppState` singleton. **Not persisted** — bots, granted steps, theme mode all reset on cold start. When wiring real SMS/Telegram, swap to Room or DataStore.

The one persisted thing is `KeepAlivePreferences` (SharedPreferences) so the foreground service auto-resumes on next launch if the user had it on.

## Keep-alive design

Not using `gyf-dev/Cactus` — that library is abandoned since 2022, all artifacts gone from Maven Central and JitPack (every build errors). The native stack here covers the same ground:

| Cactus trick | Our replacement |
|---|---|
| Foreground service | `KeepAliveService` with `foregroundServiceType="specialUse"` |
| stopWithTask=false | Same, in manifest |
| JobScheduler periodic | `WatchdogWorker` (WorkManager periodic, 15-min) |
| Boot persistence | `BootReceiver` |
| 1-pixel Activity | Skipped — blocked on Android 10+ anyway |
| Silent music | Skipped — battery drain not worth it for Samsung |

If we later need maximum reliability on Xiaomi/Huawei, the next addition is a dual-process foreground service (`android:process=":remote"` second service that ping-pongs with the main one). ~100 lines.

## ADB testing pattern

User's phone (Samsung R5CY50CMNWM, 1080x2340, One UI) is usually connected.

```bash
adb exec-out screencap -p > /tmp/x.png   # capture
# then Read /tmp/x.png to view
adb shell input tap X Y                  # tap (native coords, no scaling)
adb shell input swipe X1 Y1 X2 Y2 200    # scroll
adb shell uiautomator dump /sdcard/ui.xml && adb shell cat /sdcard/ui.xml  # exact bounds
```

Coords are 1-to-1 with the screenshot file pixels — the file IS 1080x2340.

## Open work

- Bots are entered but not persisted and not connected to anything
- SMS receive/send not wired (manifest perms declared, but no `BroadcastReceiver` or `SmsManager` calls yet)
- Telegram Bot API not wired (no `kotlin-telegram-bot` dep yet)
- The Permissions screen grants runtime perms but nothing reads SMS / call log yet

## Anti-Kill UX rules

- Notifications + BatteryOptimization auto-detect on ON_RESUME via `PermissionSync.syncAll`
- OEM steps (AutoStart, BackgroundActivity, LockInRecents) **never auto-mark as granted** — tapping "Open settings" opens the closest deep-link only; user explicitly flips the "I've done this" toggle
- The Start button on the service control card walks through ungranted steps one by one; for OEM steps it advances after one tap (session-only `oemAcknowledged` set) so the wizard doesn't get stuck, but the cards stay visually ungranted until the user toggles them
- Rationale text in step cards is OEM-specific (different on Samsung vs Xiaomi vs Huawei) via `OemHelper.current`

---
> Source: [Sir-MmD/xsms](https://github.com/Sir-MmD/xsms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
