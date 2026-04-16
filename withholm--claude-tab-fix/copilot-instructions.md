## claude-tab-fix

> This project (and any project using this hook) contains tab-indented files.

# claude-tab-fix

## File editing rules

This project (and any project using this hook) contains tab-indented files.

**Always use the `Edit` tool for targeted changes to existing files.**

Do not use `Bash` with `sed`, `awk`, `perl -i`, or similar to modify file content. Do not use the `Write` tool to rewrite files that already exist on disk unless you intend to replace the entire file.

Reason: the `Edit` tool is intercepted by the `claude-tab-fix` PreToolUse hook, which detects and corrects indentation mismatches before they reach the file system. Bash and Write bypass this hook, which can silently introduce mixed indentation or cause edits to fail.

The hook will warn you if you attempt a Bash or Write operation on a tab-indented file, but it cannot correct those operations automatically.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WithHolm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
