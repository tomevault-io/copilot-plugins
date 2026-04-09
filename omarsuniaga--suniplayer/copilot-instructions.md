## suniplayer

> Monorepo music player for live musicians/DJs.

# SuniPlayer — Claude Code Context

## Project
Monorepo music player for live musicians/DJs.
- **Web**: React 18 + Zustand 4 + Vite + Vitest 4 + TypeScript 5 (`apps/web`)
- **Native**: React Native + Expo (`apps/native`)
- **Shared core**: `packages/core` — types, stores, services, platform interfaces

Audio files live in `apps/web/public/audio/`. Built-in catalog: `apps/web/src/data/tracks.json`.

## Architecture

### Stores (packages/core/src/store/)
- **5 domain stores**: `useBuilderStore`, `usePlayerStore`, `useSettingsStore`, `useHistoryStore`, `useLibraryStore`
- **`useProjectStore`**: Composite API — combines all stores; use domain stores directly in perf-critical code
- **`usePedalBindings`**: Global keydown hook — mount once in `AppViewport`, reads state via `.getState()` to avoid stale closures

### Platform Interfaces (packages/core/src/platform/interfaces/)
- **`IAudioEngine`**: Contract all audio engine implementations must fulfill (web + native)
- Web implements via `useAudio.ts` hooks; native implements via `ExpoAudioEngine.ts`

### Audio Engine — Web (`apps/web/src/services/useAudio.ts`)
- **Dual-channel A/B**: Two HTMLAudioElement instances for seamless crossfade
- **Fade in/out**: `fadeEnabled`, `fadeInMs`, `fadeOutMs` settings; `fadeTimersRef` is a per-channel `Map` (NOT a single ref)
- **Crossfade mode**: Canal A fade-out while canal B fade-in; controlled by `crossfadeMs`
- **Volume + autoGain**: Per-track `gainOffset` applied on top of master volume
- **Analytics hooks**: `trackStart`, `trackEnd`, `trackSkip` fired from `useAudio`

### Audio Engine — Native (`apps/native/src/platform/ExpoAudioEngine.ts`)
- Implements: `load`, `play`, `pause`, `seek`, `fadeVolume`, `setVolume`, `getPosition`
- **`fadeVolume`**: fully functional — smooth volume transitions via interval-based stepping
- Callbacks: `onPositionUpdate`, `onBufferUpdate`, `onBufferingChange`, `onEnded`, `onError`
- Analytics at engine level: `trackStart`, `trackEnd`, `trackSkip` (30% rule for skip detection)
- **Pitch shift**: explicit no-op with `console.warn` — RNTP v4 does not support pitch shift; requires external library (e.g. soundtouchjs-rn)

### AudioStreamerService (`apps/web/src/services/AudioStreamerService.ts`)
- Fetch with progress tracking for loading bars
- **Blob short-circuit**: if blob URL already loaded for a track, skips re-fetch immediately
- **IndexedDB recovery**: if fetch fails, attempts to recover audio from local IndexedDB

### Blob URL Policy
- `blob_url` on `Track` is EPHEMERAL — created at runtime, NEVER persisted to localStorage
- `partialize` in stores explicitly excludes `blob_url`

### Analytics (`packages/core/src/services/AnalyticsService.ts`)
- Tracks: `playCount`, `completePlays`, `skips`, `affinityScore`
- `affinityScore` uses Laplace smoothing to avoid cold-start bias

### SQLiteStorage — Native (`apps/native/src/platform/SQLiteStorage.ts`)
- Implemented for: track analysis data + waveform data
- Binary audio storage **fully implemented**: `saveAudioFile`, `getAudioFile`, `deleteAudioFile`, `getAllStoredTrackIds`
- Audio files stored in `documentDirectory/audio_storage/`; path registered in `audio_files` SQLite table
- Base64 ↔ Blob conversion for native ↔ web transfer

