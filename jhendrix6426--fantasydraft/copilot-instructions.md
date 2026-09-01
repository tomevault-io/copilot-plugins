## fantasydraft

> Hobby project for a fantasy draft built around a tabletop CCG's ("Redemption")

# FantasyDraft

Hobby project for a fantasy draft built around a tabletop CCG's ("Redemption")
annual Nationals tournament. Players are drafted onto fantasy teams before
Nationals, then scored on their real tournament performance. The frontend is
a static site (no build step) deployed on GitHub Pages. There is now also a
small Cloudflare Worker backend (`worker/`) powering the live draft feature —
see "Live Draft system" below. Everything else (scouting, standings) stays
static/read-only with no backend of its own.

Live site: https://jhendrix6426.github.io/FantasyDraft/
Repo: https://github.com/jhendrix6426/FantasyDraft — pushing to `main` auto-deploys.

## Structure

- `index.html` — landing page: header banner + tab bar (GM Tools, Commissioner
  Tools, Live Draft, Live Scoring, Draft History, Records). "GM Tools" and
  "Commissioner Tools" are `.tab-group`s — click opens a `.tab-dropdown`
  (GM Tools: Scouting, My Draft Board, My Account, Join Live Draft;
  Commissioner Tools: Live Draft Setup, Scorekeeping) rather than loading a
  page directly; each toggle button gets `.active` whenever any of its own
  sub-tabs is loaded, same visual treatment as a normal tab. The remaining
  four (Live Draft, Live Scoring, Draft History, Records) are plain buttons.
  Tab-switching JS lives inline at the bottom of the file — clicking any
  `[data-tab]` button swaps `#tab-frame`'s `src` to `data-src` if the button
  has one, else `data-tab + '.html'`. `data-src` exists for buttons that
  don't map 1:1 onto `<data-tab>.html`: **Live Draft**/**Live Scoring** (top
  level) point at `draft-presentation.html?year=…`/`scoring-presentation.html?year=…`
  — the read-only broadcast views, which already render their own "waiting"
  placeholder when nothing's live yet, so no separate not-available handling
  was needed here — while **Join Live Draft** (GM Tools) and **Live Draft
  Setup** (Commissioner Tools) both point at `live-draft.html` with
  `?mode=gm` / `?mode=commish` to skip straight past its landing role-picker
  into the right login form (see `landingMode` in `live-draft.html`, below).
  **Scorekeeping** just points at plain `live-scoring.html`, since its
  landing view is already the scorekeeper login with no picker to skip.
  Bump the two hardcoded `year=` values here every season alongside
  `FANTASY_DRAFT_YEAR` in `draft-history.html`/`scouting.html`/`records.html`.
  Adding another plain top-level tab is a new button with a matching
  `data-tab` (+ `data-src` if it needs one); adding another dropdown sub-tab
  is a new button inside the relevant `#…-dropdown` plus adding its
  `data-tab` value to that dropdown's `…TabNames` array (`gmToolsTabNames` /
  `commishToolsTabNames`) so its toggle highlights correctly. All dropdowns
  share one open/close implementation (`.tab-toggle`/`.tab-dropdown` classes,
  not per-dropdown ids) — opening one closes any other that's open. Every
  dropdown closes on an outside click *or* on a click into `#tab-frame` — the
  latter needs its own handling (`window`'s `blur` event) since a click
  inside an iframe never bubbles to the parent document, so a plain
  document-level click listener alone can't catch it. This header only wraps
  a page when it's reached *through* the tab bar (i.e. loaded inside
  `#tab-frame`) — GMs reach `live-draft.html`/`my-board.html` via a direct
  shareable link instead, which bypasses `index.html` entirely. Those pages
  (plus `live-scoring.html`, which can also be opened directly) detect this
  with `window.self === window.top` and render their own small `.site-nav`
  back-link row when true, so a GM arriving cold via their link isn't
  stranded with no way to reach the rest of the site. Don't add this to
  `draft-presentation.html`/`scoring-presentation.html` — those are
  deliberately chrome-free full-bleed broadcast views, always opened
  directly, never wrapped by `index.html`.
- `scouting.html` — the scouting tool. Self-contained single file (HTML/CSS/JS,
  no dependencies, no build). This is where most of the historical-stats work
  has happened.
- `records.js` — shared IIFE (`window.RecordsBook`) for all-time record-book
  computation (format/day high scores, win streaks, draft steals, etc.),
  loaded by `records.html`, `draft-history.html`, and `scouting.html` so the
  logic isn't triplicated. Takes the raw `nationals-history` `db` payload
  directly via `RecordsBook.compute(db)` — see its header comment for the
  full exported API.
- `records.html` — all-time record book UI, reads `records.js`.
- `draft-history.html` — past fantasy drafts (standings, draft order, per-year
  rosters), reads the same `nationals-history` API plus `records.js` for
  badges.
- `live-draft.html` — GM- and commissioner-facing live draft tool (see below).
- `my-board.html` — standalone GM-only view of a single GM's private draft
  board (see "GM draft boards" below), for keeping it open on its own
  tab/device separate from both `live-draft.html` and `scouting.html`. Not
  part of the tab system — opened directly by URL or via a link from
  `live-draft.html`'s board panel.
- `my-account.html` — standalone GM-only page for changing your own password
  (see "Email+password" below). Same not-part-of-the-tab-system status as
  `my-board.html`.
- `draft-presentation.html` — standalone OBS-facing broadcast view for the
  live draft (see below). Not part of the tab system — opened directly by URL.
