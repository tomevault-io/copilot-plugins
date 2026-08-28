## pgcache-bench

> Context for coding agents working in this repo. Commands and procedures live

# Agent notes

Context for coding agents working in this repo. Commands and procedures live
in the READMEs — start with [README.md](README.md), and read
[bench/README.md](bench/README.md) **before quoting any benchmark numbers**.
AWS topology: [aws/README.md](aws/README.md); measurement runbook:
[aws/BENCHMARK.md](aws/BENCHMARK.md).

## Invariants — violating these produces wrong numbers, not errors

- **Never use a fixed-time warmup.** Warm until steady state: registration
  plateaued (`pgcache_cache_queries_registered` stops growing), windowed hit
  rate ≥ ~0.99, CDC queue drained (`pgcache_cache_writer_cdc_queue` ≈ 0).
  A cold or partially warm cache measures the registration storm and origin
  population, not cache serving.
- **Cross-check with origin CPU.** At read-only steady state the origin is
  near-idle. Elevated origin CPU during a read-only measure window means the
  run is invalid.
- **Don't restart a run mid-measure.** Warmup state is lost on driver or
  pgcache restart; start the warm-to-steady-state cycle over.

## Operational gotchas

- Results are **Prometheus-only** (`driver_*` metrics, `lane` label); the
  driver prints no end-of-run summary. See "Reading the results" in the README.
- Expect long-running commands: the data load is ~10-15 min (one-time — check
  whether the origin already has tables before re-running it) and real
  benchmark runs take minutes to hours. Run them in the background.
- `pkill -f` / `pgrep -f` over SSH match the launching shell's own argv —
  a silent no-op kill. Kill by explicit PID, or use `pgrep -x` with the
  15-char process name `pgcache-bench-d`.
- Shell quoting over SSH: a heredoc nested inside `bash -lc "…"` silently
  fails to write — use `ssh … 'bash -s' <<'EOS' … EOS` (quoted delimiter) or
  `scp` a script file. Parens inside `echo` under `bash -lc "…"` are a syntax
  error. zsh doesn't word-split unquoted vars — inline ssh `-o` options
  instead of expanding an options variable.
- `bench.sh warm` SIGTERMs the driver, which drains slowly under load. Before
  measuring, make sure exactly one driver process remains (`ps -C
  pgcache-bench-d`); kill stale ones by explicit PID (the older `ETIME`).
- The pinned pgcache image is multi-arch (amd64 + arm64); `PGCACHE_IMAGE`
  overrides it to benchmark a different release.

---
> Source: [PgCache/pgcache-bench](https://github.com/PgCache/pgcache-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
