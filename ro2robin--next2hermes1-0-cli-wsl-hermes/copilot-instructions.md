## next2hermes1-0-cli-wsl-hermes

> Run a WSL-native Hermes coding workflow that uses GitHub Copilot CLI as the primary executor and Codex CLI as the fallback executor, with fixed wrappers, structured outputs, and file-based auditing.


# WSL Copilot-Primary Workflow

Use this skill when:
- Hermes is running inside WSL
- the repository to operate on is also inside WSL, ideally under `/home/rorobin/code/...`
- GitHub Copilot CLI should be the default executor
- Codex CLI should only take over when Copilot fails, times out, returns unusable output, or the user explicitly asks for a second opinion

This skill is for the **native WSL path**, not the Windows-through-`cmd.exe` workaround. In the current environment, both CLIs are available natively in WSL:
- `copilot` → `/home/rorobin/.local/bin/copilot`
- `codex` → `/home/rorobin/.local/bin/codex`

## Workspace layout

Expected workflow directory:

```bash
/home/rorobin/ai-workflow/
├── bin/
│   ├── build_copilot_prompt.sh
│   ├── run_copilot_main.sh
│   ├── run_codex_task.sh
│   ├── run_agent_orchestrator.sh
│   ├── run_workflow_reply.sh
│   ├── format_orchestrator_reply.py
│   └── parse_agent_result.py
├── logs/
├── tests/
└── tmp/
```

Meaning:
- `bin/` stores wrappers, the parser, the user-reply formatter, and the one-key entry script
- `logs/` stores runtime logs
- `tests/` stores lightweight script-level verification for the orchestration flow
- `tmp/` stores prompt files, final outputs, and intermediate artifacts

## Required behavior

### 1. Keep execution inside WSL
Do not bounce the main flow into Windows commands when the repo and tools are already available in WSL.

Preferred repo root:

```bash
/home/rorobin/code/<repo>
```

Avoid defaulting to `/mnt/c/...` for normal project work unless the user explicitly wants a Windows-backed repo.

### 2. Check repository validity first
Before invoking either agent, verify:
1. `repo_dir` exists
2. `repo_dir/.git` exists
3. the repo is on a WSL-native path if possible

If any check fails, stop and report the problem instead of attempting the agent run.

## Task classification

Classify every request into one of these four types before execution:

### `analyze`
Read-only analysis.
Use for:
- root-cause investigation
- code explanation
- error analysis
- repair suggestions without editing

### `fix`
Minimal necessary change plus the most relevant validation.
Use for:
- bug fixes
- config fixes
- small targeted repairs

### `implement`
New feature or broader code addition.
Use for:
- new modules
- new endpoints
- new pages or capabilities

### `review`
Review only, no edits.
Use for:
- change review
- diff inspection
- risk assessment

## Preferred entrypoint: orchestrated workflow

For normal Hermes execution, prefer the orchestration wrapper rather than manually chaining the lower-level wrappers:

```bash
~/ai-workflow/bin/run_agent_orchestrator.sh <repo_dir> <task_type> <user_request_file> [copilot_model] [fallback_mode] [codex_model]
```

Defaults:
- `copilot_model`: `auto`
- `fallback_mode`: `allow`
- `codex_model`: `gpt-5.4`

Fallback modes:
- `allow`: retry Copilot once, then allow Codex fallback
- `no-fallback`: retry Copilot once, then fail without running Codex

The orchestrator currently does all of the following:
1. validates repo path, `.git`, task type, and dependency scripts
2. archives the user request into `tmp/`
3. runs Copilot up to two attempts
4. validates that the final output is non-empty, contains the required section headers, and produces a non-empty parsed summary
5. builds a fallback prompt and runs Codex when policy allows
6. reuses the timestamped Codex output files emitted by `run_codex_task.sh` directly, avoiding duplicate fallback archiving inside the orchestrator
7. emits machine-readable result lines such as:
   - `STATUS=...`
   - `TASK_TYPE=...`
   - `EXECUTOR_USED=...`
   - `FALLBACK_OCCURRED=...`
   - `COPILOT_ATTEMPTS=...`
   - `FINAL_MESSAGE_FILE=...`
   - `PARSED_JSON_FILE=...`
   - `LOG_FILE=...`
   - `EVENTS_FILE=...`
   - `REASON=...`
   - `MISSING_DEPENDENCY=...` (when applicable)
   - `ORCHESTRATOR_LOG=...`

Use the lower-level wrappers directly only when you intentionally want to bypass orchestration.

