## claude-code-token-count

> Read Claude Code session JSONL files and report token usage from message.usage objects. Use when the user asks how many tokens a Claude Code session used, wants to inspect usage breadcrumbs in ~/.claude/projects JSONL logs, compare session token counts, expose input/output/cache token totals from a session transcript, or install a Claude Code hook that blocks tool calls after an early context threshold.


# Claude Code Token Count

Use the bundled Python scripts to inspect Claude Code session JSONL logs and optionally install local project hooks that keep `.claude/usage.json` current and block tool calls after a token threshold.

## Quick Start

Run against a known JSONL path:

```bash
python3 claude-code-token-count/scripts/get_token_count.py /path/to/session.jsonl
```

Run against a Claude Code session id and cwd:

```bash
python3 claude-code-token-count/scripts/get_token_count.py --cwd /path/to/project --session-id <uuid>
```

Find the newest session JSONL for a cwd:

```bash
python3 claude-code-token-count/scripts/get_token_count.py --latest-for-cwd /path/to/project
```

Emit machine-readable JSON:

```bash
python3 claude-code-token-count/scripts/get_token_count.py /path/to/session.jsonl --json
```

Show one row per usage-bearing event:

```bash
python3 claude-code-token-count/scripts/get_token_count.py /path/to/session.jsonl --by-message
```

Count repeated JSONL breadcrumbs instead of deduplicating by `message.id`:

```bash
python3 claude-code-token-count/scripts/get_token_count.py /path/to/session.jsonl --count-duplicates
```

On Windows, use `py -3` instead of `python3` if that is the available Python launcher.

## Claude Code Storage Notes

Claude Code hooks receive event JSON on stdin. Official hook docs describe common hook input fields and `PreToolUse` behavior; command hooks can read stdin and return JSON decisions. The hook payload includes `session_id`, `transcript_path`, and `cwd` in current Claude Code builds. Prefer `transcript_path` inside hooks instead of re-deriving a JSONL path.

Claude Code stores local transcript JSONL files under `~/.claude/projects/<encoded-project-path>/<session-id>.jsonl`. This path shape is observed from Claude Code's local storage and is also exposed directly through hook payloads as `transcript_path`; it is not documented as a stable public API. Treat the JSONL file as read-only. In practice Claude Code continuously mirrors or rewrites session state from its own in-memory/server-side state, so editing the JSONL is not a reliable way to change a session. Reading it for current `message.usage` breadcrumbs is valid for reporting and hook gating.

Official docs to consult when wiring hooks:

- Claude Code hooks reference: https://docs.anthropic.com/en/docs/claude-code/hooks
- Claude Code settings reference: https://docs.anthropic.com/en/docs/claude-code/settings

## What It Counts

The script scans JSONL lines, reads `message.usage` objects, and sums numeric fields ending in `_tokens`, including common Claude usage fields:

- `input_tokens`
- `output_tokens`
- `cache_creation_input_tokens`
- `cache_read_input_tokens`

It reports:

- `aggregate`: sum of usage blocks, deduplicated by `message.id` by default.
- `last_usage`: the final usage block in the file.
- duplicate usage breadcrumbs skipped or counted.
- derived totals for input-side tokens and all observed token fields.

Use `aggregate` for total session activity when each usage block is per request. Use `last_usage` when you know the log records cumulative usage in the final message.

The Claude Code CLI's displayed context usage has matched `last_usage.all_observed_tokens` in real-session checks.

## Hook Setup

For per-project gating, write hooks into the project's `.claude/settings.local.json`. Claude Code documents `.claude/settings.local.json` as local, single-project, gitignored settings.

When installing, merge conservatively:

- Create `.claude/` if missing.
- Create `.claude/settings.local.json` if missing.
- If `.claude/settings.local.json` exists, parse the JSON and preserve all existing keys.
- Preserve existing hook groups and hook handlers.
- Add the `PreToolUse` and `UserPromptSubmit` handlers only if equivalent commands are not already present.
- Do not overwrite unrelated settings or user hook commands.

Example:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "python3 /abs/path/to/claude-code-token-count/scripts/token_usage_gate.py --threshold 900000"
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 /abs/path/to/claude-code-token-count/scripts/unblock_via_prompt.py"
          }
        ]
      }
    ]
  }
}
```

The `PreToolUse` hook should:

- Read the hook payload from stdin.
- Read `payload["transcript_path"]`.
- Compute `last_usage.all_observed_tokens`.
- Write/update `<project>/.claude/usage.json`.
- Stop the whole turn when `last_usage.all_observed_tokens` exceeds the threshold by returning top-level `continue: false` and a `stopReason` that includes the unblock code.

The default threshold is `900000`. Users can edit it per project by changing `--threshold` in `.claude/settings.local.json`.

Agent behavior: treat `--threshold`, `--behavior`, and `--handoff-allowance` as user-owned policy choices. Do not proactively offer to change them after installing or inspecting the hook. Do not alter them unless the user explicitly asks for a specific change. If the user asks what the settings mean, explain them without suggesting a change.

By default the gate uses hard-stop behavior:

```json
{
  "continue": false,
  "stopReason": "\\n\\nHALTED AT CONTEXT THRESHOLD (<used>/<limit>): The agent must wait and not say or do anything without further user direction\\n\\nTO UNBLOCK AND PROVIDE HAND OFF INSTRUCTION TO THE AGENT FOR THIS SESSION, type \"unblock <code> <hand off instructions>\".\\n\\nSuggested prompt: \"unblock <code> write a detailed handoff.md doc to allow another agent to continue this work\"\\n\\n"
}
```

This stops the turn entirely; the agent does not get to print its own explanation. The user sees `stopReason`, so it must include the unblock code and instructions. To use the older one-tool-call denial behavior instead, add `--behavior deny` to the `token_usage_gate.py` command.

The paired `UserPromptSubmit` hook lets the user unblock a short handoff window by typing:

```text
unblock 123456 write a handoff document
```

When blocking, the gate should tell the user directly in `stopReason`:

```text
TO UNBLOCK AND PROVIDE HAND OFF INSTRUCTION TO THE AGENT FOR THIS SESSION, type "unblock 123456 <hand off instructions>".

Suggested prompt: "unblock 123456 write a detailed handoff.md doc to allow another agent to continue this work"
```

The six-digit code is generated once and stored in `.claude/usage.json`. On a correct code, the prompt hook grants a small handoff allowance so the agent can write a summary or handoff document, then stop.

---
> Source: [RandyHaylor/claude-code-token-count](https://github.com/RandyHaylor/claude-code-token-count) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
