## streamdeck-ip-display

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Stream Deck plugin that displays IP addresses on Stream Deck buttons with configurable auto-refresh functionality. It offers four button types: Dual IP Display (both local and public), Local IP Only, Public IP Only, and IP Toggle Display (cycles between modes). Features Canvas-based rendering for pixel-perfect display, WiFi SSID display, and user-configurable refresh intervals via property inspector UI.

## Development Commands

```bash
# Build the plugin
npm run build

# Build and watch for changes (auto-restarts Stream Deck plugin)
npm run watch

# Manual Stream Deck operations (requires Elgato CLI)
streamdeck restart io.piercefamily.ip-display
streamdeck link
streamdeck dev
```

## Architecture

### Plugin Structure
- **`src/plugin.ts`** - Main plugin entry point that registers all actions and connects to Stream Deck
- **`src/actions/local-ip-display.ts`** - Dual IP display action (shows both local and public IP)
- **`src/actions/local-ip-only-display.ts`** - Local IP only display action
- **`src/actions/public-ip-only-display.ts`** - Public IP only display action
- **`src/actions/ip-toggle-display.ts`** - Toggle IP display action (cycles between modes)
- **`io.piercefamily.ip-display.sdPlugin/`** - Stream Deck plugin directory with manifest, assets, and built code
- **`io.piercefamily.ip-display.sdPlugin/ui/`** - Property inspector HTML files for settings configuration

### Key Patterns

**Action Registration**: Uses `@action()` decorator with UUID matching manifest.json:
```typescript
@action({ UUID: "io.piercefamily.ip-display.dual-ip" })
export class IPDisplay extends SingletonAction<IPSettings>
```

**Canvas Rendering**: All visual output uses Canvas instead of text to avoid truncation issues:
- 144x144 pixel resolution for Stream Deck buttons
- Text shadows for readability on transparent backgrounds
- Base64 data URI conversion for Stream Deck compatibility

**IP Address Handling**:
- Local IP: Uses `os.networkInterfaces()` to find non-internal IPv4 addresses
  - Supports user-selected network interface via dropdown (auto-detect default)
  - Settings persist with `networkInterface?: string` field
- Public IP: Fetches from ipify.org API with 5-minute caching to avoid rate limits
- Status indicator: Color-coded dot (green/orange/red) based on connection status

**WiFi SSID Detection**:
- Uses native OS commands via `child_process.exec()` with `promisify` wrapper
- Platform-specific command detection:
  - Windows: `netsh wlan show interfaces` (regex: `/^\s*SSID\s*:\s*(.+)$/m`)
  - macOS: `system_profiler SPAirPortDataType` (regex: `/Current Network Information:[\s\S]*?\n\s+([^:]+):/`)
  - Linux/other: Returns `null` (not supported)
