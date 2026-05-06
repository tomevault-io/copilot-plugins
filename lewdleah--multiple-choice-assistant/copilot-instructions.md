## playwright-mcp

> Which Playwright MCP server to use for browser testing


# Playwright MCP Server

**NEVER** use `cursor-ide-browser` MCP server. It opens Cursor's built-in browser, which has no connection to the Chrome dev profile where MCA is loaded.

**ALWAYS** use `user-playwright-extension` MCP server. This is the Playwright instance connected to the Chrome dev profile via CDP (`--remote-debugging-port=9222`).

```
# Correct
CallMcpTool server="user-playwright-extension" toolName="browser_tabs"
CallMcpTool server="user-playwright-extension" toolName="browser_snapshot"
CallMcpTool server="user-playwright-extension" toolName="browser_click"

# WRONG — never do this
CallMcpTool server="cursor-ide-browser" toolName="browser_tabs"
```

---
> Source: [LewdLeah/Multiple-Choice-Assistant](https://github.com/LewdLeah/Multiple-Choice-Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
