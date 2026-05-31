## wand-enhancer

> This repository patches the Wand Electron app from a .NET Framework WPF desktop tool. Keep changes narrow and preserve the patch pipeline invariants.

INFO ./docs/*

# Wand Enhancer Agent Notes

This repository patches the Wand Electron app from a .NET Framework WPF desktop tool. Keep changes narrow and preserve the patch pipeline invariants.

## Remote Web Panel

- The default local remote port is `3223`. Keep C# and frontend constants aligned.
- The embedded panel must stay small because the desktop patcher embeds it and then injects it into Wand's `app.asar`.
- Production builds must not include mock data, debug routes, sourcemaps, local fonts, heavy icon libraries, or runtime class helper packages.
- The Electron bridge is authored as modular CommonJS source under `web-panel/bridge/source.cjs` and `web-panel/bridge/bridge-modules/`, but production runtime must be bundled/minified into `web-panel/dist/bridge.cjs` by `pnpm run build:bridge`. Do not copy `bridge-modules` into Wand or embed them as ASAR resources.
- Mock/demo data is dev-only and must be reached through `import.meta.env.DEV` dynamic imports.
- Source can use React-compatible imports, but production runtime resolves them to Preact aliases in `web-panel/vite.config.ts`.
- UI uses Tailwind CSS and lightweight shadcn-style local primitives under `web-panel/src/components/ui/`.
- Default renderer script sources live in `web-panel/bridge/scripts/default/` and are bundled/minified into `web-panel/dist/renderer-scripts/` by `pnpm run build:bridge`. Custom user scripts are selected in the WPF patch modal and copied from `PatchConfig.CustomScriptPaths`; only existing `.js` files are accepted. A local `renderer-scripts/` folder next to the patcher exe is still copied as an advanced fallback.
- `web-panel/bridge/scripts/default/installed-apps-sync.js` resolves Wand's renderer services/store and publishes `My Games` snapshots through the `wand-remote-installed-apps` IPC channel. The synced list must mirror Wand's `my_games` source criteria: catalog games come from `installedGameVersions`, and extra installed unsupported titles come from `correlatedUnavailableTitles` whose `games[].correlationIds` match `installedApps`.
- If the injected renderer cannot read a populated `correlatedUnavailableTitles` slice from the live store, `installed-apps-sync.js` must fall back to Wand's `/v3/unavailable_titles` correlation lookup through the renderer API client instead of degrading to raw install entries or an empty `My Games` list.
- Installed app snapshots should include game artwork in `imageUrl` when possible. `installed-apps-sync.js` must prefer Wand's own client icon CDN shape `https://api-cdn.wemod.com/steam_community/<steamAppId>/client_icon/96.webp` whenever the matched title/game/version metadata contains a Steam AppID, regardless of install platform. Do not assume `steamAppId` is a flat property; search nested `steam*` metadata before falling back to installed Steam `sku`. If metadata still does not expose the icon, fall back to the rendered Wand sidebar DOM (`.sidebar-game-row-image` background-image) keyed by `titleId` parsed from `data-tooltip-trigger-for`. The web panel `GameCover` must tolerate broken artwork URLs and fall back to its text cover.
- The same renderer sync script also forwards lifecycle state through `wand-remote-game-status`: `game-launched` / `game-ended` come from Wand's launch monitor service, and trainer runtime comes from the running-trainer visibility service. The web panel consumes this as the `game_status` websocket message.
- When Wand does not emit a `game-launched` event but a trainer is already active, `wand-remote-game-status` must synthesize a running session from the running-trainer visibility payload so the remote panel does not show an idle game session next to a running trainer.
- The websocket `hello` snapshot must still send cached `installed_apps` and `game_status` even when no trainer snapshot is active yet; do not reintroduce a handshake path that returns early after `trainer_changed`.
- Remote Play/Stop uses the websocket `remote_command` message. The bridge forwards it over `wand-remote-command` / `wand-remote-command-response`, and `installed-apps-sync.js` resolves Wand's trainer API + trainer service to launch a trainer for a `gameId` or end the current trainer.
- Remote Play must construct Wand's real trainer launch request class (`69482.vO`) before calling `trainerService.launch(...)`. Passing a plain object launches the game process but breaks Wand's `getMetadata(vO)`-based trainer state, causing missing status, disappearing play/close buttons, and stuck loading behavior.

## ASAR Patch Pipeline

- Preserve and restore both `resources/app.asar` and `resources/app.asar.unpacked` backups.
- Inject `web-panel/dist` as `remote-panel/`; it must already contain `bridge.cjs` and generated default renderer scripts under `renderer-scripts/`. Selected/local custom renderer scripts are then copied under `remote-panel/renderer-scripts`.
- Do not commit extracted `.source/` or `.sources/` output. Recreate it only for reverse-engineering sessions.
- `AsarSharp.AsarExtractor.ExtractAll` must skip unpacked entries when their source path equals the destination (in-place extraction is a self-copy that fails on locked files like `TrainerLib_x64.dll`) and silently skip unpacked entries whose source is missing on disk (e.g. `auxiliary/GameLauncher.exe` removed by an installer). Do not reintroduce hard failure on either case.
- The `DevToolsOnF12` patch anchors on the Electron main-process `<app>.whenReady().then(` site and attaches a `before-input-event` hook to every `BrowserWindow.webContents`. Do not patch the renderer keydown listener — the minified `ACTION_OPEN_DEV_TOOLS` dispatch site is not stable across Wand releases.
- Cheats can be pinned per game in the web panel via `pinned-storage.ts` (`localStorage` key `wand-remote.pinned-cheats.v1:<gameId>`). Pinned cheats render as a virtual `pinned` category at the top of the list; their normal category placement is preserved.
- Custom quick presets are per trainer/game and stored by `preset-storage.ts` under `localStorage` key `wand-remote.presets.v1:<gameId-or-trainerId>`. Presets capture persistent cheat values only; do not include `button` one-shot cheats in saved presets.
- All `localStorage` access in `web-panel/src/features/remote-panel/` MUST go through the shared helpers in `storage.ts` (`loadJson` / `saveJson` / `loadStringSet` / `saveStringSet`). Do not reintroduce per-module `try/catch` + `JSON.parse` duplication in `pinned-storage`, `preset-storage`, or `game-pin-storage`. Trainer/game storage IDs are derived through the shared `getTrainerStorageId(trainer)` helper in `storage.ts`; do not re-implement the `gameId → titleId → trainerId → 'global'` precedence inline.
- All shared bridge port/path/IPC channel/WS-opcode/protocol-version constants live in `web-panel/bridge/bridge-modules/constants.cjs` (exports `IPC_CHANNEL`, `WS_OPCODE`, `BRIDGE_PROTOCOL_VERSION`, `BRIDGE_SERVER_VERSION`, `RENDERER_INJECTION_DELAYS_MS`). Do not redeclare `3223`, `/remote/*`, IPC channel strings, raw WS opcode numbers (1/8/9/10), or the 500/2000 ms injection delays inline. The renderer-script-side equivalents (e.g. `vO`/`TRAINER_LAUNCH_REQUEST_EXPORT_KEY`, snapshot key prefixes, bootstrap log throttle) live in `web-panel/bridge/scripts/default/installed-apps-sync/constants.js`.
- UI string-union types follow the `E*` enum convention from `.claude/rules/frontend-conventions.md` (currently `ECheatType` in `protocol.ts`, `EConnectionStatus` in `state.ts`); the wire string values must remain on the right-hand side of the enum members. Reducer `PanelAction` `type` tags stay as discriminated-union string literals (the union itself provides the discrimination — converting it to an enum loses pattern matching).
- Cheat input controls live one-per-file under `web-panel/src/features/remote-panel/controls/` (`ToggleControl`, `SliderControl`, `ScalarControl`, `NumberControl`, `ActionButton`, `SelectionControl`, `IncrementalControl`); shared `SliderTrack` / `StepButton` / `ControlInternalProps` are in `controls/shared.tsx` and number formatting helpers in `controls/format-number.ts`. `controls/CheatControl.tsx` is a thin dispatcher map keyed by `ECheatType` — do not inline new control bodies into it.
- Mobile drawer performance is sensitive to `backdrop-filter`. Keep drawer panels and nested glass controls blur-free under coarse pointers, and do not add per-row `backdrop-blur-*` inside drawer lists.

## Validation

- Web panel build: `cd web-panel && pnpm run build` (runs type-check, Vite build, then `build:bridge` into `dist`).
- Bridge/script syntax checks after build: `node --check web-panel/dist/bridge.cjs` and `node --check web-panel/dist/renderer-scripts/remote-popup-cleanup.js`.
- Production dist should contain only static assets and should not contain `mock-instance`, `Mock Adventure`, `Simulation`, `Debug session`, `mock=1`, `demo-session`, `vite.svg`, `tailwind-merge`, `class-variance-authority`, or `clsx`.

---
> Source: [k1tbyte/Wand-Enhancer](https://github.com/k1tbyte/Wand-Enhancer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
