## ollanta

> Multi-language static analysis platform in Go. Two deliverables:

# Ollanta - Agent Guide

Multi-language static analysis platform in Go. Two deliverables:

- **Scanner** (`ollantascanner`) - CLI: discovers files, parses (Go via `go/ast`; JS/TS/Python/Rust via tree-sitter), applies rules, writes JSON/SARIF reports to `<project>/.ollanta/`. Optional embedded UI on port 7777.
- **Server** (`ollantaweb`) - REST API on port 8080: ingests scan reports (asynchronously, via background workers), tracks issues across scans, evaluates quality gates.

Deeper docs: `docs/architecture.md`, `docs/contributing.md`, `CONTRIBUTIONS.md` (fast operational checklist). Change proposals use OpenSpec under `openspec/changes/`.

## Modules

`go.work` lists 9 modules. The root Makefile builds/tests 8 of them (all but `adapter/`, which needs a live Postgres).

| Module | Purpose | CGo |
|--------|---------|-----|
| `domain/` | Models, ports, domain services. stdlib only. | No |
| `application/` | Use cases (scan, ingest). Internal imports: `domain/` + `ollantacore/` only. | No |
| `adapter/` | Hexagonal adapters (`secondary/`: parser, rules bridge, telemetry). | Yes* |
| `ollantacore/` | Shared legacy types + `rulecatalog` (CGo-free rule catalog used by `ollantaweb`). | No |
| `ollantaparser/` | Tree-sitter bindings. **Only true CGo module.** | **Yes** |
| `ollantarules/` | Rule registry, sensors, embedded JSON metadata. | Yes* |
| `ollantascanner/` | CLI entrypoint (`cmd/ollanta`), discovery, executor, local UI server. | Yes* |
| `ollantastore/` | Postgres repos (pgx/v5), search (ZincSearch). | No |
| `ollantaweb/` | REST server (chi/v5), auth, ingest, webhooks. | No |

_*Transitive CGo via `ollantaparser`._

**Satellite modules outside `go.work`** - easy to miss; NOT covered by `make` or CI:

- `ollantaengine/` - quality gates, new-code tracking, summarizer. `ollantaweb` depends on it via `replace`. Run its tests yourself from inside the dir (set `$env:GOWORK='off'` if go complains it is not in the workspace).
- `tests/e2e/` - full scanner -> server -> ingest pipeline test with its own Makefile (`make test`). Builds the scanner, boots a disposable compose stack, skips when Docker is unavailable.

The server image (`ollantaweb/Dockerfile`, built with `CGO_ENABLED=0`) produces 5 binaries: `ollantaweb` (API), `ollantaworker` (scan-job processing - without it, pushed scans never complete), `ollantaindexer` (search projection), `ollantawebhookworker` (webhook delivery), `ollantamigrate` (one-shot schema migration).

## Commands

| Command | Does |
|---------|------|
| `make build` / `make test` | 8 workspace modules (excludes `adapter/`), CGO on |
| `make lint` | golangci-lint per-module on 7 modules - excludes `ollantaweb/` (entire dir excluded in `.golangci.yml`) and `adapter/` |
| `make run` / `run-bg` / `stop` | Scan + local UI on :7777 (vars: `PROJECT_DIR`, `PROJECT_KEY`, `PORT`) |
| `make push` | Scan + push with `-server-wait`. **Exits 3 when the server-side quality gate fails** - that is a gate failure, not an ingestion error |
| `make up` / `down` / `recreate` / `logs` | Compose `server` profile: postgres + zincsearch + web + 3 workers |
| `make smoke-local` | PowerShell end-to-end smoke (scanner -> server) on port 18080 (`SMOKE_BACKEND_PORT` to override). Preserves temp project + server log on failure |
| `make swagger` | Regenerates `ollantaweb/docs/` from swag annotations |
| `make release` / `release-dry-run` | Cross-compile the scanner for 5 platforms |

Traps:

- **Never run `golangci-lint` at the workspace root** - each module has its own `go.mod`. Run per-module, like the Makefile does.
- Windows: the Makefile prepends `C:\msys64\mingw64\bin` to PATH **for its own targets only**. For direct `go` / `golangci-lint` on CGO packages: `$env:CGO_ENABLED='1'; $env:PATH = 'C:\msys64\mingw64\bin;' + $env:PATH`.
- Scanner UI (TypeScript): `cd ollantascanner/server/static && npm test && npm run build`.
- Server SPA (vanilla JS): `node --test app.test.js` in `ollantaweb/api/static/`.
- Web e2e (`ollantaweb/e2e/`, Playwright) needs the full `server` profile up **including `ollantaworker`** - ingestion is async, so issue tests hang or show empty states without it. Seeded login: `admin` / `admin`.

## After a change, validate

| Changed area | Run |
|--------------|-----|
| Scanner / rules / parser (CGO) | `make build && make test` |
| Web / hexagonal (no CGO) | `go build ./domain/... ./application/... ./ollantastore/... ./ollantaweb/...` and `go test` on the same |
| `ollantaengine/` | `go test ./...` inside that dir (not covered by make/CI) |
| `adapter/` | `go test ./adapter/...` with `DATABASE_URL` pointing at Postgres |
| Server behavior | `docker compose --profile server build ollantaweb`, then `make smoke-local` |
| Scanner UI `src/` | `npm run build` and commit `dist/` (embedded in the binary) |

`ollantastore/postgres` tests **skip silently** unless `DATABASE_URL` (or `OLLANTA_TEST_DATABASE_URL`) is set - a green `make test` does not mean the DB layer was exercised.

## CI

