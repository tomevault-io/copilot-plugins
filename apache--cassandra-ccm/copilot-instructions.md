## cassandra-ccm

> - CLI entry points live in `ccm/` and the core implementation is under `ccmlib/` (cluster lifecycle, node management, command helpers).

# Repository Guidelines

## Project Structure & Module Organization
- CLI entry points live in `ccm/` and the core implementation is under `ccmlib/` (cluster lifecycle, node management, command helpers).
- Tests are in `tests/` and mirror library modules; add new tests alongside the code they cover.
- Packaging and install metadata: `setup.py`, `setup.cfg`, and runtime deps in `requirements.txt`. Miscellaneous utilities and docs live in `misc/`, `ssl/`, and `INSTALL.md`.

## Build, Test, and Development Commands
- Create an isolated environment: `python3 -m venv venv && source venv/bin/activate`.
- Install runtime deps and editable package: `pip install -r requirements.txt && pip install -e .`.
- Add test-only deps: `pip install mock pytest requests`.
- Run tests: `pytest` (use `pytest tests/test_node.py -k <pattern>` for focused runs). Expect integration-like cases that spawn local Cassandra nodes; keep environments clean.
- Validate a local install: `ccm --help` to confirm entry points resolve.

## Coding Style & Naming Conventions
- Python-first codebase; follow PEP 8 with 4-space indents and line-length restraint (~100 chars).
- Maintain Python 2.7 and 3.x compatibility where practical; avoid newer syntax that breaks 2.7 and gate version-specific behavior carefully.
- Use descriptive identifiers for nodes/clusters, and align CLI option names with existing verbs/nouns (`create`, `populate`, `start`, etc.).
- Prefer pure functions for helpers; keep side effects (filesystem, subprocess) localized and well-logged.

## Testing Guidelines
- Framework: pytest. Name files `test_*.py` and favor small, deterministic cases over long-lived clusters.
- When tests need Cassandra binaries, prefer the repository cache (`~/.ccm/repository`) to avoid repeated downloads.
- Add coverage when touching node lifecycle, logging, or install path resolution; include regression tests reproducing reported issues.
- Use markers or narrow selections for slow tests; do not assume network availability beyond local loopback.

## Commit & Pull Request Guidelines
- Commits are short, action-oriented summaries (e.g., `Improve jdk validation`, `Fix cleanup when node start fails`); keep a single focus per commit.
- PRs should describe the change, note risk areas (filesystem writes, subprocess calls), and link related tickets/issues.
- Include how you tested (`pytest`, manual `ccm` invocation) and any environment notes (Python version, OS). Screenshots are unnecessary; logs or command transcripts help reviewers.

---
> Source: [apache/cassandra-ccm](https://github.com/apache/cassandra-ccm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