- 5-minute caching (`SSID_CACHE_DURATION = 5 * 60 * 1000`) with timestamp tracking
- Timeout: 5 seconds per command execution
- Fails silently if command fails (Ethernet, disconnected, or unsupported platform)
- Cache structure: `{ ssid: string | null; timestamp: number }`
- Only displayed for local IP (not public IP) when `showWifiSSID !== false` (default true)
- Canvas rendering: Gray text (#999999) below LOCAL IP label
  - Font: 10-11px Arial (varies by mode)
  - Truncation: 15-20 chars depending on layout with "..." suffix
  - Position: 12-15px below label text

**Clipboard Copy Pattern**:
- Long-press detection using 800ms threshold (`LONG_PRESS_THRESHOLD`)
- `onKeyDown`: Starts timer, stores press timestamp and IPs
- `onKeyUp`: Checks duration, executes copy if ≥ threshold, else refreshes
- Uses `clipboardy` library for cross-platform clipboard access
- Visual feedback via `showOk()` (success) or `showAlert()` (failure)
- Dual IP format: `<localIP>,<publicIP>` (comma-separated)
- Single IP format: Just the IP address without any formatting

**Custom Label Support**:
- Settings fields: `customLabel`, `customLocalLabel`, `customPublicLabel`
- Max 12 characters to fit button layout
- Falls back to defaults ("LOCAL IP", "PUBLIC IP") when blank
- Renders using Canvas with dynamic font sizing

**Multi-line Display Mode**:
- Toggle via `multilineIP?: boolean` setting
- Splits IP addresses into two lines: `192.168.` / `1.100`
- Larger fonts (20px vs 16px for IPs, 13px vs 14px for labels)
- Different vertical spacing calculations for single vs multi-line
- `splitIP()` utility function handles formatting

**Custom Color Customization**:
- Settings fields: `labelColor?: string`, `ipColor?: string`
- Color picker UI using `<sdpi-color>` SDPI Components
- Canvas rendering uses fallback pattern: `settings.labelColor || '#C0C0C0'`
- Defaults: Silver (#C0C0C0) for labels, White (#FFFFFF) for IP addresses
- All color pickers implemented in all four action types
- Optional fields maintain backward compatibility
- Status dots remain hardcoded (green/orange/red) for connection status

**Status Dot Behavior**:
- Positioned inline with labels (to the left) using dynamic text measurement
- Single IP displays: Green (#00FF00) if connected, Red (#FF6B6B) if disconnected
- Dual IP displays: Green (#00FF00) both connected, Orange (#FFAA00) partial, Red (#FF6B6B) none
- Calculation: `dotX = 72 - (labelMetrics.width / 2) - offset` (offset: 8-10px)
- Dual IP: Dot only appears before PUBLIC IP label (not local IP label)
- Positioned at same Y coordinate as label text for inline appearance
- Colors hardcoded and not affected by custom color settings

### Build Process

The build uses Rollup with specific Stream Deck requirements:
- **External dependencies**: Canvas is marked external and installed separately in plugin directory
- **Output**: Bundles to `io.piercefamily.ip-display.sdPlugin/bin/plugin.js`
- **Module type**: Emits ES modules with package.json type declaration
- **Watch mode**: Automatically restarts Stream Deck plugin on changes

### Stream Deck Integration

**Manifest Configuration** (`io.piercefamily.ip-display.sdPlugin/manifest.json`):
- Defines plugin metadata, actions, and Node.js requirements
- Action UUIDs must match decorator in TypeScript code
- Supports both macOS 12+ and Windows 10+

**Event Handling**:
- `onWillAppear`: Renders IP display when button appears and starts auto-refresh timer
- `onKeyDown`: Manual refresh with cache bypass for immediate update
- `onWillDisappear`: Cleans up timer when button is removed from view
- `onDidReceiveSettings`: Restarts timer when user changes refresh interval

**Auto-Refresh System**:
- Timer-based refresh with configurable intervals (default 10 minutes)
- Efficient single timer per action type, supports multiple instances
- Manual refresh bypasses cache, automatic refresh respects cache
- Settings-driven: 0 = Manual Only, >0 = auto-refresh interval in milliseconds

**Property Inspector Integration**:
- HTML-based settings UI using SDPI Components v4 library
- Dropdown selection for refresh intervals (Manual Only, 1min-1hr)
- Settings persistence via Stream Deck SDK
- Separate UI files for different action types

**SDPI Components Datasource Pattern** (Network Interface Dropdown):
```html
<sdpi-select setting="networkInterface" datasource="getNetworkInterfaces" loading="Loading interfaces...">
  <option value="">Auto-detect (default)</option>
</sdpi-select>
```

Plugin responds to datasource requests in `onSendToPlugin`:
```typescript
override onSendToPlugin(ev: SendToPluginEvent<any, Settings>): void {
  const payload = ev.payload as { event?: string };

  if (payload.event === 'getNetworkInterfaces') {
    const nets = networkInterfaces();
    const interfaces: string[] = [];

    for (const name of Object.keys(nets)) {
      const netInterface = nets[name];
      if (!netInterface) continue;

      const hasIPv4 = netInterface.some(net => {
        const familyV4 = typeof net.family === 'string' ? 'IPv4' : 4;
        return net.family === familyV4 && !net.internal;
      });

      if (hasIPv4) interfaces.push(name);
    }

    streamDeck.ui.current?.sendToPropertyInspector({
      event: 'getNetworkInterfaces',
      items: interfaces.map(name => ({
        label: name,
        value: name
      }))
    });
  }
}
```

Key points:
- SDPI Components automatically handles request/response cycle
- `datasource` attribute triggers request when property inspector loads
- Response must include `items` array with `label`/`value` objects
- No custom JavaScript needed in HTML files
- `streamDeck.ui.current?.sendToPropertyInspector()` sends data to active PI

### Timer Management Patterns

**Lifecycle Management**:
```typescript
private refreshTimer: NodeJS.Timeout | null = null;
private visibleActions = new Map<string, WillAppearEvent<Settings>>();

// Start timer on appearance
override async onWillAppear(ev: WillAppearEvent<Settings>) {
    this.visibleActions.set(ev.action.id, ev);
    this.startRefreshTimer(ev.payload.settings);
}

// Clean up on disappearance
override onWillDisappear(ev: WillDisappearEvent<Settings>) {
    this.visibleActions.delete(ev.action.id);
    if (this.visibleActions.size === 0 && this.refreshTimer) {
        clearInterval(this.refreshTimer);
        this.refreshTimer = null;
    }
}
```

**Settings Management**:
```typescript
type IPSettings = {
  refreshInterval?: number;        // Auto-refresh interval in ms (default: 600000)
  customLabel?: string;             // For single IP actions (max 12 chars)
  customLocalLabel?: string;        // For dual IP/toggle actions (max 12 chars)
  customPublicLabel?: string;       // For dual IP/toggle actions (max 12 chars)
  multilineIP?: boolean;            // Split IP across two lines (default: false)
  networkInterface?: string;        // Specific interface or empty for auto-detect
  labelColor?: string;              // Custom label color (default: #C0C0C0 silver)
  ipColor?: string;                 // Custom IP address color (default: #FFFFFF white)
  showWifiSSID?: boolean;           // Show WiFi network name (default: true)
};

type ToggleSettings = IPSettings & {
  mode: 'dual' | 'local' | 'public';  // Toggle action current display mode
};
```

- Default interval: 600000ms (10 minutes)
- Property inspector UI updates settings via Stream Deck SDK
- `onDidReceiveSettings` event restarts timer with new interval
- All settings persist across Stream Deck restarts automatically

### Dependencies

**Runtime**:
- `@elgato/streamdeck`: Official Stream Deck SDK v1.0.0
- `canvas`: For pixel-perfect text rendering (installed in plugin directory)
- `clipboardy`: Cross-platform clipboard access for copy functionality

**Development**:
- `@elgato/cli`: Stream Deck development tools
- Rollup + TypeScript build pipeline
- Node.js 20 target configuration

## Important Notes

### Core Architecture
- Canvas dependency requires separate installation in plugin directory due to native bindings
- IP address caching prevents API rate limiting (5-minute intervals for public IP)
- Auto-refresh timers are efficiently managed with single timer per action type
- Manual button press bypasses cache for immediate refresh, auto-refresh respects cache
- Property inspector settings persist across Stream Deck restarts automatically
- Text shadows are essential for readability on transparent Stream Deck buttons
- Plugin auto-restarts during development when using `npm run watch`
- Timer cleanup is critical in `onWillDisappear` to prevent memory leaks

### Long-Press Pattern Best Practices
- Use `pressTimers` Map to track press state per action instance
- Store timestamp, timer, and any data needed for operation (IPs, mode, etc.)
- Always `clearTimeout()` in `onKeyUp` to prevent memory leaks
- Clean up press timers in `onWillDisappear` for removed actions
- 800ms threshold provides good balance between accidental/intentional presses

### SDPI Components Datasource Pattern
- Always use `datasource` attribute instead of custom JavaScript for dynamic dropdowns
- Response payload must include `event` and `items` array with `label`/`value` objects
- Use `streamDeck.ui.current?.sendToPropertyInspector()` to respond
- Optional chaining (`?`) prevents errors when no PI is open
- SDPI handles all request/response timing automatically

### Multi-line Display Considerations
- `splitIP()` splits on dots: `['192', '168', '1', '100']` → `192.168.` / `1.100`
- Font sizes must be tested to prevent edge clipping at 144x144 resolution
- Different vertical spacing for single-line vs multi-line layouts
- Status dot positioning uses dynamic text measurement for centering

### Network Interface Selection
- Filter interfaces to only show those with active IPv4 addresses
- Handle both string and numeric family types: `'IPv4'` or `4`
- Empty string setting value means auto-detect (default behavior)
- Fall through to auto-detect if selected interface becomes unavailable
- Dropdown populates on PI load, updates reflect on next refresh

### Custom Labels
- 12 character limit ensures text fits within 144px button width
- Blank/empty values default to "LOCAL IP" / "PUBLIC IP"
- Labels are optional - all actions work without customization
- Canvas text measurement can validate label width before rendering

### WiFi SSID Feature
- Native OS commands preferred over npm packages (most are unmaintained)
- macOS Sequoia 15+: Traditional `networksetup -getairportnetwork` returns `<redacted>`
- Use `system_profiler SPAirPortDataType` for modern macOS compatibility
- 5-minute cache prevents excessive command executions
- `getWifiSSID()` must be `async` and all calls must use `await`
- SSID only shown when `mode === 'dual' || mode === 'local'` (toggle action)
- Canvas layout positions SSID below label (not inline) for better screen real estate
- Fail silently with `streamDeck.logger.debug()` - never show errors to user
- Gray color (#999999) distinguishes SSID from primary content (label/IP)
- Make `generateImage()` methods async when adding SSID support

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brandonpierce) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