## One-key entrypoint

For the simplest end-to-end usage, prefer:

```bash
~/ai-workflow/bin/run_workflow_reply.sh [--json] <repo_dir> <task_type> <user_request_file> [copilot_model] [fallback_mode] [codex_model]
```

This script:
1. runs `run_agent_orchestrator.sh`
2. saves the machine-readable orchestrator output into `tmp/workflow_reply_<timestamp>_<pid>.out`
3. passes that file into `format_orchestrator_reply.py`
4. in default mode, prints the final Chinese user-facing reply to stdout
5. in `--json` mode, prints a JSON object containing the final `reply` plus key metadata such as `status`, `task_type`, `executor_used`, `reason`, and `orchestrator_output_file`
6. when `PARSED_JSON_FILE` exists and is readable, `--json` mode also inlines commonly used parsed fields such as `summary`, `root_cause`, `changed_files`, `commands_run`, `test_result`, `risks`, and `next_step`

Use `--json` when Hermes also needs structured metadata without separately re-reading the saved orchestrator output file.
This is especially useful when Hermes wants both a ready-to-send `reply` and direct access to parsed fields like `summary`, `root_cause`, and `test_result`.

## User-facing reply formatting

After `run_agent_orchestrator.sh` finishes, prefer formatting the machine-readable output with:

```bash
python3 ~/ai-workflow/bin/format_orchestrator_reply.py <orchestrator_output_file>
```

This formatter turns orchestrator key=value output into a stable Chinese reply suitable for Hermes to send to the user.

Current behavior:
- success path uses `TASK_TYPE`, `EXECUTOR_USED`, `FALLBACK_OCCURRED`, and `PARSED_JSON_FILE`
- failure path uses `REASON` and `MISSING_DEPENDENCY` when present
- output stays concise and stable, using sections like:
  - `任务判断：...`
  - `执行状态：...`
  - `根因：...`
  - `修改内容：...`
  - `验证情况：...`
  - `风险点：...`
  - `下一步建议：...`
  - or, on failure, `失败说明：...` and `建议：...`

This script is the preferred final formatting layer between orchestrator output and Hermes' user-facing message.

## Structured failure reasons

When the orchestrator fails, prefer consuming `REASON` as a stable machine-readable category instead of scraping raw logs.

Currently implemented reason categories include:
- `dependency_missing`
- `repo_dir_missing`
- `repo_not_git`
- `user_request_missing`
- `invalid_task_type`
- `invalid_fallback_mode`
- `copilot_timeout`
- `copilot_output_invalid`
- `copilot_execution_failed`
- `copilot_output_missing`
- `codex_timeout`
- `codex_output_invalid`
- `codex_execution_failed`
- `codex_output_missing`

Additional structured field:
- `MISSING_DEPENDENCY=...` when `REASON=dependency_missing`

These categories are intended for Hermes-side reply shaping. For example:
- `copilot_timeout` → explain that the primary executor timed out
- `copilot_output_invalid` → explain that Copilot returned unusable structured output
- `codex_output_invalid` → explain that fallback also ran but did not return a parseable structured result
- `dependency_missing` → explain which wrapper or parser is missing using `MISSING_DEPENDENCY`

## Primary execution path: Copilot

Use the fixed wrapper:

```bash
~/ai-workflow/bin/run_copilot_main.sh <repo_dir> <task_type> <user_request_file> [model]
```

### Prompt-building rule
Do not handcraft a deeply quoted Copilot prompt directly in Hermes.
Instead:
1. write the user request to a file
2. call the prompt builder
3. let the wrapper invoke Copilot

Prompt builder:

```bash
~/ai-workflow/bin/build_copilot_prompt.sh <task_type> <user_request_file> <output_prompt_file>
```

### Copilot tool policy encoded in the wrapper
For `analyze` and `review`:
- allow `read`
- allow limited shell reads (`git status`, `git diff:*`, `ls:*`, `cat:*`)
- deny `write`
- deny `rm`
- deny `git push`
- deny `url(*)`

For `fix` and `implement`:
- allow `read`
- allow `write`
- allow limited validation commands (`pytest:*`, `python:*`, `npm test:*`, `node:*`)
- still deny deletion, push, and arbitrary URL access

### Copilot outputs
The wrapper writes timestamped files such as:
- prompt file in `~/ai-workflow/tmp/`
- final message in `~/ai-workflow/tmp/`
- log file in `~/ai-workflow/logs/`

