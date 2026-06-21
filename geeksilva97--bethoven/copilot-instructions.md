## bethoven

> World Cup prediction pool played **over SSH**. Players connect with `ssh`, pick

# CLAUDE.md — BEThoven

World Cup prediction pool played **over SSH**. Players connect with `ssh`, pick
scores in a Bubble Tea TUI, and compete on a leaderboard. A player's **SSH public
key is their identity** — no passwords, no signup forms.

## Stack

- **Go** (module `bethoven`, single static binary).
- **charmbracelet/wish** — SSH server. **charmbracelet/bubbletea** + **bubbles** +
  **lipgloss** — TUI. **charmbracelet/wish/bubbletea** — bridges a session to a Tea program.
- **modernc.org/sqlite** — pure-Go SQLite (no cgo), via `database/sql`.

## Architecture — the layering is the point

```
cmd/bethoven        entrypoint: config -> db.Open -> seed -> service.New -> server.New
internal/config     env config (BETHOVEN_*)
internal/clock      Clock interface: Real (prod) + Fake (tests). INJECTED everywhere time matters.
internal/models     pure domain types (no behaviour, no deps) — shared by store/scoring/service
internal/db         SQLite: Open, embedded schema.sql, Store (typed queries), seed.go
internal/auth       key fingerprint, admin allowlist, invite-code check
internal/scoring    PURE Points(bet, match) — the heavily unit-tested core
internal/live       OPTIONAL live feed: ESPN adapter + in-memory Cache + Poller. Behind the service.LiveStore port.
internal/analytics  OPTIONAL usage tracking: own SQLite DB + async Recorder. Behind the service.AnalyticsSink port.
internal/ai         OPTIONAL AI player "BETanIA": Claude-API predictor + live Bettor + seeder + Monitor. Behind the service.AIMonitor port + a trigger seam. NEVER imports service (function seams).
internal/service    ALL business rules. Depends only on Store + Clock (+ optional LiveStore/AnalyticsSink/AIMonitor). The integration-test seam.
internal/server     thin wish SSH server -> resolves key to user -> launches TUI
internal/tui        presentation only; every action calls a service method
```

**Golden rule:** business logic lives in `internal/service`. `server` and `tui`
are thin. If you're tempted to put a rule in the TUI, put it in the service and
call it from the TUI — that's how it stays testable (tests drive the service
directly, with a fake clock, no terminal).

## Core rules / invariants

- **Kickoff lock (most important):** you can only bet while `clock.Now().UTC() <
  match.StartsAt`. Enforced in `service.PlaceBet` using the **server clock only** —
  never trust client time. Also rejects `match.Finished` (belt-and-suspenders for
  clock skew / early result entry). Invariant: *cannot bet an ongoing or ended match.*
- **Own-bets-only (default):** player queries are scoped to their `user_id`;
  players never see another player's individual picks. The admin `AllBets` grid
  exposes raw picks and is gated by `requireAdmin` in the service (not just hidden
  in the UI). **Exception — `public_bets`:** an admin can flip the DB-backed
  `public_bets` setting (admin TUI → Settings) to open the grid to all players via
  `service.PublicBetsGrid`. Even then, picks are revealed **only for matches that
  have kicked off or finished** (`m.Finished || now >= m.StartsAt`) — upcoming
  matches are omitted entirely, mirroring the `MatchLeaderboard` reveal rule, so
  blind betting is preserved. Enforced server-side, not just in the UI.
