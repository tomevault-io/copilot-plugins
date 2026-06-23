## mcp-browser-tools-setup

> Use of browser tools MCP for UI debugging

# MCP Browser Tools Setup & Troubleshooting Protocol

## 🚨 CRITICAL: Browser Tools MCP Requires 3-Component Setup

Sprint 10 revealed that Browser Tools MCP is **COMPLEX** and fails silently without proper setup. **ALWAYS verify all components**.

### **✅ MANDATORY 3-COMPONENT VERIFICATION:**

#### 1. Node.js Version Check (CRITICAL)
```bash
# ✅ MUST be Node.js v18+ (requires native fetch API)
node --version  # Must show v18.0.0 or higher

# ❌ If version is <18:
nvm install 20
nvm use 20
nvm alias default 20
```

#### 2. Browser Tools Server (Port 3025)
```bash
# ✅ Start the middleware server
npx @agentdeskai/browser-tools-server@latest

# ✅ Verify server is running
netstat -an | findstr 3025  # Windows
lsof -i :3025              # Mac/Linux

# ✅ Should see output like:
# TCP    0.0.0.0:3025    0.0.0.0:0    LISTENING
```

#### 3. Chrome Extension Installation
```bash
# ✅ Clone and install extension
git clone https://github.com/AgentDeskAI/browser-tools-mcp.git
cd browser-tools-mcp
```

**Chrome Extension Steps:**
1. Open Chrome → `chrome://extensions/`
2. Enable "Developer mode" (top right toggle)
3. Click "Load unpacked"
4. Select folder: `browser-tools-mcp/chrome-extension`
5. Verify "BrowserToolsMCP" appears in extensions list

### **🔍 VERIFICATION PROTOCOL:**

#### Test MCP Connection
```bash
# ✅ Test 1: Basic MCP connectivity
mcp_browser-tools_wipeLogs
# Expected: "All logs cleared successfully"

# ✅ Test 2: Server communication
mcp_browser-tools_getConsoleLogs
# Expected: [] (empty array) or actual logs

# ✅ Test 3: Full functionality (requires extension)
mcp_browser-tools_takeScreenshot
# Expected: Screenshot data or specific error message
```

#### Extension Connection Check
1. Open any webpage (e.g., https://example.com)
2. Open Chrome DevTools (F12)
3. Look for "BrowserTools" tab in DevTools
4. Verify connection status shows "Connected"

### **❌ COMMON ERROR PATTERNS & FIXES:**

#### Error: "Failed to discover browser connector server"
```bash
# ✅ FIX: Start Browser Tools Server
npx @agentdeskai/browser-tools-server@latest

# ✅ Verify server is accessible
curl http://localhost:3025/health
```

#### Error: "Chrome extension not connected"
```bash
# ✅ FIX: Reinstall extension
# 1. Remove extension from chrome://extensions/
# 2. Reload unpacked extension
# 3. Refresh target webpage
# 4. Check DevTools for "BrowserTools" tab
```

#### Error: "Error taking screenshot"
```bash
# ✅ FIX: Extension installed but not connected to active tab
# 1. Refresh the target webpage
# 2. Open DevTools (F12)
# 3. Check "BrowserTools" tab connection status
# 4. Try screenshot again
```

#### Error: Node.js version issues
```bash
# ✅ FIX: Upgrade Node.js
# Browser Tools MCP requires Node.js v18+ for native fetch API
nvm install 20
nvm alias default 20
node --version  # Verify v20.x.x
```

### **🚀 BEFORE USING BROWSER TOOLS MCP:**

**ALWAYS run this verification sequence:**

```bash
# 1. Check Node.js version
node --version  # Must be ≥18.0.0

# 2. Start server (in separate terminal)
npx @agentdeskai/browser-tools-server@latest &

# 3. Verify server
curl http://localhost:3025/health || echo "Server not running"

# 4. Test basic MCP connectivity
mcp_browser-tools_wipeLogs

# 5. Open target webpage and check extension
# - Go to target website
# - Open DevTools (F12)
# - Verify "BrowserTools" tab exists and shows "Connected"

# 6. Test full functionality
mcp_browser-tools_takeScreenshot
```

### **🔧 DEVELOPMENT WORKFLOW:**

#### Session Start Checklist
- [ ] **Node.js ≥18**: `node --version` shows v18+
- [ ] **Server Running**: Port 3025 accessible
- [ ] **Extension Loaded**: Chrome extension installed and enabled
- [ ] **Extension Connected**: DevTools shows "BrowserTools" tab
- [ ] **MCP Responsive**: `wipeLogs` returns success
- [ ] **Full Functionality**: Screenshot works

#### Debugging Browser Tools Issues
```bash
# Debug server connectivity
curl -v http://localhost:3025/health

# Debug extension installation
chrome://extensions/ # Check if "BrowserToolsMCP" is listed

# Debug extension connection
# Open DevTools → BrowserTools tab → Check connection status

# Debug MCP server communication
mcp_browser-tools_getConsoleLogs  # Should not timeout
```

### **📋 TROUBLESHOOTING FLOWCHART:**

```
Browser Tools MCP not working?
├── Is Node.js ≥18? → NO: Upgrade Node.js
├── Is server running on 3025? → NO: Start server
├── Is Chrome extension installed? → NO: Install extension
├── Is extension connected? → NO: Refresh page + DevTools
└── All YES → Should work, check specific error message
```

### **⚠️ CRITICAL NOTES:**

1. **Three Components Required**: MCP server, Browser Tools server, Chrome extension
2. **Node.js Version Critical**: v18+ required for fetch API
3. **Server Must Stay Running**: Keep `browser-tools-server` process alive
4. **Extension Per Session**: May need to refresh extension connection
5. **DevTools Required**: Extension only works with DevTools open

### **🎯 SUCCESS INDICATORS:**

- Node.js shows v18+ when running `node --version`
- Port 3025 responds to health checks
- Chrome extensions page shows "BrowserToolsMCP" installed
- DevTools shows "BrowserTools" tab with "Connected" status
- `mcp_browser-tools_wipeLogs` returns success message
- `mcp_browser-tools_takeScreenshot` returns image data

---

*Browser Tools MCP setup is complex but powerful when properly configured. Always verify all three components.*

---
> Source: [rm2thaddeus/Pixel_Detective](https://github.com/rm2thaddeus/Pixel_Detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
