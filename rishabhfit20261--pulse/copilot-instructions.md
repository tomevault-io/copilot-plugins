## pulse

> **`readme.md` is the specification. This file is how you stay inside it.**

# AGENTS.md — rules for any agent working in this repo

**`readme.md` is the specification. This file is how you stay inside it.**

If something you are about to write is not in `readme.md`, you are either drifting or
inventing. Stop and check. Section references below (§4, §9, …) point at `readme.md`.

---

## 0. Current state

**M1 (Skeleton) and M2 (Tier 1 collector + router + dedup) are complete and verified.**
In place on top of M1: the collector package (`base`, `router`, `runner`, `sources/{rss,
github_releases}`), `pulse.scheduler` tiered polling, cross-source dedup (`story_key.py`,
`dedup.py`, the `story_keys` table via revision `0002`), the Analyzer pickup guard
(`analyzer/runner.should_analyze`), and `embeddings.py`. The default test suite is offline
and keyless; `uv run pytest -m integration` exercises the real gates, including the
story-key race, against the Compose Postgres. An M2 follow-up added
`generator/tasks.draft_admission` — the fail-closed three-way draft-admission policy
(DRAFT / HOLD / KILL) enforcing the tier-2 confirmation requirement (§2 items 30–32)
before any Generator code exists to get it wrong — plus `publisher/notify.count_held_tier2`
(the age-bounded heartbeat counter, visible from M3 onward) and
`generator/tasks.sweep_held_tier2` (stale expiry + newest-first selection for M10's
confirm worker).

**M3 (Telegram heartbeat) is complete and confirmed on a phone** (four beats, 15 min
apart, 2026-07-31): `publisher/{telegram,notify,runner}.py` — Bot API client (long
polling), heartbeat with window counts + the held-tier-2 line, update acknowledgement
(callbacks plug in at M8). Post-M3 hardening from live logs: boot-time tier-1 feed
validation (4xx fails loudly), conditional feed fetches (ETag/Last-Modified, 304s skip
parsing, `cache_hits` in the cycle summary), per-request feed timeouts with per-feed
isolation, and bot-token redaction in every error path (client messages, unchained
tracebacks, and a formatter-level scrub).

**M4 (Agent harness) is built:** `agents/{base,contracts}.py` + `prompts/README.md`.
The bounded loop enforces the cap in code, times out between turns, validates output at
the boundary (exactly one retry with the error appended), writes every run — including
failures and budget-skips — to `agent_runs` with `raw_output` captured before parsing,
and returns empty when the daily token budget is spent. The model client is a Protocol;
tests drive it with stubs (no network, no keys). Prompt files land with their agents.

**M5 (Gatekeeper) is complete and verified.** 2026-07-31: v1 moved
entirely onto the Gemini free tier — `GATEKEEPER_MODEL`, `GHOST_MODEL`, and
`ANALYST_MODEL` now default to Gemini, kept as separate keys so any one can move back
to `claude-*` without code changes (readme §16 has the tradeoff and the rationale).
`agents/clients.py` holds both real providers behind the `ModelClient` protocol,
dispatched on the model-name prefix; 429s are retried with exponential backoff
(`MODEL_MAX_RETRIES`) and exhaustion degrades like the token budget —
`AgentResult.exhausted`, HOLD never KILL. `agents/gatekeeper.py` + `prompts/gatekeeper.md`
(v1) are in place with offline tests and the labelled 20-topic eval set
(`tests/fixtures/agents/gatekeeper_eval.json`, `uv run pytest -m eval`). **M5's
"Done when" passed live 2026-07-31: 20/20 classified, tragic headlines blocked, p95 under
the 3s budget.** Re-run the eval on every gatekeeper prompt version change. Note the eval
judges against its fixture's fixed profile (its labels are only valid there), never the
`.env` `COMPANY_PROFILE`; for eyeballing the real profile's filtering there is
`uv run python -m pulse.agents.inspect_gatekeeper` — newest topics, real profile, printed
verdicts, no assertions, runs kept out of `agent_runs`. Tool-call translation in the
clients is deliberately absent until M10.