- `live-scoring.html` — scorekeeper-facing live scoring entry tool (see
  "Live Scoring system" below).
- `scoring-presentation.html` — standalone OBS-facing broadcast view for live
  scoring (leaderboard + records-aware ticker). Not part of the tab system.
- `worker/` — the `fantasy-draft` Cloudflare Worker source, deployed
  separately from the site (see "Live Draft system" / "Live Scoring system").
- `assets/header.png` — transparent PNG logo mark (badge + wordmark) used in
  `index.html`'s header, displayed at 34px tall next to the tab bar. The
  header itself is a fixed 64px bar with a CSS (not image-based) glow
  backdrop — no other image assets are in use.

## Data source

Everything is fetched client-side on page load from a Cloudflare Worker:
`https://nationals-history.jhendrix6426.workers.dev/nationals/db`

That single JSON payload contains:
- `results` — per-tournament placements/points by format (`"2025_T1 2-Player"` style keys)
- `matches` — round-by-round match results (winner, scores, opponents), same key format as `results`
- `tournaments[]` — one entry per Nationals year, including `tournaments[].fantasyDraft`
  for years that had a fantasy draft (currently 2024 and 2025)
- `players` — flat name registry, not used for stats
- `multiWL` — not currently used

**Data pool policy: everything is 2022 onward, no exceptions.** There used to
be a special case excluding 2022 "Teams" format data (low turnout that year),
but that was deliberately removed — the tool now always uses the full 2022+
pool for every format, including Teams. Don't reintroduce a per-format year
exception without being asked.

Format codes used throughout: `T1` (Type 1), `T2` (Type 2), `BD` (Booster
Draft), `SD` (Sealed), `Teams`, `TA` (Type A). `FMT_MAP` in `scouting.html`
translates the API's verbose format names into these codes; `FMT_TO_DAY` maps
each format to the tournament day it's played (Thu/Fri/Sat).

Name matching between datasets (API uses full names, sometimes with nickname
variants like "Mitch"/"Mitchell") is handled by `namesMatch()` — it requires
exact last-name match plus first-name match-or-prefix. If a player's stats
seem to be missing, check whether their name needs a normalization rule added
to `normName()`.

## The player pool is fetched from the Worker, not hardcoded

`PLAYERS` in `scouting.html` (this year's Nationals attendees + their
Thu/Fri/Sat format registrations) used to be a hardcoded array, manually
kept in sync by hand with `players_<year>` in the `fantasy-draft` Worker (the
same data live-draft.html's Player Pool panel edits) — two copies of the same
list, updated separately. That's gone: `loadPlayerPool()` in `scouting.html`'s
`init()` now fetches `GET /fantasy/players/<FANTASY_DRAFT_YEAR>` directly (same
call live-draft.html/my-board.html already made), so there's exactly one
source of truth. Uploading a new registrations CSV via live-draft.html's
Player Pool panel (see "Player pool" under Live Draft system below) now shows
up in `scouting.html` on next load with no separate step. If that fetch
fails (Worker unreachable — rare, but see below), `PLAYERS` is left `[]` and
`setStatus()` shows a distinct ⚠ message rather than silently rendering an
empty table with no explanation; this doesn't fall back to a stale hardcoded
copy on purpose, since that would just reintroduce the exact drift this
change removed. Each pool entry:

```js
{name:'Player Name', thu:'BD', fri:'T1', sat:'Teams', days:3, firstNats:false}
```

`thu`/`fri`/`sat` are format codes or `null` if not playing that day; `days`
is the count of non-null days. This is a live network dependency scouting.html
didn't strictly have before (previously only used for the GM board/session
triangle, see "Live Draft system" below) — accepted deliberately, since
`live-draft.html`/`live-scoring.html`/`draft-history.html` already fully
depend on the same Worker being up, and a real Cloudflare Workers outage is
rare enough (~99.99%+ published uptime) not to be worth designing around; a
bad `worker.js` deploy is the more realistic failure mode, and that's on us
to test before shipping, same as always.

## Fantasy draft history is live, not hardcoded

`DD` (fantasy draft results by year) used to be a hardcoded object. It's now
built at runtime by `processFantasyDraft()` from `tournaments[].fantasyDraft`,
so new draft years appear automatically once the API has them — no code
changes needed. Champion is derived as the team with the highest `pts`.

## Scouting table features

- Per-player Nationals stats (personal avg vs. pool avg, by day/format),
  Win/Loss match record, and fantasy draft history — all in a sortable/
  filterable table, with a per-player modal for full detail.
- **Customizable columns**: users toggle which stat columns are visible via
  the "Customize" panel. Preference persists in `localStorage`
  (`scouting-visible-cols`).
- **Draft Score**: a weighted composite ranking (-3 to 3 weight per stat:
  Total Avg, Win%, Fantasy Slot Delta). Missing data scores as neutral (50),
  not zero — first-time attendees shouldn't get punished for lacking history.
  Weights persist in `localStorage` (`scouting-score-weights`). See
  `computeScores()` if touching this.
- Adding a new column: add an entry to `COLUMN_DEFS` (id, label, optional
  sortKey, render function) — the table header/rows/toggle panel are all
  generated from that array, nothing else needs to change.

## Design system

