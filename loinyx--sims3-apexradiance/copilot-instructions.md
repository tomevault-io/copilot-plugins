## sims3-apexradiance

> **Apex Radiance** ("Apex Radiance for The Sims 3"; file `ApexRadiance.asi`; author @loinyx; renamed 2026-09-28 from

# CLAUDE.md: Apex Radiance

## What this is
**Apex Radiance** ("Apex Radiance for The Sims 3"; file `ApexRadiance.asi`; author @loinyx; renamed 2026-09-28 from
"Sims3 Settings Setter Apex Edition" / "S3SS Apex" / `S3SSApex.asi`) is a native mod for The Sims 3 (Steam 1.67.2,
`TS3W.exe`, 32-bit). It is an ASI loaded by Ultimate ASI Loader, running next to an unmodified official
Sims3SettingsSetter. It hooks the D3D9 device (the game runs on the official DXVK 3.1.1 `d3d9.dll`) and patches game
code in memory: Detours, pattern scans, ImGui menu, TOML config. Visible names come from `apex_version.h`
(`APEX_PRODUCT_NAME`, `APEX_PRODUCT_TAGLINE`); internal identifiers keep "Apex" (namespaces, `ApexPatch`, `APEX_`
macros, `apex_*` source files).

Features: Night Lighting (rebuilt night lamp light on ground, roads, floors, walls, roofs, water, foliage, objects,
fences, snow; key `NightTerrainRelight`), Every-Story Ground Light (`SplitLevelGroundLight`, lot lamps on any story
light the ground; part of Night Lighting), Reflections, Picture filters (SDR), Edge Smoothing
(SMAA/FXAA), Depth Blur, Borderless window, Performance (Faster Game File Lookups `ResourceLookupCache`, off by default
until tested, with Remember Missing Files `ResourceLookupMisses` (negative entries + write epochs) and Faster File Lists
`FileListCache` (GetKeyList cache), both off by default until tested; Lot Lighting While Moving `LotLightingMotion`;
Wall Shading While Moving `WallShadingWhileMoving` (defers the wall AO pass while moving, on by default); Faster Texture
Compression `FastTextureCompression`, a
bit-identical rewrite of the game's CPU DXT encoders, and Faster Cache Compression `FastCacheCompression`, a faster
RefPack compressor in the game's format, both off by default until tested; Spread New Objects Over Frames
`SceneNodeBudget`, a budgeted copy of the scene's pending-node drain while the camera moves, and Faster Object Lookups
`ObjectLookupIndex`, a validated index for the object/lot lookup by ID, both experimental and off by default; offline
tests in `tools\dxt_test` and `tools\refpack_test`; `docs/features/performance.md`), Frame Profiler (dev build only), plus dev tools (Light Probe
Ctrl+Shift+F7, Light Diag Ctrl+Shift+F8, Frame Capture Ctrl+Shift+F9, Lot Map Probe, census). Menu: Violet UI
(sidebar plus feature cards), hotkey Ctrl+Shift+F11. Smooth Streaming, Script GC Scheduler and Service Frame Budget
were removed (see below).

**Scope (user decision 2026-09-28):** HDR output, Native HDR, Ambient Occlusion, Smooth Streaming, Script GC Scheduler
and Service Frame Budget are removed from the standalone. Their findings are kept in `docs/removed-features.md`; do not
bring them back without the user asking.

## State of the code
- Frozen combined build (Apex inside a fork of S3SS): `%USERPROFILE%\Desktop\S3SS-dev\Sims3SettingsSetter\`, branch
  `night-remake`, tag `combined-final` (commit 45e36e2, local only). Full copy in
  `Backups Sims 3\16-antes da separacao (codigo completo)`. Treat it as read-only reference.
- This folder (`S3SSApex\`; the folder name is not changed yet, the user decides) is Apex Radiance, the standalone ASI
  that runs next to an unmodified official S3SS. The plan is `%USERPROFILE%\Desktop\S3SS-dev\PLANO-SEPARACAO.md`,
  summarised in `docs/architecture.md`. New work goes here. The framework is rewritten from scratch (no S3SS code).
- The docs cite files by their combined-tree names; the standalone keeps the module names.

## Read before touching anything
- `docs/README.md`: index. Then `docs/architecture.md` and `docs/workflow.md`.
- Before any lighting change: `docs/features/night-lighting/README.md`, the sub-part doc, and the engine docs
  (`docs/engine/`). Each feature doc has a "Pitfalls and failed approaches" section. Do not retry what is listed there
  without new evidence.
- Raw sources, in Portuguese and chronological (later entries win): `S3SS-dev\NOTAS-ILUMINACAO.md`, `PASSO3-PLANO.md`,
  `ROADMAP-NIGHT-REMAKE.md`. Decompile: `S3SS-dev\re\out`. Game shaders: `Game\Bin\Shaders_Win32.precomp` (read-only).

## Build
MSBuild: `C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\MSBuild\Current\Bin\MSBuild.exe`, v143,
Release|Win32, C++20, static CRT, vcpkg triplet `x86-windows-static` (always `/p:VcpkgEnableManifest=false`).

Apex Radiance (this folder; `ApexRadiance.sln` / `ApexRadiance.vcxproj`, `TargetName` `ApexRadiance`). The user
compiles; do not build unless asked:
```
MSBuild ApexRadiance.sln /p:Configuration=Release /p:Platform=x86 /p:VcpkgEnableManifest=false                     -> Release\ApexRadiance.asi (dev)
MSBuild ApexRadiance.sln /p:Configuration=Release /p:Platform=x86 /p:VcpkgEnableManifest=false /p:ApexPublic=true  -> Public\ApexRadiance.asi (public)
```
`/p:ApexPublic=true` defines `S3SS_PUBLIC` (objects in `Public\obj\`). Combined tree (frozen, for reference):
`MSBuild Sims3SettingsSetter.sln ... [/p:S3SSPublic=true]` -> `Release\` / `Public\S3SSApex.asi`.

Flavours (`build_flavor.h`): dev = everything plus dev tools and "Developer" UI sections; public = `S3SS_PUBLIC` /
`kPublicBuild`, dev tools compiled out. The user plays the dev build; releases ship the public build.

## Install (only with the game closed)
1. `TS3W.exe` and `Sims3LauncherW.exe` must both be closed (`tasklist | findstr /i "TS3W Sims3Launcher"`). Never kill
   them without asking.
2. Back up the installed `C:\Games\Hydra\The Sims 3\Game\Bin\ApexRadiance.asi` (the first time: the old
   `S3SSApex.asi`) into a new numbered folder `%USERPROFILE%\Desktop\Backups Sims 3\<NN-description>\` (next number: 84; 83 = sources of the indoor light between stories, work in progress kept out of the 1.5.1 release; 82 = installed asi before the indoor light between stories; 81 = sources before the indoor light between stories; 80 = installed asi before the glossy stair shader and the review fixes; 79 = installed asi before the flicker fix and the lamp switch updates; 78 = installed asi before the smooth indoor light; 77 = installed asi before the room light map edge padding; 76 = installed asi before the light probe alpha images; 75 = installed asi before the numbered light probe captures; 74 = light_probe.cpp before the numbered captures; 73 = installed asi before the terrain light map fix; 72 = lot_light_bridge.cpp before the terrain light map fix; 71 = installed asi before the 1.5.0 test; 70 = sources before the Night Lights brightness controls (1.5.0); 69 = installed asi before 1.4.8; 68 = installed asi before 1.4.7; 66 = sources before the Reset fix; 67 = installed asi before 1.4.6; 62-64 = picture.cpp and .claude.json before the color and MCP changes; 65 = installed asi before 1.4.5; 61 = installed 1.4.1 asi before the 1.4.2 test; 60 = installed 1.4.0 asi before 1.4.1; 59 = sources before the performance translation; 58 = sources before the lighting translation; 56 = resource_cache.cpp before the convert index fix; 57 = installed asi + crash report before it; 55 = installed asi before SceneNodeBudget was suspended; 54 = installed v1.8.0 asi before the crash reporter; 53 = installed asi before v1.8.0; 52 = sources before the v1.8.0 review fixes; 50 = sources before the v1.7.0 review fixes; 51 = installed asi before v1.7.0; 23 = source before the menu UX features; 42 = source before the compression features; 46 = source before the round 3 performance features; 47 = conflicted sources before the v1.5.0 merge of perf-c6-c8).
3. Copy the dev `Release\ApexRadiance.asi` into `Game\Bin\`. Exactly one copy of the mod in `Bin`: **delete the old
   `S3SSApex.asi`** (previous standalone and combined-build name). The official `Sims3SettingsSetter.asi` stays beside
   it. A leftover `S3SSApex.asi` is detected: if it loaded first Apex Radiance idles (log error only), otherwise the old
   one idles and a banner asks to delete it; the old combined build makes Apex Radiance keep its features off.
4. Config, log and dev outputs: `Documents\Electronic Arts\The Sims 3\Apex Radiance\` (`ApexRadiance.toml`,
   `ApexRadiance_LOG.txt`, `apex_radiance_imgui.ini`, `ApexRadiance_Hitches.txt`, `ApexRadiance_FrameCapture.txt`,
   `ApexRadiance_LightDiag.txt`, `ApexRadiance_LightProbe.txt` + `LightProbe\`, `ApexRadiance_Censo.txt` + `Censo\`,
   `Profiles\<name>.toml` = the menu's profiles, see `docs/ui.md`).
   First start without `ApexRadiance.toml`: copies the previous standalone's `...\S3SS\Apex\Apex.toml` as it is (old
   folder left in place; its `apex_imgui.ini` is not copied), else migrates from `...\S3SS\S3SS.toml` (backup
   `S3SS.toml.pre-split.bak` in the new folder). Official S3SS keeps `...\S3SS\` (`S3SS.toml`, `S3SS_LOG.txt`); Apex
   Radiance never writes there.

## Rules from the user (always)
- **Back up before modifying** any game, mod, config or source file, into `Backups Sims 3\<numbered folder>`, never
  only the scratchpad.
- **No guessing.** Study before implementing. What works: F7 GPU probe, F8 diag, read the real shader and its
  constants, patch by pattern, test offline over all captured shaders, adversarial review. Mark unverified facts as
  unverified.
- **English is the source language** of UI, tooltips and status text; logs, the Developer page and file names stay
  English only. Since 1.4.1 the menu is translated into Portuguese (Brazil), Spanish and French (Settings > Menu >
  Language; automatic = Windows' language): write new texts in English and add their translations to the tables in
  `i18n/tr_*.cpp` (how: `i18n/TRANSLATING.md`; the widgets translate what they draw, raw ImGui text and run-time text
  need `I18n::Tr` / `Trf`). **The user chats in Portuguese; reply in Portuguese.**
- **Credits:** "Credits: @loinyx" only at the end of each feature description shown on hover; no visible credit lines
  outside the menu's Settings > Credits section, which lists sims3fiend (Sims3SettingsSetter, the model for the
  rewritten framework), the line "Every-Story Ground Light (lamps on upper floors lighting the ground) uses a technique from Arro's Split-Level Lighting Fix." (user-approved 2026-09-29; also in the README credits with a link to arro-now.tumblr.com; never in feature descriptions or promo text, and never implying the whole mod is based on it), FXAA and third-party code (ImGui, Detours, toml++, SMAA, Lucide icons ISC). The framework was rewritten, so
  there are no carried sims3fiend file headers. Sims3SettingsSetter is named only in compatibility notices, detection,
  the S3SS recommendation card, Credits and the S3SS.toml migration.
- **Name:** visible text says "Apex Radiance" via `APEX_PRODUCT_NAME`; never "S3SS Apex" / "Apex Edition" again.
- **Every new function (user rule, 2026-09-29):** after adding any feature or option, review the whole menu for the
  best organization (page, tab, card, group, order, labels that read alike, Advanced vs visible, Experimental badge,
  dependencies) and adjust what needs it, not only the new row. Use the `apex-menu` skill
  (`~/.claude/skills/apex-menu`: `perl audit.pl` lists untranslated texts, copy-guideline breaks, undocumented keys
  and settings missing from ResetDefaults) and report what was moved or renamed.
- **UI:** main controls visible, tuning in collapsed "Advanced", dev tools apart (Developer sections, public build
  hides them). New standalone UI theme: **Violet** (accent #7F77DD, dark #534AB7, light #CECBF6; window #15161a, cards
  #1c1d22), sidebar plus feature cards, own hotkey Ctrl+Shift+F11 (see `docs/ui.md`).
- **Git:** author the maintainer's git identity (loinyx). Never commit game shader bytecode (`*_ref.h`), dev
  leftovers (`patches/call_trace_patch.cpp`, `patches/lot_edge_lighting_patch.cpp`, `patches/light_diag_patch.cpp`,
  `trace_targets.h`), or `Public/`. Commit only when asked.
- **Publishing, pushing, releases, renames: only with the user's explicit OK.** Licensing is open (upstream has no
  license; the user should ask sims3fiend before the standalone is published).
- Offline test harnesses: read-only, never write many files from a test exe (Kaspersky flagged one as ransomware).

## Diagnosing
Ask what the user saw first. Then read `ApexRadiance_LOG.txt` (patterns found/installed; shaders that did not match;
`[Config] Migration path:`; S3SS / old-build detection), the feature's status lines, an F7 capture, the F8 diag, and
the Frame Profiler / `ApexRadiance_Hitches.txt` for performance. The docs quote the combined build's names
(`S3SS_LOG.txt`, `S3SS_Hitches.txt`, ...). See
`docs/workflow.md` section 4 and `docs/features/dev-tools/`.

## Release
GitHub `loinyx/Sims3SettingsSetter-Apex` (combined build; releases `nightremake-v0.1.0-alpha`, `apex-v0.2.0-alpha`
Latest). Push:
`git -c credential.helper= -c 'credential.helper=!"/c/Program Files/GitHub CLI/gh.exe" auth git-credential' push fork night-remake:main`.
Release asset = public `S3SSApex.asi`, English notes in the v0.2.0 shape. Details: `docs/workflow.md` section 5.
Apex Radiance releases will ship `Public\ApexRadiance.asi`; its repository (and any GitHub repo rename) is not decided
and needs the user's explicit OK.

## Map of docs
- `docs/architecture.md`: hooks, registry priorities and Skip, post-scene chain (Edge 20, DepthBlur 30), INTZ depth
  share, patch system, TOML, logger, flavours, threads, standalone split.
- `docs/engine/*.md`: TS3W.exe RE with address tables (main loop, streaming, terrain light bake, room light maps,
  light objects and rigs, shaders, camera/map view, Mono GC, timers).
- `docs/features/**/*.md`: one per feature and per Night Lighting sub-part: settings (TOML keys, defaults), how it
  works, files, addresses and patterns, interactions, pitfalls, in-game tests.
- `docs/ui.md`: the Violet menu (pages, widgets, startup banners).
- `docs/features/performance.md`: the resource lookup cache (FindProvider, package list, database classes), the lot
  lighting budget while moving, the DXT encoders (0x006152F0 / 0x006154B0, reverse-engineered step by step) and the
  RefPack stream (compressor, decompressor, callers), the scene pending-node drain (0x006E4130) and the object tree
  walk behind the lookup by ID (0x00C62D40); `framework/slot_chain.h` shares vtable slots, `framework/entry_chain.h`
  function entries and `framework/call_chain.h` CALL instructions with the Frame Profiler.
- `docs/removed-features.md`: HDR, Native HDR, AO, Smooth Streaming, Script GC Scheduler, Service Frame Budget
  (revival notes).

---
> Source: [loinyx/Sims3-ApexRadiance](https://github.com/loinyx/Sims3-ApexRadiance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