The wrapper also prints:
- `FINAL_MESSAGE_FILE=...`
- `LOG_FILE=...`

## Fallback execution path: Codex

Use the fixed fallback wrapper:

```bash
~/ai-workflow/bin/run_codex_task.sh <repo_dir> <prompt_file> [model]
```

Default model in the current script:

```bash
gpt-5.4
```

The wrapper writes timestamped files such as:
- `~/ai-workflow/tmp/codex_<timestamp>_<pid>_final.txt`
- `~/ai-workflow/tmp/codex_<timestamp>_<pid>_events.jsonl`

And prints:
- `FINAL_MESSAGE_FILE=...`
- `EVENTS_FILE=...`

The wrapper also honors `AI_WORKFLOW_HOME` when set; otherwise it defaults to `~/ai-workflow`.

## When to fall back from Copilot to Codex

Switch to Codex when any of these are true:
1. Copilot returns empty output
2. Copilot times out
3. Copilot output is structurally incomplete
4. Copilot fails repository or permission checks that make the result unusable
5. Copilot obviously drifts away from the requested task
6. the user explicitly asks for Codex
7. the user asks for a second opinion

Important exception:
- if the user explicitly says "only use Copilot", do not auto-fallback without renewed permission

## Required output format from the agent

Both executor prompts should aim for this exact structure:

```text
SUMMARY:
ROOT_CAUSE:
CHANGED_FILES:
COMMANDS_RUN:
TEST_RESULT:
RISKS:
NEXT_STEP:
```

Hermes should prefer consuming this structured output rather than raw terminal chatter.

## Structured parsing

Use the parser script after a successful run:

```bash
python3 ~/ai-workflow/bin/parse_agent_result.py <final_output_file>
```

Expected JSON keys:
- `summary`
- `root_cause`
- `changed_files`
- `commands_run`
- `test_result`
- `risks`
- `next_step`

If parsing fails:
- fall back to reading the raw final output file
- state clearly that structured parsing failed

## Recommended execution procedure

Preferred path:
1. Normalize the request into:
   - `repo_dir`
   - `task_type`
   - `user_request`
   - `constraints`
   - `preferred_agent`
   - `fallback_agent`
   - `model`
   - `timeout_seconds`
2. Write the user request to a file under `~/ai-workflow/tmp/`
3. Run `run_workflow_reply.sh`
4. In normal mode, return the script's stdout as the user-facing reply
5. In `--json` mode, consume the returned JSON object's `reply` and metadata directly
6. Inspect the saved `tmp/workflow_reply_*.out` file only when you need deeper debugging or additional metadata beyond the JSON output

Manual path when intentionally bypassing orchestration:
1. Validate the repo path and `.git`
2. Run `run_copilot_main.sh`
3. Read `FINAL_MESSAGE_FILE`
4. Parse it with `parse_agent_result.py` if possible
5. If the result is empty, broken, or untrustworthy, prepare a prompt file and run `run_codex_task.sh`
6. Read and parse the fallback result
7. Reply to the user with a clean summary instead of dumping raw logs

## Recommended user-facing response shape

Summarize results in this order:
1. task judgment
2. root cause
3. changes made
4. validation status
5. risks
6. next step suggestion

Do not paste raw logs unless the user asks for them.

## Current implementation notes discovered from inspection and follow-up work

These are important operational notes for the current workflow:

1. `run_copilot_main.sh` is timestamped and audit-friendly.

2. `run_codex_task.sh` now writes timestamped output filenames itself.
   - This removes the old fixed-name overwrite risk in direct Codex runs.
   - `run_agent_orchestrator.sh` now reuses those emitted paths directly instead of creating a second archived copy.

3. Automatic orchestration is now implemented at the higher level in `run_agent_orchestrator.sh`.
   - It retries Copilot once.
   - It validates structured output.
   - It falls back to Codex when policy allows.
   - It supports `no-fallback` mode for “Copilot only” behavior.
   - It emits structured failure reasons such as `copilot_timeout`, `copilot_output_invalid`, `codex_output_invalid`, and `dependency_missing`.

4. `parse_agent_result.py` still expects header lines in the exact form `KEY:` on their own lines.
   - If the agent adds extra prose or changes header spelling, structured parsing will degrade.

5. `logs/` may legitimately be empty before the first real execution.

6. `tmp/copilot_task.md` currently looks like a simple example prompt artifact rather than the full task-type-aware prompt produced by `build_copilot_prompt.sh`.