Dark sporty/tech theme pulled from the logo: near-black background, red/gold
gradient accents, radial glow lighting. Fonts via Google Fonts: **Oswald**
(headers, labels, buttons — condensed/athletic) and **JetBrains Mono** (all
data/stats — keeps numbers aligned, terminal/scoreboard feel). Format badges
are color-coded pills (cyan/red/green/gold/purple/orange per format). Keep
`index.html`'s tab bar styling in sync with `scouting.html` if the palette
changes — they're meant to feel like one product.

**Most pages' `.page` container is `max-width: 1480px`**, matching
`scouting.html` (chosen as the reference since its wide data table needs the
room) — `draft-history.html`, `records.html`, `live-draft.html`, and
`live-scoring.html` match it, so those don't read as narrower than one
another depending on which tab you're on. Two deliberate exceptions stay
narrower: `my-account.html` at `560px` (just a 2-3 field password form —
stretching it to full width looked worse, not more consistent), and
`my-board.html`, reverted back to its original `720px` after widening it
once — a single-column ranked list read as too spread out at 1480px, unlike
the other pages' multi-card/grid layouts which actually used the extra
room well. `draft-presentation.html`/`scoring-presentation.html` aren't part
of this at all — they're full-bleed broadcast views by design (see above),
no `.page` wrapper to match. Widening a page's container doesn't mean form
fields inside it should stretch to it: on the four 1480px pages,
`select`/`input[type=text|password|number]` are capped at `max-width: 480px`
(textareas get their own
wider `900px` cap, since the JSON pool editor benefits from the room) in
every file whose `.page` grew — otherwise a lone Roster Size input or a
login form's username field ends up absurdly wide. If you add another page
to the site, match both numbers, not just the outer one.

## Live Draft system

A real-time, multi-device live draft, separate from the read-only
`nationals-history` API used by `scouting.html` — this one has its own
backend, `worker/` (Cloudflare Worker `fantasy-draft`, deployed at
`https://fantasy-draft.jhendrix6426.workers.dev`, single KV namespace
`FANTASY_DB`). It was built from a scrapped earlier attempt at this project
(originally at `~/fantasy-draft-worker/`, now moved into this repo).

**Auth** (there is none of this on the `nationals-history` API — don't
confuse the two):
- Commissioner actions (GM roster, scoring config, player pools, draft
  start/pause/resume/undo) require an `X-Commish-Key` header matching the
  `COMMISH_KEY` Worker secret. Set via `wrangler secret put COMMISH_KEY` from
  `worker/` — it is **not** stored anywhere in this repo, only in Cloudflare.
- GM pick submission requires `X-GM-Token` + `gmId` in the body, checked
  against a `token` field on that GM's `gm_registry` entry. `GET /fantasy/gms`
  strips tokens from the response (public); `GET /fantasy/gms?full=1` (commish
  auth) returns them, for the roster editor in `live-draft.html`. Commissioner
  UI generates each GM a shareable link (`live-draft.html?gm=id&token=...`)
  rather than making them type a token.
- No auth for spectators: `live-draft.html`'s landing screen (shown when
  there's no saved session, no `?gm=&token=` link, and no `?mode=` param —
  see below) has a third "Just Watching" option alongside GM/Commissioner
  that just links straight to `draft-presentation.html?year=...` — for
  anyone who wants to follow along without a GM token or commish key. Since
  `index.html`'s top-level Live Draft tab now points directly at that same
  presentation view (see `index.html` above), this picker option mostly
  matters for someone who reached `live-draft.html` cold via a raw/shared
  link rather than through the tab bar.
- `?mode=gm` / `?mode=commish` (read into `landingMode`) skip that
  three-option picker entirely and render the matching login form straight
  away — this is what `index.html`'s "Join Live Draft" and "Live Draft
  Setup" nav items link with. It only affects what the picker *shows*; a
  saved session or a `?gm=&token=` link still short-circuits straight to a
  logged-in view exactly as before, checked earlier in `init()` than
  `landingMode` is ever consulted. `renderGmLoginForm()`/`renderCommishLoginForm()`
  (plus their `attach…Handlers()` pairs) are shared between this deep-link
  path and the normal picker-driven path, so there's one copy of each form.

**Username+password is an alternate GM login, not a replacement for tokens.**
A GM's `gm_registry` entry can carry `username` + `passwordHash`/`passwordSalt`
(PBKDF2, 100k iterations, via Workers' built-in `crypto.subtle` — no external
dependency) alongside their existing `token`. Deliberately called `username`,
not `email` — there's no email-sending infrastructure in this project, and
naming it `email` would have implied verification/recovery capabilities that
don't exist. `POST /fantasy/gms/login` (public — username+password) returns
that same `token` on success, so every other GM-authenticated endpoint
(`pick`, `board`, etc.) needs zero changes; password login is just a nicer
front door onto the identical session. The shareable token link still works
untouched and is the ultimate fallback.
- `POST /fantasy/gms/reset-password` (commish auth, body `{gmId}`) generates
  a new auto-generated `word-word-number` password (e.g. `swift-tiger-42` —
  easy to read aloud/text) and returns it **once**, for the commissioner to
  relay out of band. No email-sending infrastructure exists or is needed —
  this is the entire recovery story, from the GM Roster panel's "Reset
  Password" button in `live-draft.html`. The revealed password shows inline
  in that GM's row (`.pw-reveal-row`, transient client-side state on
  `commishGmRows[i].revealedPassword` — never persisted or sent back to the
  Worker) with a Copy button (same clipboard-then-select-text fallback
  pattern as `copyGmLink`) and a Hide button, rather than a plain `alert()`,
  specifically so it can be copied in one click.
