## aira-scanner

> **AIRA Scanner** (AI-Induced Risk Audit) is a research tool for detecting fail-soft patterns in software systems — especially those developed with significant AI assistance.

# CLAUDE.md — AIRA Scanner Development Handbook

## Project Overview

**AIRA Scanner** (AI-Induced Risk Audit) is a research tool for detecting fail-soft patterns in software systems — especially those developed with significant AI assistance.

**Core Question**: "Does the system tell the truth when it fails?"

AIRA identifies systems that:
- Return success despite incomplete or failed operations
- Degrade silently instead of failing explicitly
- Obscure true system state under error conditions
- Preserve appearance of function while weakening guarantees

**Key Facts:**
- Command-line linter + rule engine
- Detects 15 fail-soft pattern categories (C01–C15)
- Works on Python, JavaScript, and TypeScript codebases
- Published via Homebrew (`brew install BDB-Labs/aira-scanner/aira`); source installs via `pip install ./CLI`
- Research-driven: empirical validation in progress

## Repository Structure

```
aira-scanner/
├── CLI/                              # Scanner CLI + package
│   ├── aira/                         # Main package
│   │   ├── checkers/                 # Detection rule implementations
│   │   │   ├── python_checker.py     # Python AST-backed static checks
│   │   │   ├── js_checker.py         # JavaScript/TypeScript checks
│   │   │   └── test_coverage_checker.py  # Test coverage asymmetry (C14)
│   │   ├── scanner.py                # Core scanning engine (AIRAScanner)
│   │   ├── cli.py                    # CLI entry point + arg parsing
│   │   ├── __main__.py               # Entry point → cli.py:main()
│   │   ├── research.py               # Research submission (Supabase/Airtable/JSONL)
│   │   ├── llm.py                    # LLM provider routing
│   │   ├── collector.py              # Public repo collection
│   │   ├── deterministic_scan.py     # Server-side inline scan helpers
│   │   └── __init__.py               # Package init (version)
│   └── tests/                        # Test suite
│       ├── test_scanner_modes.py     # Scanner mode + edge-case tests
│       ├── test_research.py          # Research submission tests
│       ├── test_cli_scan_validation.py  # CLI validation tests
│       ├── test_cli_failure_behavior.py # CLI failure mode tests
│       ├── test_llm_routing.py       # LLM provider routing tests
│       ├── test_deterministic_scan.py   # Deterministic scan tests
│       ├── test_collector.py         # Collection tests
│       └── fixture_violations.py     # Test fixtures
├── api/                              # Web API (Vercel serverless)
├── docs/                             # Documentation (web-facing)
├── Formula/                          # Homebrew formula
├── lib/                              # Shared libraries
├── scripts/                          # Build + CI scripts
├── pyproject.toml                    # Project configuration
├── ROADMAP.md                        # Feature roadmap
├── CONTRIBUTING.md                   # Contribution guidelines
├── AIRTABLE_SCHEMA.md                # Legacy Airtable findings database
├── CHANGELOG.md                      # Version history
├── README.md                         # Quick start
├── LICENSE                           # MIT
└── .gitignore
```

## Fail-Soft Pattern Categories (C01–C15)

### C01 — Success Integrity Violations
System returns success code despite operation failure.

**Example**:
```python
try:
    db.insert(record)
except Exception:
    return True  # ✗ Returns success after failure
```

**AIRA Detection**: Flags `return True` / success-shaped dicts inside exception handlers.

### C02 — Audit & Evidence Integrity Gaps
System fails to record evidence of decision or operation outcome.

**Example**:
```python
try:
    audit_write(event)
except Exception:
    pass  # ✗ Evidence silently lost
```

**AIRA Detection**: Flags audit/write calls in try blocks with non-re-raising handlers.

### C03 — Broad Exception Suppression
Catch-all `except:` or `except Exception:` with no propagation.

**Example**:
```python
try:
    important_operation()
except:
    pass  # ✗ Silent failure
```

**AIRA Detection**: Flags bare excepts and broad Exception handlers that don't re-raise.

### C04 — Fallback & Degradation
System silently falls back to unsafe defaults instead of failing.

**AIRA Detection**: Flags `fallback`, `degraded`, `best_effort`, `fail_open` identifiers.

