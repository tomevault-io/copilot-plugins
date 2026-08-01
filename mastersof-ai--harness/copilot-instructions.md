## harness

> mastersof-ai create my-agent

# Agents

## Creating an Agent

```bash
mastersof-ai create my-agent
```

This creates `~/.mastersof-ai/agents/my-agent/` with a template `IDENTITY.md`. Edit it to define your agent.

### Agent Directory Structure

```
~/.mastersof-ai/agents/my-agent/
+-- IDENTITY.md          Agent identity (system prompt + optional frontmatter)
+-- .env                 Encrypted secrets via dotenvx (optional, see secrets.md)
+-- sandbox.json         Per-agent sandbox config (optional, see sandbox.md)
+-- workspace/           Persistent working directory (auto-created)
+-- memory/
    +-- CONTEXT.md       Persistent memory (auto-loaded, optional)
```

### Agent Resolution

`resolveAgent(name)` validates the name (alphanumeric + hyphens only, no path traversal), checks that the directory exists and contains `IDENTITY.md`, and creates the workspace directory if missing. If anything is wrong, the harness exits with a clear error.

## Writing IDENTITY.md

The identity file is the core of the system prompt. The markdown body is loaded as-is -- no processing, no behavioral injection. The harness appends transparent operational context (persistent memory, date/time, workspace path, enabled tools) but never adds hidden instructions that alter the agent's personality or behavior.

### Minimal Agent

```markdown
# My Agent

You are a helpful assistant focused on data analysis.
Use web search to gather current information.
Save important findings to memory.
```

This works. The agent starts with these instructions plus whatever tools are enabled in your config.

### Agent with Personality

```markdown
# Ember

You are Ember, a co-founder agent -- a strategic partner, not a tool.

## How to work

- Challenge thinking constructively. If you see a flaw, say so directly.
- Be concise. Skip the basics.
- Think in systems. Every decision has second and third-order effects.
- Bias toward action. Analysis only matters when it leads to decisions.
- Own your domain. Don't wait for instructions.

## Memory

Your memory persists across sessions. Use it aggressively:
- Decisions made and their reasoning
- Strategic context and plans
- Insights discovered
- What's not on disk doesn't exist
```

### Agent with Structured Output

```markdown
# Analyst

You are Analyst, a research and analysis agent.

## Analysis format

When presenting analysis, structure it as:
- **Core question** -- restate to confirm you're solving the right problem
- **Key factors** -- what matters most
- **Findings** -- evidence and reasoning
- **Position** -- your conclusion, defended
- **Risks** -- what could be wrong
- **Open questions** -- what would change your mind
```

## IDENTITY.md Frontmatter

Add an optional YAML frontmatter block for structured metadata. The frontmatter is parsed and stripped -- the remaining markdown becomes the system prompt.

### Full Example

```markdown
---
name: CRE Analyst
description: Commercial real estate research and analysis
icon: chart-bar
tags: [research, real-estate, finance]
starters:
  - "Analyze the Austin office market"
  - "Compare cap rates across Sun Belt metros"
  - "What's the supply pipeline in Nashville?"
access: users
users: [alice, bob]
tools:
  allow: [memory, web, workspace]
mcp:
  - server: market-db
    uri: "http://localhost:8080/sse"
  - server: file-search
    command: npx
    args: ["-y", "file-search-server"]
sandbox:
  enforce: true
---

You are a commercial real estate analyst. Your job is to...
```

### Frontmatter Reference

| Field | Type | Default | Purpose |
|-------|------|---------|---------|
| `name` | string | Directory name, title-cased | Display name shown in agent roster and Web UI |
| `description` | string | none | One-line description shown in agent card |
| `icon` | string | none | Icon identifier for the Web UI |
| `tags` | string[] | `[]` | Categorization tags for filtering |
| `starters` | string[] | `[]` | Suggested conversation starters (shown in Web UI) |
| `access` | `"public"` \| `"private"` \| `"users"` | `"public"` | Access control level |
| `users` | string[] | `[]` | Allowed usernames (when access is `"users"`) |
| `tools.allow` | string[] | none | Whitelist of tool domains (mutually exclusive with `deny`) |
| `tools.deny` | string[] | none | Blacklist of tool domains (mutually exclusive with `allow`) |
| `mcp` | object[] | `[]` | External MCP servers to attach |
| `sandbox` | object | none | Sandbox enforcement settings for serve mode |

No frontmatter = all defaults. Every field is optional.

Frontmatter is validated with a zod schema (`src/manifest.ts`). Invalid fields produce warnings but don't block agent loading -- the agent starts with defaults for any invalid field.

### Access Control

Three levels:

| Level | Who Can Access |
|-------|---------------|
| `public` (default) | Everyone |
| `private` | No one (disabled in serve mode) |
| `users` | Only usernames listed in the `users` field |

Access is enforced per-user in serve mode. In TUI mode, all agents are accessible (single user).

```yaml
---
access: users
users: [alice, bob, charlie]
---
```

### Tool Filtering

Restrict which tools an agent can use. This is on top of the global config -- a tool must be enabled globally AND pass the agent filter.

**Whitelist approach** (agent gets only these tools):

