## rapiscm

> Rust API scanner. Point at an API spec or a URL to scan.

# rapiscm

Rust API scanner. Point at an API spec or a URL to scan.

## Status

Working. 8 subcommands, 60+ flags, ghost mode, JS bundle scanning, task system, tracker analytics.

## Quick start

```sh
cargo build --release
./install.sh [--browser]   # install binary + optional browser tools
cargo test                 # 68 unit tests
cargo clippy               # lint (no warnings expected)
cargo fmt                  # format
```

## CLI

```
rapiscm spec <file>          # scan from OpenAPI spec (JSON/YAML)
rapiscm url <url>            # scan a URL / API base URL
rapiscm scan <target>        # auto-detect
rapiscm fuzz <target>        # fuzz endpoints
rapiscm corp <name>          # domain discovery
rapiscm session <file>       # replay JSONL session
rapiscm tasks                # manage saved scans
rapiscm capture <url>        # capture page evidence

Flags:  -H/--header, --auth, --rate-limit, --timeout, --concurrency,
        -o/--output (table|json|md|doc), --follow-redirects, -k/--insecure,
        --paths, --tags, --proxy, --crawl (html|js|full), --ghost,
        --eval <js>, --ua-rotate, --jitter, --proxy-rotate
  (browser feature): --browser (chrome|firefox), --headed
```

## Features

- `browser`: headless browser discovery via chromiumoxide + geckodriver. Enable with `--features browser`.

## Project layout

```
src/
  main.rs           # tokio::main, dispatch to all subcommands
  cli.rs            # clap CLI definition (8 subcommands, 60+ flags)
  config.rs         # ScanConfig builder from Cli
  types.rs          # Target, Endpoint, ResponseResult, Check, etc.
  error.rs          # Error enum (thiserror)
  ghost.rs          # Ghost mode: UA rotation, jitter, header randomization, proxy rotation
  check/
    mod.rs          # run_checks() + run_async_checks()
    security.rs     # security header checks (CSP, HSTS, etc.)
    cors.rs         # CORS preflight probe
    auth.rs         # Auth-enforcement probe
    schema.rs       # Response status + body validation
    trackers.rs     # Cookie analysis (Secure/HttpOnly/SameSite/expiry)
  scan/
    runner.rs       # ScanRunner: concurrent HTTP + rate limiting + ghost jitter
    spec.rs         # spec-mode: parse spec → scan
    url.rs          # url-mode: discover endpoints → scan (crawl, JS bundles, ghost)
    browser.rs      # (feature-gated) Chrome/Firefox interactive discovery + JS eval
  parser/
    spec.rs         # openapiv3 → Vec<Endpoint>
    url.rs          # HTML link extraction, wordlist, API endpoint heuristics
    js_bundle.rs    # JS bundle download + API route extraction (11 patterns)
  report/
    mod.rs          # format_results() dispatcher
    table.rs        # Terminal (ANSI) + markdown table formatters
    json.rs         # JSON formatter
    summary.rs      # Summary statistics
    doc.rs          # Structured API documentation (llm-api-style)
    site.rs         # Static HTML report generator
  analytics/
    mod.rs          # TrackerAnalysis, dispatch
    detect.rs       # ~200-entry tracker signature DB + matching
    sigdb.rs        # Tracker signature database types
  filter/
    mod.rs          # Expression filters (tag:, status:, path:, method: + AND logic)
  tag/
    mod.rs          # tag_endpoint() and tag_response()
  extract/
    mod.rs          # Content-type dispatched URL extraction
    html.rs         # <a href>, <form action> parsing
    js.rs           # JS string literal URL extraction
    json.rs         # URL values from JSON responses
    headers.rs      # Location, Content-Location, Link headers
    sitemap.rs      # robots.txt, sitemap.xml parsing
  fuzz/
    mod.rs          # run_fuzz_scan, FuzzOpts
    runner.rs       # FuzzRunner: path fuzzing loop
    matcher.rs      # MatchConfig: status/size/regex + baseline
    wordlist.rs     # built-in API path wordlist (~100 entries)
  discover/
    mod.rs          # DiscoverConfig, run_discover
    crtsh.rs        # crt.sh Certificate Transparency search
    rdap.rs         # ARIN RDAP reverse WHOIS
    asn.rs          # BGPView ASN → IP → reverse DNS
    search.rs       # Google Custom Search dork-based discovery
    favicon.rs      # Shodan favicon hash search
    gaid.rs         # Google Analytics ID pivot
  session/
    mod.rs          # SessionRunner, SessionConfig, dispatch
    parse.rs        # JSONL line parser → ResponseResult
    timing.rs       # TimingAnalytics: gaps, bursts, rate-limit events
  task/
    mod.rs          # TaskMeta, ResultSummary, StorageInfo
    store.rs        # JSON file store: save/load/list/delete/prune
    index.rs        # index.json management
    queue.rs        # queue.json management, crash recovery
    rebuild.rs      # re-run from task.json config
    diff.rs         # TaskDiff algorithm + classification
    export.rs       # export to md/sarif/html
    resume.rs       # checkpoint-based partial re-run
install.sh          # installs binary + optional browser deps (--browser)
.github/workflows/ci.yml  # check, clippy, fmt, test
.agents/SPEC.md, PLAN.md  # design documents
.agents/plan.md, todo.md  # implementation plan
CHANGELOG.md              # release history
docs/decisions/ADR-*.md   # architecture decision records
```

## Conventions

- `cargo clippy` must pass before commit
- `cargo fmt` before diff
- Errors: `anyhow`/`thiserror`, propagate with context
- CLI: `clap` derive macros
- HTTP: `reqwest` with configurable timeouts
- Minimize external deps: manual ANSI codes, std time, AtomicU64 IDs

---
> Source: [grave0x/rapiscm](https://github.com/grave0x/rapiscm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