**M6 (Writer) is complete and verified.** `agents/writer.py` + `prompts/writer.md` (v3 —
2026-08-01 added one rule: `image_brief` derives from the written `post_text`'s argument,
not the topic headline alone, so the picture reflects what the post is about):
one Gemini call with `responseSchema`, no tools, no loop, through the same bounded
harness (`WRITER_TIMEOUT_SECONDS`, default 60). Input is the topic, the verdict's
`angle` (plus the Analyst's `summary` when the verdict is an `AnalystVerdict` — it read
the story; the collector's scrape didn't), the company profile, and §9's
`brand_voice_examples`, which live in the optional file at `VOICE_EXAMPLES_PATH`
(default `config/voice_examples.md`; no placeholder ships — fake examples steer tone
worse than none). Prompt v2 (2026-08-01): when voice examples are present they are the
authority on tone and formatting and override the prompt's style bans (emoji, bullets,
question closes) — the two must never fight; structure rules always hold: hook ≤12
words standing alone, then the news in ONE short human line ("OpenAI just published X",
never the input summary recited back), then the argued angle. The profile informs the
angle and nothing else — never listed, quoted, or paraphrased in the post, never our
own service lines named (that turns commentary into a services pitch, on every post).
Hashtags are 3–5 and live ONLY in the `hashtags` field — the example posts' hashtag
footers are appended by the system, never written into `post_text`. Topics carry the
item's first-party RSS summary since 2026-08-01 (`raw_payload.summary` rides
`DedupItem` onto the topic at creation; merge rules in readme §8) — before that,
fast-lane topics had `summary=NULL` and the writer worked from bare headlines;
existing rows were backfilled from `raw_trends`. Offline tests cover the done-when (a malformed reply is caught, one
retry, then a failed topic, never a raise); a live call produced a valid in-range draft
with a populated `image_brief` on 2026-07-31.

**M7 (Image pipeline) is complete and verified.** `generator/{image_gen,compose,storage}.py`
+ three templates (`bold`, `minimal`, `data`) in `generator/templates/`. Two strict §11
stages: `ImageProvider.generate(brief, seed) -> bytes` (Pollinations default; the prompt
always appends the no-text/no-logos/no-faces exclusions) and Playwright compositing
(HTML + data-URI assets → 1200x627 PNG). Generation failure — including a dead network —
degrades to `generate_image(...) -> None` (text-only draft), while a compose failure
raises: its inputs are all local, so failing there is a bug, not weather. The logo
renders at **natural pixel size while it fits the slot** — supply `BRAND_LOGO_PATH`
pre-sized (~48–64px tall) and it is pixel-perfect by construction — but since
2026-08-03 every template clamps `.logo` to `max-width: 340px; max-height: 96px`
(aspect preserved), so an oversized source scales down instead of leaving the canvas
(`test_compose.py::test_the_logo_bounding_box_sits_fully_inside_the_canvas`). The
shipped `templates/assets/logo.png` is a neutral placeholder — the real brand asset is
`templates/assets/oyelabs.png`, wired via `BRAND_LOGO_PATH` in `.env` since 2026-08-01.
History: the first asset (235×56, wordmark cropped from a stacked lockup) was cut too
tight — glyphs touched its own edges and the y lost its descender, so every composite
read "ovelabs". Replaced 2026-08-03 with the user's horizontal lockup
(icon + wordmark + tagline, blue on transparent): source at `~/Downloads/oyelabs.png`
(598×168), prepared with `convert -bordercolor none -border 8 -filter Lanczos -resize
340x96` → 320×96, verified no opaque pixel touches any file edge and re-rendered
against all three templates. If the asset ever changes again, keep that pipeline: pad
first (a tight crop is how "ovelabs" happened), resize to fit the 340×96 slot clamp,
then run the edge check and `test_compose.py`. The `data` template's
stat callout comes from `compose.split_stat`
(deterministic regex, never a model). Uploads: `storage.upload_image` with
`image_key(topic_id, seed)` — retries overwrite, Regenerate's new seed gets a new
object; the Compose `minio-init` grants anonymous download so stored URLs render in the
dashboard. Verified live 2026-07-31: real background, all three composites at exactly
1200x627, logo crisp under 4x zoom, objects retrievable from MinIO. Gotcha recorded in
§13: blank `S3_ACCESS_KEY` in `.env` does **not** equal the Compose default
(`minioadmin`) — set both the same.

