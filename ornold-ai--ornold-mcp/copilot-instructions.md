## ornold-mcp

> You are an AI agent. A user has asked you to install Ornold MCP. Follow these steps.

# Instructions for AI Agents

You are an AI agent. A user has asked you to install Ornold MCP. Follow these steps.

## What is Ornold MCP?

Ornold MCP (`mcp-ornold-browser`) is an MCP server for browser automation: control antidetect browsers (Linken Sphere, Dolphin Anty, Wade X/Wadex, Vision Browser, AdsPower, Octo Browser, Multilogin, MoreLogin, 0DETECT, GoLogin, Undetectable, Incogniton, Indigo), solve CAPTCHAs, record and replay automation flows, and interact with web pages through vision or DOM.

npm package: `mcp-ornold-browser`

## Prerequisites

- **Node.js ≥ 18** — check with `node -v`. If missing, tell the user to install from https://nodejs.org
- **Ornold token** — starts with `orn_sk_`. Ask the user for it, or tell them to get one at https://mcp.ornold.com (sign up → Tokens page → Create token)

## Installation by Agent Type

Ask the user which antidetect browser and mode they need before installing. See "Recommended Setup" below for splitting into multiple servers.

### Claude Code
```bash
claude mcp add --transport stdio ornold-browser -- npx mcp-ornold-browser --token TOKEN --mode vision
npx mcp-ornold-browser --install-skill
```

### Cursor
Write to `~/.cursor/mcp.json` (create if missing, merge if exists):
```json
{
  "mcpServers": {
    "ornold-browser": {
      "command": "npx",
      "args": ["mcp-ornold-browser", "--token", "TOKEN"]
    }
  }
}
```
Tell user to restart Cursor.

### VS Code / Copilot
```bash
code --add-mcp '{"name":"ornold-browser","command":"npx","args":["mcp-ornold-browser","--token","TOKEN"]}'
```

### Codex
Write to `~/.codex/config.toml` (create if missing):
```toml
[mcp_servers.ornold-browser]
command = "npx"
args = ["-y", "mcp-ornold-browser", "--token", "TOKEN"]
```
Desktop apps don't inherit terminal PATH. If MCP fails, use absolute path from `which npx`.

### Windsurf
Tell user: Settings → Tools → Add MCP Server, paste:
```json
{
  "mcpServers": {
    "ornold-browser": {
      "command": "npx",
      "args": ["mcp-ornold-browser", "--token", "TOKEN"]
    }
  }
}
```

### Cline
Tell user: Cline sidebar → MCP → Add, paste:
```json
{
  "mcpServers": {
    "ornold-browser": {
      "command": "npx",
      "args": ["mcp-ornold-browser", "--token", "TOKEN"],
      "disabled": false
    }
  }
}
```

## Recommended Setup: Separate MCP Servers

**Important:** Create separate MCP server instances instead of loading everything into one. This keeps the tool list small and focused — the agent picks the right tool more reliably.

**Rule of thumb:** one MCP server = one interaction mode + one antidetect browser.

### Why split by mode?

- `dom` mode loads ~15 tools (snapshot, click, type, fill, hover, drag, select...)
- `vision` mode loads ~12 tools (screenshot, vision_analyze, click_normalized_box, flow recording...)
- `both` loads ~27 tools — the agent gets confused which click/type to use

Split them: one server for DOM work, one for vision work. If you only use one mode, one server is fine.

### Why split by antidetect browser?

- Each antidetect browser adds its own management tools (for example: Linken, Dolphin, Octo, AdsPower, Multilogin)
- If you use multiple antidetect browsers, create a separate MCP server per browser
- If you only use one antidetect browser, one server is enough

### Example: Claude Code with Linken Sphere (vision + dom)

```bash
# Vision mode — for stealth automation, flow recording
claude mcp add --transport stdio ornold-vision -- npx mcp-ornold-browser --token TOKEN --mode vision --linken-port 40080

# DOM mode — for fast scraping, form filling
claude mcp add --transport stdio ornold-dom -- npx mcp-ornold-browser --token TOKEN --mode dom --linken-port 40080
```

### Example: Cursor with two antidetect browsers

```json
{
  "mcpServers": {
    "ornold-linken": {
      "command": "npx",
      "args": ["mcp-ornold-browser", "--token", "TOKEN", "--mode", "vision", "--linken-port", "40080"]
    },
    "ornold-dolphin": {
      "command": "npx",
      "args": ["mcp-ornold-browser", "--token", "TOKEN", "--mode", "vision", "--dolphin-port", "3001", "--dolphin-token", "API_TOKEN"]
    }
  }
}
```

### Example: Single antidetect browser, single mode (simplest)

If you only use one browser and one mode, one server is fine:
```bash
claude mcp add --transport stdio ornold-browser -- npx mcp-ornold-browser --token TOKEN --mode vision --linken-port 40080
```

