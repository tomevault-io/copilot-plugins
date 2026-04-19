## depot-mcp

> To connect Cursor to the Home Depot MCP server:

# Home Depot MCP Server Configuration

## MCP Server Setup
To connect Cursor to the Home Depot MCP server:

1. **In Cursor Settings:**
   - Go to Settings → Extensions → MCP
   - Click "Add Server"
   - Use these settings:
     - **Name:** home-depot
     - **Command:** node
     - **Args:** dist/index.js
     - **CWD:** /Users/makaminski1337/Developer/depot-mcp

2. **Or create a config file:**
   ```json
   {
     "mcpServers": {
       "home-depot": {
         "command": "node",
         "args": ["dist/index.js"],
         "cwd": "/Users/makaminski1337/Developer/depot-mcp"
       }
     }
   }
   ```

## Available Resources
- `res://ir/news-releases` - Latest IR news releases
- `res://ir/events` - Upcoming IR events and presentations

## Available Tools
- `get_sec_filings` - Get SEC filings with filtering options

## Test Questions
Once connected, try these questions:
1. "Show me Home Depot's latest IR news"
2. "Get the last 5 SEC filings for Home Depot"
3. "What upcoming events does Home Depot have?"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MAKaminski) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