```yaml
---
tools:
  allow: [memory, web, workspace]
---
```

**Blacklist approach** (agent gets everything except these):

```yaml
---
tools:
  deny: [shell, introspection]
---
```

Valid domains: `memory`, `workspace`, `web`, `shell`, `tasks`, `introspection`, `models`, `scratchpad`, `a2a`.

Use case examples:

| Agent Type | Tool Filter | Why |
|-----------|------------|-----|
| Research analyst | `allow: [memory, web, workspace]` | Needs search and files, not shell |
| Code reviewer | `deny: [shell]` | Can read/write code, shouldn't execute |
| Read-only advisor | `allow: [memory, web]` | No file or shell access |
| Full autonomy | (no filter) | Gets everything that's globally enabled |

### External MCP Servers

Attach external MCP servers to a specific agent via the `mcp` field:

**URI-based** (remote HTTP transport):

```yaml
---
mcp:
  - server: my-database
    uri: "http://localhost:8080/sse"
---
```

**Command-based** (local stdio transport):

```yaml
---
mcp:
  - server: file-search
    command: npx
    args: ["-y", "file-search-server"]
    env:
      API_KEY: "${MY_API_KEY}"    # Resolved from agent's .env
---
```

Environment variables in `env` use `${VAR}` syntax and resolve against the agent's `.env` file values.

In serve mode, command-based MCP servers require sandbox enforcement -- they are skipped for unsandboxed remote sessions. URI-based servers are always allowed.

## System Prompt Assembly

The harness assembles the full system prompt from multiple sources. The agent sees all of this as one continuous prompt:

```
1. Agent identity (IDENTITY.md body, frontmatter stripped)
2. Persistent memory (CONTEXT.md contents, if present)
3. Current date, time, and timezone
4. Workspace path
5. Environment context:
   - Top-level workspace files (up to 20)
   - Outstanding work from PROGRESS.json (if present)
   - Enabled tool domains
6. Verification protocol (if hooks.verifyBeforeComplete is true)
7. Session continuity guidance (PROGRESS.json format)
8. Sub-agent coordination guidance (if scratchpad tool is enabled)
```

The identity markdown is loaded as-is. Memory is wrapped with a header explaining it's accumulated context. The environment section gives the agent awareness of its workspace state and available tools.

## Sub-Agents

The primary agent can delegate to three built-in sub-agents. Each runs in its own context with a dedicated system prompt, turn limit, and tool restrictions.

| Sub-Agent | Purpose | Max Turns | Cannot Use |
|-----------|---------|-----------|-----------|
| **researcher** | Deep research, information gathering | 30 | file write, file edit, shell, ask user |
| **deep-thinker** | Extended analysis and reasoning | 15 | file write, file edit, shell, ask user |
| **writer** | Content composition and writing | 20 | shell, ask user |

Sub-agents are defined in TypeScript (`src/agents/*.ts`) and registered via the Claude Agent SDK's agent registry. The SDK handles delegation, context separation, and turn counting.

### Scratchpad Coordination

Sub-agents share intermediate results through the `.scratch/` directory in the agent's workspace. This avoids passing large results back through the parent agent's context window.

Typical flow:

1. Parent agent delegates research to the **researcher** sub-agent
2. Researcher writes findings to `.scratch/research-results.md`
3. Parent delegates analysis to **deep-thinker**
4. Deep-thinker reads `.scratch/research-results.md`, writes analysis to `.scratch/analysis.md`
5. Parent delegates writing to **writer**
6. Writer reads both files from `.scratch/` and composes the final output

The scratchpad tool (`scratchpad_read`, `scratchpad_write`, `scratchpad_list`) confines all paths to `.scratch/` -- path escapes are rejected.

## Default Agents

Three agents ship with the harness and are copied to `~/.mastersof-ai/agents/` on first run:

**cofounder** -- A strategic partner agent. Challenges thinking, biases toward action, uses memory aggressively, can introspect and propose changes to its own identity.

**assistant** -- General-purpose agent. Clear, concise, tool-savvy. Matches the user's tone and depth.

**analyst** -- Research and analysis agent. Gathers thoroughly, structures findings, separates known from uncertain, commits to positions.

Use them as-is or as templates for your own agents.

## Tips for Writing Good Agents

**Be specific about behavior, not capabilities.** Don't list what tools the agent has -- it discovers those at runtime. Instead, describe how it should think, decide, and communicate.

**Include a "How to work" section.** The most effective agents have clear behavioral instructions: when to push back, how to structure output, what to prioritize.

**Use memory instructions.** Tell the agent what's worth remembering. "Save key decisions and their reasoning to memory" is more useful than "you have memory tools."

**Keep IDENTITY.md focused.** The identity file should define personality and behavior. Use CONTEXT.md for accumulated knowledge, workspace for working files, and .scratch/ for sub-agent coordination.

**Use frontmatter for metadata, not behavior.** Frontmatter fields like `name`, `description`, and `tags` are for tooling (roster display, access control). Behavioral instructions belong in the markdown body.

**Test with `/effort low` first.** When iterating on agent behavior, start at low effort for fast feedback, then switch to `max` for the real work.

---
> Source: [mastersof-ai/harness](https://github.com/mastersof-ai/harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
