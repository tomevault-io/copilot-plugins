## redis-search

> Response shapes for Search and Autocomplete with exact_match feature


# Exact Match Response Shapes

## Overview

The exact match feature adds a new `exact_match` property to search and autocomplete responses. When the user's query exactly matches a known entity (e.g., "the godfather" → movie "The Godfather"), that match is surfaced in a dedicated field.

---

## Non-Stream Responses (JSON)

**Endpoints:** `/api/search`, `/api/autocomplete`

### Before (current)

```json
{
  "tv": [...],
  "movie": [...],
  "person": [...],
  "podcast": [...],
  "author": [...],
  "book": [...],
  "news": [...],
  "video": [...],
  "ratings": [...],
  "artist": [...],
  "album": [...]
}
```

### After (evolved)

```json
{
  "exact_match": { "mc_id": "tmdb_movie_238", "mc_type": "movie", ... },
  "exact_matches": [
    { "mc_id": "tmdb_movie_238", "mc_type": "movie", ... },
    { "mc_id": "tmdb_tv_216645", "mc_type": "tv", ... }
  ],
  "tv": [...],
  "movie": [...],
  "person": [...],
  "podcast": [...],
  "author": [...],
  "book": [...],
  "news": [...],
  "video": [...],
  "ratings": [...],
  "artist": [...],
  "album": [...]
}
```

**`exact_match` field:**
- **Type:** `ExactMatchItem | null`
- **When present:** At least one exact match found; contains the single best match (hero pick).
- **Media tie-break:** For `movie`/`tv`, when multiple exact matches exist:
  1. **Viable filter:** Candidates without watch providers are eliminated — unless they are movies released within the last ~6 months (theatrical window) or unless no candidates survive the filter.
  2. **Sort by effective date (YYYYMM) descending:** `release_date` for movies, `last_air_date` for TV, rounded to year-month (day dropped). Most recent month-bucket wins.
  3. **Within same YYYYMM:** higher `popularity` wins.
  4. **Within same YYYYMM and popularity:** movie before TV.
- **Non-media fallback:** After media, non-media exact matches fall back to source priority (`person > podcast > book > author`).
- **When absent:** Omit key, or use `"exact_match": null` for consistent typing (recommend always include, use `null` when none)
- **Cast shape:** For `exact_match`, `cast` is now `Array<{ name: string, id: string | null }>` (name + TMDB ID).

**`exact_matches` field:**
- **Type:** `ExactMatchItem[]`
- **Contents:** All items across all sources that qualified as exact matches, ranked (media items by `score_media_result`, non-media by source priority).
- **Includes the hero pick:** The `exact_match` winner is included in this array. The UI filters it out at render time to avoid visual duplication.
- **When empty:** `[]` (no exact matches found).

---

## Streaming Responses (SSE)

**Endpoints:** `/api/search/stream`, `/api/autocomplete/stream`

### Event Types

| Event      | When                         | Data shape                                                    |
|-----------|-------------------------------|--------------------------------------------------------------|
| `result`  | Each source completes         | `{ source: string, results: MCBaseItem[], latency_ms: number }` |
| `exact_match` | When a source yields an exact match | `ExactMatchItem` (single provisional item)                      |
| `exact_match_final` | After all sources complete and the backend selects the best exact match | `ExactMatchItem` (single authoritative hero pick) |
| `exact_matches_final` | After all sources complete | `ExactMatchItem[]` (all qualifying exact matches, ranked) |
| `done`    | All sources complete          | `{}`                                                         |

### Event sequence example

```
event: result
data: {"source":"movie","results":[...],"latency_ms":12}

event: exact_match
data: {"mc_id":"tmdb_movie_238","mc_type":"movie","search_title":"The Godfather",...}

event: result
data: {"source":"tv","results":[...],"latency_ms":15}

event: exact_match_final
data: {"mc_id":"tmdb_movie_238","mc_type":"movie","search_title":"The Godfather",...}

event: exact_matches_final
data: [{"mc_id":"tmdb_movie_238","mc_type":"movie",...},{"mc_id":"tmdb_tv_216645","mc_type":"tv",...}]

event: done
data: {}
```

**`exact_match` event behavior:**
- Emitted **as soon as** a source completes and yields at least one exact match
- May fire **multiple times** per stream (e.g., movie exact match, then person exact match)
- Provisional only; clients may surface it immediately
- `exact_match` events can appear between `result` events (typically right after the `result` that contained the match)

**`exact_match_final` event behavior:**
- Emitted at most once per stream, after all sources have completed
- Uses the same backend selection logic as non-stream `exact_match`
- Intended to be authoritative; clients should replace any provisional exact match with this final one when present

**`exact_matches_final` event behavior:**
- Emitted at most once per stream, after all sources have completed
- Contains the full ranked array of all qualifying exact matches (including the hero pick)
- Clients should replace any accumulated `exact_matches` with this authoritative list

---

## TypeScript Types (Mobile / Clients)

All types are now defined in `src/contracts/*.ts` with a barrel export from `src/contracts/index.ts`.

```typescript
import type {
  SearchResponse,
  ExactMatchItem,
  ExactMatchCastEntry,
  StreamResultEvent,
  StreamExactMatchEvent,
  StreamExactMatchesFinalEvent,
  StreamDoneEvent,
} from "@/contracts";
```

### Stream event handlers

```typescript
import type { StreamResultEvent, ExactMatchItem, StreamDoneEvent } from "@/contracts";

es.addEventListener("result", (e) => {
  const data: StreamResultEvent = JSON.parse(e.data);
  // data.source, data.results, data.latency_ms
});

es.addEventListener("exact_match", (e) => {
  const item: ExactMatchItem = JSON.parse(e.data);
  // Provisional exact match while sources continue to stream
});

es.addEventListener("exact_match_final", (e) => {
  const item: ExactMatchItem = JSON.parse(e.data);
  // Final authoritative exact match for the completed stream
});

es.addEventListener("done", (e) => {
  const data: StreamDoneEvent = JSON.parse(e.data);
  // data.source_hint may be present
  es.close();
});
```

---

## Backwards Compatibility

- **Non-stream:** Adding `exact_match` is additive; existing clients can ignore it
- **Stream:** `exact_match` remains additive, and `exact_match_final` is a new optional event; clients that only listen for `result` and `done` are unchanged

## Person autocomplete matching behavior

- Person index filtering now uses a dedicated matcher for query behavior.
- Query construction and filtering for people are intentionally person-specific:
  - Multi-word person queries now require each token to match separately so names with middle words still match (`megan therese rippey`).
  - Stopword-leading phrases such as `"the rip"` still avoid broad prefix behavior.
- Other index matchers (`is_autocomplete_match`) are unchanged for non-person uses.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emgould)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/emgould)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
