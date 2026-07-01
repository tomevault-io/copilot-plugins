## hep-workflow

> > Read by all AI agents working in this repo. Claude Code loads `CLAUDE.md`;

# hep-workflow

> Read by all AI agents working in this repo. Claude Code loads `CLAUDE.md`;
> Codex and other tooling load `AGENTS.md`. The two files are kept
> **byte-identical** by `tests/contract/test_top_level_docs_byte_identical.py`.

## Project

A skill-based agent workflow for high-energy physics phenomenology — model
proposal → symbolic calculation (Package-X / Mathematica) → numerical scans
→ publication-grade exclusion plots.

The repo ships **skill definitions and validation tooling**, not a runtime
application. Most edits change schemas, skill prompts, helper scripts, or
fixtures, and the test/validator layer is the contract surface.

## Common commands

Setup (one-time):

```bash
python3 -m venv .venv && source .venv/bin/activate
python3 -m pip install -r requirements-dev.txt
```

The three validation layers — all must be green before any commit lands:

```bash
python3 scripts/validate_examples.py
python3 scripts/validate_workspace_projects.py
python3 -m pytest -q
```

Run a single test or one layer:

```bash
python3 -m pytest -q tests/contract
python3 -m pytest -q tests/integration/test_minimal_scan.py
```

End-to-end suite (requires `wolframscript`; gated):

```bash
python3 -m pytest -q tests/e2e --run-e2e
```

## Project-level contracts (load-bearing)

These rules apply project-wide. Full text in `docs/contracts/`. The summaries
here are pointers, not duplicates — never copy contract bodies into this file.

| Contract | One-line rule | Full text |
| --- | --- | --- |
| Mirror invariants | `.claude/skills/` ↔ `.agents/skills/` byte-identical; orchestrator `.md` ↔ `.toml` content-equivalent; CLAUDE.md ↔ AGENTS.md byte-identical | `docs/contracts/mirror-invariants.md` |
| Canonical name rule | Canonical parameter / function identifiers use ASCII letters, digits, and underscores only; workflow IDs (`task-001`, `analysis-001`, `c-001`) follow their own schema patterns | `docs/contracts/canonical-name-convention.md` |
| Three-validator discipline | `validate_examples` + `validate_workspace_projects` + `pytest -q` must stay green on `main` | `docs/contracts/three-validator-discipline.md` |
| Manifest history actions | New action requires synchronized update across schema + skills + orchestrator + tests | `docs/contracts/manifest-history-actions.md` |
| Skill / agent / script division | Mechanical → script; generation → skill; coordination → agent | `docs/contracts/skill-agent-division.md` |
| Honest reproduction principle | Calculations independently derived; disagreement reported as-is; never adjust tolerance to mask | `docs/contracts/honest-reproduction-principle.md` |

## Architecture (quick-glance)

Two conceptual layers separated by responsibility:

- **Agents** (`.claude/agents/` / `.codex/agents/`) coordinate project state,
  read/update `manifest.json`, dispatch skills, validate outputs.
- **Skills** (`.claude/skills/` ↔ `.agents/skills/`) own disjoint slices of a
  workspace project: `hep-idea`, `package-scribe`, `hep-numerics`, and any
  future entry skills (e.g., `hep-paper-formalize`).
- **Scripts** (`scripts/` and `<skill>/scripts/`) do mechanical work.

Source-of-truth hierarchy (when docs disagree, fix top-down):
1. `schemas/*.json`
2. `docs/contracts/*.md`
3. `.claude/skills/*/references/`
4. `.claude/skills/*/SKILL.md`
5. `.claude/agents/*.md` + `.codex/agents/*.toml`
6. `README.md`, `CONTRIBUTING.md`, this file

## Workspace project layout (sketch)

```text
workspace/projects/<project-name>/
├── manifest.json
├── idea/                       hep-idea
├── model/                      hep-idea
├── constraints/                hep-idea
├── calculations/task-NNN/      package-scribe
└── numerics/                   hep-numerics
    ├── scan-configs/<analysis-id>.json
    ├── scan-results/<analysis-id>/
    └── figures/<analysis-id>/
```

Committed workspace fixture: `workspace/projects/smoke-e2e/` (minimal, used by
e2e). Other `workspace/projects/` entries are user-local generated state and
are not part of the public release. Richer synthetic contract fixtures live
under `tests/fixtures/workspace-projects/`.

## Next-step guidance

When responding to the user during an active agent or skill turn — progress
updates, completion summaries, validation reports, blocker reports, status
answers, or questions back to the user — end with a single concise line
naming the most useful next action:

> Next step: <concrete action> (<state basis>)

Base it on the current project state: `manifest.json`, artifact status,
validation output, last completed or blocked workflow step. Do not infer
from session memory alone. Localize the line to the conversation language
(e.g. "Next step: ..." when responding in English).

Skip this line when the response is a raw artifact, JSON / YAML / TOML / code
block, machine-readable output, brief tool-call narration ("file written",
"tests passing"), or a turn unrelated to an active workflow.

## Notes

- If files matching `codex-prompts-*.md` exist in a local checkout, treat them
  as frozen execution history, not active source of truth.
- Python 3.11+ required.
- Authoritative test discipline: `python3 -m pytest -q` must stay at zero
  failures on `main`.

---
> Source: [huangzhonglv/hep-workflow](https://github.com/huangzhonglv/hep-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