**M8 (Draft review over Telegram, text only) is complete and verified** *(rescoped by the
user 2026-07-31: no images in the flow, no LinkedIn — see §15)*. `generator/runner.py` is
the polling generation loop (M9 makes it event-driven): stale fast-lane topics expire
first (`HELD_MAX_AGE_HOURS`, so a cold start never grinds a backlog through the
Gatekeeper), then gate verdicts persist onto topic rows and route through
`draft_admission`, then the Writer drafts **newest-first under `MAX_DRAFTS_PER_DAY`**
(default 5) — topics past the cap stay approved-but-undrafted, counted in the heartbeat
("awaiting a draft slot", `notify.count_awaiting_draft_slot`). Every status transition
uses `UPDATE … WHERE status=<expected> RETURNING` (rowcount is -1 under this psycopg
setup — never trust it), so re-runs and racing workers no-op. Drafts are **text-only**:
`DRAFT_INCLUDE_IMAGE=false` skips M7's provider/compose/upload entirely (`image_url`
NULL); the flag-on path is built and integration-tested, including image-failure →
text-only degrade. Delivery (`publisher/review.py`): two messages — bare post text
(clean long-press copy), then a reply with the source headline + link and
Approve/Reject buttons; `drafts.tg_message_id` (migration 0003) makes sends idempotent;
`notify.clip` keeps every message under Telegram's 4096-char limit. **Approve stores,
never publishes** (publishing is M11); decisions are once-only and edit the review
message. Verified live 2026-07-31: a real Tier-1 topic ("Building abundant
intelligence") was gated, written, and delivered to the phone as a copyable draft;
the Approve/Reject flow is unit-tested and awaits the first real tap. Two live findings
baked in: a cold-start feed backfill arrives fresh-seen but years-old-published
(~1,900 topics expired by the publish-date staleness rule the moment it landed), and
back-to-back Writer calls can exhaust the free-tier RPM — those topics HOLD as
approved-undrafted and drain on later cycles, exactly the degrade path.

**M9 first slice (images back on) is complete and verified live 2026-08-01.**
`DRAFT_INCLUDE_IMAGE=true` in `.env`; migration `0004` adds `drafts.image_brief`,
`headline_overlay` (Writer outputs stored so Regenerate never re-runs the Writer),
`tg_photo_message_id`, and `manual`. Delivery is in **LinkedIn order** (user decision
2026-08-01, readme §8): bare post text first as its own message, the composited image
BELOW it as a separate photo message — never `sendPhoto` with the post as caption, which
renders the wrong way round — then the source line with Approve / Reject / Regenerate.
The photo goes up as **multipart bytes** (`TelegramClient.send_photo` /
`edit_message_photo`): MinIO URLs are unreachable from Telegram's servers, so URL-based
sends can never work. Regenerate (`review._regenerate_image`) is **not a decision**: new
seed → `generate_image` → `render` → upload → `editMessageMedia` in place; post text and
status untouched, refused on decided drafts (an approved post's image must not drift),
failure keeps the current image and sends a notice (never silent); the callback is
answered *before* generating (the render outlives Telegram's spinner). Manual trigger:
`uv run python -m pulse.generator.trigger [--topic-id N]` — no argument picks the newest
gate-passed undrafted topic; an explicit id is honoured but never bypasses the gate. It
runs the full path **including the image regardless of the flag**, stays out of
`agent_runs` (no-op run logger) and out of `MAX_DRAFTS_PER_DAY`
(`drafts.manual=true`, excluded by `drafts_created_today`), and does draft + send +
message-id stamping in ONE transaction so a concurrently running Publisher can never
double-send. Verified live 2026-08-01: one text-only draft (Pollinations transiently
down — the degrade path, working) and one full image draft (370KB composite, delivered
with photo + all three buttons). The 8-word `headline_overlay` cap is confirmed against
all three templates by real browser measurement
(`test_compose.py::test_an_eight_word_headline_never_overflows_or_touches_the_logo`).
**Restart running generator/publisher processes** to pick up the flag (settings are
lru_cached at startup) and the regen callback handler.

**M10 (slow lane) is built and offline-verified 2026-08-03** — every §15 done-when
criterion has a test (`test_analyzer_integration.py`, `test_generator_runner_integration.py`,
`test_velocity.py`, `test_history_mcp.py`); live verification against real tier-3 traffic
is the remaining step. The pieces, and the judgment calls made:

- **Tier-3 collectors** (`collector/sources/`): `hackernews` (Firebase, top 50),
  `reddit` (PRAW hot posts; **enabled without REDDIT_CLIENT_ID/SECRET the scheduler
  refuses to boot** — reddit is therefore *out* of `ENABLED_SOURCES` in `.env` until
  creds exist), `google_trends` (RSS, geo=IN), `gdelt` (DOC 2.0, keyless, enabled),
  `youtube` (search+stats, needs `YOUTUBE_API_KEY`, disabled), and `claude_agent` —
  the Ghost as a source (adapter over `agents.ghost`, no prompts in the collector;
  results ride the router's tier-3 default; disabled by default, readme §16 lever 3).
  All tier-3 sources share **one polling job at `TIER3_POLL_SECONDS`** — per-source
  cadence would outrun the 15-minute analysis tick they feed.
- **Analyzer** (`analyzer/runner.run_analysis_cycle`, scheduled inside `pulse.scheduler`
  every `ANALYZE_INTERVAL_MINUTES` — no fifth process): cluster → score → judge.
  Clustering (`analyzer/cluster.py`) reuses `DbDedupStore` — story key first, then
  embedding at `CLUSTER_THRESHOLD` (0.85, looser than dedup's 0.90 on purpose) — so
  merge/upgrade rules exist once. Velocity (`analyzer/velocity.py`): Redis 15-min
  buckets, 24h TTL, `(recent−prior)/max(prior,1)` over 45-min windows, **weighted by
  multiplying with `source_count`** (the most literal reading of §8 — revisit against
  live data if it over-triggers). The Analyst runs hottest-first within what is left of
  `MAX_ANALYST_RUNS_PER_HOUR`; every pickup re-checks `should_analyze` against fresh
  row state.
- **Agents**: `agents/analyst.py` (full verify + confirm mode) and `agents/ghost.py`,
  prompts `analyst.md` / `analyst_confirm.md` / `ghost.md`. **The confirm/full hourly
  budgets are told apart by prompt version**: confirm prompts version as
  `confirm-*` (`analyst.CONFIRM_VERSION_PREFIX`), and `confirm_runs_last_hour` /
  `verify_runs_last_hour` count `agent_runs` on that prefix — renaming a prompt
  version breaks the budget seam, don't. Tool support landed in `agents/clients.py`:
  function tools as provider-neutral `ToolSpec`, tool turns translated per dialect;
  **web search is a server tool on both providers** (Gemini Google-Search grounding /
  Anthropic `web_search`) — it runs inside the API call, never reaches the loop's
  `ToolHandler`, and does **not** count against `max_tool_calls`; the wall-clock
  timeout is the real bound. Structured output + tools can't be combined (both
  providers) — `client_for` rejects it; tool agents rely on boundary validation.
- **History MCP** (`mcp_server/history.py`): three read-only queries, two front doors —
  in-process as the Analyst's tool handler (no transport inside a bounded loop), and
  MCP stdio via `python -m pulse.mcp_server.history` (mcp 2.0: `MCPServer`, the 1.x
  `FastMCP` path no longer exists). Repeat check counts pending/approved/published
  drafts, ignores rejected ones (a declined story may come back).
- **Tier-2 confirmation** (`generator/runner.confirm_pass`, in `run_cycle` between gate
  and draft): expiry always runs, then confirms newest-first within
  `MAX_TIER2_CONFIRMS_PER_HOUR`; unconfirmable → rejected/`'tier-1 source not
  confirmed'`; exhaustion → held, visible. **Gap found by test**: a held tier-2 topic
  upgraded to tier 1 stops matching `tier2_held_conditions` and nothing owned it —
  `confirm_pass` now approves scored/fast/tier-1/verified-NULL topics on authority
  before spending budget.
- The scheduler now **requires `COMPANY_NAME`/`COMPANY_PROFILE` at boot** (Ghost and
  Analyst both reason against it).

**M9 is complete — the headline number is measured (2026-08-03).** The last three
pieces: **event-driven enqueue** — the Celery app lives in `generator/tasks.py`
(worker: `uv run celery -A pulse.generator.tasks worker`), the Collector's
`on_fast_lane` hook enqueues `generate_topic(topic_id)` the moment the gates create or
upgrade a fast topic, and the generator poll **stays running as the reconciliation
backstop** (enqueue failure logs and falls back; every transition is status-guarded so
the two paths race safely — the loser occasionally wastes one model call, acceptable
and known). **Parallel write + image** — `draft_topic()` runs the Writer and the
background concurrently; the background prompt derives from the *headline* (the
Writer's `image_brief` doesn't exist yet; it is still stored for Regenerate), and the
daily cap is re-checked under a Postgres advisory lock at claim time so racing workers
can't overshoot. **The acceptance test** — `tests/test_latency_headline.py` (`-m eval`,
needs Compose Postgres + keys): local tier-1 feed, a real slept-through worst-case
poll interval, live models end to end.

The measured numbers, all real: eval run **174.4s** (passed; includes the full 60s
worst-case poll and a 44s Gatekeeper read-retry storm). Live-daemon runs same day:
206.5s (cold-start scheduler still grinding its first cycles — restart artifact),
183.9s (event path enqueued at +33s, but the gate call burned 48s of read-retries and
FAILED; the 30s poll backstop rescued it), 213.9s (free-tier RPM window exhausted by
the test runs themselves: two gate attempts died in 429 backoff — §16's known price),
and finally, with quota clear: **93.1s** — 55.8s near-worst-case poll wait, 9s gate,
20s writer in parallel with the background, composite + upload, image included. The
system's own overhead is ~37s past the poll; the tail risk is entirely Gemini
free-tier weather, and the degrade path (HOLD, backstop retry) carried every bad run
to a delivered draft.

**Next: M12's dashboard (user decision 2026-08-03: dashboard before LinkedIn — not
ready to publish), then M11.** M10 live verification continues alongside: watch the
analysis cycles and the confirm sweep. **Each
milestone must satisfy its own "Done when:" before the next one starts.** The fast lane
(M2, M5, M9) ships before the slow lane (M10); do not build clustering or velocity
early — Gate 2 is a dedup check against *existing* topics, not clustering; topic
formation for slow items stays in M10.

---

## 1. What this project is (memorize it)

Pulse watches curated sources, and **who published an item decides how it is handled**. A
company announcing on its own channel is treated as fact and goes straight to writing —
draft on the phone in ~2 minutes. Anything less authoritative must earn its way through
velocity and verification. Drafts land in Telegram; a human approves before anything
publishes to LinkedIn.

The whole system is optimised for **latency on the fast lane**. If a change makes the fast
lane slower, it is wrong regardless of what else it improves.

---

## 2. Hard invariants — never violate, never "improve"

These are the rules most likely to be broken by an agent trying to be helpful.

### Lanes and tiers (§4 — the section that matters most)

1. **Tier assignment is a dictionary lookup against `config/sources.yaml`.** Never ask a
   model what tier a source is. That would add seconds and cost to every item and defeats
   the fast lane.
2. **Unlisted sources default to Tier 3.** This is a safety property, not a convenience.
   Never invert it, never add a "probably trustworthy" heuristic.
3. **Never add tools to the Gatekeeper.** No tools, no loop, one call, ~3s budget. A
   Gatekeeper with tools is just a slow Analyst. If an item seems to need a lookup, it
   belongs in a slower lane.
4. **Tier 1 skips verification, never judgment.** The Gatekeeper's relevance + safety call is
   mandatory on every fast-lane item. There is no path to publication that bypasses it — a
   major company can announce layoffs, a breach, or a death.
5. **Fast-lane rows are enqueued at insert time**, not on a scheduled tick. This one
   behaviour is worth more latency than every other optimisation combined.
6. **Fast-lane rows never enter the Analyzer.** No clustering, no velocity, no Analyst.
7. **In the Generator, the Writer call and image generation run concurrently.** Do not
   serialise them.

### Structure

8. **Services never import or call each other.** Collector → Analyzer → Generator →
   Publisher communicate exclusively through row status transitions in Postgres.
9. **All model calls live in `src/pulse/agents/`.** Nowhere else. `collector/sources/claude_agent.py`
   is a thin adapter that calls `agents.ghost` and translates the result — no prompts, no
   tool definitions in it.
10. **An agent is a function.** Typed Pydantic input → bounded loop → validated Pydantic
    output. Callers must not know a model is inside.
11. **Four agents. Never a fifth, never a merged one.** `gatekeeper`, `ghost`, `analyst`,
    `writer`. They never call each other; the pipeline sequences them.
12. **Limits go in the loop, not the prompt.** Tool-call caps, timeouts, and token budgets
    are enforced in code in `agents/base.py`. A prompt asking the model to "use at most 5
    searches" is not a limit.
13. **Never persist unvalidated model output.** Parse into the Pydantic contract first. On
    failure: retry exactly once with the error appended, then fail the topic.
14. **Log every agent run, including failures**, to `agent_runs`, with `raw_output` captured
    *before* parsing.

### Safety and publishing

15. **`safety_verdict='blocked'` or `verified=false` kills the topic.** No override path, no
    flag, no admin bypass. Do not add one. When in doubt the model blocks — do not soften
    that instruction in a prompt.
16. **Nothing publishes without explicit human approval.** No auto-publish mode in v1, not
    even behind a config flag. Do not add one.
17. **Never ask the image model to render text or logos.** Background only; Playwright
    composites headline + logo. This is §11 and it is non-negotiable.

### Operations

18. **Degrade, do not stop.** Dead source → log and continue, never abort the run. Token
    budget exhausted → agents return empty, feeds keep running. Provider capacity still
    exhausted after backoff (429, or the free tier's 503/504 overload storms) → same as
    budget: empty result, `AgentResult.exhausted`, the item HOLDs and is retried later —
    exhaustion is never a KILL (§16). Image gen failed after
    retries → text-only draft, not a lost topic.
19. **Migrations from commit one.** Every schema change is an Alembic revision. Never
    `create_all()`.
20. **Idempotency everywhere.** Any service run twice must not duplicate rows
    (`ON CONFLICT DO NOTHING` on `(source, external_id)`).
21. **No secrets in code**, not even in tests. Everything through `config.py`.
22. **Prompts, templates, and `sources.yaml` are data.** Changing tone, visual style, or the
    source list must never require a Python change or a redeploy.
23. **uv is the only package manager.** No `pip install`, no `requirements.txt`, no
    `python -m venv`, no `source .venv/bin/activate`. Every command you run or write into
    docs, Dockerfiles, or CI goes through `uv run`. `uv.lock` is committed. Never hand-edit
    `pyproject.toml` or `uv.lock` — use `uv add`, `uv remove`, `uv lock`.

### Dedup and merges (§8) — added with M2

24. **`story_keys` is unbounded — no TTL, no cleanup job.** A URL that has ever
    produced a topic must never produce another, however old. `DEDUP_WINDOW_HOURS`
    bounds Gate 2's embedding lookback only. Do not "tidy" this table.
25. **A rejection sticks.** The same URL arriving after a rejected topic or draft
    hits Gate 1, attaches, and nothing regenerates. There is no re-open path.
26. **The topic keeps title and summary from its best-tier source.** A lower-tier
    merge changes neither; a strictly better-tier merge replaces the title and
    clears the summary. Never let an aggregator's paraphrase title a first-party
    story.
27. **The upgrade rule enqueues only when nothing is decided:** lane flipped to
    fast AND no draft exists AND status is `new`/`scored`. Everything else
    attaches silently.
28. **The Analyzer re-checks lane, status, and draft existence at pickup**
    (`analyzer/runner.should_analyze`) — the queue is never trusted; a topic can
    be upgraded mid-flight.
29. **The gates are DB lookups plus one local embedding.** No model calls, no
    per-item page fetches. Redirect resolution runs only for known shortener
    hosts and is off the latency budget; canonical-URL reading is out of v1 (§19).
30. **Tier 2 never drafts without a confirmed Tier-1 source.**
    `generator/tasks.draft_admission` is the only gate in front of the Writer and
    **fails closed** — but `None` and `False` are different failures:
    `verified=False` is a decision → KILL (`status='rejected'`, sticks);
    `verified=None` means the confirmation never ran (cap, budget, or not yet
    wired) → **HOLD**: `status='scored'`, `verified IS NULL`, re-attempted later,
    counted in the heartbeat. Never turn a HOLD into a silent drop or an
    automatic KILL, and never add a second path to the Writer.
31. **The tier-2 confirmation has its own budget, separate from the Analyst's.**
    Confirm-mode caps are `TIER2_CONFIRM_MAX_TOOL_CALLS=3` /
    `TIER2_CONFIRM_TIMEOUT_SECONDS=60`, at most `MAX_TIER2_CONFIRMS_PER_HOUR`.
    Never count confirms against `MAX_ANALYST_RUNS_PER_HOUR` (they'd starve each
    other) and never give the confirm call the Analyst's full 8/180 limits. On
    exhaustion: HOLD and report — do not call, do not kill.
32. **Held is not forever.** Past `HELD_MAX_AGE_HOURS` (48h from `first_seen_at`)
    a held item expires to `rejected`/`block_reason='stale'` and is **never sent
    for confirmation** — confirming old news spends budget drafting dead stories.
    The sweep (`generator/tasks.sweep_held_tier2`) expires before it selects and
    selects **newest-first**; the heartbeat count applies the same age bound.
    The held-state predicate lives once, in `models.tier2_held_conditions` —
    never redefine it in a service.

---

## 3. Anti-hallucination rules

The failure mode for this project is an agent that invents a plausible name and then builds
three files on top of it. Guard against it:

- **Do not invent table or column names.** The complete schema is §7. If you need a field
  that does not exist, say so and propose an Alembic revision — never silently add it to a
  model.
- **Do not invent status or enum values.** The complete sets are:
  - `raw_trends.status`: `new`, `routed`
  - `topics.status`: `new`, `scored`, `approved`, `rejected`, `drafted`
  - `drafts.status`: `pending_review`, `approved`, `rejected`, `published`, `failed`
  - `lane` (both tables): `fast`, `slow`
  - `source_tier` / `topics.source_tier`: `1`, `2`, `3`
  - `safety_verdict`: `safe`, `blocked`
  - `agent_runs.agent`: `gatekeeper`, `ghost`, `analyst`, `writer`
  - `drafts.template_used`: `bold`, `minimal`, `data`
- **Do not invent config keys.** The complete list is §13 and it must match `.env.example`
  exactly. A new key means adding it to `config.py` and `.env.example` in the same change.
- **Do not invent dependencies.** The stack is §5. **Ask before adding anything not listed**
  (§17); when approved, `uv add` it and commit the lockfile change in the same step. No
  LangChain, no LangGraph, no agent framework — the loop is hand-written in `agents/base.py`.
- **Do not invent commands.** If an install or run step is not `uv sync` / `uv run …` /
  `uv add …` / `uvx …`, it is wrong. A half-remembered pip-based tutorial snippet is the most
  likely source of a wrong command here.
- **Do not invent API shapes.** External endpoints change. §12 explicitly flags the Google
  Trends RSS endpoint as needing verification; Pollinations has no SLA and shifting tiers.
  Check live docs before writing a client — never write one from memory.
- **Do not invent model behavior in tests.** Record real responses into `tests/fixtures/`
  (`sources/` for feeds, `agents/` for model responses including tool turns) and replay them.
  Tests run with **no network and no API keys** — a hard requirement, not a nice-to-have.
- **When the spec is silent or ambiguous, ask.** Do not pick a plausible default and build on
  it. §19 lists four deliberately-open decisions; if your work depends on one, surface it
  rather than resolving it yourself.

---

## 4. Fixed names — use these exactly

```
pyproject.toml  uv.lock  .python-version  .env.example
docker-compose.yml  Dockerfile          # uv-based
config/sources.yaml                     # tier definitions
alembic/versions/
src/pulse/
  config.py  sources_config.py  db.py  models.py  schemas.py  embeddings.py  logging_config.py  scheduler.py
  agents/       base.py  clients.py  gatekeeper.py  ghost.py  analyst.py  writer.py  contracts.py  prompts/
  mcp_server/   history.py
  collector/    base.py  router.py  story_key.py  dedup.py  runner.py  sources/{rss,github_releases,hackernews,reddit,google_trends,gdelt,youtube,claude_agent}.py
  analyzer/     cluster.py  velocity.py  runner.py      # slow lane only
  generator/    image_gen.py  compose.py  tasks.py  templates/
  publisher/    telegram.py  notify.py  linkedin.py  runner.py
  api/          main.py  routes/dashboard.py
tests/fixtures/{sources,agents}/
```

Agent contracts (`agents/contracts.py`) — the only shapes that cross an agent boundary:

| Agent | In | Out | Model | Tools | Limit |
|---|---|---|---|---|---|
| `gatekeeper` | `Topic`, `CompanyProfile` | `GateVerdict` | `GATEKEEPER_MODEL`, default Gemini flash-lite, thinking low | **none** | 1 call, no loop, ~3s |
| `ghost` | `CompanyProfile`, `seen_titles: list[str]` | `list[DiscoveredItem]` | `GHOST_MODEL`, default Gemini | web search | 5 calls / 120s |
| `analyst` | `Topic`, `source_items`, `velocity_score`, `CompanyProfile` | `AnalystVerdict` | `ANALYST_MODEL`, default Gemini | web search + history MCP | 8 calls / 180s |
| `writer` | `GateVerdict \| AnalystVerdict`, `brand_voice_examples` | `PostDraft` | `WRITER_MODEL`, Gemini `responseSchema` | **none** | 1 call, no loop |

v1 runs every agent on the Gemini free tier (readme §16): the safety gate and the writer
are the same model family, accepted because nothing publishes without human approval, and
reversible per agent — the provider is dispatched on the model-name prefix in
`agents/clients.py`, so pointing any `*_MODEL` key back at `claude-*` is a config change,
never a code change. Do not hardcode a provider anywhere, and do not give the Writer
tools — if the Writer seems to need a lookup, the upstream verdict is missing a field (§9).

The tier-2 confirmation is the `analyst` agent in confirm-only mode — one narrow question
("does a Tier-1 source exist for this claim?") under its own caps (3 calls / 60s /
`MAX_TIER2_CONFIRMS_PER_HOUR`). It is not a fifth agent; do not create one for it.

`ghost.seen_titles` is not optional: without it Ghost re-reports the same stories hourly and
corrupts velocity counts.

The history MCP server is **read-only, without exception**, and exposed **only to the
Analyst**. The agent recommends; the pipeline writes.

---

## 5. Scope boundaries

**Non-goals for v1 (§18) — do not build these, even if they seem obviously useful:**
X/Twitter ingestion or posting, auto-publishing, multi-tenancy, any web UI beyond the
read-only status dashboard, comment replies / DMs / engagement automation, **any paid data
source**.

**Also out of scope unless asked:** refactoring code that already passes its milestone,
adding abstraction layers "for later", swapping a library listed in §5, adding retries or
caching the spec did not ask for.

Requested scope is the deliverable. Do not quietly widen it.

---

## 6. Before you finish any task

- [ ] Does it violate anything in §2 above? Re-read the list — especially the lane rules.
- [ ] Are all names — tables, columns, statuses, config keys, file paths — from the spec,
      not invented?
- [ ] Did anything you touched make the fast lane slower? `drafts.latency_ms` is the core
      health metric; treat a regression in it as a bug.
- [ ] `uv run mypy --strict src/pulse` clean; type hints on every public function.
- [ ] `uv run ruff check .` and `uv run ruff format --check .` clean.
- [ ] `uv lock --check` passes — the lockfile matches `pyproject.toml` and is committed.
- [ ] Structured JSON logs with `service`, `lane`, `topic_id`, `draft_id`.
- [ ] `uv run pytest` passes with no network and no API keys.
- [ ] Does the current milestone's "Done when:" (§15) actually hold? Verify it, don't assume.

Report honestly. If a step is skipped or a test fails, say so plainly with the output.

---
> Source: [rishabhfit20261/Pulse](https://github.com/rishabhfit20261/Pulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
