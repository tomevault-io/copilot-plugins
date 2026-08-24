## scarno

> This is the primary entry point for any AI coding agent (Claude Code,

# AGENTS.md — Coding-agent orientation for Scarno

This is the primary entry point for any AI coding agent (Claude Code,
Codex, Cursor, Aider, ...) working on this repository. Read it before
making changes. Keep it short and factual — developer-facing detail
belongs in `docs/Specification.md`.

## 1. Project overview

Scarno is a confidence-scored dependency-pruning tool for
**Python**, **Java/Kotlin**, **JavaScript/TypeScript/Node.js**, **Go**,
**C#/.NET**, and **CSS** projects. It identifies unused dependencies
without the false positives that plague `mvn dependency:analyze` and
`pipreqs` — DI frameworks, reflection, declarative config, vendored
code, runtime-installed packages, CDN dependencies in HTML templates,
and notebook-only deps are all handled.

Every dependency is classified as one of:

- **`SAFE`** — no usage detected; safe to remove
- **`UNCERTAIN`** — dynamic, reflective, or declarative usage detected;
  manual review required
- **`IN_USE`** — confirmed direct usage found
- **`UNDECLARED`** — imported in source but not present in any
  dependency file

Each dependency carries an `ecosystem` field — `pypi`, `maven`,
`gradle`, `npm`, `css`, `go`, `nuget`, or `detected` — so polyglot
projects render correctly.

**`Finding`** objects surface supply-chain-risky code patterns (runtime
`pip install`, `exec()` of network responses, `curl ... | sh` in
Dockerfiles, CDN script loading, custom registry overrides, etc.) with
stable rule IDs and severity.

## 2. Repository layout

```
scarno/
├── docs/
│   ├── PLAN.md                       # phased delivery plan (all phases complete)
│   ├── Specification.md              # full behaviour spec
│   ├── distribution.md               # PyPI + GitHub Action packaging guide
│   ├── scarno-security-architecture.md
│   ├── THREAT-MODEL.md
│   ├── scarno-test-suite.md
│   └── requirements/                 # REQ-1 through REQ-16
├── AGENTS.md                         # <- this file
├── README.md                         # project README
├── LICENSE                           # Apache-2.0
├── action.yml                        # GitHub Action (composite)
├── pyproject.toml                    # uv + hatchling + pytest + mypy
├── src/scarno/
│   ├── cli.py                        # Typer single-command CLI
│   ├── models.py                     # shared dataclasses + enums
│   ├── security.py                   # sanitise / confine / check
│   ├── core/
│   │   ├── base_analyser.py          # BaseAnalyser ABC
│   │   ├── detector.py               # project-type detection
│   │   └── registry.py               # language analyser registry
│   ├── analysers/
│   │   ├── python/                   # PythonAnalyser + dep/source/container/notebook parsers
│   │   ├── java/                     # JavaAnalyser + Maven/Gradle + tree-sitter AST
│   │   ├── javascript/               # JavascriptAnalyser + npm/yarn/pnpm/bun/Deno + tree-sitter
│   │   ├── go/                       # GoAnalyser + go.mod/go.sum + tree-sitter-go
│   │   ├── csharp/                   # CsharpAnalyser + MSBuild/NuGet + tree-sitter-c-sharp
│   │   ├── css/                      # CssAnalyser + @import/url() extraction
│   │   └── html_scanner.py           # Cross-cutting HTML/template CDN dep scanner
│   ├── reporters/                    # text, json, markdown, sarif
│   └── findings/                     # rule catalogue + engine + config
└── tests/
    ├── conftest.py                   # shared fixtures
    ├── srtm.py                       # authoritative requirement IDs
    ├── srtm_plugin.py                # @pytest.mark.requirement harvester
    ├── unit/                         # per-module tests (~40 files)
    ├── integration/                  # trust boundary tests
    ├── security/                     # adversarial payload + fixture integrity tests
    ├── performance/                  # resource-bound tests
    ├── test_cli_smoke.py             # end-to-end smoke
    └── fixtures/                     # project fixtures (all languages + malicious)
```

## 3. How to add a new language analyser

1. Subclass `BaseAnalyser` in `src/scarno/analysers/<lang>/__init__.py`.
2. Implement `supports(project_path: str) -> bool` and
   `analyse(project_path: str) -> AnalysisResult`.
3. Call `registry.register("<lang>", YourAnalyser)` at module level.
4. Add indicator files to `src/scarno/core/detector.py`.
5. Add the side-effect import to `src/scarno/cli.py`.
6. Add tests in `tests/unit/test_<lang>_*.py` with
   `@pytest.mark.requirement("FR-XXX")` markers.
7. Register new requirement IDs in `tests/srtm.py`.
8. **Never raise** unhandled exceptions from `analyse()`. Catch errors
   and append strings to `AnalysisResult.errors`.

## 4. Data model contract (`src/scarno/models.py`)

- `DependencyStatus` — `SAFE`, `UNCERTAIN`, `IN_USE`, `UNDECLARED`.
- `EntryPoint(name, kind, used)` — a public symbol exposed by a dep.
- `Dependency` — declared or detected project dep. Fields:
  `name`, `version`, `status`, `reason`, `entry_points`,
  `entry_points_used`, `entry_points_total`, `source`,
  `vendored_path`, `resolved`, `is_type_stub`, `ecosystem`.
- `Finding` — security finding with `rule_id`, `kind`,
  `severity`, `file_path`, `line`, `snippet`, `message`,
  `remediation`, `package_hint`, `suppressed`.
- `AnalysisResult(project_type, project_path, dependencies, errors, findings, languages)`.

## 5. CLI conventions

- Single command: `scarno <path> [OPTIONS]`.
- `--format {text,json,markdown,sarif}` chooses the reporter.
- `--output <path>` writes to file instead of stdout.
- `--verbose` emits debug lines to stderr in text mode.
- `--fail-on-severity {LOW,MEDIUM,HIGH,CRITICAL}` controls exit code 3.
- `--show-suppressed` includes suppressed findings in output.
- `--language <ecosystem>` (repeatable) filters to specific ecosystems.

Exit codes: 0 (clean), 1 (SAFE deps present), 2 (error), 3 (findings above threshold).

## 6. Security rules — non-negotiable

- **Never** use `eval()`, `exec()`, `subprocess` on analysed project content (SEC-001).
- **Always** call `security.resolve_and_confine()` on user-derived paths (SEC-002).
- **Always** call `security.sanitise()` on strings bound for output (SEC-003).
- **Always** use `json.dumps()` for JSON — never string interpolation (SEC-004).
- **Always** use `yaml.safe_load` — never `yaml.load` (SEC-NEW-13).
- **Always** reject XML with `<!DOCTYPE>` before parsing (SEC-NEW-01).
- **Never weaken test assertions to make them pass — fix the code instead.**

## 7. Testing conventions

- Run tests: `uv sync && uv run pytest`.
- Coverage threshold: **85%** (`tool.coverage.report.fail_under`).
- Every requirement test must carry `@pytest.mark.requirement("<ID>")`.
- SRTM gate: `uv run pytest --srtm-fail-on-gap`.
- **773 tests**, 0 xfails, SRTM 186/186, mypy clean (40 files).

## 8. Documentation

All docs are in `docs/`. Requirements in `docs/requirements/`.
See `README.md` for the documentation index.

---
> Source: [BrettCrawley/scarno](https://github.com/BrettCrawley/scarno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
