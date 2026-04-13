## rocket-path-ai

> Guidelines for using Chrome DevTools MCP for browser automation, debugging, and performance testing


# Chrome DevTools MCP - Cursor Rule

## Overview

Chrome DevTools MCP (`chrome-devtools-mcp`) provides full access to Chrome DevTools Protocol for browser automation, debugging, performance analysis, and testing. It acts as a Model-Context-Protocol (MCP) server, giving AI assistants access to Chrome DevTools capabilities.

**Repository:** [ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)  
**NPM Package:** [chrome-devtools-mcp](https://npmjs.org/package/chrome-devtools-mcp)

## Key Features

- **Performance Insights**: Record traces and extract actionable performance insights using Chrome DevTools
- **Advanced Browser Debugging**: Analyze network requests, take screenshots, check browser console
- **Reliable Automation**: Uses Puppeteer to automate Chrome actions with automatic wait for results
- **Full DevTools Access**: Complete access to Chrome DevTools Protocol capabilities

## Requirements

- **Node.js**: v20.19 or newer (latest maintenance LTS)
- **Chrome**: Current stable version or newer
- **npm**: For package management

## Configuration

### Basic Setup for Cursor

Add to `.cursor/mcp.json` or Cursor Settings → MCP → New MCP Server:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

> **Note:** Using `chrome-devtools-mcp@latest` ensures you always use the latest version.

### Advanced Configuration Options

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "chrome-devtools-mcp@latest",
        "--channel=canary",           // stable, canary, beta, dev
        "--headless=true",            // Run without UI
        "--isolated=true",            // Temporary user data dir (auto-cleanup)
        "--viewport=1920x1080",       // Initial viewport size
        "--browser-url=http://127.0.0.1:9222"  // Connect to running Chrome
      ]
    }
  }
}
```

### Configuration Options Reference

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--browserUrl`, `-u` | string | - | Connect to running Chrome via port forwarding |
| `--wsEndpoint`, `-w` | string | - | WebSocket endpoint (e.g., `ws://127.0.0.1:9222/devtools/browser/<id>`) |
| `--wsHeaders` | string | - | Custom headers for WebSocket (JSON format) |
| `--headless` | boolean | `false` | Run in headless (no UI) mode |
| `--executablePath`, `-e` | string | - | Path to custom Chrome executable |
| `--isolated` | boolean | `false` | Use temporary user-data-dir (auto-cleanup) |
| `--userDataDir` | string | `$HOME/.cache/chrome-devtools-mcp/chrome-profile-$CHANNEL` | Custom user data directory |
| `--channel` | string | `stable` | Chrome channel: `stable`, `canary`, `beta`, `dev` |
| `--logFile` | string | - | Path to debug log file |
| `--viewport` | string | - | Initial viewport size (e.g., `1280x720`) |
| `--proxyServer` | string | - | Proxy server configuration |
| `--acceptInsecureCerts` | boolean | `false` | Ignore self-signed/expired certificates |
| `--chromeArg` | array | - | Additional Chrome arguments |
| `--categoryEmulation` | boolean | `true` | Include emulation tools |
| `--categoryPerformance` | boolean | `true` | Include performance tools |
| `--categoryNetwork` | boolean | `true` | Include network tools |

## Available Tools

### Input Automation (8 tools)
- `click` - Click elements on the page
- `drag` - Drag and drop elements
- `fill` - Fill input fields
- `fill_form` - Fill entire forms
- `handle_dialog` - Handle browser dialogs (alert, confirm, prompt)
- `hover` - Hover over elements
- `press_key` - Press keyboard keys
- `upload_file` - Upload files to file inputs

### Navigation Automation (6 tools)
- `close_page` - Close a page
- `list_pages` - List all open pages
- `navigate_page` - Navigate to a URL
- `new_page` - Open a new page
- `select_page` - Switch to a specific page
- `wait_for` - Wait for conditions (element, text, time)

### Emulation (2 tools)
- `emulate` - Emulate devices (mobile, tablet, etc.)
- `resize_page` - Resize the viewport