- `POST /fantasy/gms/change-password` (GM-token auth, body `{gmId,
  currentPassword, newPassword}`) is GM self-service, from `my-account.html`
  — requires knowing the current password (whatever the last reset issued),
  not just an active session, so an unlocked device alone can't take over the
  account. Min 6 characters, no other complexity rules (small friend-group
  hobby app, not worth the UX friction).
- Password hashes never leave the Worker in any API response, including to
  commissioner tooling — `GET /fantasy/gms?full=1` returns `hasPassword`
  (boolean) instead. Because of that, `PUT /fantasy/gms` (the roster editor's
  save) merges each incoming row against the *previously stored* entry by
  `id` to preserve `passwordHash`/`passwordSalt` server-side — the client
  literally cannot round-trip a hash it was never given, so without this
  merge, saving the roster for an unrelated reason (e.g. adding a new GM)
  would silently wipe everyone else's password.
- `my-account.html` — new standalone GM-only page (same login pattern as
  `my-board.html`: direct `?gm=&token=` link, shared `localStorage` session,
  or a manual login form with both password and token options) for changing
  your own password. Reachable via the GM Tools dropdown in `index.html`.

**Turn order is derived, never stored** — `computeTurn()` in `worker.js`
recomputes the on-the-clock GM from `picks.length` and `draftOrder` (snake:
reverses every round) on every request. There is no `currentPick` field to
desync from the picks array.

**Key endpoints** (see `worker/worker.js` for the full list): `GET
/fantasy/livedraft/:year` (public, includes derived `onTheClock`/`round`/
`pickNumber`/`isComplete`/`onDeck`), `POST .../start|pause|resume|undo`
(commish), `POST .../pick` (GM token — validates turn order, player-pool
membership, and de-dupes via a client-generated `pickRequestId` so a
retried/double-submitted request replays the same result instead of erroring
or double-picking).

**Race conditions**: KV has no compare-and-swap. Given a small trusted GM
group and turn-gating already limiting writes to one authorized GM at a time,
this is handled pragmatically rather than with Durable Objects — idempotent
`pickRequestId` replay covers accidental double-submits, and commissioner
`undo` is the manual fallback for the rare residual case. Both `live-draft.html`
and `draft-presentation.html` poll every 2-3s, so a bad pick would surface
almost immediately.

**Player pool**: `live-draft.html`'s Player Pool panel updates
`players_<year>` (`PUT /fantasy/players/:year`), primarily via uploading a
registrations CSV export — see `handlePoolFileUpload()`/`diffPlayerPool()`
in `live-draft.html`, which diffs the upload against the currently saved
pool and only adds/updates what actually changed, never silently dropping
an existing player (a raw full-JSON-paste box still exists under "Advanced"
for deliberate edits/removals). `GET /fantasy/players/:year` is public (no
auth), which is what lets a GM's own draft board (below) work before the
draft has even started — and, since `scouting.html` fetches this same
endpoint (see "The player pool is fetched from the Worker, not hardcoded"
above), it's also what keeps the scouting list in sync automatically now,
with no separate step.

**GM draft boards**: each GM can pre-rank a private wishlist of players from
their own `live-draft.html?gm=...&token=...` link, usable both before the
commissioner starts the draft and during it. Stored server-side (not
`localStorage`) at KV key `board_<year>_<gmId>`, authenticated the same way
picks are (`X-GM-Token`), via `GET/PUT /fantasy/livedraft/:year/board` — this
is deliberate so the board follows the GM's link to whatever device they
actually draft from, not just the device they built it on. It's never
included in the public `/fantasy/livedraft/:year` response (that's read by
every GM and the broadcast view), only fetchable with that GM's own token, so
one GM's strategy stays invisible to the others. When it's that GM's turn,
board rows for still-available players get an inline Draft button, so the
board doubles as a fast-pick tool, not just a reference list. Every board
row's name (`renderBoardRow()` here, and `my-board.html`'s own equivalent
row markup) is a link to `scouting.html?player=<name>` opened in a new tab —
`scouting.html`'s `init()` reads that `?player=` param and auto-opens the
player's modal once data loads (`PLAYERS.find(...)`, falling back to an
inline "not in this year's scouting list yet" status note rather than
silently doing nothing if the name isn't an exact match — a real
possibility now that the CSV pool-merge tool can add a player to
`players_<year>` without `scouting.html`'s separately-maintained `PLAYERS`
array being updated too). Scouting's modal, not `draft-history.html`'s, is
the deliberate target here — draft-history's modal only has data once a
year's draft is finalized (see "Draft History Is Sourced From This Worker"
below), which is exactly when a GM is *not* using their board; scouting's
modal works pre-draft since it's keyed off historical/registration data,
not that year's fantasy results, and is already board-aware (the same
Add/Remove toggle used elsewhere in it appears when a logged-in GM opens a
player this way).

On `live-draft.html` itself, the board renders in two places sharing one
`renderBoardRow()` helper so they can't drift apart: an inline preview
(top `BOARD_PREVIEW_SIZE` players, no reorder/remove — just Draft when
eligible) so it doesn't dominate the main draft screen, and a "See Full
Draft Board" button opening the complete ranked list (with reorder/remove)
in a modal. The modal (`#board-modal-bg`) is deliberately placed **outside**
`#app` in the DOM, not inside it — `render()` replaces `#app`'s entire
`innerHTML` on every poll/pick/board edit, which would otherwise blow away
the modal's open/closed state each time. Because it lives outside `#app`,
nothing refreshes its content automatically, so `render()` explicitly calls
`renderBoardModalContent()` at the end whenever the modal is open, and
`isInteractingWithForm()` (see the polling note above) checks both `#app`
*and* `#board-modal-bg` for focus — without that second check, a background
poll could destroy the modal's own "add player" `<select>` mid-click, the
exact bug that check exists to prevent elsewhere. This replaced a plain
"Open as Standalone Page" link out to `my-board.html`, which is still
reachable on its own via the GM Tools dropdown or `scouting.html`'s hero
pill — it just isn't linked from `live-draft.html` directly anymore.

