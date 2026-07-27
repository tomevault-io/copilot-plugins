## watch-n-learn

> This file is a short guide for AI coding agents (and the people running them). Watch & Learn is packaged as a Claude Code plugin, but the skill inside it is written in an open format that several other agents understand too. Here's how to use it wherever you work.

# AGENTS.md — using Watch & Learn with any AI coding agent

This file is a short guide for AI coding agents (and the people running them). Watch & Learn is packaged as a Claude Code plugin, but the skill inside it is written in an open format that several other agents understand too. Here's how to use it wherever you work.

## What's in this repo

- `skills/watch-and-learn/` — the skill itself: a `SKILL.md` (plain instructions), a couple of Python scripts, and some reference notes. This folder is the portable part. Everything else is packaging.

The skill takes a how-to video and turns it into a reusable skill your agent can follow later. It watches the video locally (still frames plus a transcript), writes up the procedure, and checks that the result actually works.

## Install, by agent

**Claude Code** — install it as a plugin:

```
/plugin marketplace add axel-pm/watch-n-learn
/plugin install watch-and-learn@wnl
```

**OpenAI Codex, and other agents that read the open skill format** — copy (or symlink) the skill folder into the place that agent looks for skills. For Codex and several others, that's an `.agents/skills/` folder in your project or home directory:

```sh
git clone https://github.com/axel-pm/watch-n-learn
mkdir -p ~/.agents/skills
cp -R watch-n-learn/skills/watch-and-learn ~/.agents/skills/
```

**Any other agent** — you can still use it. Point your agent at `skills/watch-and-learn/SKILL.md` and tell it to follow the steps. The instructions are plain Markdown and the scripts are plain Python, so nothing is locked to one vendor.

## What it needs to run

- **Python 3** on the machine.
- A **one-time setup** the first time: `bash skills/watch-and-learn/scripts/install-tools.sh`. This installs the local video tools (for reading frames and transcribing audio) into `~/.video-watch-tools`. It needs internet once, and downloads a ~150MB transcription model on first use. After that it runs offline for local video files.
- **No API keys, no paid services.**

## One thing that varies between agents

The skill suggests running the "watch the video" step in a **subagent** (a separate, throwaway helper) so a pile of images doesn't clog the main conversation. If your agent can't spawn subagents, that's fine. The `SKILL.md` says exactly what to do instead: run it inline, summarize, and clear the images from context before moving on. Same outcome.

## Why the format travels

The `SKILL.md` format is an [open standard](https://agentskills.io/specification) (published December 2025). Claude Code, OpenAI Codex, Cursor, Gemini CLI, and others read it natively. Writing to that shared format is why one folder works in many places.

---
> Source: [axel-pm/watch-n-learn](https://github.com/axel-pm/watch-n-learn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
