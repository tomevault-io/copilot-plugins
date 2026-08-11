## cadence

> Cadence is a **zero-dependency Swift 6 concurrency library** for pacing outbound work: rate

# Cadence

Cadence is a **zero-dependency Swift 6 concurrency library** for pacing outbound work: rate
limiting, debouncing, and request coalescing. It was extracted and generalized from a production
weather app and deliberately decoupled from any HTTP client.

This file is the always-on brief for AI agents. It holds essentials and pointers; deep rationale
lives in `docs/adr/` and step-by-step workflows live in `.claude/skills/`.

## Knowledge map — where the details live

- **Design rationale** → `docs/adr/` (Architecture Decision Records). Read the relevant ADR
  before changing a foundational choice (dependencies, Get-decoupling, actor model, rate-limit
  algorithm).
- **Architecture walkthrough** → `docs/architecture.md`.
- **HTTP-client usage** → `Examples/GetIntegration.md`.
- **Workflows** (invoke as `/name`) → `.claude/skills/`: `red-gate` (TDD), `release` (cut a
  version), `add-rate-limit-strategy` (add a `RateLimiter`), `mutation-review` (Muter-driven test
  quality).
- **Validators / triagers** (read-only subagents) → `.claude/agents/`:
  `concurrency-invariant-validator`, `public-api-doc-checker`, `mutation-survivor-triage`.

## Quick start

```bash
swift build          # compile (Swift 6 language mode, strict concurrency)
swift test           # run the Swift Testing suite (fast, deterministic)
swiftlint lint       # style (also runs in CI; optional locally)
```

There is no Xcode project — this is a pure SwiftPM package. Everything is driven from the CLI.

## Architecture

Three pillars, all under `Sources/Cadence/`:

1. **Scheduling** (`Scheduling/`) — `RequestScheduler` (actor) is the entry point:
   `schedule` (rate-limited one-off), `debounce` (collapse rapid repeats), and `coordinate`
   (coalesce concurrent similar requests). `RequestScheduling` is its protocol;
   `RequestCombinable` is the strategy for keying/merging/splitting coalesced requests.
2. **Rate limiting** (`RateLimiting/`) — `SlidingWindowRateLimiter` (actor) enforces any number
   of `RateLimitRule`s, each `.wait` or `.throw`. `RateLimiterStorage`
   (`UserDefaults`/in-memory) persists the window across launches. `RateLimiter` is the protocol.
3. **Support** (`Support/`, `Logging/`) — `CadenceClock` (time abstraction; `SystemClock` in
   prod, virtual clock in tests), `CadenceLogger` (pluggable, `NoopLogger` default),
   `Array.unique()`.

The library returns **generic `T`** from `schedule`/`coordinate` — it knows nothing about HTTP.
That decoupling is intentional; see `docs/adr/0002-decouple-from-get.md`.

## Conventions

- **Swift 6, strict concurrency.** Both targets use `.swiftLanguageMode(.v6)`. Code must compile
  with no concurrency warnings. Shared mutable state lives inside actors.
- **Tests use Swift Testing**, never XCTest: `import Testing`, `@Test`, `#expect`, `Issue.record`.
  Prefer the injected `TestClock` over real sleeps so tests are deterministic and instant. Real
  timing is acceptable only for the coalescing/debounce concurrency tests where virtual time
  can't model the race (see `RequestSchedulerTests`).
- **Document every `public` symbol** with a doc comment explaining *why* it exists, plus a short
  example on the important ones. `public-api-doc-checker` enforces this.
- **No runtime dependencies.** A dependency-free core is a feature (ADR-0001). Don't add one.
- **Style**: `.swiftlint.yml` (120-col warning). CI runs `swiftlint --strict`.

## Critical invariant — coalescing lifecycle

`RequestScheduler.startNewBatch` clears the shared batch state (`activeRequests` /
`activeParameters`) **before** the underlying `execute` call, never in a `defer`. Clearing late
would let a request arriving during execution merge into an already-captured batch and receive an
empty result. If you touch coordination, keep this ordering and keep its tests
(`coordinateCoalesces`, `coordinateStartsFreshBatch`). Run `/concurrency-invariant-validator`
after any change under `Scheduling/`.

## Releasing

Use the `/release` skill: bump the version, move `CHANGELOG.md` **Unreleased** entries under the
new version + date, tag `MAJOR.MINOR.PATCH`, and push. Follow SemVer — any change to a `public`
signature is at least a minor, a breaking one is a major.

## Directory map

```
Sources/Cadence/
  Scheduling/     RequestScheduler, RequestScheduling, RequestCombinable
  RateLimiting/   SlidingWindowRateLimiter, RateLimiter, RateLimitRule, RateLimitError,
                  RateLimiterStorage (+ UserDefaults / InMemory)
  Logging/        CadenceLogger (+ Noop / Print)
  Support/        CadenceClock (+ SystemClock), Array+Unique
Tests/CadenceTests/
  *Tests.swift            one suite per area
  TestSupport/            TestClock, PassthroughRateLimiter, ImmediateScheduler, IntBatchService
docs/adr/          Architecture Decision Records
Examples/          HTTP-client integration guides
.claude/           agents, skills, settings
```

---
> Source: [peteroettl/Cadence](https://github.com/peteroettl/Cadence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