## Testing
- **Web**: `npm test` — Vitest 4, 25 test files, acceptance tests across areas F1–F9
- **Native**: Jest, 3 test files (`ExpoAudioEngine`, `LocalFileAccess`, `SQLiteStorage`)
- `globals: true` in `vitest.config.ts` — required for `@testing-library/react` auto-cleanup (`afterEach`)
- Reset stores in tests: `localStorage.clear()` + `store.setState(store.getInitialState(), true)`
- Use `queryAllByText()` not `queryByText()` when multiple elements may match (throws on multiple in v10.4.1)

## Zustand Patterns
- **Stale closure prevention**: Read store state inside event handlers with `useStore.getState()`, not from React closures
- **`partialize`**: Only persistent _data_ fields — never actions, never ephemeral UI state (e.g. `learningAction`, `blob_url`)
- **Persist keys**: `suniplayer-builder`, `suniplayer-player`, `suniplayer-settings`, `suniplayer-history`, `suniplayer-library`

## Set Builder
- Algorithm: Monte Carlo (600 iterations) + Greedy fallback
- Filters: BPM range, venue energy bias, mood transitions
- BPM filter has graceful fallback: if filter leaves < 3 tracks, uses full catalog (intentional design)
- `setTrackTrim()` recalculates `tTarget` from scratch via `getEffectiveDuration()` — not a delta
- `energy` field is musical energy (0.0–1.0), NOT audio RMS — assign semantically per genre/feel

## Audio Analysis
- `pip install librosa` — available for BPM/chroma analysis of MP3s in `public/audio/`
- `pip install mutagen` — for reading MP3 duration/metadata without full decode
- `librosa.beat.beat_track()` returns ndarray — use `float(np.asarray(tempo).item())` to extract scalar
- Slow songs (~76 BPM) may be detected at 2x tempo by librosa — divide by 2 if musically implausible

## Key Files
- `apps/web/src/services/useAudio.ts` — Web audio engine (dual-channel A/B, fade, crossfade, analytics)
- `apps/web/src/services/AudioStreamerService.ts` — Streaming, blob short-circuit, IndexedDB recovery
- `apps/native/src/platform/ExpoAudioEngine.ts` — Native audio engine (Expo AV)
- `apps/native/src/platform/SQLiteStorage.ts` — Native storage (analysis + waveforms; no binary audio)
- `apps/native/src/screens/PlayerScreen.tsx` — Native player UI
- `packages/core/src/platform/interfaces/IAudioEngine.ts` — Audio engine contract
- `packages/core/src/services/setBuilderService.ts` — Monte Carlo set building algorithm
- `packages/core/src/services/AnalyticsService.ts` — Affinity scoring + play analytics
- `packages/core/src/store/` — 5 domain stores
- `packages/core/src/types.ts` — Shared types (Track, Show, SetEntry, TrackMarker)
- `apps/web/src/data/tracks.json` — Built-in catalog (edit to add/update tracks with real BPM/energy/key)
- `packages/core/src/store/useSettingsStore.ts` — includes `PedalAction`, `PedalBinding`, `PedalBindings` types
- `apps/web/src/services/usePedalBindings.ts` — Learn Mode + global keydown dispatch
- `apps/web/src/components/settings/PedalConfig.tsx` — Bluetooth pedal UI
- `apps/web/src/__tests__/features.test.ts` — Feature acceptance tests across 9 areas (F1–F9)
- `docs/superpowers/plans/` — Implementation plans (pedal bindings: done; iOS migration: pending)

## Feature Status

| Feature | Web | Native |
|---------|-----|--------|
| Reproducción básica | ✅ | ✅ |
| Fade In/Out | ✅ | ✅ |
| Crossfade | ✅ | N/A |
| Set Builder Monte Carlo | ✅ | ✅ core |
| Analytics (affinityScore) | ✅ | ✅ |
| Pedal Bluetooth | ✅ | N/A |
| Background audio | ✅ | ✅ |
| Pitch shift | ✅ web | ❌ native stub |
| Waveform | ✅ | ⚠️ parcial |
| Multi-set Shows | ✅ | ✅ core |
| Buffering bar | ✅ | ✅ |
| Blob URL fix (ERR_FILE_NOT_FOUND) | ✅ | N/A |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/omarsuniaga)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/omarsuniaga)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
