## swarmdirector

> Before using ANY Browser Tools MCP functions, you **MUST** execute in a new terminal:

# Browser Tools MCP Usage Requirements

## **MANDATORY Prerequisite**
Before using ANY Browser Tools MCP functions, you **MUST** execute in a new terminal:

```bash
npx @agentdeskai/browser-tools-server@latest
```

## **Browser Tools MCP Functions that Require the Server**
- `mcp_Browser_Tools_getConsoleLogs`
- `mcp_Browser_Tools_getConsoleErrors`
- `mcp_Browser_Tools_getNetworkErrors`
- `mcp_Browser_Tools_getNetworkLogs`
- `mcp_Browser_Tools_takeScreenshot`
- `mcp_Browser_Tools_getSelectedElement`
- `mcp_Browser_Tools_wipeLogs`
- `mcp_Browser_Tools_runAccessibilityAudit`
- `mcp_Browser_Tools_runPerformanceAudit`
- `mcp_Browser_Tools_runSEOAudit`
- `mcp_Browser_Tools_runNextJSAudit`
- `mcp_Browser_Tools_runDebuggerMode`
- `mcp_Browser_Tools_runAuditMode`
- `mcp_Browser_Tools_runBestPracticesAudit`

## **Process**
1. **ALWAYS check if browser tools server is running first**
2. If not running, execute: `npx @agentdeskai/browser-tools-server@latest` as background process
3. Wait for server to start before attempting any Browser Tools MCP calls
4. Use Puppeteer tools for basic navigation/screenshots if browser tools server is not available

## **Alternative**
If Browser Tools MCP is not available, use Puppeteer MCP tools:
- `mcp_Puppeteer_puppeteer_navigate`
- `mcp_Puppeteer_puppeteer_screenshot`
- `mcp_Puppeteer_puppeteer_click`
- `mcp_Puppeteer_puppeteer_fill`
- `mcp_Puppeteer_puppeteer_evaluate`

## **Important Notes**
- Browser tools server runs as a background process
- Must be running throughout the session for Browser Tools MCP to work
- Failure to start server will result in "Failed to discover browser connector server" errors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blopit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