- **Scoring — admin-selectable mode (`scoring_mode` setting; default Classic).**
  Three pure functions in `internal/scoring`, dispatched by `scoring.Score(mode, …)`:
    - **Classic** (default, `scoring.Points`): exact = 3; correct result (W/D/L)
      only = 1; wrong = 0. Tiers are **mutually exclusive** — exact is 3, not 3+1.
    - **Proximity** (`ProximityPoints`): 0 for a wrong result, else
      `max(1, 5 − distance)` where `distance = |predA−scoreA| + |predB−scoreB|`.
      Exact → 5, one goal off → 4, …, floor of 1 for calling the winner. (Kicktipp's
      "deduction per goal" model.)
    - **Scarcity** (`ScarcityPoints`): Proximity **plus** a contrarian bonus —
      +2 for a correct result <25% of the match's bets picked, +2 for a correct
      exact score <10% picked. **Quorum gate (`scarcityQuorum`, 8):** the bonus
      only applies once a match has ≥8 bets; below that "rare" is just noise (a
      lone picker in a 4-person field isn't a contrarian), so Scarcity scores
      identically to plain Proximity. **Pool-relative**, so it takes a
      `scoring.Pool` (counts of same-result / same-exact bets + total) the
      **service** computes; the pure function never reads the DB.
  Knockouts store the **regulation 90' score** in every mode, so ET/penalties are
  ignored and a 1-1 a.e.t. scores as a 1-1 draw.
  **Plumbing:** the service builds a `scorer` (`service_scoring.go`) once per
  read; it carries the mode and, **only for Scarcity**, the per-match pick
  distribution (built from `AllBets`). All point-bearing reads — `Leaderboard`
  (incl. the synthetic-match live fold), `MyResults`, `MatchLeaderboard`,
  `buildBetsGrid` — call `sc.points(b, m)`, never `scoring.Points` directly.
  Admin picks the mode in the TUI **Settings** screen; players see the active
  mode and its rules in **"How scoring works"** (`scoring_rules.go`). Mode is
  stored/read like `public_bets` (KV `settings` table; absent ⇒ Classic), so
  existing pools are unaffected until an admin opts in.
- **Leaderboard tiebreak.** Players tied on total points are ordered by
  **exact-score hits, then correct-result (W/D/L) hits, then display name** — the
  single comparator `betterRank` (`service_results.go`). The exact/result tallies
  are mode-agnostic (`scoring.IsExact`/`IsCorrectResult`) and computed in the same
  loop that totals points, so no new storage. The live board folds in-play
  provisional outcomes into the tiebreak (mirroring `Total`), while the
  settled-rank sort behind `LiveRankDelta` subtracts the live portion so the
  "nothing live ⇒ every delta 0" invariant holds. `StandingsHistory` (`rankUsers`)
  mirrors the same comparator on finished matches so BETanIA's comment movement
  matches the board. Per-match ranking, public-bets, and the admin grid keep
  name-only ordering — they aren't the overall leaderboard.
- **Identity = SHA256 key fingerprint.** Set once at registration, immutable after.
- **Live scores (optional, `internal/live`).** A background `Poller` fetches ESPN's
  keyless scoreboard (`fifa.world`), resolves each event to a stored match (by
  team-pair + date, with an alias map), and writes an **in-memory** `Cache`
  (`service.LiveStore` port). Nothing live is persisted — a restart re-fetches.
  The service folds **provisional** points into `Leaderboard` for in-play matches
  (via a synthetic finished match fed to the active-mode `scorer.points`) and
  overlays read-time `Live*` fields onto `models.Match` (the store never reads/writes
  them). **Auto-finalize:** `FinalizeFromFeed` writes the official result on a
  `post` event **only if `!Finished`**, so it never clobbers a settled result;
  admin `EnterResult` always overrides. Disable with `BETHOVEN_LIVE_ENABLED=false`
  (a nil LiveStore ⇒ behaviour identical to no feed). Picks stay hidden pre-kickoff;
  only the live score is shown, preserving blind betting. **The feed is untrusted**:
  `decodeEvents` rejects impossible scores (outside 0–99) and sanitizes the display
  clock to a digit/punctuation whitelist (`cleanClock`) — it renders into every
  player's terminal, so it's the same ANSI-injection boundary as display names
  (`cleanName`). Feed team names are only used for resolution, never rendered.
  **Odds:** `decodeEvents` also parses the per-match `odds` array (the priority-1
  provider's `details` + `overUnder`) into a short string (`cleanOdds`, sanitized
  like `cleanClock`), carried as `Score.Odds`/`Match.LiveOdds`. It's never rendered
  directly — it's grounding fed to BETanIA's live commentary (see below). Present
  pre-match and may drop in-play; absent ⇒ empty string, no behaviour change.
  **Key events (goals/cards):** for **in-play** events only, the provider also hits
  ESPN's per-match **summary** endpoint (`fetchKeyEvents`/`decodeKeyEvents`) and
  pulls `keyEvents[]` — the curated goal/card list — into `[]models.MatchEvent`
  (`Clock`/`Type`/`Text`/`Scoring`), carried as `Score.Events`/`Match.LiveEvents`.
  The scorer's name lives in the **prose `text`** (ESPN's structured athlete refs are
  empty for `fifa.world`), so it's sanitized with `cleanEventText` — a prose stripper
  mirroring `ai.sanitizeText` (whole-CSI strip + C0/C1 drop + length cap), since it
  reaches both terminals and the model. Capped at the most-recent `maxKeyEvents`; one
  extra ~350 KB GET per live match per poll, tolerated on failure. Fed into BETanIA's
  live commentary so it can name the scorer (see below).
  **Phase (halftime/ET/penalties):** the coarse `state` stays `in` at the interval,
  so `decodeEvents` also reads `status.type.name` and maps it via `ParsePhase` to OUR
  controlled label (`PhaseHalftime`/`PhaseExtraTime`/`PhasePenalties`; "" ⇒ ordinary
  play — never raw feed text), carried as `Score.Phase`/`Match.LivePhase`. The TUI
  shows **"HT"** instead of the stale stoppage clock (`45'+8'`) at halftime
  (`liveScore`), and the phase is fed to BETanIA so the live line reacts to the break.
  **Full ESPN feed reference: `docs/espn-api.md`** (endpoints, every field, what's
  used vs available, sanitizers).
