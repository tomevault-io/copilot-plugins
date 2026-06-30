## laskobot

> **ALWAYS USE THIS FOR DEPLOYMENT:** `./scripts/deploy`

# Browser MCP - Enhanced Browser Automation

## 🚀 Quick Deploy
**ALWAYS USE THIS FOR DEPLOYMENT:** `./scripts/deploy`
- **IMPORTANT:** Always use `./scripts/deploy` for ANY browser MCP deployment
- Smart version checking (compares source vs deployed)
- Automatic version bumping with validation (patch/minor/major)
- Creates timestamped backups (keeps last 5)
- Rollback capability: `./scripts/deploy --rollback`
- Configuration management: `./scripts/deploy --config`
- Robust error handling and status checks
- **DO NOT MANUALLY COPY FILES** - Let the script handle everything

## Architecture Modes

### 🎯 **UNIFIED MODE** (Recommended - NEW!)
**Industry-standard single-listener architecture**
- **Entry:** `dist/index-unified.js`
- **Port:** Single port 8765 for ALL instances
- **Benefits:**
  - ✅ No port scanning needed
  - ✅ Scales to unlimited Claude instances
  - ✅ Simpler firewall rules (1 port)
  - ✅ Follows Selenium/Playwright/Chrome DevTools pattern
  - ✅ Lower memory footprint
- **Connection:** `ws://localhost:8765/session/<instanceId>`

### Multi-Instance Mode (Legacy)
- **Entry:** `dist/index-multi.js`
- **Ports:** 8765-8775 (11 pre-created servers)
- **Limit:** Max 11 concurrent instances
- **Note:** Will be deprecated in future release

### HTTP Mode
- **Entry:** `dist/index-http.js`
- **Port:** HTTP 3000 + dynamic WebSocket allocation
- **Use Case:** Alternative transport for specific setups

## Components

### 1. MCP Server
📦 **Location:** `/home/david/.local/lib/browsermcp-enhanced/`
- **Running:** Via Claude's MCP integration (configured in `~/.claude/mcp_servers.json`)
- **Recommended Entry:** `/home/david/.local/lib/browsermcp-enhanced/dist/index-unified.js`
- **Manual Update:**
  1. Bump version in `package.json`
  2. Build: `npm run build`
  3. Copy: `cp -r dist/* /home/david/.local/lib/browsermcp-enhanced/dist/`
  4. **RESTART CLAUDE** - Required for new MCP code to load!

### 2. Chrome Extension
🔧 **Location:** `/home/david/.local/lib/browsermcp-enhanced/chrome-extension/`
- **Install:** Load unpacked in `chrome://extensions/`
- **Purpose:** Enables browser automation from MCP server
- **Features:**
  - Smart click detection (automatically uses trusted clicks for OAuth/popups)
  - Enhanced element detection with component-based capture
  - Accessibility-aware element selection
- **Manual Update:** 
  1. Copy: `cp -r chrome-extension/* /home/david/.local/lib/browsermcp-enhanced/chrome-extension/`
  2. Run: `./scripts/chrome-canary-restart.sh` to restart browser
  3. Reload extension in `chrome://extensions/`

## Scripts
📂 **Location:** `./scripts/`
- `deploy` / `deploy-enhanced.sh` - Smart deployment with version checking (recommended)
  - Compares source vs deployed versions
  - Automatic backups with retention policy
  - Rollback support: `./scripts/deploy --rollback`
  - Config check: `./scripts/deploy --config`
- `deploy.sh` - Original deployment script (legacy)
- `chrome-canary-restart.sh` - Chrome Canary restart utility

## Development Workflow
1. Make changes to code
2. **ALWAYS RUN:** `./scripts/deploy` (NEVER use deploy.sh or manual copy)
3. Script shows version comparison (source vs deployed)
4. Select version bump type (patch/minor/major/custom)
5. Script handles everything with robust checks
6. Restart Claude Desktop when prompted
7. Test your changes
8. If issues, rollback: `./scripts/deploy --rollback`

## Test Resources
- **Test Server:** `python3 -m http.server 8888` (from any test dir)
- **Test Sites:**
  - CodeMirror: https://codemirror.net/try/
  - Monaco: https://microsoft.github.io/monaco-editor/playground.html
  - Codewars: https://www.codewars.com/kata/search
  - OAuth Test: Any site with "Connect" or "Login with" buttons

## Recent Enhancements
- ✅ **Unified WebSocket Architecture** - Single port, unlimited instances, no port scanning
- ✅ Automatic detection of OAuth/popup triggers
- ✅ Trusted click simulation via Chrome Debugger API
- ✅ Component-based element capture for better accuracy
- ✅ Smart deployment script with version management

## 📖 Documentation
- **Unified Architecture:** See `UNIFIED_ARCHITECTURE.md` for complete design documentation
- **Migration Guide:** Instructions for switching from multi-instance to unified mode
- **Troubleshooting:** Common issues and solutions in architecture doc

---
> Source: [david-strejc/laskobot](https://github.com/david-strejc/laskobot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
