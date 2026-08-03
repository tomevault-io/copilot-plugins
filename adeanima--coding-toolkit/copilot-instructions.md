## coding-toolkit

> After any change to a skill, hook, command, or `plugin.json`/`marketplace.json`:

# coding-toolkit — project instructions

## Ship-and-verify workflow (mandatory)

After any change to a skill, hook, command, or `plugin.json`/`marketplace.json`:

1. **Deploy the update** so it is live, not just committed:
   - merge the worktree branch to `main` and push (only with explicit user consent), or
   - reinstall / reload the plugin so the running session picks up the change.
   Do not declare work done while it still sits only in a worktree branch.
2. **Hand the user a concrete test scenario** to execute themselves — the exact
   repo to open, the exact phrase to type or command to run, and the expected
   result. Skill triggering, hook firing, and slash-command behaviour can only be
   verified in a fresh session the agent cannot drive; the user runs the test.

Never close out a change with "it should work" — deploy it, then give the test
the user runs to confirm it.

---
> Source: [AdeAnima/coding-toolkit](https://github.com/AdeAnima/coding-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