### Performance (3 tools)
- `performance_analyze_insight` - Analyze performance and get insights
- `performance_start_trace` - Start performance tracing
- `performance_stop_trace` - Stop performance tracing

### Network (2 tools)
- `get_network_request` - Get details of a specific network request
- `list_network_requests` - List all network requests

### Debugging (5 tools)
- `evaluate_script` - Execute JavaScript in the page context
- `get_console_message` - Get a specific console message
- `list_console_messages` - List all console messages
- `take_screenshot` - Capture a screenshot
- `take_snapshot` - Capture accessibility snapshot

## When to Use Chrome DevTools MCP

### ✅ DO: Use For

1. **Performance Testing**
   - Record performance traces
   - Analyze page load times
   - Identify performance bottlenecks
   - Get actionable performance insights

2. **Browser Automation**
   - End-to-end testing workflows
   - Form filling and submission
   - Multi-step user journeys
   - Cross-browser testing (via emulation)

3. **Debugging**
   - Console error investigation
   - Network request analysis
   - JavaScript execution testing
   - Visual debugging with screenshots

4. **Accessibility Testing**
   - Take accessibility snapshots
   - Verify ARIA attributes
   - Test keyboard navigation
   - Check semantic HTML structure

5. **Local Development Testing**
   - Test localhost applications
   - Verify build outputs
   - Compare with production
   - Visual regression testing

### ❌ DON'T: Use For

- Testing on untrusted websites (security risk)
- Bypassing security measures
- Scraping sensitive data
- Automating actions without approval

## Best Practices

### ✅ DO: Performance Testing Workflow

```typescript
// 1. Navigate to the page
await navigate_page({ url: "https://developers.chrome.com" });

// 2. Start performance trace
await performance_start_trace();

// 3. Interact with the page (if needed)
await click({ selector: "button.load-more" });

// 4. Stop trace and analyze
await performance_stop_trace();
const insights = await performance_analyze_insight();

// 5. Review insights for:
// - Long tasks
// - Layout shifts
// - Slow network requests
// - JavaScript execution time
```

### ✅ DO: Console Error Checking

```typescript
// Always check console after navigation
await navigate_page({ url: "http://localhost:3000" });

// Get console messages
const consoleMessages = await list_console_messages();
const errors = consoleMessages.filter(msg => msg.level === 'error');

if (errors.length > 0) {
  // Log errors for debugging
  console.error("Browser console errors:", errors);
  
  // Take screenshot for context
  await take_screenshot();
  
  // Get detailed error info
  for (const error of errors) {
    const details = await get_console_message({ id: error.id });
    console.error("Error details:", details);
  }
}
```

### ✅ DO: Network Request Analysis

```typescript
// Monitor network requests
await navigate_page({ url: "http://localhost:3000/api/test" });

// List all network requests
const requests = await list_network_requests();

// Filter failed requests
const failed = requests.filter(req => req.status >= 400);

if (failed.length > 0) {
  // Get details of failed requests
  for (const req of failed) {
    const details = await get_network_request({ id: req.id });
    console.error("Failed request:", details);
  }
}

// Check for slow requests (> 1 second)
const slow = requests.filter(req => req.duration > 1000);
if (slow.length > 0) {
  console.warn("Slow requests detected:", slow);
}
```

### ✅ DO: Form Testing Pattern

```typescript
// Comprehensive form testing
async function testFormSubmission() {
  // 1. Navigate to form
  await navigate_page({ url: "http://localhost:3000/booking" });
  
  // 2. Fill form fields
  await fill({ 
    selector: "input[name='email']", 
    value: "test@example.com" 
  });
  
  await fill({ 
    selector: "input[name='name']", 
    value: "Test User" 
  });
  
  // 3. Or use fill_form for multiple fields
  await fill_form({
    fields: [
      { selector: "input[type='email']", value: "test@example.com" },
      { selector: "input[type='text']", value: "Test User" }
    ]
  });
  
  // 4. Submit form
  await click({ selector: "button[type='submit']" });
  
  // 5. Wait for response
  await wait_for({ condition: "text", value: "Success" });
  
  // 6. Verify result
  await take_screenshot();
  const consoleLogs = await list_console_messages();
  const network = await list_network_requests();
}
```

