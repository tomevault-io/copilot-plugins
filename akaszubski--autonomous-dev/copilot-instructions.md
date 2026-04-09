## autonomous-dev

> Development harness for Claude Code. Deterministic enforcement, specialist agents, alignment gates — 12 elements of harness engineering.

# autonomous-dev

Development harness for Claude Code. Deterministic enforcement, specialist agents, alignment gates — 12 elements of harness engineering.

## Critical Rules

- **NEVER direct-edit without `/implement`**: `agents/*.md`, `commands/*.md`, `hooks/*.py`, `lib/*.py`, `skills/*/SKILL.md` — these are functional infrastructure. Hook-enforced: `unified_pre_tool.py` blocks Write/Edit to these paths outside the pipeline.
- **Direct editing is only for**: user-facing docs (README.md, CHANGELOG.md, docs/*.md), config (.json/.yaml), typos (1-2 lines).
- **After plan mode approval → use `/implement`**: The plan IS the input to `/implement`, not a license to bypass it.
- **Run `/improve` after `/implement` sessions.** Use `--auto-file` to create GitHub issues.
- **Use `/clear` after each feature.** Prevents context bloat.
- **Deploy with `bash scripts/deploy-all.sh`** — never manual `cp -rf`. Script handles local, remote (Mac Studio), validation, and integrity checks.
- **Don't simplify, redesign, or consolidate agents.** The pipeline, hooks, and enforcement are validated over months of real use. The cost is tokens, not complexity. Complexity is the mechanism.

## Build & Test

```bash
# Run full test suite (from repo root)
pytest --tb=short -q

# Run specific test file
pytest tests/unit/hooks/test_native_tool_auto_approval.py -v

# Run with coverage
pytest --cov=plugins/autonomous-dev/hooks --cov=plugins/autonomous-dev/lib --cov-report=term-missing

# GenAI tests (LLM-as-judge, not included in default runs)
pytest tests/genai/ --genai
```

## Architecture

- **Pipeline**: 8-step SDLC (15 internal steps) — alignment → research → plan → acceptance tests → implement → validate → verify → git
- **Enforcement**: 25 hooks with JSON `{"decision": "block"}` hard gates (not prompt-level nudges)
- **Agents**: 14 specialists with fresh context per invocation, model-tiered (Haiku/Sonnet/Opus)
- **Skills**: 17 domain packages, progressively injected per-step to prevent context bloat

Component counts: 14 agents, 17 skills, 22 commands, 25 hooks, 185 libraries.

## Commands

`/implement` (full, --light, --batch, --issues, --resume, --fix) | `/create-issue` (--quick) | `/plan-to-issues` (--quick) | `/align` (--project, --docs, --retrofit) | `/audit` (--quick, --security, --docs, --code, --tests) | `/setup` | `/sync` (--github, --env, --all, --uninstall) | `/health-check` | `/advise` | `/worktree` (--list, --status, --merge, --discard) | `/scaffold-genai-uat` | `/status` | `/refactor` (--tests, --docs, --code, --fix, --quick) | `/sweep` | `/improve` (--auto-file) | `/retrospective` | `/mem-search` | `/skill-eval` (--quick, --skill, --update) | `/autoresearch` (--target, --metric, --iterations, --min-improvement, --dry-run)

## Key Paths

| What | Where |
|------|-------|
| Alignment source of truth | [.claude/PROJECT.md](.claude/PROJECT.md) |
| Pipeline command | `plugins/autonomous-dev/commands/implement.md` |
| State machine | `plugins/autonomous-dev/lib/pipeline_state.py` |
| Hook enforcement | `plugins/autonomous-dev/hooks/unified_pre_tool.py` |
| Agent definitions | `plugins/autonomous-dev/agents/` |
| Test suite | `tests/` (unit, integration, regression, security, hooks, genai) |
| Activity logs | `.claude/logs/activity/` |
| Architecture details | [docs/ARCHITECTURE-OVERVIEW.md](docs/ARCHITECTURE-OVERVIEW.md) |
| Troubleshooting | [plugins/autonomous-dev/docs/TROUBLESHOOTING.md](plugins/autonomous-dev/docs/TROUBLESHOOTING.md) |

## Session Continuity

`SessionStart-batch-recovery.sh` auto-restores batch state after `/clear` or auto-compact. Activity logged to `.claude/logs/activity/` by `session_activity_logger.py`.

**Last Updated**: 2026-03-30

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akaszubski) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
