## logstash-vector

> This repository contains configuration, documentation, and sample data for a Logstash → Vector migration. There is no compiled application or conventional test suite; most work involves editing YAML, VRL, and documentation.

# AGENTS.md

## Purpose
This repository contains configuration, documentation, and sample data for a Logstash → Vector migration. There is no compiled application or conventional test suite; most work involves editing YAML, VRL, and documentation.

## Repository Layout
- `impl/vector.yaml`: Primary Vector configuration (includes 35 unit tests).
- `impl/vrl/`: Externalized VRL transform files (8 files, one per transform step).
- `doc/requirements.md`: Detailed migration requirements and parsing rules.
- `doc/todo.md`: Task checklist and validation notes.
- `doc/testing-procedures.md`: Testing procedures and validation guidelines.
- `impl/implementation-summary.md`: Implementation summary and current status.
- `INTEGRATION_TESTING_COMPLETE.md`: Integration testing framework summary.
- `tests/integration/`: Complete integration testing framework with scripts, test data, and documentation.
- `sample/`: Sample Logstash/Fluentd configs and example log file.
- `tmp/`: Local Vector data directory (runtime output).

## Build / Lint / Test Commands
### Summary
- No build system (no `package.json`, `pyproject.toml`, `Cargo.toml`, `Makefile`, or `go.mod`).
- No lint config detected.
- **Testing**: Vector built-in unit tests (35 tests) + integration testing framework.

### Validation Commands
These commands are available if the `vector` CLI is installed locally.

- Validate config syntax:
  - `vector validate --config impl/vector.yaml`
- Run Vector unit tests:
  - `vector test --config impl/vector.yaml` (runs 35 built-in tests)
- Run Vector with the config:
  - `vector --config impl/vector.yaml`
- Run integration tests:
  - `cd tests/integration && ./run_all_tests.sh`

### Testing Infrastructure
- **Unit Tests**: 35 tests embedded in `impl/vector.yaml` covering all parsing logic
- **Integration Tests**: Complete framework in `tests/integration/` with:
  - Baseline generation (Logstash comparison)
  - Output comparison tools
  - Elasticsearch validation
  - Test data for all scenarios
- **Documentation**: See `tests/integration/README.md` for comprehensive testing guide

## Code Style Guidelines
### General
- Prefer minimal, focused edits to configs and docs.
- Keep changes aligned to the migration requirements in `doc/requirements.md`.
- Avoid unrelated refactors; keep changes small and traceable.

### YAML (Vector Config)
- Use 2 spaces for indentation.
- Keep component names short and descriptive (e.g., `parse_fields`, `ap_log_files`).
- Group `sources`, `transforms`, and `sinks` in that order.
- Use explicit scalar values (avoid implicit YAML booleans like `on/off` if unclear).
- Maintain consistent quoting style; use single quotes for regex patterns.

### VRL (Vector Remap Language)
- Prefer explicit `to_string!()` and `to_int!()` conversions when types are expected.
- Use `parse_grok!()` only when parsing is required; otherwise use `parse_grok()`.
- Guard parsing with `is_null()` checks to avoid overwriting existing fields.
- Use `del()` for conditional field removal instead of assigning `null`.
- Keep VRL blocks in a logical order: enrich → parse → derive → convert → cleanup.

### Imports / Dependencies
- There are no language-level imports or dependencies in this repo.
- Do not add new build tools or dependency manifests unless explicitly requested.
- **Python dependencies**: Python scripts in `tests/integration/` use `uv` for dependency management. Always use `uv venv` and `uv pip install` rather than system `pip`.

### Naming Conventions
- Match existing field names exactly (case-sensitive): e.g., `maskGroupId`, `MaskListNo`, `IsQueryPhase`.
- Prefer `snake_case` for Vector component names and local VRL variables.
- Do not rename fields that are referenced by downstream systems.

### Error Handling
- Parsing failures should not stop the pipeline. Use `parse_grok()` or guarded `parse_grok!()` calls.
- Keep transforms resilient: check for `null` and handle missing fields gracefully.
- Avoid throwing errors unless a field is truly required for correctness.

### Formatting
- Keep lines readable; wrap long regex patterns only if doing so preserves YAML/VRL validity.
- Preserve the existing ordering of grok fallback patterns.
- Avoid adding inline comments unless needed to clarify non-obvious logic.

### Documentation Updates
- Update `doc/requirements.md` or `impl/implementation-summary.md` when behavior changes.
- Keep documentation factual, not speculative.
- Testing documentation is in `tests/integration/` - update README.md, TESTING_SUMMARY.md, or QUICKSTART.md as needed.

## Cursor / Copilot Rules
- No `.cursor/rules/` or `.cursorrules` found.
- No `.github/copilot-instructions.md` found.

## Change Safety Checklist
- Confirm changes preserve the parsing patterns defined in `doc/requirements.md`.
- Validate config syntax with `vector validate` if available.
- Ensure multiline regexes and grok patterns remain unchanged unless explicitly required.
- Verify field removal logic for query-phase events is intact.

## Notes for Agentic Changes
- Prefer modifying `impl/vector.yaml` over adding new files.
- Sample log files in `sample/` should be treated as reference data; do not edit unless requested.
- Keep `tmp/` free of committed artifacts.

---
> Source: [YH-Hung/logstash-vector](https://github.com/YH-Hung/logstash-vector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
