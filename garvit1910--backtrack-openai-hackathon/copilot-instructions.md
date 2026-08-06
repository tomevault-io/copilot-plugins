## backtrack-openai-hackathon

> **Pitch:** Backtalk reads real customer feedback, clusters it into revenue-weighted

# Backtalk — closed-loop ad engine (hackathon)

**Pitch:** Backtalk reads real customer feedback, clusters it into revenue-weighted
pain points with vector embeddings, researches the live market with Octen, compiles
everything into a **versioned brand context layer**, generates fully-cited ad
variants *through* that layer, runs per-segment Thompson-Sampling bandits to find
which message converts for which audience, and writes the winners back into the
layer — so cycle 2's ads are provably sharper — before shipping the winning brief
to Slack (Composio) on one human approval. The demo proof: the loop runs **twice**,
and cycle 2's ads are visibly different per segment and convert better.

**Demo brand:** *Crewkit* — a B2B team-merch subscription platform (companies order
branded apparel/swag for their teams on monthly plans). This reconciles e-commerce
complaint themes (shipping, sizing, returns) with SaaS-style accounts that carry
`segment` (starter|growth|enterprise) and `mrr`. Person 2's `fixtures/brand.yaml`
should describe Crewkit.

## The contract — `lib/schemas.ts` is FROZEN

All cross-zone types live in [lib/schemas.ts](lib/schemas.ts) and are frozen as of
Gate 1. Shapes: `Ticket`, `Cluster`, `EvidenceCard`, `ContextRow`, `ContextPack`,
`Creative`, `BanditState`, `BanditReport`, `StageEvent`, `ShipBrief` — plus the
function contracts Persons 2/3 implement: `ContextStoreApi` (append/compile),
`GenerateCreatives`, `RunLearner`, `RunBandit`.

`StageEvent.status` is `start | item | tick | done | awaiting_approval | shipped`
(per the brief — note this supersedes the early shim's `"complete"`). `item`
events stream one payload at a time so the UI can animate. Bandit ticks are
`status: "tick"` with payload `TestTickPayload` (`{ states: BanditState[] }`).

## Ownership zones — never modify another zone

| Person | Owns |
|---|---|
| **1** | `lib/signals/`, `lib/schemas.ts`, `lib/events.ts`, `app/api/`, `fixtures/tickets.json`, `fixtures/cache/`, `scripts/`, repo config |
| **2** | `lib/context/`, `lib/agents/creative.ts`, `lib/agents/learner.ts`, `fixtures/brand.yaml` |
| **3** | `lib/bandit.ts`, `lib/agents/mediaBuyer.ts`, `app/page.tsx`, `components/` |

The only cross-zone seam is [lib/signals/registry.ts](lib/signals/registry.ts)
(Person 1's file): it binds the orchestrator to Person 2/3 exports, using stubs in
`lib/signals/stubs/` until the real modules land. When your module is ready, tell
Person 1 and they flip one import line.

**Rule: no dynamic imports of cross-zone modules anywhere** (`import(someVar)`
breaks the Next build when the target doesn't exist yet). Static imports only,
through the registry.

## Conventions

- `DEMO=replay|live` (env, overridable per-request with `/api/run?mode=`).
  **Every stage dumps its output to `fixtures/cache/`.** Live runs also record
  every emitted event to `fixtures/cache/events.jsonl`; replay mode re-emits that
  log through the same SSE stream with tuned pacing — the UI cannot tell the
  difference. `fixtures/cache/` is committed: replay must work on a fresh clone
  with zero API keys.
- Cache files: `clusters.json`, `evidence.json`, `packs-c1.json`,
  `creatives-c1.json`, `bandit-c1.json`, `learnings.json`, `packs-c2.json`,
  `creatives-c2.json`, `bandit-c2.json`, `ship.json`, `ship.result.json`,
  `embeddings.json`, `events.jsonl`.
- Never put embedding vectors in `StageEvent.payload` (megabytes over SSE) —
  vectors live only in cache files.
- Pipeline order: cycle 1 `ingest → understand → research → compile → create →
  test → learn`, cycle 2 `compile → create → test → ship`. Compile runs once per
  segment against the top revenue-at-risk cluster. The run stream ends with the
  ship stage's `awaiting_approval` event; `POST /api/ship` (approve button) posts
  the brief to Slack for real — even in replay mode.
- Keys live in `.env.local` (gitignored): `OPENAI_API_KEY`, `OCTEN_API_KEY`,
  `COMPOSIO_API_KEY`, plus `DEMO`, `REPLAY_SPEED`. See `.env.example`.
- Models: `text-embedding-3-small` for embeddings, `gpt-5-mini` for labeling and
  distillation.

## Gates

1. ~~schemas + tickets.json + CLAUDE.md on main~~ ✅
2. ~~`fixtures/cache/clusters.json` exists and matches the planted themes~~ ✅
3. ~~Full pipeline runs end-to-end (P3's real mediaBuyer + P2 stubs); replay
   streams both cycles (328 events, ~36s)~~ ✅ — `DEMO=replay` is now default
4. Real P2 modules swapped into `lib/signals/registry.ts`; final live run
   refreshes cache. **Blocked on user:** valid `COMPOSIO_API_KEY` (current one
   401s) or `SLACK_WEBHOOK_URL` in `.env.local`, then `POST /api/ship` goes live

---
> Source: [garvit1910/backtrack-openai-hackathon](https://github.com/garvit1910/backtrack-openai-hackathon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
