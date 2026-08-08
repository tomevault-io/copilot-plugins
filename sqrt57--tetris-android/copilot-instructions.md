## tetris-android

> handles rotation in-process (`MainEvent::WindowResized`/`ConfigChanged`)

# tetris-android — Agent Instructions

Tetris in native Rust/NDK, built directly on the stack Bevy uses on Android
(`android-activity` + `wgpu`, no winit, no engine) to hit Bevy's Android pain
points firsthand. Background and rationale: [design/README.md](design/README.md).
Current implementation status against the build-steps plan: [README.md](README.md#status).

## Build & test

```bash
# Pure game logic (board, pieces, gravity, line clear) — no Android deps,
# runs on the host. Always run this after touching src/game.rs.
cargo test

# Cross-compile the Rust cdylib for device (arm64) and emulator (x86_64) into
# app/src/main/jniLibs. This is the fastest way to validate that any change to
# lib.rs/renderer.rs/lifecycle.rs/input.rs actually compiles against the real
# android-activity/wgpu/ndk API surface — `cargo check` on the host skips all
# of it, since those deps are gated to cfg(target_os = "android").
cargo ndk -t arm64-v8a -t x86_64 -o app/src/main/jniLibs build

# Full APK (runs cargoBuild automatically via app/build.gradle.kts, then Gradle/Kotlin)
./gradlew assembleDebug
```

No AVD is configured, but a physical device is sometimes connected via
`adb` (check `adb devices` — don't assume either way). When one is present,
`adb install -r app/build/outputs/apk/debug/app-debug.apk` and
`adb shell am start -n com.sqrt57.tetris/.MainActivity` actually exercise the
gesture/rendering paths instead of leaving them as compiled-but-unverified.
`adb install` can fail with `INSTALL_FAILED_USER_RESTRICTED` if "Install via
USB" is off or an on-device confirmation dialog is pending — that needs a
human to clear on the device itself, agents can't dismiss it.

## Toolchain (already set up on this machine as of 2026-07-26)

- Rust targets: `aarch64-linux-android`, `x86_64-linux-android` (`rustup target add`)
- `cargo-ndk` (`cargo install cargo-ndk`)
- Android SDK at `%ANDROID_HOME%`, NDK `28.2.13676358`, platform `android-36`,
  build-tools 35/36/37, `cmdline-tools` (so Gradle can auto-accept/download
  missing SDK components, as it did for build-tools 34 on first run)
- Gradle: no system install: `gradlew`/`gradlew.bat` vendor Gradle 8.10.2 via
  the wrapper (`gradle/wrapper/gradle-wrapper.jar` is committed)
- No AVD created; a physical device is sometimes attached — check
  `adb devices` before assuming a target is available either way

## Architecture notes / known gotchas

- `Cargo.toml` gates `android-activity`, `ndk`, `wgpu`, `pollster`,
  `raw-window-handle` behind `[target.'cfg(target_os = "android")'.dependencies]`
  so `cargo test`/`cargo check` stay fast and dependency-free on the host.
  Only `src/game.rs` is host-compiled; everything else in `src/` is
  `#[cfg(target_os = "android")]`-gated in `lib.rs`.
- `ndk::native_window::NativeWindow` implements `HasWindowHandle` but **not**
  `HasDisplayHandle`, which `wgpu::Instance::create_surface` requires. See the
  `SurfaceWindow` wrapper in `src/renderer.rs` — it supplies the empty
  `AndroidDisplayHandle` marker alongside the real window handle. Don't pass a
  bare `NativeWindow` to `create_surface` again; it won't compile.
- `android-activity`'s input API is a *lending* iterator:
  `app.input_events_iter()` returns something you drive with
  `iter.next(|event| -> InputStatus { .. })` in a loop until it returns
  `false`, not a normal `Iterator`. See `src/input.rs`.
- Only one consumer per frame may drain `app.input_events_iter()` — a second
  call in the same frame just sees an empty buffer. IME events
  (`InputEvent::TextEvent`/`TextAction`, driven by `src/text_entry.rs`) are
  forwarded from the same `input.rs` loop that classifies touch gestures,
  not polled separately.
- `app/build.gradle.kts` compiles against `compileSdk = 36`, one version ahead
  of what AGP 8.7.2 was tested against — `gradle.properties` has
  `android.suppressUnsupportedCompileSdk=36` for that; it's intentional, not
  a bug to "fix" by downgrading.
- `GameActivity` (via `AppCompatActivity.setContentView`) throws
  `IllegalStateException: You need to use a Theme.AppCompat theme` unless the
  app's manifest theme descends from `Theme.AppCompat`. See
  `app/src/main/res/values/themes.xml` (`Theme.Tetris`) and its
  `android:theme` reference in `AndroidManifest.xml` — don't drop either,
  the app crashes on launch (before any Rust code runs) without them.
- `AndroidManifest.xml`'s `<activity>` has no `android:screenOrientation`
  (unlocked — follows the device's sensor/rotation-lock setting) and declares
  `android:configChanges` covering orientation/screenSize/etc., so Android
  handles rotation in-process (`MainEvent::WindowResized`/`ConfigChanged`)
  instead of recreating the Activity. `game`/`name_entry` state is untouched
  across rotation; don't reintroduce `screenOrientation="portrait"` — that's
  exactly the config-change path Step 8 exercises.
- The physical test device seen so far is MIUI/Xiaomi-based (Android 16). Its
  logcat is noisy with device-specific lines that look alarming but aren't
  app bugs: `avc: denied ... miuilog`/`vendor_displayfeature_prop` (SELinux
  denials for paths the app was never granted, and doesn't need),
  `MiuiFreeFormManagerService`/`BarFollowAnimation` freeform-window chatter,
  and `FileUtils: Failed to chmod(/data/miuilog/...)`. Don't chase these when
  triaging a real crash — filter logcat by `--pid` and look for
  `FATAL EXCEPTION` / `panicked` instead.
- Git identity in this repo should be `Dmitry Grigoryev
  <1461123+sqrt57@users.noreply.github.com>` (personal GitHub identity, not
  the machine's global work email) — check `git config --local user.email`
  matches before committing if it's ever unset.

## Where things are

```
src/
  game.rs        Pure Tetris logic — board, pieces, gravity, line clear. Unit-tested.
  lib.rs         android_main entry point, event loop.
  renderer.rs    wgpu surface bound to the current ANativeWindow.
  lifecycle.rs   Owns the renderer across surface-destroyed/recreated events.
  input.rs       Touch gesture classification (zone tap / swipe) and IME event
                 forwarding into app-level actions.
  text_entry.rs  Soft-keyboard-backed name entry (high-score prompt on game over).
app/             Gradle module: manifest, GameActivity Kotlin stub, resources.
design/          LLM research/design docs behind this project.
```

---
> Source: [sqrt57/tetris-android](https://github.com/sqrt57/tetris-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
