## music-quiz

> Music Quiz v4.0.0 — multiplayer music quiz party game powered by Apple Music.

# Music Quiz — Project Instructions

## Current Status (April 2026)

Music Quiz v4.0.0 — multiplayer music quiz party game powered by Apple Music.
Core quiz + DJ Mode + Party Sessions + MusicKit JS + AI Trivia working end-to-end.

## Monorepo Layout (v4)

pnpm + Turborepo workspace. Run `pnpm install` then `pnpm build` from the root.

```
packages/
  shared/        @music-quiz/shared      — pure types (was src/quiz/types.ts)
  quiz-engine/   @music-quiz/quiz-engine — game engine, Express routes, WS handlers,
                                            Apple Music client, OAuth, MCP token,
                                            home-ws, browser-ws, playback providers,
                                            and the vanilla quiz UI under src/public/
  mcp-server/    @music-quiz/mcp-server  — MCP entry point + Express/Next bootstrap.
                                            Contains src/index.ts and server.js.
  web/           @music-quiz/web         — Next.js 16 frontend (App Router)
home/            standalone Mac agent, NOT in the workspace (own package.json)
apps/tvos/       NATIVE SwiftUI display client (Apple TV) — TestFlight live, NOT in workspace
```

**Dependency graph (one-way, no cycles):**
- `shared` → no deps
- `quiz-engine` → `shared`
- `mcp-server` → `quiz-engine` + `shared`
- `web` → `shared`

**Deviation from the original migration doc:** the doc placed `apple-music.ts`,
`oauth.ts`, `token.ts`, `token-store.ts`, `browser-ws.ts`, `home-ws.ts`,
`quiz.ts` and `quiz-manager.ts` inside `mcp-server`. Doing so would have
created a circular dependency (quiz-engine needs all of those, and mcp-server
imports from quiz-engine). They now all live in `quiz-engine` and are
re-exported via `packages/quiz-engine/src/index.ts` so `mcp-server` consumes
everything through the single workspace name `@music-quiz/quiz-engine`.

**Key build commands:**
- `pnpm install`                       — install all workspace deps
- `pnpm build`                         — turbo build (shared → quiz-engine → mcp-server + web)
- `pnpm --filter @music-quiz/mcp-server dev`  — start backend (tsc + node server.js)
- `pnpm --filter @music-quiz/web dev`         — Next.js dev server
- `pnpm start`                         — production node packages/mcp-server/server.js

`home/` is built independently with its own `tsconfig.json` + `package.json`
(plain `npm install && npx tsc` inside `home/`).

## tvOS App (apps/tvos/) — Status (April 2026)

**Live on TestFlight as build 0.1.0 (2)** under WebHouse Apple Developer team
`7NAG4UJCT9`. Bundle ID `dk.webhouse.music-quiz.tvos`. Confirmed running on
real Apple TV hardware. Display name in App Store Connect: **Quiz Mash Music**.

### Architecture
- 100% **native SwiftUI** (NOT a WKWebView shell — WKWebView doesn't exist on
  tvOS, that was the original plan and got revised in Phase 2-ny).
- **Display client only** — read-only spectator. Connects to `/quiz-ws` as
  `register_display` role. Never sends host commands. All quiz control still
  happens via web admin / mobile PWA.
- **Native MusicKit playback** via `MusicKit.framework` (when entitlement is
  re-enabled — see "Outstanding issues" below). Acts as the consumer of the
  existing `MusicKitWebProvider` outbound channel — zero server-side changes
  needed for tvOS-specific routing.
- See `docs/TVOS-WS-PROTOCOL.md` for the canonical wire contract (owned by
  the web/server side).

### Implemented features
- All quiz states render: Idle, Lobby (QR + players + join code),
  Countdown, Question (artwork + 4 colored options), Reveal (answer card +
  fun fact), Scoreboard (gold/silver/bronze ranks), Final Podium (with
  confetti + gold-pulse), DJ Mode (queue + per-player picks), Now Playing
  (vinyl sphere with pulsing red glow halo).
