## benchmark

> Benchmarking playbook — HTTP load testing with oha and Server-Timing profiling. Update this rule when new benchmarking lessons are learned.


# Benchmarking Playbook

When you discover a new benchmarking technique, pitfall, or best practice, add a concise bullet to the relevant section below in the same session.

## Tools

- **CPU microbenchmarks**: mitata (`bun add -d mitata`) — picosecond-accurate, JIT-aware, boxplot/lineplot/barplot/summary visualizations
- **HTTP load testing**: oha — Rust-based, real-time TUI, percentile distribution, supports POST/headers/JSON bodies
- NEVER roll your own `performance.now()` loops — mitata handles warmup, GC, JIT deopt detection, and batching correctly

## mitata essentials

- Always use `do_not_optimize(returnValue)` to prevent dead code elimination — JIT will remove pure functions with no side effects
- Use generator functions with `state.get()` + `.args()` / `.range()` for parameterized benchmarks instead of duplicating bench calls
- Use computed parameters (`[0]() { return val }, bench(p0) { ... }`) to prevent loop invariant code motion optimization
- Use `.gc('inner')` for allocation-heavy benchmarks to get consistent results
- Wrap related benchmarks in `summary(() => { ... })` to get automatic comparison (Nx faster/slower)
- Wrap summaries in `boxplot()` for distribution visualization — use `lineplot()` for scaling curves
- Run with `--expose-gc` on Node for accurate GC metrics; Bun exposes GC natively
- Install `@mitata/counters` on Apple Silicon or Linux for IPC and cache stats
- Watch for the `!` warning in output — it means the benchmark was likely optimized out (dead code elimination)

## oha essentials

- Basic: `oha -n 1000 -c 50 URL` (1000 requests, 50 concurrent)
- Duration-based: `oha -z 30s -c 100 URL`
- POST with JSON: `oha -m POST -d '{"key":"value"}' -T application/json URL`
- Custom headers: `oha -H "Authorization: Bearer token" -H "x-api-key: key" URL`
- Rate-limited: `oha -q 200 -z 30s URL` (200 req/sec for 30s)
- JSON output for CI: `oha -n 500 --json URL`
- Disable TUI for scripting: `oha --no-tui -n 500 URL`
- Use `--latency-correction` for coordinated omission correction

## Writing good benchmarks

- Test one thing per bench — isolate the function under test from setup, I/O, and side effects
- Always benchmark both the hit path AND the miss path (match vs no-match, cache hit vs miss)
- Vary input sizes — use `.args()` or `.range()` to expose scaling characteristics (O(n) vs O(1))
- Include a trivial baseline (e.g. a simple boolean flag eval) so relative cost is obvious
- Pre-build fixtures outside bench scope — never allocate inside the measured function unless testing allocation
- Test with realistic data shapes and sizes, not just minimal inputs
- Test worst-case scenarios explicitly (max rules, max batch values, deepest nesting)
- Simulate real production workloads (mixed flag types, multiple contexts)

## Structuring benchmark files

- Name files `*.bench.ts` next to the code they test
- Group related benchmarks with `summary()` inside `boxplot()` blocks
- Order sections from primitives → single operations → scaling → bulk → worst case → production simulation
- Use factory functions for test data to keep the file readable
- Export nothing — the file should be self-contained and runnable with `bun path/to/file.bench.ts`

## Interpreting results

- Sub-nanosecond (< 1 ns) results often indicate dead code elimination — add `do_not_optimize()`
- If p99 is wildly higher than avg, GC pressure or JIT recompilation is happening
- Compare within the same `summary()` block — cross-block comparison is unreliable due to CPU thermal state
- Linear scaling in `lineplot()` is expected for O(n) operations — unexpected curves indicate algorithmic issues
- For HTTP tests, focus on p99 latency under load, not just throughput

## Server-Timing profiling

- Use `@elysiajs/server-timing` to get per-phase durations in response headers — no code instrumentation needed
- Read with `curl -sf -D - -o /dev/null URL` to see `Server-Timing` header breakdown
- Phases: `request` (CORS/parsing), `beforeHandle` (auth/middleware), `handle` (your handler), `afterHandle` (response transform), `total`
- Use to identify whether latency is in middleware, cache lookup, DB query, or evaluation logic
- Run locally with `dotenv -e .env -- bun --hot apps/api/src/index.ts --port 3001` to test with real env vars
- Compare cold (first request, cache miss) vs warm (subsequent, cache hit) to see cache effectiveness

## HTTP benchmarking with oha against real servers

- Prefer hitting the actual running server (dev or prod) over building mock servers — measures the real stack
- Warm up caches before benchmarking by sending a few throwaway requests first
- Subtract DNS+dialup time from response time to see actual server processing time
- p99 tells you more than average — a 130ms avg with 400ms p99 means tail latency issues
- Use `unset NO_COLOR` before oha if running in shells that set it (breaks `--no-tui`)
- Keep the oha script as a `.sh` file next to the route it tests, taking clientId as an argument
- Always test error paths (missing params, 404s) — they should be faster than happy paths
- p99 spikes on otherwise fast endpoints usually mean cache TTL expiry hitting Redis/DB — tune TTL or use stale-while-revalidate

## Common pitfalls

- Benchmarking with module-level side effects (DB connections, Redis) — importing a route file that calls `cacheable()` at module level triggers Redis connections. Don't extract pure functions into separate files just for benchmarks — keep the code simple, use oha against the running server instead
- `Vary: *` header from CORS plugins kills CDN caching — override with `Vary: Origin` on cacheable endpoints
- Comparing results across different machines or thermal states
- Not varying input sizes — a function may be O(1) for small inputs but O(n^2) at scale
- Running benchmarks while other processes compete for CPU
- Over-engineering benchmark setups — a simple shell script with oha against prod is better than a mock server with fake data
- Running local dev without `dotenv` — Redis/DB errors from missing env vars make results meaningless

---
> Source: [databuddy-analytics/Databuddy](https://github.com/databuddy-analytics/Databuddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