A GM's session (`{gmId, gmToken, gmName, year}`) lives in `localStorage` under
the key `livedraft-gm`, set on login in `live-draft.html` or `my-board.html`.
Because `scouting.html`, `live-draft.html`, and `my-board.html` are all
same-origin, this session is shared across all three without any extra
plumbing — logging in on one logs you in on the others. `scouting.html`'s
per-player modal reads this session (read-only glance, no login flow lives
there) and shows an "Add to My Draft Board" / "Remove" toggle scoped to
whichever GM is logged in, so a GM can build their board while looking at the
real stats, not just names. The same toggle also has a compact one-click
equivalent right in the table — a small circular `+`/`★` button
(`renderBoardIcon()`) next to each player's name in the `player` column,
for adding without opening the modal every time. Both controls share the
same `gmBoard` array and stay in sync with each other: `toggleBoardIcon()`
(row → modal) and `toggleBoard()` (modal → row) each patch the *other*
control in place via `outerHTML`/`CSS.escape`-scoped `querySelector`,
rather than a full `renderTable()`, so toggling doesn't reset sort/scroll
position or require a page-load-order-safe re-render like the initial
icon appearance does (see `loadGmBoardSession()`'s trailing `renderTable()`
call, needed because it races the main Nats-history fetch in `init()`).
`my-board.html` is the same board panel as `live-draft.html`'s (full
add/reorder/remove), minus the Draft button and turn-order UI, for GMs who
want their board open separately from both the stats table and the draft
itself — it also accepts a direct `?gm=...&token=...` link, not just the
shared session, so it works standalone on a fresh device. Reachable via
`index.html`'s GM Tools dropdown.

**"Scouting Portal" and "My Draft Board" in `scouting.html`'s header are a
view toggle, not navigation** (the gold pill used to be a link to
`my-board.html` — it isn't anymore). `currentView` (`'scouting'` | `'board'`)
picks which row set `renderTable()` builds, but both views share the exact
same `COLUMN_DEFS`-driven header/row markup, so switching feels like the data
changing under an otherwise-identical table rather than a different page:
- Board mode filters `PLAYERS` down to `gmBoard` (in board order, not
  re-sorted — the whole point of the board is manual ranking) instead of the
  usual search/day/format filters, and skips the sort-column logic entirely
  in favor of `gmBoard`'s own order. The search box still applies (handy once
  a board gets long); the day/format/history/draft/sort filters are quietly
  ignored rather than hidden, to avoid extra DOM-toggling complexity.
- Two columns get bolted onto the same `activeColumns` render pass: `#`
  (rank) and `Actions` (↑/↓/✕, plus a Draft button when this GM is on the
  clock in an active draft — same idea as the board panel elsewhere, but
  this page never polls, so `boardLivedraft` is fetched fresh only when
  `switchToBoardView()` runs, not kept live).
- There's no separate "add player" control in board view — adding still only
  happens via the row `+`/`★` icon over in Scouting Portal, by design: the
  toggle's whole framing is "browse & add in Scouting, review & rank in My
  Draft Board," not two competing ways to add.
- Toggling a player on/off the board from *within* board view (the ★ icon or
  a modal opened from a board row) needs a full `renderTable()`, unlike the
  lightweight `outerHTML` patch used in scouting view — removing a board
  member here means the row disappears and every rank below it shifts, which
  a single-button patch can't do. Both `toggleBoardIcon()` and `toggleBoard()`
  branch on `currentView === 'board'` for this.

**Resetting a draft**: `POST /fantasy/livedraft/:year/reset` (commish auth)
hard-resets `livedraft_<year>` back to the pre-draft default (no draft order,
roster size, player pool, or picks) — wired to a "Reset Draft to Pre-Draft"
button in the commissioner's Danger Zone panel, gated by typing the year to
confirm. It deliberately does **not** touch `gm_registry` or any
`board_<year>_<gmId>` entries, so the commissioner can rehearse a full mock
draft on the real year with the real GM links, then reset cleanly right
before the actual event without invalidating anyone's link or wiping the
boards they built during the rehearsal.

**Draft finalization**: `POST /fantasy/livedraft/:year/finalize` (commish
auth) is a permanent, one-way lock on `livedraft_<year>` — sets
`state.finalized = true` (409s if the draft isn't complete yet, or is
already finalized). Wired to a "Finalize Draft" card in the commissioner
view, shown once `ld.isComplete`, gated the same type-the-year way as Reset.
Once finalized, `/undo`, `/reset`, and the raw `PUT /fantasy/livedraft/:year`
all reject with 409 (`worker.js` — search `state.finalized`); `/pick` is
already blocked once `status !== 'active'`, which finalizing doesn't change,
but it double-checks the flag anyway for clarity. **There is deliberately no
`/unfinalize`** — a genuine post-event correction is a manual KV edit via
`wrangler`, outside the app on purpose, not an in-app undo. Sequencing:
finalize happens once, right after the draft completes and before any
scoring — it's unrelated to and doesn't touch the separate per-day
`livescore_<year>` finalize/unfinalize in the Live Scoring system below.

