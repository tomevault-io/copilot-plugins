## skil-creator

> This workspace contains a refactored version of the `skill-creator` Claude Code skill, extended to also work as a skill for [LangChain deepagent-cli](https://docs.langchain.com/oss/python/deepagents/cli/overview).

# skill-creator: Implementation Details

## Project Overview

This workspace contains a refactored version of the `skill-creator` Claude Code skill, extended to also work as a skill for [LangChain deepagent-cli](https://docs.langchain.com/oss/python/deepagents/cli/overview).

The skill lives at `skills/skill-creator/`. All Python scripts that support the skill (evals, benchmarking, description optimization) live in `skills/skill-creator/scripts/`.

---

## Environment Setup

This project uses [uv](https://docs.astral.sh/uv/) for Python environment management (Python 3.13).

```bash
# Create and activate the virtual environment
uv venv
source venv/bin/activate

# Install project + dev dependencies (pytest, pytest-cov)
uv pip install -e ".[dev]"
```

Always activate the venv before running any script:
```bash
source venv/bin/activate
python -m scripts.run_loop ...
```

---

## Architecture Overview

```
User query
    │
    ▼
Agent (Claude Code or deepagent-cli)
    │  reads name+description from skill discovery paths
    ▼
Skill triggers? ──No──► Agent answers directly
    │
   Yes
    ▼
SKILL.md body loaded into context
    │
    ├─► Subagent spawned per test case (Agent tool / task tool)
    │       └─► outputs saved to <workspace>/iteration-N/eval-*/
    │
    ├─► Grader subagent scores assertions → grading.json
    │
    ├─► aggregate_benchmark.py → benchmark.json + benchmark.md
    │
    └─► generate_review.py → browser viewer (or --static HTML)
            │
            ▼
        User reviews → feedback.json
            │
            ▼
        improve_description.py (via claude -p or deepagents -n)
            │
            ▼
        run_loop.py iterates until score peaks or max_iterations
```

The description optimization path is separate from the skill-creation loop — it runs *after* the user is satisfied with the skill content, to tune the SKILL.md `description` field for better trigger accuracy.

---

## Skill File Structure

```
skills/skill-creator/
├── SKILL.md                    # Main skill instructions (YAML frontmatter + markdown)
├── agents/
│   ├── analyzer.md             # Subagent: analyze benchmark results
│   ├── comparator.md           # Subagent: blind A/B comparison
│   └── grader.md               # Subagent: grade eval assertions
├── assets/
│   └── eval_review.html        # Template for description eval review UI
├── eval-viewer/
│   ├── generate_review.py      # Launches the eval results viewer
│   └── viewer.html             # Viewer UI template
├── references/
│   └── schemas.md              # JSON schemas for evals.json, grading.json, etc.
└── scripts/
    ├── __init__.py
    ├── aggregate_benchmark.py  # Aggregates run results into benchmark.json
    ├── generate_report.py      # Generates HTML report for description optimization
    ├── improve_description.py  # Calls claude -p or deepagents -n to improve skill description
    ├── install.sh              # Cross-platform installation helper
    ├── package_skill.py        # Packages skill into .skill archive
    ├── quick_validate.py       # Validates a skill directory against the spec
    ├── run_eval.py             # Trigger evaluator — Claude Code backend (claude -p)
    ├── run_eval_deepagents.py  # Trigger evaluator — deepagent-cli backend (deepagents -n)
    ├── run_loop.py             # Optimization loop (calls run_eval or run_eval_deepagents)
    └── utils.py                # Shared: parse_skill_md()
```

---

## Platform Compatibility

Both Claude Code and deepagent-cli implement the [Agent Skills specification](https://agentskills.io/specification). The `SKILL.md` format is identical between platforms.

### Allowed SKILL.md Frontmatter Keys

deepagent-cli **rejects unknown frontmatter keys**. The complete allowed set is:

| Key | Required | Notes |
|-----|----------|-------|
| `name` | Yes | Must match directory name exactly; lowercase alphanumeric + hyphens, ≤64 chars |
| `description` | Yes | ≤1024 chars |
| `license` | No | SPDX identifier |
| `compatibility` | No | ≤500 chars |
| `metadata` | No | Arbitrary key-value dict |
| `allowed-tools` | No | Space-delimited list of tool names |

### Skill Discovery Paths

| Platform | User-scope | Project-scope |
|----------|-----------|---------------|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| deepagent-cli | `~/.deepagents/<agent>/skills/` | `.deepagents/skills/` |
| Both (shared standard) | `~/.agents/skills/` | `.agents/skills/` |

Install to `~/.agents/skills/skill-creator` to make the skill available across both platforms.

---

## Scripts Reference

### `run_eval.py` — Claude Code Trigger Evaluator

Tests whether a skill description causes `claude -p` to trigger (read the skill) for a given query.

**Mechanism:**
1. Writes a temporary command file to `.claude/commands/` with the test description
2. Runs `claude -p <query> --output-format stream-json --include-partial-messages`
3. Detects triggering via stream events: `Skill` tool call containing the command name, or `Read` tool call on the skill path
4. Returns early on detection; kills process on timeout

**Usage (via run_loop.py):**
```bash
python -m scripts.run_loop \
  --eval-set trigger-evals.json \
  --skill-path skills/skill-creator \
  --model claude-sonnet-4-6 \
  --agent claude \
  --max-iterations 5
```

### `run_eval_deepagents.py` — deepagent-cli Trigger Evaluator

Mirrors `run_eval.py` but uses `deepagents -n '<query>'` (non-interactive mode).

**Mechanism:**
1. Writes a temporary skill directory to a temp path discoverable by deepagents (`.deepagents/skills/`)
2. Runs `deepagents -n '<query>'` and captures output
3. Detects triggering by checking whether the skill name appears in the agent's tool calls or reasoning output
4. Cleans up temp directory after each run

**Usage (via run_loop.py):**
```bash
python -m scripts.run_loop \
  --eval-set trigger-evals.json \
  --skill-path skills/skill-creator \
  --model <deepagents-model-id> \
  --agent deepagents \
  --max-iterations 5
```

### `run_loop.py` — Optimization Loop

Runs the full description optimization loop: eval → improve → eval → ... up to `--max-iterations`.

**Key flags:**
- `--agent claude` (default) | `--agent deepagents` — selects the trigger evaluator backend
- `--model` — model ID for the evaluator; use the model powering the current session
- `--holdout 0.4` — fraction of eval set held out as test set (prevents overfitting)
- `--verbose` — print progress to stderr

### `install.sh` — Cross-Platform Installer

```bash
./skills/skill-creator/scripts/install.sh --shared     # ~/.agents/skills/ (both platforms)
./skills/skill-creator/scripts/install.sh --claude     # ~/.claude/skills/
./skills/skill-creator/scripts/install.sh --deepagents # ~/.deepagents/agent/skills/
```

---

## Platform-Specific Behavioral Differences

### Subagents

| Claude Code | deepagent-cli |
|-------------|---------------|
| `Agent` tool (spawn subagent) | `task` tool (spawn subagent) |
| Notification on completion includes `total_tokens` and `duration_ms` | Same convention |

In `SKILL.md`, the eval run instructions reference the `Agent` tool. deepagent-cli users should use the `task` tool instead — the `SKILL.md` deepagent-cli section documents this.

### Description Optimization

The `run_eval.py` script detects Claude Code-specific stream events (`Skill`/`Read` tool use via `content_block_start`). The deepagent-cli evaluator (`run_eval_deepagents.py`) uses a different detection strategy appropriate to the deepagents output format.

### Memory Files

- Claude Code: `CLAUDE.md` (this file)
- deepagent-cli: `AGENTS.md`

When the skill-creator skill itself runs inside deepagent-cli and needs to save persistent context, it should use `AGENTS.md` instead of `CLAUDE.md`.

---

## Testing

Tests live in `tests/` at the workspace root. Run with:

```bash
source venv/bin/activate
pytest -v
```

Install dev dependencies first:

```bash
uv pip install -e ".[dev]"
```

| Test file | What it covers |
|-----------|----------------|
| `tests/test_utils.py` | `parse_skill_md()` — YAML frontmatter parsing |
| `tests/test_quick_validate.py` | Validator against deepagent-cli spec; regression guard on the skill itself |
| `tests/test_run_eval_deepagents.py` | Trigger detection logic (subprocess mocked) |
| `tests/test_run_loop.py` | `--agent` flag routing + train/test split logic |
| `tests/test_integration.py` | Non-mocked smoke tests (validate + install) |

---

## Extension Points

To add support for a third agent platform (e.g., a hypothetical `myagent`):

1. **Trigger evaluator** — copy `scripts/run_eval_deepagents.py` to `scripts/run_eval_myagent.py`. Adapt:
   - `find_project_root()` — look for the platform's config dir (e.g., `.myagent/`)
   - Temp skill placement — write to whatever directory the platform auto-discovers
   - CLI invocation — replace `["deepagents", "-n", query]` with the platform's non-interactive command
   - Detection — adjust `_skill_triggered_in_event()` or the plain-text fallback to match that platform's output format

2. **Backend registration** — in `run_loop.py`, add `"myagent"` to `_get_eval_backend()` and to the `--agent` choices list in `main()`.

3. **LLM caller** — in `improve_description.py`, add `_call_myagent()` and handle `agent == "myagent"` in `_call_llm()`.

4. **SKILL.md section** — add a `## myagent-Specific Instructions` section mirroring the deepagent-cli one, documenting any behavioural differences (subagent tool name, memory file name, etc.).

5. **Tests** — add `tests/test_run_eval_myagent.py` following the pattern in `tests/test_run_eval_deepagents.py`.

---

## Development Notes

- Run `quick_validate.py` after any `SKILL.md` change to confirm the file is valid for deepagent-cli
- `tests/test_quick_validate.py::test_skill_creator_itself` acts as a CI regression guard for frontmatter validity
- The `eval-viewer/generate_review.py` works on both platforms (pure Python, no Claude Code dependencies)
- `package_skill.py` also works on both platforms

---

## Related Resources

- deepagent-cli docs: https://docs.langchain.com/oss/python/deepagents/cli/overview
- deepagent-cli skills docs: https://docs.langchain.com/oss/python/deepagents/skills
- Agent Skills specification: https://agentskills.io/specification
- deepagents GitHub: https://github.com/langchain-ai/deepagents/tree/main/libs/cli

---
> Source: [erhwenkuo/skil-creator](https://github.com/erhwenkuo/skil-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
