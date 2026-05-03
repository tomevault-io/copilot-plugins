## elite-dangerous-colonization-tracker

> If you find yourself reading the same file or making the same tool call more than 3 times in a row with no progress, **STOP IMMEDIATELY**. You are stuck in a loop.

# Agent Guidelines

## Loop Detection (CRITICAL)

If you find yourself reading the same file or making the same tool call more than 3 times in a row with no progress, **STOP IMMEDIATELY**. You are stuck in a loop.

What to do:
1. Pause and acknowledge the loop to the user
2. Ask for help — describe what you're trying to do and what's going wrong
3. Wait for the user's response before continuing

Signs you are in a loop:
- You've read the same file offset more than twice without making edits
- Your tool calls are returning the same content repeatedly
- You're "trying again" with the same approach that already failed
- Your read offset is resetting to the start of a file you've already seen

**Do not silently repeat failing operations.** Each wasted tool call costs the user money. If something isn't working after 2-3 attempts, ask for help.

## General

- Always read a file with a specific offset if you've already seen the beginning — don't re-read from line 1 unless you need to
- Batch independent reads/operations together rather than doing them sequentially
- If a tool is consistently returning unexpected results, stop and ask the user rather than retrying

---
> Source: [nkkromhof/elite-dangerous-colonization-tracker](https://github.com/nkkromhof/elite-dangerous-colonization-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
