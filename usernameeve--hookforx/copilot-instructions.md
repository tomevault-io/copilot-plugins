## hookforx

> LSPosed/Xposed (Kotlin, API 82) module that removes ads from X (Twitter) (`com.twitter.android`). Single module `:app`. AGP 8.1.0 / Kotlin 1.9.0 / Gradle 8.2 / JDK 17, minSdk 27, target 34.

# AGENTS.md

LSPosed/Xposed (Kotlin, API 82) module that removes ads from X (Twitter) (`com.twitter.android`). Single module `:app`. AGP 8.1.0 / Kotlin 1.9.0 / Gradle 8.2 / JDK 17, minSdk 27, target 34.

## Build / deploy / verify

```bash
./gradlew assembleRelease                          # outputs UNSIGNED apk
apksigner sign --ks ~/.android/debug.keystore --ks-pass pass:android \
  --out app/build/outputs/apk/release/app-release.apk \
  app/build/outputs/apk/release/app-release-unsigned.apk
adb install app/build/outputs/apk/release/app-release.apk
adb logcat -s HookForX
```

- **After every `adb install -r`, re-enable the module in LSPosed Manager** (scope: `com.twitter.android`).
- Expected startup log: `HookForX loaded into process: com.twitter.android` + `HookResolver: <target> resolved via cache|known|scan → <class>` + `RepositoryFilter: k.b/emit ad-block installed` + `l.v presenter blocker installed` + `PremiumProbe: installed (override=..., flag getters=N, subscription checks=N)` + `HookRegistry: 4 hookers processed`.
- Tool paths: adb = `C:\Users\Administrator\Android\platform-tools\adb.exe`, jadx = `C:\Users\Administrator\jadx\bin\jadx.bat`. JDK 17 required (`C:\Program Files\OpenJDK17`). SDK located via `ANDROID_HOME`.
- No tests, no lint/CI — verification is manual on-device via logcat + uiautomator.
- **Logcat buffer floods fast** (X logs + emit counters): enlarge with `adb logcat -G 32M` before long verification sessions; keep emit/k.b logs throttled (every 20th).

## Architecture