Finalizing is also the trust boundary for `GET /fantasy/livedraft/:year/history`
(public) — see "Draft History Is Sourced From This Worker" further down —
which 409s until `state.finalized` is true, so a draft's roster/scores only
ever surface as history once the commissioner has explicitly locked them in.

**Deploying worker changes**: `cd worker && wrangler deploy` (manual, no CI —
matches how the rest of this project deploys).

## Live Scoring system

A manual, round-by-round scoring tracker for Nationals weekend itself —
separate from (but complementary to) the Live Draft system above, sharing
the same `worker/` Worker and `FANTASY_DB` KV namespace. Built because there
was no reliable way to pull live results automatically from either a
spreadsheet or the tournament host's own site, so a human (a "scorekeeper,"
not necessarily the site owner) enters results into this tool as they
happen, regardless of where the official record lives.

`live-scoring.html`'s login screen used to have a "Just Watching?" option
below the scorekeeper passphrase field, mirroring the equivalent link on
`live-draft.html`'s landing screen — removed once `index.html`'s top-level
Live Scoring tab started pointing directly at `scoring-presentation.html`
(see `index.html` above), since `live-scoring.html` itself is now only ever
reached via Commissioner Tools → Scorekeeping, where a spectator option
doesn't belong. Spectators use the top-level tab; this page is
scorekeeper-only.