- Blind Round + Steal Round (april 2026 game modes) — banner badges,
  hidden options when blind, 5s steal countdown ring with pulsing red
  banner, steal-winner highlight in reveal.
- WebSocket auto-reconnect with exponential backoff.
- Demo mode (`SIMCTL_CHILD_MUSIC_QUIZ_DEMO=1`) cycles through every state
  in ~50s for visual smoke tests without a server.
- App icon + Top Shelf (regular + wide) generated from
  `logo/music-quiz-logo.svg` via `pnpm tvos:icon`.
- Fastlane pipeline: `pnpm tvos:build` (unsigned simulator smoke build),
  `pnpm tvos:beta` (full TestFlight upload via cert+sigh+gym).

### Outstanding issues
1. ~~**MusicKit entitlement is currently DISABLED**~~ — **RESOLVED april 2026.**
   `com.apple.developer.musickit` is **NOT required for tvOS** — that
   entitlement is iOS / macOS only. The previous session chased a phantom
   requirement based on iOS docs; the "Apple bug" where sigh's profiles
   strip the entitlement is actually correct Apple behaviour. Verified by
   Xcode 26's capability picker which lists no MusicKit option when the
   target platform is tvOS (only Sign In with Apple, App Groups, etc.).

   **What tvOS apps actually need for MusicKit:**
   - `MusicKit.framework` linked in the build target
   - `NSAppleMusicUsageDescription` in Info.plist (already present)
   - Runtime authorization via `MusicAuthorization.request()` in Swift
   - **No entitlement key** in `.entitlements` file
   - The `MusicQuiz.entitlements` file is intentionally empty
     (`<dict></dict>`) and that is correct.

   The existing manual signing flow (sigh + cert + build_app) works fine
   without MusicKit in entitlements. Build 3 (april 2026) shipped with
   the entitlement removed and the ASC API silent strip is now a non-issue.
2. **Release build hardcodes `https://music.quiz-mash.com`** as server URL.
   For LAN dev testing against real Apple TV hardware, either deploy
   main to production (so the TestFlight build sees the new code) or
   build a separate "dev" Release variant pointing at LAN.
3. **First TestFlight upload was painful** — see
   `memory/tvos_signing_setup.md` for the hard-won manual signing
   incantation. Don't let future sessions re-discover that.
4. **Apple Developer Team:** WebHouse `7NAG4UJCT9` (separate from
   fysiodk-aalborg-sport `MPV8D57VRK`). ASC API key `VMM43993MK` lives in
   `apps/tvos/fastlane/AuthKey_VMM43993MK.p8` (gitignored).

### tvOS dev workflow
```bash
# Generate app icon from SVG
pnpm tvos:icon

# Regenerate xcodeproj (after Swift file changes)
pnpm tvos:gen

# Unsigned simulator smoke build (CI-friendly, no secrets needed)
pnpm tvos:build

# Full TestFlight upload (requires apps/tvos/fastlane/.env)
pnpm tvos:beta

# Live test against LAN dev server (host bot creates a real session)
node apps/tvos/scripts/host-bot.mjs 192.168.39.140:3000 3
node apps/tvos/scripts/player-bot.mjs <joinCode> 192.168.39.140:3000 3

# Boot simulator + launch app
xcrun simctl boot 7F7E85BF-2387-48DD-9129-EC2AE1CC6DB5
SIMCTL_CHILD_MUSIC_QUIZ_DEV_HOST=192.168.39.140 \
  xcrun simctl launch 7F7E85BF-2387-48DD-9129-EC2AE1CC6DB5 dk.webhouse.music-quiz.tvos
```