5 jobs on push/PR to `main`: `test-scanner` (CGO, `-race`), `test-web` (`CGO_ENABLED=0` + Postgres service), `test-adapter` (CGO + Postgres), `lint` (golangci-lint v2 on the 4 scanner modules only, plus gofmt check), `docker-build` (scanner + server images).

## Boundaries (enforced in review)

- **CGo:** only `ollantaparser`. `ollantaweb` must never import `ollantaparser`/`ollantarules`, even transitively. Domain uses `any` for tree-sitter types.
- **Rings:** `domain/` imports stdlib only. `application/` never imports adapters. Outer rings import inward only. Every port implementation carries a compile-time check: `var _ port.IAnalyzer = (*Impl)(nil)`.
- **Bridge:** legacy `ollantacore/domain` types and hexagonal `domain/model` types are distinct. Convert via `adapter/secondary/rules/bridge.go`; never mix.
- **No duplicated data or types:** one canonical location per struct/data file; share via `go:embed` or an API. Rule metadata JSONs in `ollantarules` are canonical; `ollantacore/rulecatalog/catalog.go` is the CGo-free mirror for `ollantaweb` - update both.

## Adding a rule

4 steps (3 new files + 2 edits):

1. Logic: Go rules in `ollantarules/languages/golang/rules/my_rule.go`; JS/TS/Python rules in `ollantarules/languages/treesitter/<name>_<lang>.go` (flat package, not per-language dirs).
2. Metadata JSON next to the logic (`go_<key>.json` or `<lang>_<key>.json`).
3. Register the rule variable in that package's `embed.go` `MustRegister(...)` call (panics on missing or duplicate MetaKey).
4. Mirror the entry in `ollantacore/rulecatalog/catalog.go`.

Hard requirements:

- **Negative test** (`..._NoFalsePositive`) alongside the positive one. Tree-sitter predicates (`#eq?`, `#match?`, ...) are evaluated by the Go binding layer - if that step breaks, every predicate rule silently false-positives. Follow existing examples in `sensor_test.go`.
- **`reference_url`** in the JSON pointing to an authoritative source (MITRE CWE, OWASP, official language/API docs, Effective Go, golangci-lint docs). Never SonarQube RSPEC (copyright). No reference = do not register the rule. Maintenance helpers: `scripts/validate_references.py`, `scripts/sync_cwe_tags.py`.

## Frontends (there are two)

1. **Scanner UI** - `ollantascanner/server/static/`: TypeScript bundled by esbuild into `dist/app.js`, which the scanner binary embeds. After any edit under `src/`, run `npm run build` and commit `dist/`. Editing `src/` without rebuilding silently ships the old UI.
2. **Server SPA** - `ollantaweb/api/static/`: vanilla JS modules under `app/`.
   - New reactive state must be added in BOTH `createInitialState()` and `resetProjectState()` (`app/core/state.js`). State created ad-hoc in handlers misbehaves when users navigate between projects/scopes.
   - A new tab/route must be wired in four places: tab bar, content renderer, lazy loader, event binders. Missing any one makes it unreachable or permanently empty.

## Code style

- **Never discard errors** - the `_, _ = f()` pattern is forbidden. Sole exception: `defer f.Close()` (excluded in `.golangci.yml`).
- Logging: `slog` with structured fields (`slog.With("file", path, "rule", key)`). Info = operational events (scan started/finished, gate evaluated, webhook fired); Debug = per-file/per-rule trace; Error = failures. Remove ad-hoc `log.Printf` / `fmt.Printf` before committing.
- API handlers: set `Content-Type: application/json` before writing; error bodies are `{"error": "message"}`.
- chi: for a resource with public GETs plus admin-only writes, nest `r.Group` (with the admin middleware) inside `r.Route`. GET handlers placed outside `r.Route` on the same path cause 405s.
- When extracting setup code out of `main()`: keep `defer x.Close()` and shutdown hooks in the owner's scope, or return the resource explicitly - do not leave defers behind in helpers.
- Naming: `I`-prefixed interfaces, `New`-prefixed constructors, `Test{Type}_{Scenario}`, rule keys `{lang}:{kebab-name}`, snake_case JSON tags, `omitempty` for optional fields, `json:"-"` for secrets, `ctx` as first arg, sentinel errors (`var ErrNotFound = errors.New("not found")`), package-level doc comments.
- DRY: one source of truth per concept; the third repetition of a pattern gets extracted. KISS: manual DI in `main.go` (no containers), direct SQL via pgx (no ORM), flat over nested. YAGNI: no single-implementation interfaces outside hexagonal ports, no speculative config knobs, no feature flags for unfinished work. One file = one concern; split files past ~400 lines.

## Gotchas

- **Push 401:** `OLLANTA_SCANNER_TOKEN` (on `ollantaweb`) must equal `OLLANTA_TOKEN` (scanner/push side). Dev default on both: `ollanta-dev-scanner-token`.
- **Rule keys contain `:`** (`go:no-large-functions`): frontend must `encodeURIComponent`, backend must `url.PathUnescape` wildcard route params before lookup.
- Local compose seeds `admin`/`admin` and dev secrets. For any shared environment, override `PG_PASSWORD`, `OLLANTA_JWT_SECRET`, `OLLANTA_SCANNER_TOKEN` in `.env`.
- Config precedence - scanner: defaults < `config.toml` < CLI flags; server: defaults < `config.toml` < `OLLANTA_*` env vars.

## Commits

- Branch: `username/brief-description`. Conventional prefixes: `feat:`, `fix:`, `chore:`, `test:`, `docs:`.
- Before pushing: `make lint` and `make test`. Flag security implications in the PR description.

---
> Source: [ollantacorp/Ollanta](https://github.com/ollantacorp/Ollanta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
