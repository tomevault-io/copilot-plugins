## polybot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm test                              # node:test over test/**/*.test.ts via tsx
node --import tsx --test test/race-engine.test.ts   # one test file
node --import tsx --test --test-name-pattern "idempotent" test/*.test.ts
npm run build                         # tsc -p tsconfig.json (type-check + emit to dist/)
npm run check                         # test + build; run this before committing
npm run dev                           # arena API only, 127.0.0.1:3001
npm run dev:all                       # arena API (3001) + deterministic course (4000)
npm run smoke:steel                   # live Steel check: 4 real sessions; needs real keys
```

`npm test` and `npm run build` use fakes and never touch Steel or OpenRouter. Only
`smoke:steel` and a live `POST /races` require credentials. Copy `.env.example` to `.env`.

## Architecture

A backend for a four-racer browser-agent race. Four LLM agents drive four Steel cloud
browser sessions through the same course while a master LLM sabotages them and spectators
trade a virtual prediction market on the winner. The spectator web app is in `web/`
(Vite + React + TypeScript; see `web/README.md`).

Layering is strict and dependency-inverted: `domain` knows nothing about I/O, `application`
depends only on interfaces declared in `src/application/contracts.ts`, and `agents`,
`infra`, `course`, and `persistence` supply the implementations. Tests exercise the
application layer with hand-written fakes rather than mocks; keep new I/O behind a contract
interface so this stays possible.

```
api/server.ts (Fastify)     → RaceRegistry → RaceCoordinator (one per race)
                                                 ├─ RaceEngine        pure state machine, emits RaceEvent[]
                                                 ├─ VirtualPredictionMarket
                                                 ├─ RacerSessionManager   → SteelSessionManager (+ SteelKeyPool)
                                                 ├─ CompetitorAgentRunner → PlaywrightCompetitorRunner (+ OpenRouter model per racer)
                                                 ├─ CourseVerifier        → DeterministicCourseVerifier → HttpCourseStateGateway
                                                 ├─ ObstacleProvider      → MasterObstacleProvider → CdpObstacleProvider
                                                 └─ RaceEventStore        → JsonlRaceEventStore
```

`createProductionRaceCoordinator` (`src/application/production-race-factory.ts`) is the only
place that reads env vars and wires the real implementations; `buildApi` takes the factory
as a parameter so tests inject their own.

### Race lifecycle

`prepareAndStart` creates all four Steel sessions, prepares all four agents, arms the
sabotage plan, passes the readiness barrier, then calls `RaceEngine.start` so all racers
begin at the same timestamp. `prepare` runs everything before the start on its own (once):
with `startHoldMs` (`FIGHT_INTRO_HOLD_MS`, 10 s in live mode, 0 simulated) the registry
prepares a fight created to start now, publishes `startsAt` as ready + hold, and starts it
then on a timer (`startTimer`; `tickAll` is the fallback). Its market stays `pending` (no
trading) until that start, so the intro video (played over the lobby's featured card) ends
exactly as the agents and the market start; scheduled fights keep pre-fight trading. Agent loops then
run detached; a rejected loop marks that racer `failed` rather than failing the race. An
API-level ticker calls `tick` every second.

The ticker calls `RaceEngine.tick`, but elapsed time no longer freezes hazards or ends
a race. The first racer whose finish passes verification wins; the coordinator then
freezes and resolves the market, stops the other runners, and releases every Steel
session. A race is explicitly aborted if startup fails or all runners fail.

Every method takes an explicit `now` parameter defaulting to `Date.now()`. Tests pass fixed
timestamps; preserve this when adding time-dependent logic.

### Verification and idempotency

Racer self-reports are never trusted. `recordCheckpoint`/`recordFinish` call the
`CourseVerifier` first and throw if the course's own state disagrees, and the verifier also
checks that raceId, racerId, courseId, seed, and Steel session id all match the run. The
engine separately enforces idempotency: a repeated `racerId:checkpoint` claim is a no-op,
checkpoints must advance by exactly one, duplicate observations are idempotent, and sabotage
fires at most once per racer. Transient course-state transport failures are retried by the
deterministic verifier; hard authorization or run-proof failures are not hidden.

The master completion judge (`completionJudge`, the master model) is consulted only for a
run the verifier says it does not cover (`CourseVerifier.coversRun` returns false). A
verifier without that method covers every run, and `DeterministicCourseVerifier` covers every
run, so no course-server race (arena-shop or the test course) ever reaches the judge: its
workers get no judge and no page review, and a checkpoint or finish reported with source
`"master"` is still verified against the course. A run on a site the course does not serve
fails closed (it never shows progress) unless it is wired with a verifier that declines it.

### Sabotage

One immutable `SabotagePlan` is selected per race *before* the start (`armSabotage`), not per
checkpoint — the older per-checkpoint policy path survives only as a fallback. The master
model picks a tier (`basic`/`intermediate`/`difficult`) and a `DisruptionCommand`; selection
is bounded by a 2.5 s timeout and any failure falls back to a seed-derived deterministic
tier, so a live race never blocks on the model. The plan fires per racer at that racer's
first verified checkpoint (checkpoint 1 for the default enabled plan), gated on
`verifyTargetOpening`.

Disruptions are compiled to a JS string by `buildDisruptionScript` and injected over CDP
into one racer's page. `validateDisruptionCommand` runs on every path — on construction, on
model output, and again at apply time — and tier limits cap `durationMs`/`intensity`. A
blocking modal has no visible Close control; recovery requires an active bounded DOM action.
Hazards do not self-clear from `durationMs`.
Injected scripts target elements only by `[data-arena-role="…"]`, the same attribute the
competitor runner uses for clicks, and each carries a `data-arena-disruption-id` marker plus
a registry-backed recovery helper. CDP commands for one racer are serialized through a
`SessionCommandQueue`; different racers stay parallel.

### Competitor agents

`PlaywrightCompetitorRunner` runs a bounded observe→decide→act loop (`COMPETITOR_MAX_ACTIONS`,
default 40: winning shop runs took 13–30 steps, a racer stuck in sabotage recovery 77). The
production factory passes it as `maxActions`, the runner declares it as `maxSteps` so
spectators see the real budget from the start, and a racer past it fails ("exceeded N
actions"). The model returns one `AgentDecision` at a time via a forced tool call; parsing
goes through `parseAgentDecision`. Actions are deliberately narrow: clicks and typing resolve
only through `data-arena-role`, navigation is same-origin-only, and waits are rejected while
a disruption is active. OpenRouter tool JSON gets one bounded repair retry, and a reply
without the tool call is asked for once more with `tool_choice: "required"`; after that the
turn becomes a safe `inspect` marked `decisionIssue.fallback`. From the third `inspect` in a
row of an unchanged page, that step's history entry (and its `modelError`) tells the model to
act; the step stays an action.

Every configured model, the racers' and `MASTER_LLM_MODEL` alike, gets one shared sliding
window (`OPENROUTER_MODEL_MAX_CALLS_PER_MINUTE`, default 20 per 60 s); the limiter's own
defaults cover `openai/gpt-5.6-luna` and `anthropic/claude-haiku-4.5`, keyed by the
configured id in any case. The master never queues for a slot (`tryAcquireShare`): it takes
one only when it is free, holds at most a quarter of a window it shares with a racer
(`MASTER_CAPACITY_SHARE`), and sends each request once (`maxRetries: 0`, so no hidden SDK
retry skips the limiter). Without a slot its call fails with `ModelCapacityError` and the
caller falls back. A 429, 408, 5xx or dropped connection becomes a `DecisionRetryError`:
`prepareForCall` waits out the Retry-After or reset hint, else backs off from 1 s, for at
most 30 s before the provider answers again. The runner notes each pause and adds it to the
step's `rateLimitWaitMs`; paced retries never count against its decision-failure limits (3 in
a row, 6 per run). Auth errors, a spent budget and aborts get no paced retry (the first two
end the racer through those limits), and the SDK's own silent retries are off for
competitor calls. The racers and the master
draw on one `OpenRouterUsageBudget` (`RACE_LLM_BUDGET_USD`, default $100) cut into shares
(`budget.share`, `raceBudgetShares`): 10% for the master and an equal part of the rest per
racer, so a looping racer spends only its own share and stops alone. The race total
(`llmUsage`) counts every share. Each share is a soft stop — in-flight calls can overshoot
it, so keep a hard limit on the OpenRouter key too.
`src/agents/anthropic-models.ts` is a retained direct-provider alternative to the OpenRouter
adapters and is not wired into the production factory.

### Steel sessions and keys

`SteelKeyPool` rotates through `STEEL_API_KEYS`; a key is benched permanently on 401/402/403
or a credit/quota message and temporarily on 429, and the call retries on the next key. A
live session must be released with the same client that created it, which is why
`SteelSessionManager` keeps a per-racer client map.

Steel closes a session once the `timeout` it was created with passes, whatever the race is
doing. The factory creates a race's sessions with `raceSessionTimeoutSeconds`, using the
configured duration as a provider-session lifetime plus 180 s for preparation and release,
never below 300 s (480 s by default). A racer whose browser dies anyway fails with a
readable cause, and only its own session is released.

Steel browsers run remotely and cannot reach your machine. A live race needs a public
`startUrl`, though `COURSE_BASE_URL` can stay on localhost because verification is
server-to-server.

### Course server

`src/course/course-server.ts` is a self-contained deterministic course used for tests and
local development: it serves the racer-facing HTML and owns the authoritative per-run state
that the verifier reads back from `/arena/state`. That one endpoint is the only route gated
by `COURSE_VERIFIER_TOKEN`; the browser UI stays open so agents can use it.

### Persistence and DTOs

`RaceEngine` accumulates events in memory; the coordinator appends only the newly added
suffix to the store after each transition, tracked by `persistedEventCount`. The JSONL store
serializes writes through a promise chain and filters by `raceId` on read.

`src/api/dto.ts` is the types-only contract shared by the server and the `web/` app, which
imports it as `@contract`. Keep it free of runtime exports: adding one would break the web
build. `docs/frontend-contract.md` describes that surface (simulated mode, SSE, the shared
credit ledger, `/api/meta`), all of which exists in `src`.

## Conventions

- ESM with NodeNext resolution: every relative import must carry a `.js` extension even
  though the source is `.ts`. `strict` is on and there is no linter.
- Never commit Steel or model API keys, CDP URLs containing credentials, or secrets in event
  logs; `.env` is gitignored and `.env.example` is the checked-in reference.
- Commit in small, runnable increments with Conventional Commit subjects (`feat:`, `fix:`,
  `test:`), matching the existing history.

---
> Source: [tkachyn/polybot](https://github.com/tkachyn/polybot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
