## subtube

> A personal, subscription-driven YouTube reader. Pulls recent uploads (or

# subtube — project context

A personal, subscription-driven YouTube reader. Pulls recent uploads (or
playlists) from the channels you actually subscribe to (official Data API),
applies rich per-channel filters (title/description **regex**, min duration,
live/VOD, Shorts), offers per-channel **pages**, tracks **watched** state
yourself (YouTube exposes no watch history via API), and plays videos through the
official **IFrame embed** so ads still serve. No algorithm, no recommendations,
no comments — just consumption.

## Stack & conventions

Mirrors `../hafaio.github.io` (tooling) and `../done` (Firebase): Next 15 App
Router with `output: "export"` (static), React 19, Tailwind 4 (CSS-first:
`@import "tailwindcss"`), biome (double quotes, organized imports, 2-space), bun.
Deploys to GitHub Pages via `.github/workflows/` (reusable `build` + manual
`deploy`). Firebase **Functions** (Node 24, ESM) hold the YouTube OAuth refresh
flow + the Shorts probe.

**Comments**: `/** */` documents a *declaration* — a function, component, class,
type or interface, **each of their fields**, and module-level constants — so
editors surface it on hover. `/* */` for a formal explanation belonging to no
single declaration (a module preamble, a cluster of constants). `//` for short
notes and anything inside a function body. A comment states a constraint the code
can't — not what the next line does, not why a change was made, and never the
reasoning that belongs in a commit message. Don't comment the obvious.

- `bun dev` — dev server. NOTE: the dev port must be an **Authorized JavaScript
  origin** *and* **redirect URI** on the OAuth client, or the connect popup
  fails. 3000 is often taken → it falls back to 3001, so authorize whichever port
  it uses.
- `bun run lint` — `tsc && biome check`. `bun run fmt` — biome format.
- `bun export` — static export to `out/` (don't run while `bun dev` is up; it
  clobbers `.next`).
- Functions: `firebase deploy --only functions` (needs `functions/.env` with
  `YT_OAUTH_CLIENT_ID`/`YT_OAUTH_CLIENT_SECRET`, gitignored).

## Architecture

- `src/config.ts` — `firebaseConfig` + `oauthClientId` (public web config; safe
  to commit — Firestore is gated by `firestore.rules`, not secrecy).
- `src/firebase-app.ts` — the Firebase app, plus `firestoreDb()`: Firestore with
  its **persistent (IndexedDB) cache**, so listeners paint before the server
  answers and a local write shows up immediately.
- `src/firebase.ts` — Firebase Auth (identity) + Firestore helpers. `channels/`,
  `watched/` (owner-only). `loadChannelFilters` is a one-shot read per feed load
  (not a listener — a remote filter edit lands on the next refresh, not live);
  `loadWatchedFor` is likewise a one-shot query over only the loaded ids
  (`documentId() in`, 30/chunk) instead of the whole collection.
- `src/youtube-auth.ts` / `src/youtube-token.ts` — server-side OAuth: the connect
  popup runs the Authorization-Code flow, the code is exchanged in a callable,
  the refresh token is stored server-only, and the in-memory access token is
  silently re-minted via the backend (GIS can't re-mint client-side anymore).
- `functions/src/index.ts` — `exchangeYouTubeCode` / `refreshYouTubeToken` /
  `disconnectYouTube`, plus `classifyShort`: a Firestore **trigger** on
  `videoMeta/{id}` that **drains the whole queue**, not just the doc that woke it
  — it listens for `isShort == null` and probes 12 at a time until the queue has
  been quiet for `DRAIN_IDLE_MS`. `maxInstances: 1` serializes everything: a load
  creating 300 docs still fires 300 events (a doc trigger fires per doc, no way
  around it), but the first drains all of them and the other 299 find an empty
  queue and cost milliseconds. No lock, no leader election. It self-heals too — a
  doc stranded by a dropped event is swept up by the next drain. An inconclusive
  probe **deletes** the doc rather than cache a guess.
- `src/shorts.ts` — the client half: `watchShortsVerdicts` listens to the
  `videoMeta` docs it needs (one already classified by anyone costs a read and
  never reaches the backend), and `requestShortsClassification` *creates the doc*
  with `isShort: null`, which both fires the trigger and puts the video in the
  queue it drains (Firestore can't query for an absent field). Rules let a client
  ask, never answer. Requests are deduped per session. A **cache-served** snapshot
  reports no missing ids: the first snapshot of a listener precedes the server's
  answer, and trusting it would ask about videos already classified.
- `src/youtube.ts` — Data API: subscriptions, uploads (`UC`→`UU` playlist),
  playlists, and per-video duration + live status (`videos.list`).
- `src/types.ts` — `FeedItem = Video | Playlist` (discriminated by `kind`).
- `src/filters.ts` — compile + apply per-channel filters (regex/scope/mode,
  duration, live, Shorts) to a `FeedItem`; gates are video-only.
- `src/router.ts` — query-string router: a `channel` background + an optional
  open `item` (video/playlist). `?channel=x&v=y` keeps the channel behind the
  player; Back closes.
- `src/feed-cache.ts` — IndexedDB stale-while-revalidate cache (v3) for what
  Firestore's own cache can't paint synchronously with the items: the YouTube
  items + the subscribed channels **with their filters**, so a reload filters the
  cached feed on the first frame instead of flashing it unfiltered while a read
  resolves. Shorts verdicts are folded in as they arrive (`cacheShortsVerdicts`),
  since they land *after* the load that wrote the cache — a cache that never
  learned them replays those videos as unclassified, and a channel gating on
  Shorts hides those, so every reload would open on a hole that fills in a moment
  later.
- `components/feed.tsx` — workhorse: hydrate from the cache (items **and**
  filters, so the first frame is already filtered), then load → `enrichItems`
  (watched + known Shorts verdicts) → filter/sort grid; channel pages (scoped
  feed, on-demand fetch for disabled); mid-load 401 silent-refreshes once. A load
  runs once on connect and thereafter **only when asked** — the Refresh button or
  a page reload; returning to the app shows the last-loaded feed, it doesn't
  re-fetch. Filters are snapshotted at each load (one-shot `loadChannelFilters`),
  not streamed: an edit applies to `channels` locally and saves to Firestore, but
  a remote change waits for the next load — so the feed never re-filters on its
  own. Shorts stay listener-driven (`watchShortsVerdicts`), the one thing that
  arrives after the load and patches the cards in place. A watched toggle made
  mid-load is re-applied over the load's result (that read is a point-in-time
  snapshot).
- `components/{video-card,playlist-card,channel-filters,player,login,...}.tsx` —
  UI. `next/image` with `images.unoptimized`; thumbnails guarded against empty
  src.

## Key decisions

- **Server-side OAuth** (refresh token in `users/{uid}/private/**`, no client
  rule) instead of GIS silent re-mint, which Google removed. Browser holds only
  short-lived access tokens.
- **Watched is self-tracked** in Firestore (YouTube removed the history API), and
  read per-feed-window rather than listened to: Firestore bills a re-listen after
  a 30-min disconnect as a fresh query, so a collection listener would re-read the
  whole lifetime history every session.
- **Shorts are database-driven, end to end.** The client only ever reads and
  writes Firestore: it listens to the `videoMeta` docs for the videos on screen,
  and *creating* a doc is how it asks for a missing one. The trigger fills it in;
  the listener delivers it. No callable, no request collection, no crawler, no
  scheduled sweeper — work exists only because a user loaded a video nobody had
  classified. **Probes are batched** by draining the queue in one invocation:
  billing is by the second and a probe is a second of waiting on YouTube, so 12
  concurrent probes share one billed second instead of each buying its own (~5×
  cheaper than a probe per invocation; both are pennies, but the drain is also
  what makes it self-healing).
- **Only channels that keep or drop Shorts get classified at all.** A channel whose
  Shorts filter is `all` never consults the verdict, so its videos are neither read
  from `videoMeta` nor probed — for most channels the whole mechanism costs nothing
  and never runs. (Trade: those cards carry no "Short" badge, since nothing knows.)
- **A channel that gates on Shorts hides what it can't yet judge**: an
  unclassified candidate is held back until a verdict lands, so a Short never
  blinks into (or out of) a feed that filters them. The cost is that a video
  whose verdict never comes (an inconclusive probe deletes the doc) stays hidden,
  so the Refresh icon keeps spinning while any candidate is outstanding — and
  stays *clickable*, since a reload is the only way out of that state. Channels
  with `shortsFilter: "all"` never consult a verdict and so are never affected.
- **Playlists/channel pages reuse the feed pipeline**; the broadcast/Shorts gates
  no-op on playlists.
- **Keep the official IFrame player** so ads serve and views count (intentional,
  creator-friendly; not an ad-stripping client like FreeTube/Invidious).

## Status

Feature-complete for the intended scope; lint/typecheck/tests pass. The scaffold
and the Functions backend are merged to `main`; the reader app is the open PR.
Functions (`subtube-dev`, us-central1) and Firestore rules are deployed.

Future work: auto-generate a per-channel regex from a few picked videos (greedy
set-cover over title tokens; minimal-exact is NP-hard); a paginated / infinite
feed (older uploads beyond the first page per channel).

## Gotchas

- Watch-history is self-tracked, not real YouTube history.
- Silent refresh needs an active Google session + prior consent; otherwise a
  one-click interactive Reconnect appears.
- Esc can't reach the app while the cross-origin player iframe has focus — Back
  (URL-driven) closes the player reliably.
- Shorts detection relies on undocumented `/shorts/{id}` redirect behavior;
  changing the probe logic means clearing `videoMeta` (no cache version).
- Comparable tools: Feedvault (paid, no regex/own-sync), FreeTube (ad-stripping
  desktop), Unhook (extension). subtube's niche = rich per-channel filters + own
  cloud sync + ad-serving player.

---
> Source: [hafaio/subtube](https://github.com/hafaio/subtube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