7. `*:Zone.Identifier` files under `bin/` are Windows download metadata and are not part of the executable workflow.

8. The orchestration wrapper now has lightweight script-level verification at:
   - `~/ai-workflow/tests/test_run_agent_orchestrator.sh`
   Verified scenarios include:
   - direct Copilot success
   - Copilot failure followed by Codex fallback success
   - orchestrator reusing Codex wrapper output paths directly without duplicate archival copies
   - `TASK_TYPE` emission for downstream formatters
   - `copilot_output_invalid` classification when fallback is disabled
   - `copilot_timeout` classification when fallback is disabled
   - `codex_output_invalid` classification when fallback output is not parseable
   - `dependency_missing` classification with `MISSING_DEPENDENCY`
   - no-fallback mode refusing to run Codex

9. The user-facing formatter has lightweight verification at:
   - `~/ai-workflow/tests/test_format_orchestrator_reply.sh`
   Verified scenarios include:
   - success reply formatting from `PARSED_JSON_FILE`
   - failure reply formatting from `REASON`
   - dependency failure formatting from `MISSING_DEPENDENCY`

10. The one-key entry script has lightweight verification at:
   - `~/ai-workflow/tests/test_run_workflow_reply.sh`
   Verified scenarios include:
   - running orchestrator then formatter in sequence
   - saving orchestrator output into `tmp/workflow_reply_*.out`
   - printing the final Chinese reply to stdout
   - `--json` mode returning a JSON object with `reply` and workflow metadata
   - `--json` mode inlining parsed fields like `summary`, `root_cause`, `changed_files`, `test_result`, and `next_step` when available

## Pitfalls

- Do not run either agent outside a git repository.
- Do not rely on raw shell logs as the primary source for the user-facing answer.
- Do not silently switch to dangerous permissions.
- Do not use the Windows CLI workaround when WSL-native CLIs are already present and working.
- If you bypass `run_agent_orchestrator.sh` and call `run_codex_task.sh` directly, the wrapper is now safe for repeated runs because it uses timestamped raw output filenames; still consume the emitted `FINAL_MESSAGE_FILE` and `EVENTS_FILE` instead of guessing paths.
- The orchestrator's validation currently assumes the exact section headers are present and that `summary` is non-empty; if executor prompt formats drift, fallback behavior may trigger more often than expected.

## Minimal examples

### Example 1: preferred orchestrated analyze
```bash
cat > ~/ai-workflow/tmp/user_request.txt <<'EOF'
请分析当前仓库登录失败问题，只定位根因，不要直接修改文件。
EOF

~/ai-workflow/bin/run_agent_orchestrator.sh ~/code/myrepo analyze ~/ai-workflow/tmp/user_request.txt auto allow gpt-5.4
```

### Example 2: preferred orchestrated fix
```bash
cat > ~/ai-workflow/tmp/user_request.txt <<'EOF'
请修复当前仓库中的登录失败问题，做最小必要修改，并运行最相关的验证命令。
EOF

~/ai-workflow/bin/run_agent_orchestrator.sh ~/code/myrepo fix ~/ai-workflow/tmp/user_request.txt auto allow gpt-5.4
```

### Example 3: Copilot-only mode with retry but no Codex fallback
```bash
~/ai-workflow/bin/run_agent_orchestrator.sh ~/code/myrepo review ~/ai-workflow/tmp/user_request.txt auto no-fallback gpt-5.4
```

### Example 4: manual direct Copilot path
```bash
~/ai-workflow/bin/run_copilot_main.sh ~/code/myrepo fix ~/ai-workflow/tmp/user_request.txt auto
```

### Example 5: parse result
```bash
python3 ~/ai-workflow/bin/parse_agent_result.py ~/ai-workflow/tmp/<final_output_file>
```

## Rules for Hermes

1. Prefer `run_agent_orchestrator.sh` as the default entrypoint.
2. Prefer Copilot first.
3. Use Codex only as fallback or explicit second opinion.
4. Keep the repo and execution path in WSL.
5. Use the fixed wrappers rather than ad-hoc shell command construction.
6. Parse structured output when possible.
7. Summarize cleanly for the user.
8. Mention clearly when fallback occurred.
9. Mention clearly when structured parsing failed.
10. Use `no-fallback` when the user explicitly wants Copilot only.

---
> Source: [Ro2robin/Next2Hermes1.0-Cli-Wsl-Hermes](https://github.com/Ro2robin/Next2Hermes1.0-Cli-Wsl-Hermes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