### ✅ DO: Local Development Testing

```typescript
// Test localhost application
async function testLocalApp() {
  // 1. Verify server is running
  // (Check via terminal or curl)
  
  // 2. Navigate to localhost
  await navigate_page({ url: "http://localhost:3000" });
  
  // 3. Check for errors
  const consoleMessages = await list_console_messages();
  const errors = consoleMessages.filter(msg => msg.level === 'error');
  
  if (errors.length > 0) {
    await take_screenshot();
    return { success: false, errors };
  }
  
  // 4. Test functionality
  await click({ selector: "button.start-project" });
  await wait_for({ condition: "selector", value: ".booking-wizard" });
  
  // 5. Verify state
  await take_screenshot();
  return { success: true };
}
```

### ✅ DO: Performance Analysis

```typescript
// Comprehensive performance analysis
async function analyzePerformance(url: string) {
  // 1. Navigate
  await navigate_page({ url });
  
  // 2. Start trace
  await performance_start_trace();
  
  // 3. Interact (if needed)
  await click({ selector: "button.load-content" });
  await wait_for({ condition: "time", value: 3000 }); // Wait 3 seconds
  
  // 4. Stop and analyze
  await performance_stop_trace();
  const insights = await performance_analyze_insight();
  
  // 5. Review insights:
  // - Long tasks (> 50ms)
  // - Layout shifts (CLS)
  // - Network bottlenecks
  // - JavaScript execution time
  // - Rendering performance
  
  return insights;
}
```

## Connecting to Running Chrome Instance

### Use Case: Connect to Existing Chrome

If you want to use your existing Chrome profile or connect to a Chrome instance started manually:

**Step 1: Start Chrome with Remote Debugging**

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome-profile-stable

# Linux
/usr/bin/google-chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome-profile-stable

# Windows
"C:\Program Files\Google\Chrome\Application\chrome.exe" \
  --remote-debugging-port=9222 \
  --user-data-dir="%TEMP%\chrome-profile-stable"
```

> **⚠️ WARNING:** Enabling remote debugging opens a debugging port. Any application on your machine can connect. Don't browse sensitive websites while debugging port is open.

**Step 2: Configure MCP to Connect**

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "chrome-devtools-mcp@latest",
        "--browser-url=http://127.0.0.1:9222"
      ]
    }
  }
}
```

**Step 3: Test Connection**

```
Check the performance of https://developers.chrome.com
```

## Security Considerations

### ⚠️ Important Security Notes

1. **Remote Debugging Port**
   - Opening `--remote-debugging-port` allows any app on your machine to control Chrome
   - Don't browse sensitive websites while debugging port is open
   - Use isolated user data directory when enabling remote debugging

2. **Data Exposure**
   - Chrome DevTools MCP exposes browser content to MCP clients
   - Don't share sensitive or personal information you don't want exposed
   - Use `--isolated=true` for temporary, auto-cleaned sessions

3. **Sandboxing**
   - Some MCP clients sandbox MCP servers
   - If sandboxed, Chrome may not start (needs sandbox permissions)
   - Workaround: Use `--browser-url` to connect to Chrome started outside sandbox

## User Data Directory

Chrome DevTools MCP uses the following user data directories:

- **Linux/macOS**: `$HOME/.cache/chrome-devtools-mcp/chrome-profile-$CHANNEL`
- **Windows**: `%HOMEPATH%/.cache/chrome-devtools-mcp/chrome-profile-$CHANNEL`

**Important:**
- User data directory is **not cleared** between runs by default
- Shared across all `chrome-devtools-mcp` instances
- Use `--isolated=true` for temporary, auto-cleaned sessions

## Troubleshooting

### Common Issues

1. **Chrome Won't Start in Sandboxed Environment**
   - **Solution**: Use `--browser-url` to connect to Chrome started manually
   - **Alternative**: Disable sandboxing for `chrome-devtools-mcp` in MCP client

