## slowjson

> > Slow JSON Stream: low-bandwidth DoS research tool against HTTP APIs with JSON bodies.

# slowjson — Claude Code Context

> Slow JSON Stream: low-bandwidth DoS research tool against HTTP APIs with JSON bodies.
> 32 framework testbed, full experiment matrix, LaTeX paper.
> Last updated: 2026-04-20

## Commands

```bash
make install     # pip install -e 'cli/[dev]' — required before any CLI use
make up          # docker compose up -d --build (all 32 servers)
make down        # stop + remove containers and volumes
make rebuild     # force rebuild every server image (--no-cache)
make smoke       # quick sanity: starts python-fastapi, runs 16-conn 20s attack
make matrix      # full experiment — ~12 h, writes results/runs/<RUN_ID>/
make analyse     # summarise a run: RUN_DIR=results/runs/<id>
make test        # pytest in cli/
make attack      # one-off cell: override TARGET, PROBE, CONT, PAYLOAD, CONN, DUR
make clean       # remove __pycache__, build artefacts
```

Matrix example (note: `--results-dir`, NOT `--output`):
```bash
slowjson matrix --scenarios orchestrator/scenarios.yaml --results-dir results/ --i-own-this-server
```

One-off attack example:
```bash
slowjson attack http://localhost:8001/ingest \
    --payload A --bytes-per-tick 1 --tick-interval-ms 1000 \
    --connections 64 --duration 90 \
    --probe-url http://localhost:8001/healthz \
    --docker-container slowjson-python-fastapi \
    --output results/single.json \
    --i-own-this-server          # ← required, safety gate
```

## Extended testbed (proxies / WAFs / API gateways)

9 additional targets on ports 8033–8041. All route to `stub-backend` (internal service, no port mapping).
Use the standalone scenario file — no `--scenario` filter exists in the CLI:

```bash
docker compose up -d stub-backend nginx-proxy caddy-proxy haproxy traefik \
    waf-modsecurity-nginx waf-apache-modsec waf-coraza kong-ce envoy-proxy
slowjson matrix --scenarios orchestrator/scenarios_extended.yaml \
    --results-dir results/ --i-own-this-server
```

Anti-FP rule for WAFs/proxies: `first_kill_ms ≤ 3000 AND error_rate=1.0 AND rss≈0` → Tier 4 (defensive kill, not crash).

## Architecture

```
cli/                 # slowjson Python package (entry point: slowjson.cli:main)
  slowjson/
    cli.py           # argparse subcommands: attack, matrix, analyse
    attack.py        # core attack engine (chunked drip, probe, kill tracking)
    report.py        # build_report() + evaluate_verdict() — the 4 verdict rules
    orchestrator.py  # matrix runner: expands scenarios.yaml, calls attack.py
    metrics.py       # docker stats collector + probe summariser
    payloads.py      # 4 JSON payload shapes: A (array), N (nested), O (object), S (string)
    safety.py        # --i-own-this-server guard — do not remove
orchestrator/
  scenarios.yaml     # full 32×4 matrix definition
servers/             # 32 Dockerfiles, one per framework variant
results/             # gitignored output (runs/, summary.csv, report.md)
paper/
  paper.tex          # main LaTeX source — compile with pdflatex (two passes)
  tables/            # generated .tex tables — gitignored, use git add -f to commit
  figures/           # generated PDFs — gitignored
  disclosure.md      # responsible disclosure timeline
docs/
  attack-diagram.svg # branded attack-flow diagram (820×470)
scripts/
  generate_figures.py        # reads results/summary.csv → paper/figures/*.pdf
  generate_attack_diagram.py # regenerates docs/attack-diagram.svg
```

## Verdict Rules (report.py)

Four rules classify each experiment cell:

| Rule | Condition | Signal |
|------|-----------|--------|
| 1 | probe p99 ≥ 5× baseline | latency degradation |
| 2 | probe error_rate ≥ 10% | service errors |
| 3 | RSS slope ≥ 5 MB/min | unbounded memory growth |
| 4 | no connection killed within 90 s | no body timeout |

**Tiers** (not the same as VULN/not-VULN):
- **Tier 1** — observed degradation at C=64 (Rules 1/2/3): PHP, .NET, Java (7 frameworks)
- **Tier 2** — Rule 4 only, latent exposure: async/goroutine runtimes (23 frameworks)
- **Tier 3** — hard crash at C=64: python-flask, ruby-rails
- **Tier 4** — resistant by default: go-fiber (buffer ceiling), python-fastapi-streaming (streaming parser)

`go-fiber` raw verdict is VULN (error_rate=100%) but is Tier 4 — fasthttp's read-buffer kills slow connections in ~2 s; probe errors are a side-effect of the defence, not failure.

## Paper

```bash
cd paper
pdflatex -interaction=nonstopmode paper.tex   # pass 1 — builds .aux
pdflatex -interaction=nonstopmode paper.tex   # pass 2 — resolves cross-refs
```

`paper/tables/*.tex` and `paper/figures/*.pdf` are in `.gitignore`.
To commit them: `git add -f paper/tables/ paper/figures/`.

Regenerate tables + figures after a new matrix run:
```bash
python3 scripts/generate_figures.py          # needs results/summary.csv
# tables are written by: slowjson analyse --run-dir ... --out-dir results
```

## Gotchas

- **`--i-own-this-server` is mandatory** for all `slowjson attack` and `make matrix` calls. The safety module blocks execution without it.
- **Go servers `go.sum`**: `servers/*/go.sum` is gitignored. Run `go mod tidy` locally inside the server dir first, then `git add -f servers/<name>/go.sum`. Dockerfile must copy both: `COPY go.mod go.sum ./`.
- **Coraza v3 API**: `Interruption.Action` is a plain `string` field (`"deny"`), not a method. `types.ActionTypeDeny` does not exist. `ProcessRequestHeaders()` returns `*Interruption` (no error return).
- **Kong Docker image**: use `FROM kong:latest` — tagged minor versions like `kong:3.7-alpine` are not published on Docker Hub. for all `slowjson attack` and `make matrix` calls. The safety module blocks execution without it.
- **Matrix duration**: full 32-target × 4-payload × 3-repeat run takes ~12 h on a single machine. Use `make smoke` for a quick sanity check.
- **Java JVM warmup**: baseline p99 for java-spring and java-spring-webflux is 300–450 ms due to JIT warmup. Attack p99 appears *lower* than baseline — this is not improvement, it is warmup completing. RSS slope is the reliable signal for Java.
- **Docker memory**: PHP/Laravel hits 258 MB/min RSS growth at C=64. Ensure the host has ≥ 2 GB free RAM before running the full matrix.
- **Embargo**: research is under embargo until **2026-07-19**. Do not publish results, CVE details, or the paper before that date.

## Environment

- Python ≥ 3.11, Docker with compose plugin
- No `.env` file needed — all config is in `orchestrator/scenarios.yaml` and Makefile vars
- `results/` is gitignored; commit `results/summary.csv` and `results/report.md` explicitly if needed

## License & disclosure

| Component | License |
|---|---|
| Code (`cli/`, `servers/`, `orchestrator/`) | Apache-2.0 |
| Paper and figures (`paper/`) | CC-BY-4.0 |

Disclosure plan: `paper/disclosure.md`. Notifications sent 2026-04-20, embargo lifts 2026-07-19.

---
> Source: [cr0hn/slowjson](https://github.com/cr0hn/slowjson) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
