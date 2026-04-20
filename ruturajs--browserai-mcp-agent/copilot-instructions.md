## browserai-mcp-agent

> If you want to use the **Browser Automation MCP Server** with the Claude Desktop app, you can copy-paste the template below into your configuration file.

# Claude Desktop Configuration Sample

If you want to use the **Browser Automation MCP Server** with the Claude Desktop app, you can copy-paste the template below into your configuration file.

### 📍 Configuration File Location:
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
- **Mac**: `~/Library/Application Support/Claude/claude_desktop_config.json`

---

### 📝 Sample Configuration Template
Replace `C:/path/to/your/project/BrowserAutoMCP/` with your actual project directory path.

```json
{
  "mcpServers": {
    "browser-auto": {
      "command": "npx",
      "args": [
        "-y",
        "ts-node",
        "C:/path/to/your/project/BrowserAutoMCP/src/index.ts"
      ],
      "env": {
        "NODE_PATH": "C:/path/to/your/project/BrowserAutoMCP/node_modules"
      }
    }
  },
  "preferences": {
    "sidebarMode": "chat"
  }
}
```

---

### 🚀 Troubleshooting Fix:
If your server says **"disconnected"**, it is likely because:
1.  **Node.js is not present** in your system path.
2.  **The path to `src/index.ts` is wrong**.
3.  **The browser is not installed**. Ensure you run:
    ```powershell
    npx playwright install chromium
    ```

**After saving the config, RESTART Claude Desktop to apply the changes.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RuturajS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