2. **Port Already in Use**
   - **Solution**: Change port number or kill existing process
   ```bash
   lsof -i :9222  # Find process using port
   kill -9 <PID>  # Kill process
   ```

3. **VM-to-Host Port Forwarding Issues**
   - See troubleshooting guide: [docs/troubleshooting.md](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/troubleshooting.md#remote-debugging-between-virtual-machine-vm-and-host-fails)

4. **Performance Trace Fails**
   - Ensure page is fully loaded before starting trace
   - Wait for network idle before stopping trace
   - Check console for Chrome DevTools errors

## Quick Reference

### Basic Navigation
```typescript
await navigate_page({ url: "http://localhost:3000" });
await take_screenshot();
```

### Console Error Check
```typescript
const messages = await list_console_messages();
const errors = messages.filter(m => m.level === 'error');
```

### Network Monitoring
```typescript
const requests = await list_network_requests();
const failed = requests.filter(r => r.status >= 400);
```

### Performance Analysis
```typescript
await performance_start_trace();
// ... interact with page ...
await performance_stop_trace();
const insights = await performance_analyze_insight();
```

### Form Filling
```typescript
await fill({ selector: "input.email", value: "test@example.com" });
await click({ selector: "button.submit" });
```

### Wait for Conditions
```typescript
await wait_for({ condition: "selector", value: ".loaded" });
await wait_for({ condition: "text", value: "Success" });
await wait_for({ condition: "time", value: 2000 }); // 2 seconds
```

## Integration with Local Development

### Testing Localhost Applications

```typescript
// 1. Verify dev server is running
// (Check via terminal: curl http://localhost:3000)

// 2. Navigate and test
await navigate_page({ url: "http://localhost:3000" });

// 3. Check for errors
const consoleMessages = await list_console_messages();
const errors = consoleMessages.filter(msg => msg.level === 'error');

// 4. Test functionality
await click({ selector: "button.start-project" });
await wait_for({ condition: "selector", value: ".wizard" });

// 5. Verify state
await take_screenshot();
```

### Comparing Local vs Production

```typescript
// Test localhost
await navigate_page({ url: "http://localhost:3000" });
await take_screenshot();
const localConsole = await list_console_messages();

// Test production
await navigate_page({ url: "https://fashionos100.vercel.app" });
await take_screenshot();
const prodConsole = await list_console_messages();

// Compare:
// - Visual differences (screenshots)
// - Console errors
// - Network requests
// - Performance metrics
```

## When to Use Chrome DevTools MCP vs Cursor Browser

### Use Chrome DevTools MCP When:
- You need **performance analysis** (traces, insights)
- You need **advanced network debugging** (request details, timing)
- You need **JavaScript evaluation** in page context
- You need **device emulation** (mobile, tablet testing)
- You need **programmatic control** via MCP tools

### Use Cursor Browser When:
- You need **quick visual verification** (screenshots)
- You need **simple interaction testing** (click, type)
- You want **integrated browser window** in Cursor
- You need **session persistence** across Cursor sessions

## Resources

- **GitHub Repository**: [ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)
- **NPM Package**: [chrome-devtools-mcp](https://npmjs.org/package/chrome-devtools-mcp)
- **Tool Reference**: [docs/tool-reference.md](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/tool-reference.md)
- **Troubleshooting**: [docs/troubleshooting.md](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/troubleshooting.md)
- **Changelog**: [CHANGELOG.md](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/CHANGELOG.md)

## When Implementing Chrome DevTools MCP Tests

1. **Configure MCP server** - Add to `.cursor/mcp.json` or Cursor settings
2. **Choose configuration** - Headless, isolated, viewport size, etc.
3. **Navigate to page** - Use `navigate_page` with target URL
4. **Check console** - Always check for errors after navigation
5. **Monitor network** - Check for failed or slow requests
6. **Test interactions** - Click, fill forms, navigate workflows
7. **Verify state** - Take screenshots, check console, verify network
8. **Performance analysis** - Start/stop traces, analyze insights
9. **Document issues** - Note errors, visual discrepancies, performance problems
10. **Clean up** - Close pages, reset state if needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amo-tech-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
