## claude-jev

> A Claude Code plugin. Five hooks send small judgments to TypeSafe's Jev, a

# claude-jev

A Claude Code plugin. Five hooks send small judgments to TypeSafe's Jev, a
System One model that returns typed answers instead of text. `README.md`
explains what each hook decides and why. Read it before changing behavior.

## Layout

| Path | What lives there |
|---|---|
| `scripts/jev.py` | API client and CLI. Every other script imports it. |
| `scripts/prompt_router.py` | `UserPromptSubmit` — routing hint |
| `scripts/subagent_router.py` | `PreToolUse` on `Agent\|Task` — sets subagent model, denies a file-changing brief that omits paths, acceptance, verification, or commit policy |
| `scripts/rules.py` | `PostToolUse` on edits, and `Stop` — rule enforcement |
| `scripts/compactor.py` | The `rows` bridge behind `session.compact`; `judge` is kept for the eval |
| `hooks/register.ts` | Experimental function-hooks module: `session.compact` -> `compactor.py rows`. A bridge, not a second implementation. |
| `scripts/comparators.py` | ast-grep lookups the rule hook adds to a judgment |
| `scripts/observed.py` | Scores what a past turn actually did |
| `scripts/stats.py` | `/claude-jev:stats` — scores live decisions from the three logs under `~/.claude`: `jev-router-log.jsonl` (router, subagent, rules), `jev-compact-log.jsonl`, `jev-calls.jsonl` (every API call, written by `jev.ask`) |
| `scripts/check_no_comments.py` | Hard ban: fails if `scripts/`, `eval/`, or `hooks/` source has a `#` / `//` / `/* */` comment |
| `eval/` | Offline measurement. See `eval/README.md`. |
| `skills/` | User-facing entry points shipped with the plugin. `/claude-jev:<dir name>`. |
| `.claude/skills/` | Maintainer skills for this repository, not shipped. `release` cuts a version: changelog section to bump, tag, push, GitHub release. |
| `hooks/hooks.json` | Hook registration. New hook means an entry here. `modules` names the function-hooks module; older Claude Code ignores the key. |

## Invariants

- **Hooks fail open.** A failure never blocks or alters a session. Command
  hooks exit 0 and print nothing on any error, missing key, or timeout; keep
  the `except Exception: return` at the top of every hook `main`. The `rows`
  bridge answers `{"fallback": ...}` on stdout and exits 0, and
  `hooks/register.ts` then calls `next(e)` so the built-in summary runs.
- **The one external program is ast-grep**, pinned by version and sha256 in
  `scripts/comparators.py`, fetched to `~/.claude/jev-bin` by a detached
  process outside the hook's budget, and never required: every comparator
  answers `""` without it, and the judgment proceeds as before.
- **Python 3 standard library only.** No dependency file, no third-party
  imports. `urllib.request` is the HTTP client. The one non-Python file,
  `hooks/register.ts`, exists because Claude Code loads function-hook modules
  as JavaScript; it holds no judgment, only the call into `compactor.py rows`
  and the fail-open fallthrough to `next(e)`. Even its debug-log line is the
  `summary` string Python sends. Keep it that way.
- **One environment variable:** `TYPESAFE_API_KEY`. Do not add another, and
  do not add a fallback name. Every other tunable is a module-level constant.
  `CLAUDE_CONFIG_DIR` is Claude Code's own variable, not a plugin tunable:
  `jev.config_dir()` honors it and every path under the user's config
  directory goes through that helper, never through a literal `~/.claude`.
- **No code comments.** Line, block, and JSDoc comments are banned in
  `scripts/`, `eval/`, and `hooks/` source (`.py`, `.ts`, `.js`). Shebangs
  and LICENSE text stay. Python module/function docstrings are documentation
  strings, not comments, and stay where Conventions require them. Markdown
  under `docs/`, `skills/`, `README.md`, and `AGENTS.md` is prose, not code
  comments — leave it. Rationale for thresholds and constants (`ACT`,
  `MIN_CONFIDENCE`, `KEEP_THRESHOLD`, and the rest) lives in eval results,
  docs, or PR evidence — not inline comments. Changing a threshold without
  eval evidence is still a guess. Enforce with
  `python3 scripts/check_no_comments.py` (exit 1 on any hit).
- **`scripts/observed.py` is the shared scorer.** `eval/replay.py` and
  `scripts/stats.py` both call it. Editing it moves every accuracy number in
  `README.md`.
- Hook scripts import siblings through `sys.path.insert(0, dirname(__file__))`.
  Keep that, because Claude Code runs them from arbitrary directories.

## Conventions

- Each script's module docstring states the hook it serves, the fail-open
  contract, and the env var. Keep that shape when you add one.
