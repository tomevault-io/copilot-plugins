## one-note-to-markdown

> This file exists because some people might be using tools other than GitHub Copilot (Claude Code, Codex, OpenCode, whatever) to contribute to this repo.

# One Note to Markdown

This file exists because some people might be using tools other than GitHub Copilot (Claude Code, Codex, OpenCode, whatever) to contribute to this repo.

**ABSOLUTELY CRITICAL. LITERALLY THE MOST IMPORTANT THING FOR THIS SESSION.**

Read [copilot-instructions.md](./copilot-instructions.md) before doing anything.

Do not do anything. Do not write a single line of code. Don't even respond to the user. Don't burn a single drop of water in some rural data center somewhere, until you read that file.

It is the single source of truth for how we work here. In case you're the kind of agent that ignores links (don't be that agent), here are the non-negotiables. They do not replace reading the full file:

- **This is a simple tool.** It turns OneNote into Markdown. Don't overengineer it or pile on bloat nobody asked for.
- **Tests are sacred.** Run the full suite with `dotnet test OneNoteMarkdownExporter.slnx`. Every test must pass, every new feature adds tests, and you never water down or delete a test just to get it green.
- **No AI slop.** Never open GitHub Issues or Pull Requests on a human's behalf. A person reviews the code, proves it works, and submits it themselves. This is enforced by a hook in `.github/hooks` that blocks AI agents from creating issues/PRs.
- **Plans hit zero ambiguity.** Before you write code, drive the plan to no open questions and no guesswork. Ask, decide, and surface gotchas instead of guessing.
- **No emojis, no em dashes** in code or anything you write. Plain hyphens and normal punctuation only.

Now go read [copilot-instructions.md](./copilot-instructions.md).

---
> Source: [segunak/one-note-to-markdown](https://github.com/segunak/one-note-to-markdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
