## ai-os-starter

> > This is an **AI OS cockpit** built on the **FRAME** method. It holds one or more OSes — each OS does one job.

# AI OS — cockpit

> This is an **AI OS cockpit** built on the **FRAME** method. It holds one or more OSes — each OS does one job.
> Any AI tool that opens this folder should read this file first.
> *(Tool-neutral: Claude Code, Codex, Cursor, or a hosted agent can all run this. Each just needs to be pointed at this file.)*

## This folder is the cockpit — always launch here

The user opens their AI tool **inside this folder**, never above it or inside a sub-folder. This is the launch point, so this is where everything must live to be reachable:
- **Skills + agents** → `.claude/skills/` and `.claude/agents/` **here at the cockpit root**.
- **Connections (MCP)** → `.mcp.json` **here at the cockpit root**.
- **Secrets (API keys)** → `.env.local` **here** (gitignored, never committed). `.env.example` documents which keys are needed.
- **Each OS** → its own folder in the cockpit (e.g. `content-os/`), holding only its **brain** (`FOCUS.md`, `resources/`, `ACCESS.md`, `ENGINE.md`). An OS folder never holds tools — tools nested in a sub-folder don't load.

## If the user wants to build or change an OS → run `build-os`

When the user says **"build-os"**, *"build me an OS"*, *"add to my OS"*, *"add a skill"*, *"connect a tool"*, or describes a job they want set up, run the **`build-os`** skill (`.claude/skills/build-os/`). It reads what already exists, then either creates a new OS or extends one — building the brain, the skills, and the connections in the right places. Don't hand-roll this; use the skill so everything lands correctly and stacks.

## How an OS works (FRAME)

Each OS folder spells FRAME:
- **F · Focus** → `FOCUS.md` — the one job + what "done" looks like.
- **R · Resources** → `resources/` — what the AI must know (voice, rules, examples). Plain text files the user owns and can edit.
- **A · Access** → `ACCESS.md` + the cockpit's `.mcp.json` (keys in `.env.local`) — what it can reach.
- **M · Make** → the skills/agents in the **cockpit's** `.claude/` that do this OS's work.
- **E · Engine** → `ENGINE.md` — how it runs (start by hand; automate later).

To run an OS's job, use its skill and point it at that OS's folder by path.

## How to work here

- Stay on the job the user asked for. One OS, one job.
- Be direct and friendly. Explain what you're doing in a line as you go — the user is often new to this.
- Start manual. Don't automate anything until the by-hand version is trusted.
- Keep API keys in `.env.local` only — never write a raw key into a committed file.
- When unsure, ask rather than guess — especially before anything that sends, posts, deletes, or spends.

---
*Built on FRAME · Focus · Resources · Access · Make · Engine*

---
> Source: [mrdainami/ai-os-starter](https://github.com/mrdainami/ai-os-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