### What Works
- **Multiplayer Quiz:** WebSocket game engine, QR join, Kahoot-style scoring, AI answer evaluation
- **Party Sessions (Events):** One join code per evening, players persist, picks accumulate across rounds
- **Events Management:** Create/edit/end events, scheduled/active/completed, disk-persisted (`quiz-events.json`)
- **DJ Mode:** Players earn picks through quiz → search Apple Music → add to shared queue
- **Global Search (Cmd+K):** Command palette searching Apple Music catalog — artists, albums, songs
- **Artist Page:** `#artist/{id}` — hero, top songs 3-col grid, albums grid
- **Album Page:** `#album/{id}` — hero, numbered tracks with hover play, Play All, Add All to Playlist
- **Song Context Menu:** ··· → Play, Go to Album, Go to Artist, Add to Playlist dropdown
- **MusicKit JS:** Browser-based Apple Music playback — no Mac/Home Controller needed (F17+F18)
- **PlaybackProvider Abstraction:** Swappable playback engines (MusicKit JS, Home Controller, future Spotify)
- **AirPlay:** HC: device list with selected status via osascript. MusicKit: Safari native picker
- **Exact Match Playback:** `play-exact` with full track name first, simplified fallback. Verified with play-log
- **Quiz Engine v2:** AI trivia (country-of-origin, band-members, artist-trivia, film-soundtrack, tv-theme), fact-checked by Sonnet
- **Verified Pool:** Massive curated song pool (150+ searches), batch download, verify BEFORE quiz starts — zero runtime failures
- **Global Question Bank:** 74+ validated trivia questions, grows with every quiz, persisted to disk
- **AI Enrichment:** Claude Haiku generates trivia, Claude Sonnet fact-checks. Fun facts shown on reveal
- **Song Dedup:** No song plays twice, no artist appears twice (normalized: strips Live/Remastered/English Version)
- **Diverse Sources:** Film soundtracks, TV themes, decades (60s-2010s), countries (Swedish, British, Korean, Danish...), deep genres
- **"Researching..." Modal:** Live seconds countdown during pool building + AI trivia generation
- **Pre-download:** All quiz songs downloaded + verified before quiz starts (progress modal with theme music)
- **Theme Songs:** Hardcoded Apple Music IDs — "Theme from New York, New York" (prep), "We Are the Champions" (victory)
- **Howler.js:** Sound manager with real applause.mp3 at podium
- **Waiting Room:** Late arrivals wait, auto-join when next lobby opens
- **Player Reconnect:** Rejoin DJ Mode seamlessly after page navigation
- **Library Cleanup:** Tracks quiz-added songs, deletes on DJ Mode end (never theme songs, never user's own music)
- **Now Playing:** Embedded screen in Host (no navigation needed), plus standalone page for display
- **Admin Hub:** Tabbed layout (Recently Played, Playlists, Favorites, Events), provider toggle, mini player
- **Universal Player:** `player.js` — single module for all playback (MusicKit JS / Home Controller)
- **Playlists:** Create, add/remove songs, Play All, Start Quiz from playlist
- **Favorites:** Heart button on all views, dedicated tab, stored as special playlist
- **Mini Player:** Track info, progress, play/pause/next/stop — works with both providers
- **Volume Slider:** Controls Music.app + all active AirPlay devices
- **Custom Dialogs:** All confirm/alert dialogs are custom dark-theme modals (never native)
- **Screen Recording:** ScreenCaptureKit Swift CLI with system audio + `--crop` flag
- **E2E Testing:** Playwright, 4-window ultrawide, 2-round + 5-round tests with Waiting Room
- **Play/Track Logs:** Server-side logging of requested vs actual playback + all track changes
- **PWA:** Service Worker v2 (network-first, auto-update), Wake Lock, NoSleep video fallback, 18 avatars
- **MUTE_ALL:** Env var for silent testing (no music, no sound effects)

### Architecture
- **Server:** Node.js + Express + WebSocket (`server.js` → `src/`)
- **PlaybackProvider:** Abstraction layer (`src/quiz/playback/`) — MusicKit JS or Home Controller
- **MusicKit JS:** Browser-based Apple Music via Apple's CDN, auth via developer token (`.p8` key)
- **Home Controller:** Legacy Mac agent (`home/`) — osascript playback, AirPlay control (fallback)
- **Universal Player:** `player.js` — single module for all playback, provider-aware, used by all pages
- **Host UI:** Vanilla HTML/JS (`src/quiz/public/host.*`) — fullscreen on Mac/TV, embedded Now Playing
- **Player UI:** Vanilla HTML/JS PWA (`src/quiz/public/play.*`) — mobile phones
- **Now Playing:** Embedded screen in Host + standalone page (`now-playing.html`) for display only
- **Quiz Builder:** Vanilla HTML/JS (`src/quiz/public/builder.*`) — curate custom playlists, MusicKit playback
- **Admin:** Vanilla HTML/JS (`src/quiz/public/admin.*`) — audio setup, recently played, mini player, Now Playing overlay
- **Frontend:** Next.js (`web/`) — original Now Playing page (being phased out for vanilla)
- **All quiz UI is currently vanilla** (not Next.js). Originally for tvOS WebView compatibility, but tvOS is now native SwiftUI so this constraint is **ophævet** — host UI may be migrated to React/Next.js when convenient.

### Key Files
| File | Purpose |
|------|---------|
| `src/quiz/engine.ts` | Game engine: sessions, scoring, question flow, Party management |
| `src/quiz/ws-handler.ts` | WebSocket handler: host/player messages, DJ Mode, provider routing |
| `src/quiz/dj-mode.ts` | DJ Mode state: picks, queue, autoplay, calculatePicksForRank |
| `src/quiz/types.ts` | All TypeScript interfaces (Party, CompletedRound, PartyState) |
| `src/quiz/routes.ts` | Express routes + MusicKit token + now-playing + play-log + events API + artist API |
| `src/quiz/event-store.ts` | Disk persistence for events (scheduled/active/completed) |
| `src/quiz/playback/types.ts` | PlaybackProvider interface |
| `src/quiz/playback/home-controller.ts` | Home Controller provider (wraps sendHomeCommand) |
| `src/quiz/playback/musickit-web.ts` | MusicKit JS provider (server→browser WS proxy) |
| `src/quiz/playback/provider-manager.ts` | Active provider management + fallback chain |
| `src/quiz/public/musickit-player.js` | Shared client-side MusicKit JS module (all pages) |
| `src/quiz/ai-enricher.ts` | AI trivia generation (Haiku) + fact-checking (Sonnet) |
| `src/quiz/ai-evaluator.ts` | Claude haiku for free-text answer evaluation |
| `src/quiz/question-bank.ts` | Global question bank — persisted validated trivia |
| `src/quiz/playlist-store.ts` | Disk persistence for custom playlists |
| `src/browser-ws.ts` | Now Playing WebSocket broadcaster + track change log |
| `home/server.ts` | Home Controller: osascript commands, WebSocket agent |
| `server.js` | Main server: routing between Express and Next.js |
| `scripts/e2e-full-flow.js` | Full E2E test: 2 rounds, Waiting Room, DJ Mode |
| `scripts/e2e-5rounds.js` | 5-round test with accumulated playlist verification |
| `scripts/e2e-screenshot-test-3players.js` | Screenshot test: 3 players, configurable Q count, post-test validation |
| `scripts/e2e-observer.js` | Headless observer — screenshots during manual testing |
| `scripts/screen-record/` | ScreenCaptureKit Swift CLI for video + audio recording |

## Playback Provider Architecture

### Provider Interface (`src/quiz/playback/types.ts`)
```
PlaybackProvider: playExact, pause, resume, setVolume, nowPlaying,
                  checkLibrary, addToLibrary, deleteFromLibrary, searchAndPlay
```

### Fallback Chain
1. **MusicKit JS** (browser) — primary, cross-platform, no Mac needed
2. **Home Controller** (osascript) — legacy, Mac only, AirPlay control
3. **Preview clips** (30s) — no login required (future)

### Key Endpoints
- `GET /quiz/api/musickit-token` — Developer token (JWT from .p8 key) for MusicKit JS
- `POST /quiz/api/set-provider` — Switch active provider (musickit-web / home-controller)
- `GET /quiz/api/playback-provider` — Current active provider
- `POST /quiz/api/now-playing` — Push now-playing data from browser to server
- `POST /quiz/api/admin/play` — Play track (play-exact with full name, addToLibrary fallback)
- `GET /quiz/api/admin/play-log` — Last 50 play requests (requested vs actual)
- `GET /quiz/api/admin/track-log` — Last 100 track changes (everything that played)
- `GET /quiz/api/artist/:id` — Artist info + top songs + albums
- `GET/POST/PUT/DELETE /quiz/api/events` — Event CRUD (disk-persisted)

### MusicKit JS Auth Flow
1. Page loads → `player.js` dynamically injects MusicKit CDN **only if preferred provider is musickit-web**
2. User clicks "Connect Apple Music" (on Admin) → Apple login popup
3. Auth persists via Apple cookies — all pages in same browser auto-authorize
4. Server notified via `POST /quiz/api/set-provider` → switches from Home Controller
5. HC mode: MusicKit CDN never loaded (prevents Safari autoplay dialog)

### AirPlay (Safari only)
- Find MusicKit's internal audio element → `webkitShowPlaybackTargetPicker()`
- Requires a song to have been played first (element created lazily)
- Non-Safari: toast recommends Safari or macOS Sound output settings

## Party Session (Events)

**Implemented.** See [docs/PARTY-SESSION.md](docs/PARTY-SESSION.md) for design.

- **Party** (Event) = entire evening — one join code, one playlist, multiple rounds
- **Round** = one quiz game within a Party
- **Playlist** = immutable, accumulates songs across rounds
- **Round #** visible in UI (host top-left badge + player lobby)
- **Picks** accumulate across rounds (shown on podium + player final screen)
- **"New Round"** button in DJ Mode, **"End Event"** to stop everything
- Party states: `playlist` | `lobby` | `quiz` | `ceremony`
- Verified with 5-round E2E test (28 songs accumulated, same join code)

## Song Playback

**ALDRIG fuzzy søgning.** Brug altid `playExact` via provider.

### Playback Chain — Quiz (via PlaybackProvider)
1. **Pre-download:** `addToLibrary(songId)` via Apple Music API under preparation modal
2. **Verify:** `provider.checkLibrary(name, artist)` confirms availability
3. **Play:** `provider.playExact(name, artist, { retries, randomSeek })` — exact match
4. **Fallback:** Try simplified name (without parentheses/remaster tags)
5. **Alt swap:** If both fail, swap question with pre-prepared alternative — NEVER silence
6. **Verify playing:** Poll `provider.nowPlaying()` with exponential backoff
7. **Nudge:** If still not playing, send `provider.resume()`

### Playback Chain — Admin (casual play from Recently Played, Playlists, Search)
1. **Try full name:** `playExact(fullName, artist)` — matches library entries with "(Remastered 2003)" etc.
2. **Try simplified:** `playExact(simpleName, simpleArtist)` — strips parentheses, splits on comma
3. **If not found + has songId:** `addToLibrary(songId)` → wait 3s → retry playExact
4. **Verify:** Server logs requested vs actual via play-log
5. **Known limitation:** osascript `play-exact` searches local Music.app library only. Songs must be in library.
6. **ALDRIG `open location`** — brings Music.app to foreground, unreliable

### Theme Songs (protected from cleanup)
- **Preparation:** "Theme from New York, New York" — Frank Sinatra (+ backups: Every Breath You Take, Message In A Bottle)
- **Victory:** "We Are the Champions" — Queen

## Game States & Join Rules

| Session State | New Player | Existing Player |
|---|---|---|
| **Lobby** | ✅ Join | ✅ Rejoin |
| **Game (playing/countdown/etc.)** | → Waiting Room | ❌ Closed |
| **Finished / DJ Mode** | → Waiting Room | ✅ Rejoin DJ Mode |
| **New Lobby opens** | Waiting Room → auto-join | Auto-join via `lobby_open` |

## DJ Mode Queue Rules
- Queue is immutable — songs are never deleted during a party
- Picks: #1=5, #2=3, #3=2, rest=1, streak bonus +1
- Picks accumulate across rounds
- 0 picks = search hidden, queue tab only
- Queue survives between quiz rounds

## Volume & Music
- **No fade-volume** — removed entirely, caused persistent volume=0 bugs
- Volume set to 75 at quiz start (via provider)
- Music paused (not faded) between songs and before countdown
- Champions plays async after results (non-blocking)

## Danish Language Support
- `looksLikeDanish()` detects æøå in text
- Option generation prefers Danish alternatives for Danish songs
- Year options never exceed current year

## Engine: Question Generation
- Generates **3x** requested count (e.g., 9 for 3 questions)
- Primary questions get artwork + options
- Alternatives ready for runtime swap if primary fails playback
- `excludeRecentPlays` checkbox controls recently-played exclusion

## Testing
- **E2E full flow:** `node scripts/e2e-full-flow.js` — 2 rounds, Waiting Room, DJ Mode
- **E2E 5 rounds:** `node scripts/e2e-5rounds.js` — 5 rounds, playlist accumulation
- **E2E screenshots:** `node scripts/e2e-screenshot-test-3players.js 20` — 3 players, screenshots, post-validation
- **E2E headless 100:** `MUTE_ALL=true node scripts/e2e-screenshot-test-3players.js 100` — stress test, grows bank
- **Manual test:** `node scripts/manual-test.js` — opens windows, user controls
- **Quiz log:** Saved to `recordings/quiz-log-{timestamp}.json`
- **Screen recording:** `recordings/` dir (in .gitignore)
- **MUTE_ALL=true** in `.env` — disables all music + sound effects for silent testing
- **Server must be fresh** for clean `usedSongIds`
- Servers: `NODE_ENV=development node server.js` (+ optional Home Controller)

## Documentation
- [docs/ROADMAP.md](docs/ROADMAP.md) — Milestones (done + planned)
- [docs/FEATURES.md](docs/FEATURES.md) — Feature list (F01-F20)
- [docs/features/](docs/features/) — Individual feature specs
- [docs/features/F20-global-search.md](docs/features/F20-global-search.md) — Global Search spec
- [docs/PARTY-SESSION.md](docs/PARTY-SESSION.md) — Party Session architecture
- [docs/QUIZ-PATCH-001.md](docs/QUIZ-PATCH-001.md) — Commercial platform & multi-provider plan
- [docs/NEXT-SESSION-PROMPT.md](docs/NEXT-SESSION-PROMPT.md) — Handoff for next session

## Hard Rules
1. **Fortæl brugeren hvad du laver FØR du laver det**
2. **ALDRIG fuzzy søgning** — exact match or silence (but swap alternative first)
3. **ALDRIG fade-volume** — caused cascading volume=0 bugs
4. **Playlist er immutable** under en party — kun tilføjelser
5. **Picks tildeles synkront** — før DJ Mode kan aktiveres
6. **Ingen join-skærm glimt** — blank skærm under auto-rejoin
7. ~~**Vanilla HTML/JS** for quiz UI~~ — **OPHÆVET (april 2026):** tvOS bygges som 100% native SwiftUI app, ikke WKWebView (WebKit findes ikke i AppleTVOS SDK). Quiz UI må refaktoreres til Next.js/React frit. tvOS connecter som `register_display` WS-rolle.
8. **Now Playing er read-only** — viser kun hvad host'en spiller, afspiller aldrig selv
9. **Én connect-knap** — Apple Music connects via Admin, alle andre sider auto-detecter
10. **ALDRIG native confirm/alert/prompt** — brug altid custom dark-theme modal dialogs
11. **ALDRIG `pm2 kill` eller global PM2-shutdown.** PM2-daemon hoster også andre apps fra andre repos (`cms-docs`, `sproutlake`, `webhouse-site`, m.fl.) som IKKE er en del af dette projekt. Brug KUN `pm2 delete <name>` eller `pm2 restart <name>` for vores egne entries (`music-quiz`, `home-controller`). Hvis TCC/permission-issues kræver respawn af daemon, så **spørg brugeren først** — han vil koordinere med en anden session der ejer de andre sites.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cbroberg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
