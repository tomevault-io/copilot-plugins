## localquiz

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
make repl       # Start nREPL with CIDER middleware (clj -M:repl:dev)
make test       # Run tests once (clj -X:dev:test)
make retest     # Run tests on file changes (clj -M:dev:test-refresh)
make build      # Build uber-JAR to target/app.jar (clj -Srepro -T:build uber)
```

There is no single-test command; use the REPL to run individual tests.

## Architecture

**localquiz** is a server-driven, real-time multiplayer quiz engine. The server renders all HTML (Hiccup/Chassis), pushes UI updates to clients via SSE using the [Datastar](https://data-star.dev/) library.

### Request/Update cycle

1. HTTP request → `handler.clj` (Reitit router) → middleware → action or view
2. Actions (in `actions/`) transact to the Datahike database
3. `db_listener.clj` subscribes to Datahike transactions and publishes to a core.async pub/sub topic per game
4. `sse.clj` subscribes to those topics (throttled to 100ms), re-renders the view, diffs the HTML hash, and sends `patch-elements!` deltas to all connected clients

### State management

- **Mount** manages stateful components: HTTP server, Datahike connection, DB listener, async channels
- **Datahike** (embedded Datomic-like DB with LMDB backend) is the single source of truth
- Game state machine: `:new` → `:question` → `:show-answers` → `:leaderboard` (cycles back for next question)

### Key namespaces

| Namespace | Role |
|-----------|------|
| `core` | Entry point, Mount lifecycle, virtual thread setup |
| `handler` | Ring handler, Reitit router, middleware stack |
| `routes` | All URL route definitions |
| `middleware` | CSRF, session, language, Datastar signals parsing |
| `game` | Core game mechanics: state transitions, player/answer management |
| `scoring` | Multimethod dispatch on question type for score calculation |
| `question_spec` | Clojure spec definitions for question EDN format |
| `db` | Datahike schema and connection |
| `db_listener` | Database transaction listener → async pub |
| `async` | core.async pub/sub channels for game refresh events |
| `sse` | SSE handler, view hashing/diffing, Brotli compression |
| `actions/moderator` | Game creation, question flow control |
| `actions/player` | Player joining, answer submission |
| `views/` | Hiccup HTML generation (common, moderator, player) |

### Question types

Six types supported: `:multiple` (multiple choice), `:yesno`, `:open` (fuzzy text match via Jaro-Winkler), `:percent-range` (numeric with tolerance), `:sort` (order items), `:consensus` (crowd-based correctness). Each has a scoring multimethod in `scoring.clj`.

## Configuration

- `resources/default_config.edn` — built-in defaults (port 8080, 45s timeout, 100ms SSE throttle)
- `resources/.config.edn` — runtime overrides, git-ignored
- `test/resources/.config.edn` — test overrides
- JVM property `app.env=dev` enables hot reload and localhost URLs

## Questions

Question sets are EDN files in `resources/questions/`. Each file contains a vector of question maps conforming to the spec in `question_spec.clj`.

## Testing

Tests use cognitect test-runner. `test_fixtures.clj` provides a Datahike in-memory database. Handler tests use ring-mock. Integration tests live in `test/src/.../integration/`.

## Constraints

- A game has one moderator and at least two players.

---
> Source: [jindrichmynarz/localquiz](https://github.com/jindrichmynarz/localquiz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
