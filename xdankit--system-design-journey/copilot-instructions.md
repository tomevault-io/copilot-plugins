## bench

> k6 bench inputs and the result table


# Bench

Use k6. Do not add autocannon, vegeta, or wrk2.

The CLI asks, in order: scaling, processes, req/s, duration, compression. Phase 1 runs vertical only.

Compression choices: `all`, `off`, `gzip`, `brotli`, `zstd`. `all` means `off`, `gzip`, `brotli`, and `zstd`. Level is 1.

Each server gets its own table. Do not mix servers in one table. Do not mix vertical and horizontal in one table.

| Column | Meaning |
|---|---|
| Compression | `off` or the algorithm at level 1 |
| Req/s | Achieved rate |
| Data out | Bytes on the wire per second |
| CPU | Server process CPU, not the k6 process |
| p95 | Latency |
| Pass | Responses that succeeded |
| Fail | Responses that failed |

After the tables, show a winner: fail rate under 1 percent, then the highest req/s. If req/s ties, the lower p95 wins.

---
> Source: [xDAnkit/system-design-journey](https://github.com/xDAnkit/system-design-journey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
