## cerf

> WinCE ARM emulator. Runs Windows CE / Windows Mobile ARM binaries on x64 Windows. See [agent_docs/subsystems.md](agent_docs/subsystems.md) for how CERF works.

# CERF — Windows CE ARM Runtime

WinCE ARM emulator. Runs Windows CE / Windows Mobile ARM binaries on x64 Windows. See [agent_docs/subsystems.md](agent_docs/subsystems.md) for how CERF works.

## MOST IMPORTANT RULES

These override ALL other instructions. Read these FIRST. Violating these is a fireable offense.

- **Slow is correct. Fast is wrong.** — You are trained to produce output quickly and look productive. That training is your enemy. Every urge to write code quickly is the exact moment you are about to guess. One correct function per hour beats ten guessed functions per minute. See [agent_docs/rules.md](agent_docs/rules.md) § Speed Is The Enemy.
- **Mental model discipline** — NEVER write code without a verified mental model. Claim → verify (IDA decompile + runtime log) → THEN code. "I think I know" is not verified. "It's probably X" is not verified. Only an IDA decompile output visible in this conversation, or a log line from a diagnostic you ran, counts as verification. See [agent_docs/workflow.md](agent_docs/workflow.md).
- **Before writing any function body, the ida_decompile output must be visible above.** If you have not pasted the decompile result for that function in this session, you may NOT write the function. No exceptions.

## Rules (Summary)

- **IDA citations required** — every non-trivial behavior needs `IDA: 0xNNNNN` offset
- **No hacks** — replicate exact WinCE behavior from IDA decompilation
- **Full rules** — see [agent_docs/rules.md](agent_docs/rules.md)

## Reference Pages

Read these BEFORE starting work:

- **[agent_docs/workflow.md](agent_docs/workflow.md)** — Mental model discipline: how to investigate, verify, and implement. Core operating method for every task.
- **[agent_docs/subsystems.md](agent_docs/subsystems.md)** — CERF component fact sheet: what exists, what's thunked, where files live.
- **[agent_docs/rules.md](agent_docs/rules.md)** — All project rules: WinCE accuracy, architecture, communication patterns, git, subagents.
- **[agent_docs/debugging.md](agent_docs/debugging.md)** — All debugging: log reading, crash investigation, memory corruption playbook (phases 1-7, techniques 1-10).

## Build

```bash
powershell -ExecutionPolicy Bypass -File build.ps1
```

Output: `build/Release/x64/cerf.exe`. Pre-commit hook rejects .cpp/.h files over 500 lines (see `.githooks/pre-commit`). The `.vcxproj` uses `**\*.cpp` glob — new .cpp files are automatically included in the build. No manual project file edits needed.

Build might take 5-10 minutes to complete.

## Testing

```bash
python e2e_tests/wince5/<test_name>.py   # runs cerf, generates own log
./roast.sh --diff                        # code quality audit on changed files
```

## IDA MCP

gwes.exe and all WinCE DLLs are loaded in IDA instances accessible via MCP tools. Use `mcp__ida__ida_list_instances` to find them, `mcp__ida__ida_decompile` to decompile any function. Always verify behavior from IDA before implementing.

If a binary isn't loaded in any IDA instance, open it with: `python tools/open_ida.py --wait <path-to-file>`. The `--wait` flag blocks until IDA finishes analysis and the MCP server is registered, so the instance is immediately usable via MCP tools after the command returns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gweslab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
