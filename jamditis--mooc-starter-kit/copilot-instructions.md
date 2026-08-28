## mooc-starter-kit

> This file is for students using Google's Gemini CLI instead of (or

<!--
This file is for students using Google's Gemini CLI instead of (or
alongside) Claude Code. The MOOC is taught with Claude Code, but the
patterns in this repo work with any agentic CLI that reads a project
context file.
-->

# GEMINI.md

This repo works with Gemini CLI too.

Gemini CLI reads `GEMINI.md` the same way Claude Code reads `CLAUDE.md`.
If you want to use Gemini on this repo, you have two options:

1. Symlink it: `ln -s CLAUDE.md GEMINI.md` (then delete this file)
2. Copy it: keep both files in sync by hand

The symlink is less effort. The copy lets you tune one for Claude and
one for Gemini if you find the models respond to different framing.

## Free tier

Gemini CLI has a free tier (1,000 requests per day at the time this kit
was put together). That's a lot of room to experiment without putting a
card down. Worth knowing if you're price-sensitive or running a lot of
batch jobs.

## One-shot runs

Gemini CLI supports a `-p` flag for non-interactive prompts:

```
gemini -p "draft a short brief from the docs in sample-docs/"
```

That's the pattern Module 4 uses for cross-model review — fire the same
prompt at Claude, Gemini, and Copilot CLI and compare what each catches.

## Worked example

See `examples/worked-claude-md/CLAUDE.md` for a fully filled-in context
file. The same content drops into `GEMINI.md` without changes.

## More

- Course site: https://mooc.amditis.tech
- Cross-model review walkthrough: `docs/cross-model-review.md`

---
> Source: [jamditis/mooc-starter-kit](https://github.com/jamditis/mooc-starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