### C05 — Bypass & Override Paths
Escape hatches that weaken guarantees (testing, debug, force flags).

**AIRA Detection**: Flags `skip_validation`, `bypass_governance`, `force_pass`, etc.

### C06 — Ambiguous Return Contracts
Function return types don't distinguish success/failure cases.

**AIRA Detection**: Flags functions with ≥2 `return None` in different contexts.

### C07 — Parallel Logic Drift
Same operation implemented differently in different code paths. **Human review only.**

### C08 — Unsupervised Background Tasks
Async/background work with no monitoring or result collection.

**AIRA Detection**: Flags `create_task()`, `ensure_future()` fire-and-forget calls.

### C09 — Environment-Dependent Safety Drift
Safety behavior changes based on debug/dev/staging environment variables.

**AIRA Detection**: Regex-based scan for env-conditional safety bypasses.

### C10 — Startup Integrity Weaknesses
Configuration validation missing at startup (fails at runtime).

**AIRA Detection**: Flags non-re-raising except handlers in startup/init functions.

### C11 — Determinism Drift
Non-deterministic reasoning (non-zero temperature in model calls).

**AIRA Detection**: Regex-based scan for temperature settings in AI call sites.

### C12 — Source-to-Output Lineage Gaps
Missing tracing of where output came from. **Human review only.**

### C13 — Confidence Misrepresentation
System claims result without confidence/certainty metadata.

**AIRA Detection**: Flags predict/assess/classify functions without confidence fields.

### C14 — Test Coverage Asymmetry
Happy-path tested, error paths untested.

**AIRA Detection**: `scan_test_files()` analyzes test files for coverage gaps.

### C15 — Retry & Idempotency Assumption Drift
Retry logic near write operations without idempotency keys.

**AIRA Detection**: Regex-based scan for retry + write without idempotency safeguards.

## Development Workflow

### Setup
```bash
cd CLI/
pip install -e ".[dev]"
# or: uv sync --extra dev
```

### Running the Scanner
```bash
# Scan a single file
cd CLI/ && python3 -m aira scan target.py

# Scan a directory
python3 -m aira scan target_dir/

# Scan with specific engine
python3 -m aira scan target_dir/ --engine static
python3 -m aira scan target_dir/ --engine llm --provider ollama
python3 -m aira scan target_dir/ --engine hybrid

# Exclude patterns
python3 -m aira scan target_dir/ --exclude "tests/,vendor/"

# Output JSON
python3 -m aira scan target_dir/ --output json

# Check provider health
python3 -m aira health
python3 -m aira health --check-research
```

### Running Tests
```bash
cd CLI/

# All tests
python3 -m pytest tests/ -v

# Coverage
python3 -m pytest --cov=aira tests/

# Specific test file
python3 -m pytest tests/test_scanner_modes.py -v
```

### Adding a New Check

1. **Add check metadata** in `CLI/aira/scanner.py` — `CHECKS` dict + `CHECK_ID_BY_KEY` + `CHECK_NAME_BY_KEY`

2. **Implement detection** in `CLI/aira/checkers/python_checker.py`:
```python
def _check_my_pattern(self):
    for node in ast.walk(self.tree):
        if matches_pattern(node):
            self._add("C16", "MY PATTERN NAME", "HIGH", node.lineno, "What went wrong")
```

3. **Add tests** in `CLI/tests/` with appropriate fixtures.

## Key Files

| File | Purpose |
|------|---------|
| `CLI/aira/scanner.py` | Core scanner, `AIRAScanner`, `ScanResult`, check registry |
| `CLI/aira/checkers/python_checker.py` | Python static checks (C01–C15), `PythonChecker`, `Finding` |
| `CLI/aira/checkers/js_checker.py` | JavaScript/TypeScript checks |
| `CLI/aira/cli.py` | CLI argument parsing, output formatting, exit codes |
| `CLI/aira/research.py` | Research backend submission (Supabase, Airtable, JSONL) |
| `CLI/aira/llm.py` | LLM provider routing (OpenAI-compatible, Ollama, NVIDIA NIM, Groq, OpenRouter) |
| `CLI/aira/deterministic_scan.py` | Server-side inline `scan_inline_sources()` |
| `CLI/aira/collector.py` | Public repo collection against manifests |
| `CLI/aira/checkers/test_coverage_checker.py` | C14 test coverage analysis |

