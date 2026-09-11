## claudestatsbar

> ClaudeStatsBar - a Claude Code status line showing the per-turn cost of an

# CLAUDE.md

## Project

ClaudeStatsBar - a Claude Code status line showing the per-turn cost of an
accumulated context, so the user knows when to `/clear` or `/compact`.

Single file: `statsbar.py`. Pure Python stdlib. No dependencies, no network,
no transcript parsing.

## Hard constraints

- **Never break the host session.** Every code path exits 0 and prints a line,
  even on malformed input. The top-level `except` is load-bearing; do not
  remove it.
- **Stay fast.** The status line is debounced at 300ms and re-runs on every
  turn. Budget ~25ms. No file reads beyond the tiny per-session state file, no
  subprocesses, no imports outside the stdlib.
- **Stdlib only**, and no reads of the transcript. Everything needed arrives as
  JSON on stdin; see `docs/statusline-stdin.md` for the fields and their
  version gates.
- **Degrade, never guess.** Any field can be absent on older Claude Code
  versions or early in a session. Use `dig()` and render without it. Never
  print a figure the data does not support - the suppressed `1.0×` multiplier
  is the reference case for this.

## Numbers in the README are measured, not estimated

The token figures come from a real two-week sample (916 transcripts, 57,451
requests, 11.6B tokens) taken from `~/.claude/projects/**/*.jsonl`. If you
change a claim, re-measure it. Do not round them into round-sounding numbers.

## Testing

No framework. Pipe synthetic payloads at the script and read the output:

```bash
echo '{"session_id":"t","context_window":{"total_input_tokens":186000,
  "context_window_size":200000,"used_percentage":93}}' | ./statsbar.py
```

Always cover: absent `context_window`, `{}`, non-JSON, a bogus far-future
`resets_at`, and a 5-hour percentage that drops (rolling window reset).
Delete `~/.claude/.statusline-state/t.json` between runs that test the
baseline logic.

## Style

British English in prose. Hyphens, never em dashes.

---
> Source: [Field-Logic-Ltd/ClaudeStatsBar](https://github.com/Field-Logic-Ltd/ClaudeStatsBar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