- **Analytics (optional, `internal/analytics`).** A usage tracker behind the
  `service.AnalyticsSink` port (nil ⇒ disabled ⇒ behaviour identical to no
  analytics, mirroring the live feed). Enable with `BETHOVEN_ANALYTICS_ENABLED=true`
  (default **off**, opt-in — note the inverted env test vs `LiveEnabled`).
  **Two rules keep it a side concern that can never affect bets/scores:**
    - **Separate DB.** Events go to their own SQLite file
      (`BETHOVEN_ANALYTICS_DB_PATH`, default `analytics.db`) on a **separate
      connection**, so analytics writes never contend with the domain DB's
      single-writer lock. Losing it loses only usage history.
    - **Async, zero-cost emit.** `analytics.Recorder.Track` drops the event on a
      buffered channel and returns; a background goroutine does the insert. Full
      buffer ⇒ **drop** (counted), never block. The emit *helpers* in the service
      (`track` / `trackByID` / `Track`) do **NO domain-DB reads** — they capture
      only data already in hand; the actor's display name is resolved **at read
      time** (`AnalyticsRecent` / `AnalyticsPerPlayer`), on the admin's own
      session, never on the betting hot path.
  Events: `session_start` (server, every connect), `registered`/`bet_placed`/
  `result_entered`/`match_added`/`setting_changed` (service, after the rule
  succeeds), `view` (TUI, on the menu transition — **not** the leaderboard tick,
  which would inflate counts). Admin reads via gated `Analytics*` service methods
  (`requireAdmin`); the **⚙ Admin: analytics** TUI screen shows KPIs, an
  accesses/day sparkline, per-player engagement, and a recent-activity feed.