- Question definitions live in `scripts/jev.py` (`intent_bundle`,
  `subagent_bundle`) or next to the hook that asks them. Put the meaning in
  the `instructions` and `criteria` text — Jev never sees the key names.
- Write rules in this file as bullets, one instruction each. `rules.py` parses
  instruction files bullet by bullet, and a rule buried inside a prose
  paragraph classifies poorly.
- User-facing entry points are skills under `skills/`, not commands. There is
  no `commands/` directory; do not add one.

## Verify

There is no test suite. The local / script contract for this repo is:

```bash
python3 -m compileall -q scripts eval
python3 scripts/check_no_comments.py
echo '{"prompt":"hi","transcript_path":""}' | python3 scripts/prompt_router.py; echo "exit=$?"
```

`compileall` plus stdin hook smoke (exit 0, fail open) is what a default
cloud agent or Grok Bot run can claim. Live Claude Code session verification
is out-of-band: it needs a machine with `claude` and `TYPESAFE_API_KEY`, and
is not claimed as proved by those local checks alone.

For a scripted launch → doctor → drive → evidence → cleanup loop (isolated
`HOME`, feature map under `.cursor/skills/verify-claude-jev/features/`), use
the project-local Cursor skill:

```bash
.cursor/skills/verify-claude-jev/bin/control-jev launch
.cursor/skills/verify-claude-jev/bin/control-jev doctor
# then drive one feature from features/; evidence under
# .cursor/skills/verify-claude-jev/artifacts/<RUN_ID>/
.cursor/skills/verify-claude-jev/bin/control-jev cleanup
```

See `.cursor/skills/verify-claude-jev/SKILL.md`. Keep the map honest with
`/maintain-verification-skill` when hooks or skills change.

`python3 scripts/comparators.py which` prints the ast-grep the rule hook
would run, or `(none)`; `fetch` downloads the pinned one. The hook must exit
0 within its budget either way — check with the binary renamed away.

Every hook must exit 0 on a malformed or empty event. Feed the script you
changed a matching JSON event on stdin and check the exit code.

The `rows` bridge answers bad input with `{"fallback": ...}` and exit 0:

```bash
echo '' | python3 scripts/compactor.py rows
```

Out-of-band (not claimed on Grok Bot / default cloud agents): exercise the
function-hooks module end to end on a machine with Claude Code and a real
key — load the plugin from disk, compact, and read the debug log:

```bash
export CLAUDE_CODE_ENABLE_FUNCTION_HOOKS=1
claude -p "..." --session-id "$ID" --plugin-dir "$PWD"
claude -p "/compact" --resume "$ID" --plugin-dir "$PWD" -d
grep 'jev-compact\|core never ran' ~/.claude/debug/"$ID".txt
```

"a hook's N messages stand ... core never ran" means the summary was
replaced; "built-in summary runs" means the bridge fell through.

To measure a routing or threshold change, run the relevant eval and compare
against the table in `README.md`:

```bash
python3 eval/replay.py run --variant v7_no_unclear --sample 250
python3 eval/rules_eval.py run --sample 250 --seed 0
python3 eval/rules_eval.py report --sweep
python3 eval/compare.py compact --synth 60
```

The compaction command is the only way to measure `scripts/compactor.py`.
It prints re-fetch coverage and planted-constraint survival in one gate and
exits 2 below either floor, so a wording that keeps paths but drops what the
user said, or the reverse, cannot pass on one number. `eval/sweep.py` and
`eval/planted.py` are diagnostics for reading a change, not gates.

`--seed 0 --sample 250` selects the same real edits as the numbers in
`README.md`; keep it when comparing. A rule-question wording change misses
the answer cache and re-costs the run.

The rules eval judges each record at its `sha`, and reads
`eval/global_CLAUDE.md` in place of `~/.claude/CLAUDE.md`. Change a case's
sha only when a rule-file change in that repo is the thing being measured.

These call `api.typesafe.ai` with real past prompts and cost money. Ask before
running a full sweep.

## Data and numbers

- `eval/data/` and `eval/private/` are gitignored. They hold extracted
  transcripts and repo-specific cases, and they do not survive a clone. Do not
  commit them or write code that assumes they exist.
- Numbers in `README.md` and `eval/README.md` come from eval runs. Change one
  only with a run behind it, and say which run.
- Bump `version` in `.claude-plugin/plugin.json` for a behavior change, and
  add the change under `## [Unreleased]` in `CHANGELOG.md`. Releases go
  through `/release` (`.claude/skills/release`), which refuses an empty
  section.

---
> Source: [0x7067/claude-jev](https://github.com/0x7067/claude-jev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
