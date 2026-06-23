## framecast

> Fully-local screen + camera recorder web app (Chrome-only APIs). Everything — capture, compositing, encoding, muxing, trim/convert/audio-enhance — runs in the browser. Nothing leaves the machine. MIT, owned by Amit Rawat (@sahajamit), part of the Agentic Engineer brand.

# framecast

Fully-local screen + camera recorder web app (Chrome-only APIs). Everything — capture, compositing, encoding, muxing, trim/convert/audio-enhance — runs in the browser. Nothing leaves the machine. MIT, owned by Amit Rawat (@sahajamit), part of the Agentic Engineer brand.

## Commands

| Task | Command |
|------|---------|
| Dev server | `npm run dev` (open in Chrome/Edge 122+) |
| Unit tests (vitest) | `npm test` |
| E2E (Playwright + real Chrome) | `npm run e2e` |
| Typecheck | `npx tsc -b` |
| Lint / format | `npm run lint` / `npm run format` |
| Build (includes typecheck) | `npm run build` |

CI (`.github/workflows/ci.yml`) runs lint + build + unit + e2e. `netlify.yml` deploys `dist/` to **https://framecast.amitrawat.dev** (Netlify site `framecast`, headers in `netlify.toml`) on every push to main. The old github.io URL serves a static redirect.

## Architecture (the 60-second version)

```
getDisplayMedia / getUserMedia
  → MediaStreamTrackProcessor readables (transferred to worker)
  → compositor.worker.ts: OffscreenCanvas draw (frame-driven + 1 Hz heartbeat)
  → MediaStreamTrackGenerator
  → mediabunny MediaStreamVideo/AudioTrackSource (WebCodecs H.264 + AAC|Opus)
  → fragmented MP4 (1 s fragments) via StreamTarget
  → disk.worker.ts: OPFS createSyncAccessHandle, flush every 2 s
  → on stop: finalize + copy into the user's library folder (FSA dir handle)
```

Module map: `src/capture` (device acquisition) · `src/audio` (mix graph + meters) · `src/compositor` (worker, **pure geometry in `layout.ts`**, shared renderer in `scene.ts`, code-drawn backdrops in `backdrops.ts`) · `src/recorder` (session orchestrator, encoder presets, OPFS writer, crash recovery) · `src/convert` (trim + MP4/WebM/MOV via mediabunny Conversion) · `src/enhance` (RNNoise + BS.1770 loudness, video packets pass through untouched) · `src/library` (FSA folder, scan, thumbs) · `src/pip` (Document PiP deck) · `src/state` (zustand; **live objects go in `src/recorder/runtime.ts`, never the store**) · `src/app` (screens + `controller.ts` orchestration).

## Invariants — do not break these

1. **Never record through FSA `createWritable()`.** It commits only on `close()`; a crash loses everything. Live recording writes through OPFS `createSyncAccessHandle` in `disk.worker.ts` (durable, flushed every 2 s), then `promotePartToLibrary` copies the finished file out. Crash recovery (`recovery.ts`) depends on `.part.mp4` files in OPFS.
2. **The user-gesture chain.** One click cannot both open the Document PiP deck and call `getDisplayMedia`, so the two privileged calls live on separate gestures: the preflight "Select screen" click acquires the display stream (and the live preview shows the real surface), and the "Start recording" click only opens the deck and runs the countdown. Don't merge them back into one click.
3. **Pause = `videoSource.pause()` + `audioSource.pause()` in the same microtask** (mediabunny offsets timestamps for a gapless file). Mic mute is gain = 0, never `track.stop()` — silence must keep flowing for sync. The UI clock (`accumulatedMs`) is display-only.
4. **Compositing is frame-driven, never rAF/timer-driven** (hidden tabs throttle timers to 1 Hz; capture frame pumps keep flowing). The 1 Hz heartbeat redraw exists so static screens still emit fragments.
5. **Bubble geometry has one source of truth**: `compositor/layout.ts` pure functions, consumed identically by preflight canvas, PiP deck overlay and the worker. Change the math once, everywhere follows. These are unit-tested; keep them DOM-free.
6. **Recordings are fragmented MP4; every post-op output (trim/convert/enhance/recover) is standard MP4** (`fastStart: false`) for player compatibility.
7. AAC is probed at boot (`getFirstEncodableAudioCodec`); Opus is the fallback. Never hardcode AAC (Linux CI has no AAC encoder — e2e asserts accept both).

## Invariant #8 (added after issue #4)

**Never let a constructed recording session idle.** `beginRecording()` runs the countdown FIRST, then builds the pipeline and calls `output.start()` immediately. A session constructed before the countdown (encoders + track sources waiting ~3 s for `start()`) crashes Chrome's renderer when a mic is involved. Regression-guarded by `e2e/real-flow.spec.ts`, which records in real mode (3 s countdown, real PiP, folder-mode library via an IDB-seeded handle) and fails on any `page.crash`. A `?cd=N` query param overrides the countdown length for debugging.

