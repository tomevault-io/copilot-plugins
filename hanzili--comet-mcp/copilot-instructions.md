## comet-mcp

> MCP server connecting Claude Code to Perplexity's Comet browser via Chrome DevTools Protocol (CDP).

# CLAUDE.md

## What This Is
MCP server connecting Claude Code to Perplexity's Comet browser via Chrome DevTools Protocol (CDP).

## Architecture
```
Claude Code → MCP Server (index.ts) → CometAI (comet-ai.ts) → CDP Client (cdp-client.ts) → Comet Browser
```

## 6 Tools
- `comet_connect` - Start/connect to Comet browser
- `comet_ask` - Send prompt, wait for response (15s default, use poll for longer)
- `comet_poll` - Check status of long-running tasks
- `comet_stop` - Stop current task
- `comet_screenshot` - Capture current page
- `comet_mode` - Switch Perplexity modes (search/research/labs/learn)

## Key Implementation Details

**Response extraction** (`comet-ai.ts:getAgentStatus`):
- Takes LAST prose element (not longest) - conversation threads show newest last
- Filters out UI text (Library, Discover, etc.) and questions (ends with ?)

**Follow-up detection** (`index.ts`):
- Captures old prose count/text before sending
- Waits for NEW response (different text or more elements)

**Prompt normalization**:
- Strips bullet points, collapses newlines to spaces

## Build & Test
```bash
npm run build
pgrep -f "node.*comet-mcp" | xargs kill  # Restart MCP
```

Manual testing only (integration code, external DOM dependency).

## Test Cases
1. **Quick queries** - Simple questions (math, facts) should return within 15s
2. **Non-blocking** - Short timeout returns "in progress", use poll to get result
3. **Follow-up** - Second question in same chat detects NEW response correctly
4. **Agentic task** - "Take control of browser and go to X" triggers browsing
5. **newChat after agentic** - `newChat: true` resets CDP state after browser control
6. **Mode switching** - `comet_mode` changes search/research/labs/learn

## Known Edge Cases
- **Prompt not submitted**: If response shows 0 steps + COMPLETED, prompt may not have been submitted. Retry or use newChat.
- **Stale poll response**: If poll returns unrelated response, the previous prompt failed. Send again.
- **Research mode**: Takes longer than search mode, may need multiple polls.

---
> Source: [hanzili/comet-mcp](https://github.com/hanzili/comet-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