**Scoring model** — `scoring_config` (`GET /fantasy/config`, previously dead
scaffolding from the earlier scrapped project attempt, now live):
```js
{ win: 3, timeoutWin: 2, timeoutTie: 1.5, timeoutLoss: 1, loss: 0, dnp: 0, rosterSize: 9, countPerDay: 7, ... }
```
A scorekeeper picks a player and a round result (Win/Timeout Win/Timeout
Tie/Timeout Loss/Loss/Did Not Play — Redemption auto-awards byes as a full
Win, so there's no separate bye option); the point value is looked up
server-side, never trusted from the client. Spot-checked against real 2025
historical data (`db.matches` + known `breakdown[].pts`) and confirmed to
reproduce the exact known totals once true draws (`winner: null`) are
correctly read as a Timeout Tie rather than a loss.

**Entries are per-round, not just per-player** — each entry carries a
`round` number (`POST .../entry` requires `{day, player, round, result}`),
and the Worker rejects (409) a second entry for the same player+round —
correcting a result means removing that entry and adding a new one, same
create/delete-only pattern as everywhere else in this app, never an in-place
update. `dnp` (Did Not Play, 0 pts by default, same as `loss`) is a distinct
result from `loss`, specifically for a legitimate drop/no-show — it fills a
round slot honestly (a real loss didn't happen) while still counting as
"accounted for" by the round-completeness check below.

**Expected round counts, and the hard-block on finalize**: `livescore_<year>`
carries a `roundsByFormat` map (`{BD: 8, T2: 6, ...}`), set live by the
scorekeeper per format (`PUT .../rounds`, body `{format, rounds}`, merges
into the existing map rather than requiring all six every time) once that
format's field/pairings are known — usually right before that round starts,
same moment as entering scores, which is why this isn't commissioner-only
config set in advance. `computeMissingRoundsForDay()` (duplicated in
`worker.js` and `live-scoring.html`, same reason other small helpers are
duplicated per-file in this project) compares each rostered, day-registered
player's filled round numbers against `roundsByFormat[format]` — a format
with no round count set yet also counts as incomplete (`expected: null`),
since "unknown" can't be told apart from "missing" otherwise. `POST
.../:day/finalize` is a **hard block**: it re-runs this check server-side
(authoritative, not just a client-side disabled button) and 409s with the
full list of incomplete players if anything's missing. There's deliberately
no override/force-finalize path — the `dnp` result above is the intended
escape hatch for a day that can't otherwise reach 100% completion.

**Daily team total = sum of only the top `countPerDay` (7) of a team's 9
rostered players that day** — the bottom 2 are dropped, recomputed fresh
each day. A roster member with no entries that day counts as 0, same as a
bad performance; this is deliberate so players who only attend some days
don't inherently hurt a team, as long as 7 others are producing.

**Rosters come from the Live Draft system** (`livedraft_<year>`'s completed
`picks`, grouped by `gm`) — this feature assumes the year's draft was run
through `/fantasy/livedraft`, not a separately-defined roster. `players_<year>`
supplies each player's thu/fri/sat format registration, used to determine who's
eligible to score on a given day and which format their day's points count
toward.

**Auth is a separate credential from the commissioner key** — a
`SCOREKEEPER_KEY` Worker secret (`X-Scorekeeper-Key` header, same
`wrangler secret put` process as `COMMISH_KEY`), deliberately independent so
whoever's keeping score doesn't get commissioner powers over the live draft.
There's no dedicated login-verification endpoint; a wrong key is only
discovered on the first real write, surfacing as a 401 that forces
`live-scoring.html` back to its login screen.

**KV schema** — `livescore_<year>`:
```js
{
  thu: { status: 'active'|'final', entries: [ {id, player, round, result, pts, ts} ], finalizedAt },
  fri: { ... }, sat: { ... },
  roundsByFormat: { BD: 8, T2: 6, T1: 10, TA: 6, SD: 8, Teams: 5 },
}
```
`GET /fantasy/livescore/:year` (public) joins this with `livedraft_<year>`
and `players_<year>` and returns the full computed rollup
(`teams[].dayTotals`, `seasonTotal`, plus `roundsByFormat` passed through) —
this is centralized server-side in `computeLivescoreState()` so
`live-scoring.html` and `scoring-presentation.html` don't each reimplement
the top-7-of-9 math. `POST .../entry` (scorekeeper), `DELETE .../entry/:id`
(scorekeeper — corrections matter more here than in the draft, since live
scorekeeping under time pressure produces more mistakes than the slower,
deliberate draft did), `PUT .../rounds` (scorekeeper — see above), and
`POST .../:day/finalize|unfinalize` (scorekeeper) round out the endpoints.
`buildRosterList()` (joins `livedraft_<year>.picks` with `players_<year>`
into one `{name, thu, fri, sat}` list per rostered player) is shared between
`computeLivescoreState()` and the finalize hard-block, so there's one
definition of "who's on a roster" for scoring purposes.

**Entry screen is two columns, one per that day's concurrent format** — each
day always runs exactly two formats side by side (`DAY_FORMATS` in
`live-scoring.html`: Thu = Booster Draft/Type 2, Fri = Type 1/Type A, Sat =
Sealed/Teams, same pairing as `FMT_CODE_TO_DAY` in `worker.js`), so
`renderFormatColumn()` renders one column per format, each with its own
player search (`state.searchByFormat`, keyed by format code) and its own
inline round-count input/Set button. A player row collapses to one line
(`X/Y rounds`, ⚠ if incomplete or the format's round count isn't set) and
expands (`renderPlayerRoundRow()`, click anywhere on the row) into one row
per expected round — filled rounds show the result + a Remove; an empty
round shows the six result-choice buttons directly, no extra step. This
replaced a single flat searchable list mixing both formats together with no
round awareness at all.

**`scoring-presentation.html`'s ticker** cross-references live entries
against `records.js`'s all-time `formatHighScore` thresholds (grouping a
day's entries by each player's registered format for that day) to surface
"closing in on / matched the all-time record" callouts, alongside a live
individual point leader and (once a day is finalized) a recap of who led
it. The historical `pts` values and this live formula were confirmed to come
from the same underlying scoring model, but treat exact-value matches as
approximate, not guaranteed identical.

`live-scoring.html` reuses `live-draft.html`'s focus-preservation `render()`
pattern (snapshot the focused element's value/selection before an
`innerHTML` swap, restore after) plus model-syncing the player-search input
via `oninput` — without both halves of that fix, the 3s poll loop steals
focus and blanks the search box mid-keystroke, exactly like the bug already
hit and fixed in the draft's commissioner GM-roster editor.

**Four related bugs found during pre-event testing, all variations on the
same root cause** (something rebuilds part of the DOM from stale state while
the user is mid-interaction) **— fixed in `live-draft.html` and
`my-board.html`:**
- The GM player-search box wasn't actually model-synced (unlike
  `live-scoring.html`'s, which was correct from the start) — `renderGmView()`
  called `renderPlayerRows(available)` with no filter, so every 3s poll
  quietly replaced the filtered list with the full unfiltered pool while the
  search box still showed the typed text, unnoticed until a GM drafted
  whatever the top row now was. Fixed by adding `state.search`, using it in
  both the template's `value=` and the `renderPlayerRows` call, so the
  correct filtered list renders no matter what triggered that render.
- A background poll can also destroy a `<select>` while its native dropdown
  is open (e.g. the commissioner's draft-order picker), which reads as the
  selection/cursor randomly glitching. Fixed generically: `fetchLivedraft()`
  (and `my-board.html`'s poll) takes an `isBackgroundPoll` flag and skips the
  render — not the fetch, so data still stays fresh — whenever
  `document.activeElement` is an `INPUT`/`SELECT`/`TEXTAREA` inside `#app`.
  Direct, user-triggered renders (clicking a button, selecting a GM,
  submitting a pick) are unaffected and still render immediately; only the
  timer-driven background tick defers. If you add another poll loop or
  another dropdown/text field to either file, this guard already covers it —
  no per-widget patching needed.
- The commissioner's GM Roster editor's `id`/`name` inputs each got an
  `oninput` handler writing straight to the working `commishGmRows[i]` row so
  a re-render reflects what's actually typed — but the `username` field (once
  added for password login) was missed. Any *direct* action-triggered
  render() — clicking "+ Add GM", "Reset Password" on any row, anything — not
  just a background poll, rebuilds that input from the stale `r.username`
  and silently discards whatever was typed but not yet saved. This one isn't
  poll-timing-specific like the two above; it happens on the very next
  render() regardless of cause, since nothing was capturing the typed value
  at all. Fixed the same way as `id`/`name`: an `oninput` writing to
  `r.username`. If you add another editable field to a GM row, it needs this
  same handler, or it has this bug from day one.
- Three scrollable lists — `live-draft.html`'s board modal `.board-list`
  and `#player-list` (Available Players), plus `my-board.html`'s own
  `.board-list` — reset to `scrollTop: 0` on every poll tick (3s in
  `live-draft.html`, 5s in `my-board.html`), reading as "scrolling down snaps
  back to the top after a few seconds." Replacing an element's `innerHTML`
  always resets its scroll position, and scrolling isn't "interacting with a
  form" so the `isInteractingWithForm()` guard above doesn't help here — it
  only skips a render entirely, it doesn't make an *executed* render
  scroll-safe. Fixed by snapshotting `scrollTop` right before the `innerHTML`
  rebuild and restoring it right after, in `live-draft.html`'s `render()`
  (for `#player-list`), its `renderBoardModalContent()` (for
  `#board-modal-body` *and* the `.board-list` nested inside it — two separate
  scroll containers, both need it), and `my-board.html`'s own `render()` (for
  its `.board-list`). Any new scrollable list rebuilt on a timer needs this
  same snapshot/restore, same as any new form field needs the
  model-sync/`oninput` fixes above — this bug recurred a second time
  specifically because `my-board.html` has its own independent poll loop
  that the first round of fixes didn't touch.