## Invariant #9 (added with issue #5 — scene framing)

**Scene framing lives entirely in the shared renderer, never the pipeline.** `drawScene` order is: backdrop (`compositor/backdrops.ts`, painted to fill the canvas) → inset screen/camera in a rounded, optionally shadowed frame (`screenFrameRect` + `containRect`) → bubble on top (still clamped to the **full canvas**, so it can straddle the frame edge). Three rules:

- **Resolution-independent so preview == recording.** The 1280-wide preflight canvas and the full-res output must render the same relative frame: `frame.pad` is a fraction of output height; `frame.radius`, shadow and blur sizes are authored at a 1080p reference and scaled by `outH/1080` (`frameRadiusPx`). Never store frame sizes in raw output px.
- **Backdrops are theme-invariant and code-drawn.** Colors are hardcoded in `backdrops.ts` (never CSS tokens) because the backdrop is part of the recording, not the chrome, so it must not flip with the app theme. No image assets (keeps the PWA flat); the `blur` backdrop samples the live screen via a downscaled scratch buffer.
- **`backdrop:'none'` + pad 0 + radius 0 reproduces the original full-bleed output.** e2e runs this raw path by default (`freshApp` sets it); `e2e/scene.spec.ts` opts into framing and samples a decoded corner pixel to prove the backdrop is baked into the MP4. Framing is on by default for real use (`DEFAULT_FRAME`, charcoal preset).

## Testing conventions

- E2E mode is `?e2e=1` (`src/library/fsAccess.ts isE2E()`): library backed by OPFS (no native picker), deck rendered inline (no PiP), 1 s countdown. Playwright launches real Chrome with `--use-fake-device-for-media-stream --use-fake-ui-for-media-stream --auto-select-tab-capture-source-by-title=framecast`.
- Test hooks live on `window.__framecast` (`src/app/testHook.ts`), only in e2e mode: `listLibrary()`, `inspectFile(name)` (re-opens output with mediabunny and returns duration/codecs — this is how specs assert real file contents), `setFrame(patch)` (drive scene framing deterministically), `sampleTopLeft(name)` (decode a frame and read a corner pixel).
- CDP `Page.crash` never resolves its promise; send it fire-and-forget (see `e2e/recovery.spec.ts`).
- Unit tests cover pure math only (layout, BS.1770 loudness with sine reference vectors, encoder presets). Keep them free of DOM/media APIs.

## Brand & theming — the CONSOLE system

- The UI is a piece of recording hardware: bevelled modules, inset slots, throw switches, a punch record key. Design source of truth: `src/tokens.css` (verbatim Claude Design deliverable; `:root` dark + `[data-theme="light"]` color-only overrides + `.force-dark` pin for video surfaces) and the component layer in `src/index.css`.
- **Red (`--color-rec*`) is sacred**: record/stop/clipping/destructive only. The interactive accent is LED green (`--color-accent`). Never both on one control.
- Video surfaces (`--color-video*`) are theme-invariant near-black; anything showing or framing video sits in a `.force-dark` scope (stage, deck, players, filmstrips, thumbs). The PiP deck window is pinned dark.
- Type: Big Shoulders Display = engraving (uppercase + tracked, headings/wordmark only) · Familjen Grotesk = UI body · IBM Plex Mono = every changing value (timecode, dB, %, GB), tabular.
- Depth = bevels (`--shadow-module`, `--bevel-top`) and slots (`--surface-slot`, `--shadow-slot`), never flat 1px-border cards. Presses travel 1–2px down (`--t-tap`), nothing scales. Countdown beat is 1000 ms, mechanical.
- Voice is studio/broadcast: Roll tape, On air, Stand by, Take, Tape library. e2e selectors follow this voice (`/roll tape/i`, `/recover take/i`).
- Logo = hardware badge (engraved frame + LED lens, `.fcmark`); the lens goes red on air — header badge AND favicon (`icon.svg` ↔ `icon-onair.svg` swap in App.tsx). README lockups render via `brand/render.mjs`.

## Writing style (README, release notes, anything outward)

No em dashes. Use periods, commas, colons or parentheses. (House rule for everything shipped under Amit's byline.)

## Phase 2 backlog

Tracked in README "Roadmap": teleprompter in the deck, dual-mic, background blur, live zoom (issue #6), GIF export, 9:16 vertical, keyframe-snapped head-trim, Electron wrapper. (Scene framing / "wallpaper padding" shipped via issue #5.)

---
> Source: [sahajamit/framecast](https://github.com/sahajamit/framecast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