- **BETanIA — the AI player (optional, `internal/ai`).** An AI competitor
  (display name **"BETanIA 🤖"**, reserved fingerprint `bethoven:ai-betania` — the
  non-`SHA256:` prefix means it can never collide with a real SSH key, so it's a
  pure system account with no login). It predicts regulation-90' scorelines via the
  **Claude API** (`github.com/anthropics/anthropic-sdk-go`; model
  `BETHOVEN_AI_MODEL`, default `claude-sonnet-4-6`; `ANTHROPIC_API_KEY` read by the
  SDK) and competes on the **same leaderboard** as humans. Like `live`/`analytics`,
  the package **never imports `service`** — the `Bettor` takes function seams
  (`Deps{Fixtures, MyBets, PlaceBet, Now}`), mirroring `live.MatchesFunc`. Opt-in,
  off by default (`BETHOVEN_AI_ENABLED`, inverted env test like analytics; nil
  monitor/trigger ⇒ behaviour identical to no AI). **Two tracks, split by who runs
  them so the server stays thin:**
    - **Seed (one-time, the `bethoven ai-seed` CLI — `cmd/bethoven/ai_seed.go`).**
      Creates the player and backfills **already-played** matches (`now >=
      m.StartsAt`) with **web-search-OFF** predictions — pure pre-tournament
      knowledge, unbiased since the 2026 results post-date the model's training
      cutoff and there's no internet to look them up. Writes via **`store.UpsertBet`
      directly** — the **one sanctioned kickoff-lock bypass** (sibling of the
      `place-bet` escape hatch), justified because those games are already locked.
      **Idempotent:** skips matches BETanIA already bet (no API cost on re-run).
      User creation lives **here, in the CLI** — never in server startup.
    - **Live worker (in-server, gated by `BETHOVEN_AI_ENABLED`).** `ai.Bettor` runs
      on a timer (`BETHOVEN_AI_INTERVAL_SECONDS`, default 6h; fires once immediately)
      and bets **upcoming** matches through **`service.PlaceBet`** — so the **kickoff
      lock fully applies** (no exemption for the live path). Panic-recovered;
      per-match `context` timeout; a mid-pass kickoff race is recorded as `locked`,
      not an error. Uses the **basic** web-search tool (`web_search_20250305`, not
      the slow dynamic-filtering `20260209`) capped by `maxWebSearches`.
      **`BETHOVEN_AI_MAX_PER_RUN` caps bets per pass** (e.g. 4): since
      `service.Fixtures()`→`store.ListMatches` is `ORDER BY starts_at`, the worker
      bets the **N soonest unbet upcoming matches** soonest-first, so it picks "the
      next few games" each pass rather than the whole fixture list — and never
      misses an imminent one. **`BETHOVEN_AI_LOOKAHEAD_HOURS` (default 72) bounds
      the horizon**: it won't bet a match kicking off further out than the window,
      so it stays on the near-term slate instead of reaching days/weeks ahead.
      Because fixtures are chronological, the first match past the horizon ends the
      pass early; matches enter the window on a later pass as kickoff nears (so set
      lookahead ≥ interval, else a match could slip past unbet). The server only
      **resolves** BETanIA (`svc.Lookup`);
      if it's not onboarded it logs "run `bethoven ai-seed` first" and stays off.
  **Untrusted model output is sanitized** (`ai.sanitizeText`): the model's
  rationale/confidence render into the admin terminal and `ai_bets.log`, the same
  ANSI-injection boundary as display names — `sanitizeText` strips whole CSI escape
  sequences + C0/C1 controls (strip-don't-reject, like `cleanClock`), applied at the
  prediction source so log and TUI both get clean text. Scores are clamped 0–99
  (`clampScore`, and again in `PlaceBet`). **Logging:** every pick (seed + live) is a
  JSON line in `BETHOVEN_AI_LOG_PATH` (default `ai_bets.log`; **must** be under the
  systemd `ReadWritePaths` dir in prod, i.e. `/opt/bethoven/data/ai_bets.log`) — not
  the DB schema. **Admin observability:** gated `Analytics`-style methods behind the
  `service.AIMonitor` port (`AIStatus`/`AIActivity`/`TriggerAI`, all `requireAdmin`).
  The **⚙ Admin: BETanIA** TUI screen is **tabbed** (`tab` cycles Betting →
  Comments → Usage). The **Betting** tab shows model/schedule/last+next run/totals plus
  **"Picks on record"** — BETanIA's bets sourced from the DB via
  `service.AIBets` (admin only), so they **survive a restart**, unlike the volatile
  in-memory `AIActivity` ring (which only fills as the worker *places* a bet this
  process lifetime and is wiped on restart — that was the "Recent picks: nothing yet"
  bug). When this session's `AIActivity` has the rationale for a pick, it's overlaid
  under the row. **`r` triggers an immediate pass**
  (`service.TriggerAI` → `Bettor.Trigger`, a non-blocking coalesced channel send).
  Live bets emit the `bet_placed` analytics event; the seed (writing straight to the
  store) does not.
  - **Token usage / estimated cost — the Usage tab (`internal/ai/usage.go`).** Every
    Claude call (bets via `Predict`, comments via `runTool`) records one JSON line —
    `{category, model, calls, input/output tokens, web_searches, latency_ms}` — to a
    **persistent `ai_usage.log`** (the `*ai.UsageLog` shared by predictor + commenter;
    sums usage **across the agentic loop**, so a multi-call web-search pick is one
    line). Path is `BETHOVEN_AI_USAGE_LOG_PATH`, defaulting **beside `ai_bets.log`**
    (`filepath.Dir(AILogPath)/ai_usage.log`) so prod's data-dir log path carries it
    automatically — no new env var, binary-only deploy works; **must** be under the
    systemd `ReadWritePaths` dir in prod like the other AI logs. Categories: `bet`
    (seed + live), `comment` (the two-call per-player pass), `live`. The admin
    **Usage** tab reads `service.AIUsage` (`requireAdmin`, behind the
    `service.AIUsageSource` port) which calls `UsageLog.Report()` → the pure
    `aggregateUsage` to roll the log up **at read time** by category + grand total
    with an **estimated USD cost** (editable per-Mtok `modelPrices` table + web-search
    rate; unknown models flagged, cost under-counted not wrong) and **avg latency**.
    Because it's log-backed it **survives restarts** — the whole point, unlike the
    volatile monitor rings. Latency uses wall-clock `time.Now()` inside `ai` (pure
    observability, never touches bets/scores — outside the injected-`Clock` rule).
  - **Leaderboard commentary ("roasts") — a second worker (`ai.CommentWorker`).**
    Gated by `BETHOVEN_AI_COMMENTS_ENABLED` (default **true** once `AIEnabled`),
    on a timer (`BETHOVEN_AI_COMMENT_INTERVAL_SECONDS`, default 6h; fires at
    startup; that interval is also the comment TTL). It writes **one short
    second-person comment per player**, grounded in *ranking narratives*, via a
    **two-stage Claude pipeline, no web search**: stage 1 `DetectNarratives`
    (closed `narrativeTypes` vocabulary, "never invent facts") → stage 2
    `WriteComments` in the active tone. Each line is **second person** ("you…")
    except **BETanIA's own row, which is first person** ("I'm…") — the worker is
    given her display name (`self`, from the resolved AI user) and the prompt flips
    that one line. Like the rest of `ai`, it takes function seams
    (`CommentDeps{History, Tone, Now}`) and **never imports `service`**.
    - **No new persistence for scores/positions — those are never stored.**
      `service.StandingsHistory` reconstructs the per-matchday standings series
      (positions + `Movement`/`PointsGained` deltas) by folding FINISHED matches by
      UTC kickoff date — the same pure computation `Leaderboard` already does live,
      just per round.
    - **Comments ARE persisted (the `leaderboard_comments` table).** The in-memory
      `ai.CommentCache` stays the hot read path, but it's a **write-through** cache:
      `pass()` mirrors the full set to the DB via the `CommentDeps.SaveComments` seam
      (`service.SaveComments` → `store.ReplaceLeaderboardComments`, a one-tx delete+insert
      so a dropped player leaves no stale row), and `RegenerateOne` upserts one row via
      `SaveComment`. At boot `main` calls `cache.Replace(svc.LoadComments())`, and
      `CommentWorker.Run` **skips the initial regeneration pass when the cache is
      non-empty** (`cache.Empty()`) — so a restart/deploy no longer re-spends tokens
      recreating unchanged comments (no match settles while the process is down, so they
      aren't stale). Fresh state (empty table) still fills the board once; the next
      `onMatchSettled` trigger and admin regen still regenerate + re-persist. Best-effort:
      a persist fault logs and never blanks the cache. One row per user holds the latest
      comment; `ExpiresAt` is zero (per-player comments never expire on a clock), stored
      as empty text. This is a SEPARATE concern from the volatile live-commentary KV
      (`live_comment_state`) — that one is intentionally left as-is.
    - **Visibility** in `service.LeaderboardComments`: with the cycle off, **everyone
      sees only their own** comment (under their own leaderboard row). The full set is
      exposed by **`service.AllLeaderboardComments`** — **NOT gated**: comments are a
      shared, fun feature any player can cycle through (bets stay private — that's a
      separate boundary). The TUI renders the comment beside the row in a right-hand
      column (`leaderCommentCol`), width-capped (`leaderCommentMaxWidth`) so it wraps into
      a squarer, readable block, falling back to stacking under the row on a narrow
      terminal, `helpStyle`, prefixed 🤖 (`internal/tui/results.go`). **Per-viewer
      opt-out:** **`h`** toggles a persisted "hide BETanIA comments on my leaderboard"
      preference (`LeaderboardCommentsHidden`/`SetLeaderboardCommentsHidden`, KV
      `lb_comments_off_u_<id>`) — distinct from `mute` (which stops BETanIA commenting ON
      a player). When hidden, no comment shows and the cycle doesn't run.
    - **Comment cycle (leaderboard, ON by default for everyone).** The board rotates
      which single player's comment is shown — own first, then **another at random** —
      auto-advancing in place every `cycleRefresh` (12s, paced for reading) via a
      `cycleTickMsg` loop (own `cycleEpoch`, self-stops on leave/toggle-off/superseded
      epoch, mirroring `leaderTick`). It pulls the full non-muted set from
      `AllLeaderboardComments`. **`c`** toggles it off → the board falls back to the
      viewer's own comment. **`space`** (`cycleNext`) steps MANUALLY to the **next**
      player in standings order, rotating last→first; it turns the cycle on if it was
      off and bumps `cycleEpoch` so the 12s auto-advance restarts from the manual step
      (a tap isn't instantly overridden). **Muted players are the sole exception**
      (`service.IsMuted`, cached as `selfMuted` on entry): no comment of their own and
      no cycle at all (space/`c` are no-ops for them, and when comments are hidden).
    - **Tone** is the DB-backed `comment_tone` setting (default `playful`, or
      `savage`; absent ⇒ playful), like `scoring_mode`. Toggled with **`t`** on the
      admin panel (`SetCommentTone`). **Per-player override:** each player can be
      `default`/`playful`/`savage`/`mute` via `comment_tone_u_<id>` settings
      (`SetUserCommentTone`; absent ⇒ inherit). **Mute** ⇒ that player gets **no
      comment at all, anywhere**, enforced at every surface so muting takes effect
      immediately without waiting for the next regeneration pass: the writer skips
      them, the worker never caches them, **`LeaderboardComments`/`AllLeaderboardComments`
      drop them at READ time** (so they never show on the board or in the admin cycle),
      and **`AICommentActivity` drops them at READ time** too (a pre-mute entry lingering
      in the in-memory ring never reaches the admin feed/detail). Edited with **`u`** on
      the panel (`screenAITones`).
    - **Rivalry / house context** (`x` → `screenAIContext`): a single
      `comment_context` JSON setting holds rivalry pairs (two user ids + note,
      resolved to names) and free-text house notes. Fed into the stage-2 prompt as
      *context, not instructions* (output still sanitized; admin-trusted but the
      prompt is explicit it never overrides "don't invent results"). Built into
      `ai.CommentConfig` (`{DefaultTone, Self, ToneByName, Rivalries, Notes, DerivedNotes, PromptOverride}`)
      by `service.CommentConfig` — the worker seam (replaces the old `Tone` seam).
      In the list, **`enter`** opens a row's **full untruncated content** (`ctxModeDetail`,
      wrapped) where **`e`** edits it in place (`EditRivalry`/`EditCommentNote` — a
      rivalry's two players stay; only the note text changes) and `d` deletes.
      Derived notes are read-only there (auto-generated — delete/compact instead).
    - **Auto-rivalries — BETanIA's self-managed rivalry tier (`comment_context.auto_rivalries`).**
      A SEPARATE tier inside the same `comment_context` blob, which the **admin tier
      never touches** and vice-versa. After a match **settles**, the same comment pass
      that refreshes the derived notes also runs `CommentWorker.refreshAutoRivalries`
      → `AnthropicCommenter.UpdateRivalries` (one no-web-search Claude call, usage
      category `digest`): it reads the standings (positions/movement/points from
      `StandingsHistory`) + the derived-note story + her CURRENT auto-rivalries, and
      returns the **full desired set** (declarative — add/update/delete all fall out of
      replacing it). The worker sanitizes notes (`sanitizeText`) and persists via the
      `CommentDeps.SetAutoRivalries` seam, then **re-reads `Config()`** so THIS pass's
      per-player lines already weave the new set in. `service.SetAutoRivalries` resolves
      names→ids (drops unknown players — "never invent"), keeps **pinned** entries
      verbatim, dedups unordered pairs against pinned + **admin** pairs (admin always
      wins), and caps the stored set. `service.CommentConfig` merges admin + auto into
      `cfg.Rivalries` (admin first), so **no comment-prompt change is needed** — the
      existing "riff on rivalries" instruction picks them up in both per-player comments
      and live commentary. Always-on once `AICommentsEnabled`; nil seams ⇒ tier off.
      **Admin CRUD on `screenAIContext`** (gated `AutoRivalriesView`/`EditAutoRivalry`/
      `DeleteAutoRivalry`/`PinAutoRivalry`/`ClearAutoRivalries`): `enter`→detail, `e`
      edits the note **and pins** (so the edit sticks), `p` toggles **pin** (a pinned
      rivalry is kept verbatim — BETanIA never drops/rewrites it; 📌 in the list), `d`
      deletes (a non-pinned pair may reappear next pass — pin to make removal stick),
      `R` clears the whole auto tier.
    - **Derived notes — BETanIA's auto "story of the game" tier (`comment_derived_notes`).**
      A SEPARATE tier from the admin's house `Notes`, never mixed. **One note per
      finished match** — a per-game diary. When a match **settles** — `EnterResult`
      (admin) or `FinalizeFromFeed` (feed, only on the poll that actually transitions
      it) — `service.onMatchSettled` fires the comment trigger (coalesced, like the
      admin "regenerate" key). That pass calls the worker's `refreshDerivedNotes`,
      which asks `service.PendingDigests` for finished matches **with no note yet** and
      makes ONE Claude call **per such game** (`AnthropicCommenter.DigestResults`, usage
      category **`digest`**, capped `derivedPendingCap`/pass as a burst backstop) to
      condense **that game's result + every player's pick + its live-commentary "story"**,
      then stores it via `AddDerivedNote(matchID, text)`. **Each game is noted exactly
      once** (`storedDerived.Done` tracks match ids). **No backfill:** the first pass
      (`Seeded`) adopts the already-finished slate as done and narrates only games
      finishing afterwards — so enabling/`Clear`ing mid-tournament never re-narrates the
      past. The combined stories (last `derivedNoteFeedCap`) feed the per-player prompt
      as a distinct tier (always appended, even under a prompt override). **The live
      story survives the game:** the live worker discards its in-memory lines the moment
      a match ends, so `matchDigestData` recovers them from the comment log
      (`ai.RecentLiveComments` reads `source:"live_comment"` lines logged since that
      match's kickoff — `service.SetAICommentLogPath` gives it the path).
      **Worker seams** (`CommentDeps.PendingDigests`/`DerivedNotes`/`AddDerivedNote`); the
      `ai` package still never imports `service`. **Admin curation** on `screenAIContext`
      (gated `DerivedNotes`/`DeleteDerivedNote`/`CompactDerivedNotes`/`ClearDerivedNotes`):
      `d` deletes the selected note, `c` **fuses** the whole diary into ONE
      consolidated narrative via a single Claude call (`CommentWorker.CompactNotes`
      → `AnthropicCommenter.CompactNotes`, usage category `digest`) that weights
      recent games most — not a trim-to-latest; the synthesized note carries no match
      id so it never collides with per-game dedupe, and `Done` is kept intact so
      compacting never re-narrates past games. A model error leaves the diary
      untouched. With no comment worker attached it degrades to trim-to-latest. The
      call is slow, so the TUI runs it off-thread (`compactNotesCmd`). `C`
      clears it (and re-seeds, so no past-game burst).
    - **Full prompt override** (`s` → `screenAIPrompt`): the DB-backed
      `comment_prompt_override` setting (`CommentPromptOverride`/`SetCommentPromptOverride`,
      admin only) lets an admin **replace BETanIA's entire instruction body**. Empty ⇒
      the built-in `commentPrompt` verbatim (zero change for existing pools). When set,
      `commentPrompt` swaps the persona/tone/rules body for the override but **always
      still appends the fixed plumbing** — the untrusted-data note, standings JSON, and
      the `submit_comments` tool-call trailer (without which the tool pipeline returns
      nothing). **Mute survives a careless override** because it's enforced at READ
      time; per-player tone overrides and the first-person self line are only honoured
      if the admin's text includes them. The override also feeds the live commentary
      below (as a steering preamble — its prompt is structurally different).
    - **Same sanitization boundary:** comment text is untrusted model output rendered
      into every terminal — `ai.sanitizeText` is applied in the worker before the
      cache + log, so both get clean text.
    - **Observability:** `service.AICommentMonitor` port
      (`AICommentStatus`/`AICommentActivity`/`TriggerAIComments`, all `requireAdmin`).
      The **⚙ Admin: BETanIA** panel is **tabbed** (`tab` cycles Betting → Comments →
      Usage; see the BETanIA bullet above). On the **Comments** tab the recent-comments feed
      is a **selectable list** (`↑↓`/`jk` move, `enter` opens `screenAICommentDetail`
      with the full untruncated text), and **`c` regenerates ALL comments at once** (one
      worker pass → full cache rewrite, coalesced). On `screenAICommentDetail`, **`r`
      regenerates just THAT player's comment** — `service.RegenerateComment(by, name)`
      (admin) resolves the name→user and calls the `CommentWorker.RegenerateOne(ctx,
      userID)` seam (`SetCommentRegen`), which runs a detect→write pass but `Upsert`s
      ONLY that player into the cache (every other line untouched). It's synchronous
      (two model calls), so the TUI runs it off-thread via a `tea.Cmd`
      (`regenCommentMsg`) and updates the open detail view in place. **Logging:** each
      comment is a JSON line in `BETHOVEN_AI_COMMENT_LOG_PATH` (default `ai_comments.log`;
      **must** be under the systemd `ReadWritePaths` dir in prod, like `ai_bets.log`).
    - **Live top-of-board commentary (`ai.LiveCommentWorker`).** A THIRD worker
      (gated by `AICommentsEnabled`, alongside the per-player one) writes a single
      general line about the **in-play slate** — who's nailing the scoreline, who's
      climbing/falling, which side the **odds** favour (the model is told to
      translate the American moneyline into plain favourite/underdog language, never
      quote the raw `-180`), and **who scored** (from the feed's key events) — shown
      at the **top** of the live section as a headline (`results.go`, 🤖-prefixed,
      **ungated** like `AllLeaderboardComments`: the per-viewer comment hide (`h`)
      does NOT cover it). Like the rest of `ai` it takes function seams
      (`LiveCommentDeps{Situation, Config, Now}`) and never imports `service`;
      `service.LiveSituation` builds the snapshot (live matches + closest picks +
      **key events** + movers, from `LivePicks`/`Leaderboard`), `service.LiveCommentary` reads the
      cache via the `LiveCommentSource` port. **Halftime pivot:** when every in-play
      match is at the interval (`halftimeFocus(sit)` in `ai` → all `Phase == halftime`),
      she's already narrated the half, so the prompt **switches** (not appends) to a
      leaderboard-dynamics body — who's climbing/sliding on live points, shrinking
      gaps, who leads the match's picks, who's stuck on zero. `Standings` is attached
      to the snapshot **always** (see Variety below), so the same data grounds both
      modes; the only thing that changes at halftime is which prompt body is chosen.
      Works under an admin override too. **Throwaway by design:** one Claude
      call (no web search, tone-aware, honours the prompt override), sanitized via
      `sanitizeText`, cached in `ai.LiveCommentCache` (current line + expiry + a short
      history ring fed back so it doesn't repeat itself). **Cadence — on-change +
      heartbeat:** ticks every `liveTick` (30s), regenerates when the situation
      **signature** (scores + movers + the key-event tail, *not* the clock) changes OR the heartbeat
      (`BETHOVEN_AI_LIVE_COMMENT_INTERVAL_SECONDS`, default 300s) elapses, never two
      lines closer than `liveFloor` (120s). **Cleared entirely when nothing is live**,
      so a game's lines are discarded the moment it ends. Logged to the same file as
      the per-player comments, tagged `source:"live_comment"`.
      **Variety / context:** the snapshot also carries `standings` (rank-sorted
      positions + totals) so the line can riff on the **title race / shrinking gaps**,
      and the prompt is fed the admin **rivalries** (`cfg.Rivalries`); the prompt
      explicitly tells her to ROTATE players/angles (title race, a rivalry, a
      climber/faller, who nailed it or whiffed) and not fixate on the one standout
      pick, with the recent lines fed back for anti-repeat.
      **Survives a version swap (no loose file — it's in the DB):** the in-memory
      cache is throwaway, but on shutdown `main` snapshots it to the `settings` KV
      `live_comment_state` via `service.SaveLiveCommentState` and reloads it at boot
      (`LoadLiveCommentState` → `LiveCommentCache.LoadJSON`), so the SIGTERM that
      `systemctl restart` sends preserves the current line + anti-repeat history
      across deploys. The `ai` package only marshals the JSON (`SnapshotJSON`/`LoadJSON`)
      — it never touches the DB; `main` bridges to the service (no import cycle). A
      stale snapshot is self-correcting: the next pass sees nothing live and clears it.

## Onboarding & admin

- First connect with an **unknown key** -> registration screen (invite code +
  display name). Wrong code -> rejected, no user created. Admin keys skip the code.
- **Admins** are set via `BETHOVEN_ADMINS` (comma-separated fingerprints from
  `ssh-keygen -lf key.pub`). `service.Resolve` **reconciles the role both ways** on
  connect: an allowlisted key is promoted to admin, and a stored admin no longer in
  the list is **demoted** to player. The env allowlist is the single source of
  truth — add a fingerprint and connect (order doesn't matter); remove it and the
  next connect revokes admin.
- **Display names** are validated server-side in `Register`: rejected (not silently
  stripped) if they contain control chars/ANSI escapes (`ErrBadName`) or duplicate an
  existing name case-insensitively (`ErrNameTaken`). They're rendered into other
  players' terminals, so this is a security boundary, not cosmetics.

## Run / test / build

```sh
make run              # build + serve on :2222, invite code "letmein"
make test             # go test -race ./...  (hermetic: temp DBs, ephemeral ports)
make build-linux      # static linux/amd64 (cross-compiles cleanly — no cgo)

# BETanIA onboarding (one-time, by hand; opens its own DB connection like place-bet):
ANTHROPIC_API_KEY=… BETHOVEN_DB_PATH=… bethoven ai-seed   # create the player + backfill played games (web search OFF)
```
Connect: `ssh -p 2222 localhost` (or set up a `~/.ssh/config` alias; see README).

## Tests

- `scoring` — table-driven unit tests covering the three tiers (exact 3 / result 1
  / miss 0) and the knockout regulation-90' rule.
- `service/*_test.go` — integration vs a **real temp SQLite** + **fake clock**. The
  headline is `TestKickoffLock`. Shared harness `newTestService` lives in `service_test.go`.
- `server/server_test.go` — real wish server on `:0` + real `x/crypto/ssh` client;
  asserts registration vs menu rendering. Uses a sleep-then-quit pattern to capture
  the first frame.
- `tui/tui_test.go` — `teatest` drives the Tea model directly (registration flow).

## GOTCHAS (read before you trip)

- **wish writes a stray host key.** `wish.NewServer` generates its own
  `id_ed25519` in the **current working dir** if no host signer is set *at
  construction time*. We pass the key via `wish.WithHostKeyPEM(...)` as an option
  (see `server.New` / `hostkey.go`). **Do NOT** switch to `AddHostKey` after
  `NewServer` — that reintroduces the leak (it's what put `id_ed25519` in the repo
  root and `internal/server/` originally). `id_ed25519`/`*.pem` are gitignored as a backstop.
- **Store methods called from `service` must be EXPORTED.** Go can't call an
  unexported method across packages. An unexported helper on `*db.Store` used by
  the service won't compile (this bit `BetsForMatch`, originally `queryBetsForMatch`).
- **SQLite is single-writer.** `db.Open` sets `SetMaxOpenConns(1)` + WAL +
  busy_timeout. Don't bump max conns — concurrent writers cause "database is locked".
  This is fine: one tiny server, low traffic.
- **Don't call `time.Now()` in business logic.** Use the injected `Clock`
  (`service.Now()` / `s.clock.Now()`), or the kickoff-lock tests can't control time.
  `clock.Real` is the only place wall-clock is read in the service path.
- **No TLS, by design.** SSH provides encryption + server identity (the host key).
  There is no cert. The **persistent host key** matters: if it changes, every client
  gets a scary "host key changed" warning. It lives at `BETHOVEN_HOST_KEY_PATH`
  (default `host_key`; `/opt/bethoven/data/host_key` in prod) — keep it on durable disk.
- **Run on port 2222, not 22.** Port 22 is the VM's own sshd; clashing locks you out.
- **`activeterm` requires a PTY.** SSH transport tests must `RequestPty` before
  `Shell()`, or the connection is rejected.
- **Bubble Tea Model is passed by value.** `Update` returns `(tea.Model, tea.Cmd)`.
  Mutating helpers take a value receiver and return the modified `Model` (e.g.
  `goMenu`, `openBet`); pointer-receiver helpers (`setStatus`, `focusReg`) mutate in
  place — don't mix them up or edits silently vanish.
- **Times are RFC3339 UTC text in SQLite.** Always `.UTC()` before storing; the store
  formats/parses with `time.RFC3339`.
- **`fixtures.json` is placeholder data.** Replace with the official 2026 schedule
  before launch. Seeding is idempotent — it only imports into an *empty* tournament,
  so editing the file after first boot does nothing; knockouts are added via the admin TUI.

## Backup (do this BEFORE risky ops)

**Rule:** any operation that restarts the VM, changes the running model/binary, or
mutates data (migrations, manual data edits, schema changes, restoring fixtures,
**`bethoven ai-seed`** — it inserts BETanIA's bets) **must be preceded by a
database backup.** No exceptions — the DB is the only non-reproducible state
(apostas + results); the binary and `fixtures.json` aren't.
(The optional `analytics.db` is **out of scope** for this rule — it's a separate,
non-critical file holding only usage history; back it up if you like with the same
recipe, but it never gates a risky op.)

**How — single consistent file, WAL-safe:**
```sh
sqlite3 /opt/bethoven/data/bethoven.db \
  ".backup '/opt/bethoven/backups/bethoven-$(date +%F-%H%M%S).db'"
```
The DB runs in **WAL mode** (`journal_mode(WAL)`, see `db.Open`), so committed
data may live in the `-wal` file, not yet in `bethoven.db`. **Do NOT `cp` the
`.db` alone** — you'd capture a stale snapshot. `.backup` (or `VACUUM INTO`) reads
a consistent snapshot *including the WAL* and writes one self-contained file, so
you never have to copy `-wal`/`-shm`. Works with the server running or stopped.

## Deploy (short)

Static binary + `fixtures.json` to `/opt/bethoven`, install
`deploy/bethoven.service` (systemd, unprivileged user, env vars), open TCP 2222,
`systemctl enable --now bethoven`. DB + host key persist under
`/opt/bethoven/data`. No domain/TLS needed. Full steps in README.md.

**`deploy/local-deploy.sh` rewrites `bethoven.env` wholesale** (it's gitignored —
holds the invite code + admin FP). It re-reads `ANTHROPIC_API_KEY` from the deploy
shell, so running it **without that key set clobbers the working key on the VM**.
For a code-only change that needs no new env (e.g. a new setting with a default),
prefer a **binary-only deploy** that leaves `bethoven.env` untouched: WAL-safe
`.backup` → `scp` the new binary → `mv` into `/opt/bethoven/bethoven` →
`chown bethoven:bethoven` → `systemctl restart bethoven`.

**Verifying BETanIA on the VM (post-deploy or when debugging the AI).** The
GCP deploy is `gcloud --project=edy-ai-playground compute ssh bethoven --zone=us-central1-a`.
On the box:
  - `sudo journalctl -u bethoven --since '2 min ago' --no-pager` — the startup
    lines confirm which workers armed (`BETanIA live betting/commentary/live
    commentary enabled …`). Absence of a line = that worker is off (check the env
    flag); an `ANTHROPIC_API_KEY` problem shows up as API errors here.
  - `sudo tail /opt/bethoven/data/ai_bets.log` — every pick (seed + live) as JSON.
  - `sudo tail /opt/bethoven/data/ai_comments.log` — every comment, incl.
    `source:"live_comment"` for the top-of-board line. Both logs live under the
    systemd `ReadWritePaths` data dir, so they need `sudo` to read.

---
> Source: [geeksilva97/bethoven](https://github.com/geeksilva97/bethoven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