## Antidetect Browser Flags

Ask user which antidetect browser they use. Add flags to the args array:

| Browser | Flags |
|---------|-------|
| Linken Sphere | `"--linken-port", "40080"` |
| Dolphin Anty | `"--dolphin-port", "3001", "--dolphin-token", "API_TOKEN"` |
| Wade X / Wadex | `"--wadex-port", "8080"` |
| Vision Browser | `"--vision-token", "X_TOKEN", "--vision-port", "PORT"` |
| AdsPower | `"--adspower-url", "http://local.adspower.net:50325"` and optionally `"--adspower-key", "API_KEY"` |
| Octo Browser | `"--octo-token", "OCTO_API_TOKEN", "--octo-url", "http://127.0.0.1:58888"` |
| Multilogin | `"--multilogin-token", "API_TOKEN", "--multilogin-url", "http://127.0.0.1:35000"` |
| MoreLogin | `"--morelogin-port", "40000"` or `"--morelogin-url", "http://127.0.0.1:40000"` |
| 0DETECT | `"--detect0-port", "PORT"` or `"--detect0-url", "URL"`, optionally `"--detect0-token", "TOKEN"` |
| GoLogin | `"--gologin-token", "API_TOKEN"` |
| Undetectable | `"--undetectable-port", "25325"` or `"--undetectable-url", "http://127.0.0.1:25325"` |
| Incogniton | `"--incogniton-port", "35000"` or `"--incogniton-url", "http://127.0.0.1:35000"` |
| Indigo Browser | `"--indigo-token", "API_TOKEN"` |

Provider tools appear only when the matching provider is enabled by flags or environment variables. If the user asks for Octo, AdsPower, Multilogin, MoreLogin, 0DETECT, GoLogin, Undetectable, Incogniton, or Indigo and the tools are missing, tell them to add the matching flags and restart the MCP server; do not say the browser is unsupported.

### Common provider examples

```bash
claude mcp add --transport stdio ornold-octo -- npx -y mcp-ornold-browser@latest --token TOKEN --mode vision --octo-token OCTO_TOKEN --octo-url http://127.0.0.1:58888
claude mcp add --transport stdio ornold-adspower -- npx -y mcp-ornold-browser@latest --token TOKEN --mode vision --adspower-url http://local.adspower.net:50325
```

## Mode Selection

Add `"--mode", "MODE"` to args:
- `dom` — fast, DOM-based interaction (snapshot → click by selector → type text)
- `vision` — screenshot + AI detection, best for antidetect stealth (vision_analyze → click by coordinates)
- `both` — both available (not recommended — too many tools, agent gets confused)

Recommend `vision` for antidetect browser work. Use `dom` for speed-critical tasks where stealth is not a concern.

## Verify Installation

After setup, call the `browser_status` tool. If it responds, the MCP server is running correctly.

## Install Skills (Recommended)

Skills teach the AI agent how to use Ornold tools correctly — proper typing (character by character), vision workflow, anti-detection rules, flow recording. Without skills, agents may use wrong tools (e.g. JavaScript injection instead of human-like typing).

### Claude Code
```bash
npx mcp-ornold-browser --install-skill
```
Installs to `~/.claude/commands/ornold-browser.md`. Invoke with `/ornold-browser` in chat.

### Cursor
Download `skills/.cursorrules` from the repo and copy to project root:
```bash
curl -o .cursorrules https://raw.githubusercontent.com/ornold-ai/ornold-mcp/main/skills/.cursorrules
```
Or place in `~/.cursor/rules/ornold-browser.mdc` for global access.

### Windsurf
Download `skills/.windsurfrules` from the repo and copy to project root:
```bash
curl -o .windsurfrules https://raw.githubusercontent.com/ornold-ai/ornold-mcp/main/skills/.windsurfrules
```

### Cline
Download `skills/.clinerules` from the repo and copy to project root:
```bash
curl -o .clinerules https://raw.githubusercontent.com/ornold-ai/ornold-mcp/main/skills/.clinerules
```

### VS Code / GitHub Copilot
Download and place in `.github/copilot-instructions.md`:
```bash
mkdir -p .github
curl -o .github/copilot-instructions.md https://raw.githubusercontent.com/ornold-ai/ornold-mcp/main/skills/copilot-instructions.md
```

### Codex
Codex reads `AGENTS.md` automatically — no extra skill file needed.

### Full Reference
For a complete tool reference with examples of every tool, see `skills/ornold-browser.md` in the repo.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `npx: command not found` | Node.js not installed → https://nodejs.org |
| `Token invalid` | Token may be revoked → check https://mcp.ornold.com |
| MCP won't start in desktop app | Use absolute npx path: run `which npx`, use that in config |
| Connection timeout | Check if antidetect browser is running and port is correct |

---
> Source: [ornold-ai/ornold-mcp](https://github.com/ornold-ai/ornold-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