**Drafting a player requires confirming a native `confirm()` dialog** ("Draft
{name}?") in `submitPick()` — covers both the main available-players list and
drafting straight from a GM's board, added after a mock draft produced a
misclick under the search bug above. Keep this even now that the search bug
is fixed — it's cheap insurance against fat-fingering the wrong row during a
live event.

## Draft History Is Sourced From This Worker, Verified Against Official Results

For years run through the Live Draft/Live Scoring systems above, the
`tournaments[].fantasyDraft` data `draft-history.html`, `scouting.html`, and
`records.html` render no longer has to be hand-transcribed into the separate
`nationals-history` API after the fact (that was the old workflow, still
true for the hardcoded 2024/2025 seasons that predate this).

- `worker/worker.js`'s `computeFantasyDraftHistory(env, year)` (called by the
  public `GET /fantasy/livedraft/:year/history`, 409 until the draft is
  finalized — see "Draft finalization" above) joins `livedraft_<year>.picks`
  (roster + `draftPick`), `gm_registry` (id → display name), and
  `livescore_<year>` + `players_<year>` (per-player, per-format point
  breakdown, plus the team `pts` total using the same top-`countPerDay`-of-roster
  rule as `computeLivescoreState`) into exactly the shape
  `records.js:51-55` already expects from a hardcoded year.
- `fantasy-history.js` (new shared module, same `window.X` IIFE pattern as
  `records.js`, loaded by all three consuming pages) has
  `mergeWorkerDraftHistory(db, year, apiBase)` — fetches that endpoint and,
  if it 200s, writes the result into `db.tournaments[]` for that year before
  `RecordsBook.compute(db)`/`processFantasyDraft(db)` run. Fails soft (not
  finalized yet, network error) — same fail-soft convention as every other
  Worker call in `scouting.html`.
- Each of the three pages defines its own `FANTASY_DRAFT_YEAR` constant near
  the top (next to `FANTASY_API_BASE`/`NATS_URL`/`API`) — **bump this every
  year** once that year's draft is live in `live-draft.html`. This is the one
  year sourced from the Worker; every other year keeps coming from
  `nationals-history` untouched.

**Official-results verification**: `fantasy-history.js`'s
`reconcileWithOfficialResults(db, year)` runs right after the merge above.
Once `nationals-history` has that year's official match results loaded (a
separate, later, manual step — see "Data source" up top), it recomputes each
player's per-format point total straight from `db.matches` and compares it
to the value already in `breakdown[]`. A match makes the official number
canonical (`breakdown[].verified = true`); a real disagreement leaves the
hand-entered value in place with `verified: false` + `officialPts`, which
`draft-history.html`'s `breakdownEntryHtml()` renders as a ⚠ flag with both
numbers rather than silently trusting either one. Validated against every
real 2024/2025 `breakdown[]` entry (145/145 exact match) before shipping —
the recompute:
- infers Win/Timeout Win/Timeout Tie/Timeout Loss/Loss from score vs. each
  format's target score (5 for T1/Teams/Type A/Booster Draft/Sealed, 7 for
  T2) rather than an explicit field — a winner reaching target is a clean
  Win/Loss, below target is a Timeout Win/Loss, no `winner` at all is a
  Timeout Tie for both;
- **excludes Top Cut matches** (`match.topCut === true`, null scores) —
  Top Cut only happened in 2024 and was removed after that, so it's a
  one-time historical fixture, not modeled in the ongoing engine (per John:
  each bracket win was a flat 3 pts, plus +3 for the official 3rd-place game
  winner, if that year's breakdown is ever spot-checked by hand);
  Top-Cut/bonus `breakdown` lines are left untouched (`verified` stays
  unset) rather than flagged, since this recompute can't evaluate them;
- **dedupes Teams (doubles) format's duplicate match rows** — the same
  physical game gets logged once per teammate as `playerA`, both rows
  carrying an identical score, so rows sharing a `round` collapse to one;
- **detects byes** (no match row at all — Redemption auto-awards a bye as a
  full Win) by comparing a player's round count against the max round
  number seen across the whole format+year, crediting one Win per gap.

Team `pts` is left as whatever `mergeWorkerDraftHistory` computed (already
correct via the top-N-per-day rule) — reconciliation only touches
player-level `breakdown`/`pts`, on the view that a real disagreement is
exactly what the ⚠ flag exists to surface for manual resolution, not
something to silently rebalance into the team total.

## Working on this project

No build step — edit the HTML files directly, then verify in a real browser
before pushing (Playwright + a local `python3 -m http.server` works well; the
scouting page needs to be served over http:// for its `fetch()` to behave
consistently, though it does work in most cases over file:// too). Push to
`main` to deploy; GitHub Pages typically takes 1-2 minutes to update.

---
> Source: [jhendrix6426/FantasyDraft](https://github.com/jhendrix6426/FantasyDraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
