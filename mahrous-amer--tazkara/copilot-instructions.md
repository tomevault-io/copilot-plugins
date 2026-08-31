## tazkara

> Instructions for AI coding agents (and fast-moving humans) working in or with

# Agent guide

Instructions for AI coding agents (and fast-moving humans) working in or with
this repo. The [README](README.md) explains what the tool does; this file
covers how to call it programmatically and how the code is organised.

## Calling the CLI programmatically

```bash
node tazkara.js --movies --json                  # discover valid --movie values
node tazkara.js --movie "<title>" --date <d> --seats <n> --json
```

- Always pass `--json` when consuming output from a program. stdout carries
  exactly one JSON object; diagnostics (chain failures, ambiguous titles,
  skipped chains) go to stderr. Exit code 0 on success — including "no
  sessions matched", which is `"suggestion": null`, not an error. Exit code 1
  with `Error: ...` on stderr for fatal problems (unknown flag, bad date, movie
  not found anywhere). The movie-not-found error message lists every currently
  showing title — use it to correct the query.
- `--movie` is fuzzy (case/punctuation-insensitive; exact > prefix > substring
  > all-words). When unsure of a spelling, run `--movies --json` first and pick
  a title from the listing.
- The JSON schemas for both modes are documented in the README under *Using
  tazkara from a script or AI agent*.
- Every run hits the two cinema sites live (catalogue + showtimes + one seat
  map per candidate session). Don't call it in a tight loop; nothing is cached.
- The `checkoutUrl` in the suggestion is for a human to open in a browser.
  The tool deliberately never reserves seats or pays, and agents driving it
  should preserve that boundary.

## Repo map

| File | Role |
|---|---|
| `tazkara.js` | CLI entry: arg parsing, movie resolution, orchestration, report/JSON output |
| `premiere.js` | Adapter for premiere-cinemas.com (encrypted JSON API + homepage catalogue) |
| `scene.js` | Adapter for district5.scenecinemas.com (HTML scraping + seat-plan endpoint) |
| `movies.js` | Pure fuzzy title matcher shared by both chains |
| `seats.js` | Seat-map normalisation to a common grid + best-seats picker |
| `dates.js` | Loose date token → `YYYY-MM-DD` |
| `config.js` | Site constants (API keys shipped in the sites' own JS), default movie, format ranking |
| `test.js` | Offline unit tests (`npm test` / `node --test`) |

## Conventions

- Plain CommonJS, Node ≥ 18, **zero runtime dependencies** — keep it that way.
  `fetch` and `crypto` are the built-ins.
- Adapters return a common session shape: `{ chain, cinemaName, format, date,
  time (24h "HH:MM"), ... }` plus chain-specific IDs the seat/checkout steps
  need. Catalogue entries are `{ id, title }` regardless of chain.
- Network code and parsing are separated: parsers (`parseCatalogue`,
  `parseShowtimes`, normalisers) are pure and exported so tests run offline
  with inline fixtures. If you change a parser, update its fixture in
  `test.js` from a real response.
- Warnings use `note()`/`warn()` in `tazkara.js` and go to stderr so stdout
  stays parseable.
- Run `npm test` before committing. Tests must not touch the network.

## Things that break

- The sites control both HTML and crypto material. If Premiere rotates the
  AES key/IV/JWT in its bundle, extract the new values into `config.js`
  (see comments there for where they live in the bundle).
- Premiere's catalogue comes from the Next.js flight payload embedded in the
  homepage HTML (`parseCatalogue` in `premiere.js`) — records open with
  `movie_unique_identifier` and carry `movie_title` in the same object. If the
  homepage structure changes, fix that parser first; everything else flows
  from the catalogue.

---
> Source: [mahrous-amer/tazkara](https://github.com/mahrous-amer/tazkara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