## Configuration

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `SUPABASE_URL` | Supabase research backend URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase service role key |
| `SUPABASE_TABLE` | Supabase submissions table (default: `aira_submissions`) |
| `AIRTABLE_BASE_ID` | Airtable base ID (legacy) |
| `AIRTABLE_TOKEN` | Airtable token (legacy) |
| `AIRA_RESEARCH_JSONL` | JSONL research sink path |
| `AIRA_RESEARCH_BACKEND` | Force backend: `supabase`, `jsonl`, `airtable` |
| Aira LLM providers | `AIRA_OPENAI_BASE_URL`, `AIRA_OLLAMA_MODEL`, `AIRA_GROQ_API_KEY`, etc. |

### pyproject.toml
Project metadata and dependencies are in `pyproject.toml` at repo root.

## Common Tasks

### Scanning a Codebase
```bash
cd CLI/
python3 -m aira scan /path/to/codebase/ --engine static
python3 -m aira scan /path/to/codebase/ --exclude "vendor/,dist/"
```

### Exclusions
```bash
# Built-in skip dirs (always excluded):
# .git, node_modules, __pycache__, .venv, venv, env, dist, build, .tox, coverage, .mypy_cache

# User exclude patterns (comma-separated):
python3 -m aira scan target/ --exclude "tests/,migrations/,legacy/"
```

### Interpreting Results

Each finding includes:
- **check_id**: C01–C15 (or `SCANNER` for scanner errors)
- **check_name**: Human-readable category
- **severity**: HIGH, MEDIUM, or LOW
- **file**: Relative file path
- **line**: Line number
- **description**: What the pattern is
- **snippet**: Relevant source line

### Submitting Research Data
```bash
python3 -m aira scan target/ --submit-research-aggregate --sample-name my-project
```

### Use as Library
```python
from aira.scanner import AIRAScanner

scanner = AIRAScanner("my_project/")
result = scanner.scan(mode="static")

for finding in result.findings:
    if finding["severity"] == "HIGH":
        print(f"HIGH: {finding['file']}:{finding['line']} — {finding['description']}")
```

### Use in CI/CD
```yaml
# .github/workflows/aira-scan.yml
name: AIRA Scanner
on: [pull_request]
jobs:
  aira:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -e CLI/
      - run: cd CLI && python3 -m aira scan .. --fail-on high
```

## Research & Publication

- **arXiv**: https://arxiv.org/abs/2604.17587 (published research paper)
- **Findings Database**: Submissions go to Supabase (preferred), JSONL files, or Airtable (legacy)

## Conventions

- **Check IDs**: C01–C15 for checks, SCANNER for scanner errors
- **Severity levels**: HIGH (fix before merge), MEDIUM (tech debt), LOW (nice to have)
- **C07 and C12**: Always UNKNOWN from automated checks — require human review
- **Static checks**: Python uses `ast` module; JS/TS use structured checks
- **Exit codes**: 0 = pass, 1 = findings threshold exceeded, 2 = input/usage error, 3 = operational failure

## Contributing

1. **Identify a pattern** from real-world audits
2. **Add check metadata** in `CLI/aira/scanner.py` (CHECKS dict)
3. **Implement detection** in `CLI/aira/checkers/python_checker.py` (and js_checker.py for JS)
4. **Add tests** in `CLI/tests/`
5. **Update ROADMAP.md**
6. **Submit PR** with research findings if applicable

See `CONTRIBUTING.md` for detailed guidelines.

## Getting Help

- Check IDs and categories → `CLI/aira/scanner.py` (`CHECKS` dict)
- Pattern details → Items 1–15 in this document
- CLI usage → `python3 -m aira scan --help`
- Web API → `api/` directory

---

**Last Updated**: 2026-07-04
**Website**: https://aira.bageltech.net
**Paper**: https://arxiv.org/abs/2604.17587
**Active Researchers**: Bill P + team

---
> Source: [BDB-Labs/aira-scanner](https://github.com/BDB-Labs/aira-scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
