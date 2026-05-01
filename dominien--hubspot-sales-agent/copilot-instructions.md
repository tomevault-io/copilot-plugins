## hubspot-sales-agent

> This Sales Agent is **harness-agnostic**. The same skill files can run on any local agent harness — Claude Code, Cursor, Continue, Windsurf, OpenCode, Aider, custom Node.js/Python agents, or anything else that can read markdown and execute shell commands.

# Agent Harness Compatibility

This Sales Agent is **harness-agnostic**. The same skill files can run on any local agent harness — Claude Code, Cursor, Continue, Windsurf, OpenCode, Aider, custom Node.js/Python agents, or anything else that can read markdown and execute shell commands.

---

## How It Works

Every skill file (`skills/*.md`) references **two interchangeable tool paths**:

- **Path A — MCP tools:** `mcp__hubspot__*`, `mcp__gmail__*` (prefix varies by harness — see CLAUDE.md)
- **Path B — Local CLI tools:** `npx tsx src/tools/hubspot.ts`, `npx tsx src/tools/gmail.ts`, `npx tsx src/tools/webfetch.ts`

Both paths return JSON. The agent's logic (classify, extract, draft, log) is identical regardless of which path you use.

---

## Path A — MCP (any MCP-capable harness)

MCP ([Model Context Protocol](https://modelcontextprotocol.io)) is an open protocol. Any harness that implements an MCP client can use MCP servers — it's not exclusive to Claude Code.

**MCP-capable harnesses include:**
- Claude Code
- Cursor
- Continue
- Windsurf
- Zed
- Custom harnesses using any MCP client library

**Setup:**
1. Install the HubSpot MCP server and Gmail MCP server in your harness
2. Load `skills/<skill>.md` + `CLAUDE.md` into the agent context
3. The agent calls MCP tools as it processes the loop

**When MCP is the right choice:** You're already using an MCP-capable harness and have the servers installed. MCP handles auth via the harness itself — no `.env` needed for the MCP path.

---

## Path B — Local CLI tools (universal fallback)

For any harness without MCP support, or when you prefer a simpler setup:

**Setup:**
1. `npm install`
2. `cp .env.example .env` and fill in credentials (HubSpot + Google OAuth)
3. Verify: `npx tsx src/tools/hubspot.ts --help`

**The agent's loop shells out to `npx tsx src/tools/*.ts` commands and parses JSON from stdout.**

**When CLI is the right choice:**
- Your harness doesn't support MCP yet
- You want to debug tool calls directly in your terminal (`npx tsx src/tools/hubspot.ts contacts list --limit 5`)
- You prefer a minimal dependency footprint
- You're building a custom Node.js or Python agent loop

---

## Mixing Paths

You can mix both. For example: use MCP for HubSpot (fast, no auth hassle) but CLI for webfetch (no MCP server for generic webfetch exists). The skill files reference both, so the agent picks whichever is available.

---

## Example: Minimal Harness Loop (CLI path)

Pseudocode for running `follow-up-loop` on any harness without MCP:

```
1. Read skills/follow-up-loop.md + CLAUDE.md + knowledge/*.md
2. Execute: npx tsx src/tracker.ts read → skip set
3. Execute: npx tsx src/tools/hubspot.ts contacts list --limit 100
4. For each contact not in skip set:
   a. Execute: npx tsx src/tools/hubspot.ts notes list --contact-id <id>
   b. Check skip flags in notes
   c. Generate email (LLM call using CLAUDE.md rules)
   d. Execute: npx tsx src/tools/gmail.ts draft create --to X --subject Y --body Z
   e. Execute: npx tsx src/tracker.ts append "..."
5. Continue until contacts exhausted or interrupted
```

Your harness handles the LLM calls and the tool execution loop. The CLI tools handle the HubSpot/Gmail API plumbing.

---

## Adding a New Harness Adapter

If your harness needs different tool plumbing or conventions:

1. Keep `skills/*.md`, `CLAUDE.md`, and `knowledge/*.md` unchanged (they're harness-agnostic)
2. Add a new adapter doc: `docs/harness-<name>.md`
3. Optionally add thin wrappers in `src/tools/` if your harness has different API conventions
4. Open a PR — we welcome contributions

---

## Why Both Paths?

**MCP is great when available:**
- Handles auth through the harness
- Integrated into the chat context
- No `.env` management

**CLI is great as a fallback:**
- Works on harnesses without MCP clients
- Debuggable directly in your terminal (`npx tsx src/tools/hubspot.ts contacts list`)
- Language-agnostic — any harness that can exec shell commands can use it
- Same logic works in Node.js, Python, Bash, or any orchestration tool

Use whichever matches your setup — or mix them.

---
> Source: [Dominien/hubspot-sales-agent](https://github.com/Dominien/hubspot-sales-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