`MainHook` (entry, declared in `assets/xposed_init`) → `DexKitLoader.load()` (`System.loadLibrary("dexkit")` first — LSPosed mounts the module's native libs onto X's linker path; falls back to extracting `libdexkit.so` from the module APK into X's code_cache + `System.load`; failure degrades to known-targets-only) → `HookRegistry.install()` → each `BaseHooker` independently in its own try/catch (one failure never blocks the others):
- **`resolver/HookResolver.kt`** — version-resilience layer resolving every obfuscated hook target structurally. Chain per target: `hookforx_force_scan` ON → skip to scan; else disk cache `files/hookforx/obfs_v<TABLE_VERSION>.json` (composite-CRC-keyed, sub-ms) → KnownTargets table (`NameMapping.kt` `KNOWN`, composite CRC matches a verified version) → DexKit fingerprint scan / chained structural derivation → structural validator (4 targets) → on failure `XLog.e` (target + fingerprint + reason) + null (soft fail, other hookers unaffected). Chained targets are structure-derived, never name-matched: `presenter_a` ← `lv.v(int, UrtTimelineItem)` return type; `urt_t0` ← first GENERIC type argument of `lv.v(int, UrtTimelineItem)`'s generic return type (`com.x.presenter.a<t0>` — a `ParameterizedType`, `actualTypeArguments[0]` is the real `com.x.urt.t0`); presenter_a's own sole abstract method return type is R8-erased to `java.lang.Object` and is only a bounds-resolved fallback (see Gotchas); `preroll_metadata` ← `getPrerollMetadata` return type. `resolver/Fingerprints.kt` holds 6 matchers (≥3 structural criteria each, obfuscated names never used as search criteria). Debug prefs: `hookforx_force_scan` (skip cache/known, pure scan — self-heal proof) and `hookforx_inject_bad_target` (fault-inject a wrong class to prove the validator's reject→fallback path), both written via MainActivity intent extras.
- `RepositoryHooker` — **the primary ad filter**, three verified layers:
  1. `com.x.repositories.urt.k.b(List, ...)` — shared list-processing entry, strips promoted items
  2. `com.x.repositories.urt.i$c$a.emit(Object, Continuation)` — Room→flow collector, home-timeline StateFlow injection point
  3. `com.x.urt.l.v(int, UrtTimelineItem)` — presenter factory; comment-section ads (promoMeta=true) are replaced with a **blank presenter** (runtime Proxy of `com.x.presenter.a` whose `c()` returns a no-op `com.x.urt.t0`; non-null so Compose LazyColumn doesn't crash). Detection: `getClientEventInfo().getComponent()` promoted/promo_ OR `getPromotedMetadata() != null` (via invokeOriginalMethod). Reflection Methods cached in a ConcurrentHashMap (hot path).
- `TimelineHooker` — `UrtTimelinePost.getPrerollMetadata()` → null (blocks video preroll ads; callers null-check it). View-layer fallback (`ViewGroup.addView` + RecyclerView `bindViewHolder`, obfuscated `RecyclerView$e0`) for classic non-Compose screens.
- `VideoAdHooker` — constructor counters on `PrerollMetadata`/`SspAdPodMetadata` (stats only).
- `PremiumProbeHooker` — **experimental** ad-relevant feature-flag / subscription-check probe. Default dump-only: DexKit-discovers X's boolean flag read points (`com.x.featureswitches` `(String, boolean)→boolean`) and subscription-state checks (`com.x.subscriptions`), hooks and logs unique ad-relevant keys once (`XLog.d`, throttled). `hook_premium_probe_override` (default off, opt-in) forces a single decompile-unambiguous key `feature/premium_plus` → true for A/B; if A/B shows no effect/breakage, empty `OVERRIDES` and keep dump-only (legal terminal state). Soft-fails cleanly when DexKit unavailable — never affects the three production hookers.

`mapping/NameMapping.kt` centralizes remaining obfuscated class/method names + the `KNOWN` KnownTargets table. `stats/StatsManager` counts blocked ads in-process. `ui/MainActivity` is a launcher screen for the module — Xposed code never runs there (also the debug-prefs writer: any boolean/string intent extra lands in `hookforx_prefs`, the same file `ModulePrefs` reads).

## Gotchas

- **`X/` at repo root is a gitignored reverse-engineering scratch dir** (jadx output of the X APK, ~32k files). Its `AGENTS.md` describes a stale `NetworkHook→DataHook→UiHook` 3-layer design that does NOT exist here — ignore it.
- **Obfuscated names are X-version-specific.** Before hooking, re-verify names by decompiling the target APK with jadx and update `NameMapping.kt`. `getField("d")` / `"e"` style short names are real obfuscated fields, not typos.
- **X 12.11.1 (installed on the test device) renders its main timeline with Compose.** `addView`/`bindViewHolder` hooks never fire for timeline items — View-layer hooking only covers classic screens (settings etc.). R8 also renames `RecyclerView$Adapter`→`RecyclerView$e0` (method `bindViewHolder` survives), `ViewHolder`→`RecyclerView$h`. TimelineHooker falls back to the obfuscated name and finds `itemView` by View-typed field scanning.
- **KnownTargets `recyclerview_adapter` resolved correctly on 12.11.1.** Decompilation shows `RecyclerView$e0` is the ViewHolder (declares itemView/FLAG_*) and the class actually declaring `bindViewHolder` is `RecyclerView$f` (dexdump: only `f` in the APK declares it). `KnownTargets` records `RecyclerView$f`; the resolver scan path also returns `f`. `TimelineHooker`'s declaredMethods guard skips safely if a wrong class (e.g. the `$e0` dual-name fallback) were ever returned.
- **Resolver soft-fail is by design** — a target failing cache/known/scan/validation logs `HookResolver: <target> ...` at error level and returns null; never crash, never block other hookers. `hookforx_force_scan` proves the scan path works without code changes; `hookforx_inject_bad_target` proves the validator's reject→fallback path (set to a target id like `lv`, then clear with an empty string).
- **`urt_t0` must be resolved from the GENERIC return type, NEVER `Method.getReturnType()` on presenter_a.** `com.x.presenter.a<T>` is a GENERIC interface — its sole abstract method `c()` returns the type variable `T`, which R8 erases to `java.lang.Object` at the bytecode level. `getReturnType()` on that method therefore returns `java.lang.Object`, useless for building the blank-presenter Proxy. Instead resolve `t0` from `lv.v(int, UrtTimelineItem)`'s `genericReturnType`: it is a `ParameterizedType` (`com.x.presenter.a<t0>`) whose `actualTypeArguments[0]` is the real `com.x.urt.t0` interface. See `resolveUrtT0` in `resolver/HookResolver.kt` (primary generic-arg path + bounds-resolved fallback); the `isInterface` guard rejects (and never caches) any `java.lang.Object`-like result — expected logcat line is `urt_t0 resolved via cache|known|scan → com.x.urt.t0`, NOT `→ java.lang.Object`.
- **Compose LazyColumn crashes if a renderer returns null** — never `param.result = null` on `com.x.urt.a` dispatcher / `com.x.urt.l.v(...)`. The blank-presenter Proxy (non-null, no-op render) is the verified-safe replacement.
- **Comment-section ads never pass `i$c$a.emit`/`k.b`** (independent data path, verified 2026-08-07 discrimination test). They render through `postdetail.o.v → com.x.urt.l.v`; entryIds look like `conversationthread-<id>-promoted-tweet-<id>-<hash>` or `...-rtb-image-ad-...`. `l.v()` is the only reliable intercept point (AD-STACK probe verified; `m0.a` interface hook does NOT fire).
- **JSON `parse()` hooks are unreliable**: X renders the timeline from local cache, so parse hooks may never fire after startup. Prefer data/presenter-layer hooking.
- Xposed API is `compileOnly` (provided by LSPosed at runtime) — never bundle it.
- Test device: KernelSU-Next + LSPosed JingMatrix 1.10.2 (manager = `com.rifsxd.ksunext`, no `org.lsposed.manager` package), adb has no `su` (`persist.sys.root_access=0`). Verify module state via manager UI / uiautomator dumps instead.

## Conventions

- **Performance is critical** (runs in X's main process): cache Method/Field refs at install time or in a ConcurrentHashMap (hot-path `isPromotedItem` does `javaClass.methods` scans otherwise), avoid I/O in hot paths. `XLog.d()` writes logcat only (XposedBridge.log is I/O) — use it for hot-path debug. `ModulePrefs` caches values after first read.
- Add new hookers as `BaseHooker` objects registered in `HookRegistry.hookers`.
- **Version-resilience rule**: when adding a hook on an obfuscated target, register the target in `resolver/Fingerprints.kt` (≥3 structural criteria, no obfuscated names as search criteria) and, if it has a stable structural criterion, add it to `HookResolver.VALIDATED_TARGETS`. New debug prefs go through the MainActivity intent-extra mechanism.
- **Idiom / module conventions**: no third-party deps, minimal code, `ponytail:` comments mark deliberate shortcuts. **One approved exception**: DexKit 2.2.0 (`implementation` in `build.gradle.kts`) — required for structural fingerprint scanning (resolution degrades to known-targets-only if its native lib fails to load). Do not add further dependencies.

## Process rules

- Ask before any git operation and report which commands were used; never commit without confirmation.
- No batch file deletion (`del /s`, `rm -rf`, `Remove-Item -Recurse` forbidden) — delete one file at a time.
- Before code changes, analyze impact and propose a plan first; keep changes minimal and consistent with existing patterns.

---
> Source: [usernameeve/HookforX](https://github.com/usernameeve/HookforX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
