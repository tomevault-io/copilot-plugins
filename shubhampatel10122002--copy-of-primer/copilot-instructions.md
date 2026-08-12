## copy-of-primer

> Read **PLAN.md** first. Follow its build order and conventions.

# Primer

Read **PLAN.md** first. Follow its build order and conventions.

## The one rule

**Deterministic code decides what happens; the LLM only decides what words to say.**

Mode transitions, scoring, coaching thresholds, mastery math, and safety gates are
plain TypeScript. The narrator LLM never chooses the next mode — it is told the
mode and asked for words.

## Layout

| Path | What lives there |
|---|---|
| `server/index.ts` | WebSocket server (port 3001). One session per connection. |
| `server/session.ts` | The state machine. Owns modes, timers, half-duplex gate, persistence. |
| `server/tracker.ts` | Word-by-word passage following: best-attempt scoring, repeats, reading ahead. |
| `server/azure.ts` | Pronunciation assessment (reading) + plain STT (talking). |
| `server/cartesia.ts` | Streaming TTS over the raw WebSocket, with per-context cancel for barge-in. |
| `lib/leniency.ts` | Developmental-speech table. Extend this during kid testing. |
| `lib/pedagogy.ts` | Mastery math and target selection. Pure, no LLM. |
| `lib/skills.ts` | The skill list and word→skill mapping. |
| `lib/llm/*` | Narrator, planner, intent router, safety pass, consolidation. |
| `app/api/*` | REST surface (§13). |
| `components/*` | Session UI + debug panel. |

## Conventions

- **Model IDs** live in `lib/env.ts` (`MODELS`). Sonnet for anything the child
  hears; Haiku for classification and the safety pass.
- **Never call the Anthropic API without `lib/llm/client.ts`.** It normalises
  `ANTHROPIC_BASE_URL`, which is set without `/v1` on any machine with Claude
  Code installed and otherwise 404s every request.
- **Audio**: mic is 16kHz mono PCM16 (Azure's requirement); TTS is 44.1kHz mono
  float32 (Web Audio's native format). Constants in `lib/env.ts` (`AUDIO`).
- **Wire protocol**: binary frames are audio (mic up, TTS down), text frames are
  JSON `ClientMessage` / `ServerMessage` from `lib/types.ts`.
- **`reading_events` is append-only.** Never UPDATE or DELETE.
- Only `attempt = 1` results update mastery, so coached retries can't inflate it.

## Commands

```bash
npm run db:reset   # drop, migrate, seed the demo child
npm run selftest   # deterministic core, no network or DB needed
npm run smoke      # verifies Postgres, Anthropic, Azure, Cartesia credentials
npm run dev        # Next.js on :3000 + WS server on :3001
```

Run `npm run selftest` after touching `tracker.ts`, `leniency.ts`, `pedagogy.ts`,
or `skills.ts` — those four files carry the behaviour that is hardest to eyeball
and easiest to break.

## Gotchas found the hard way

- The published `@cartesia/cartesia-js` pins `Cartesia-Version: 2024-06-10`,
  which predates the `sonic-3` family. We talk to the WebSocket directly.
- Azure's typed `detailResult` omits per-phoneme scores. Parse the raw
  `SpeechServiceResponse_JsonResult` instead (`server/azure.ts` does).
- Leniency false positives are expensive: forgiving a word silently switches
  coaching off. `lib/leniency.ts` only forgives with evidence of the actual
  substitution, or from a short list of the best-attested ones.

---
> Source: [shubhampatel10122002/Copy-of-Primer](https://github.com/shubhampatel10122002/Copy-of-Primer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
