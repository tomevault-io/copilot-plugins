## kens-chrome-extension

> This is a production-ready Chrome Extension boilerplate using Manifest V3, TypeScript, React, Vite, and Tailwind CSS.

# Chrome Extension Boilerplate - Development Guide

This is a production-ready Chrome Extension boilerplate using Manifest V3, TypeScript, React, Vite, and Tailwind CSS.

## Project Structure

```
├── src/
│   ├── background/index.ts    # Service worker - runs in background
│   ├── content/index.ts       # Content script - runs on web pages
│   ├── popup/                 # Popup UI (React) - toolbar icon click
│   ├── options/               # Options page (React) - extension settings
│   └── lib/
│       ├── storage.ts         # Type-safe Chrome storage
│       └── messaging.ts       # Type-safe message passing
├── public/icons/              # Extension icons (16, 32, 48, 128px)
├── manifest.json              # Chrome extension manifest (MV3)
└── vite.config.ts             # Build configuration
```

## Architecture Overview

### Communication Flow
```
┌─────────┐     messages      ┌────────────┐     messages      ┌─────────┐
│  Popup  │ ←───────────────→ │ Background │ ←───────────────→ │ Content │
│ (React) │                   │  (Worker)  │                   │ Script  │
└─────────┘                   └────────────┘                   └─────────┘
     ↓                              ↓                               ↓
 User clicks                  Handles all                    Runs on web
 toolbar icon                 Chrome APIs                    pages, can
                              and storage                    modify DOM
```

### Key Files to Modify

| Task | File(s) |
|------|---------|
| Add new setting | `src/lib/storage.ts` → StorageSchema |
| Add new message type | `src/lib/messaging.ts` → MessageTypes |
| Handle new message | `src/background/index.ts` → createMessageHandler |
| Modify popup UI | `src/popup/Popup.tsx` |
| Modify options page | `src/options/Options.tsx` |
| Add page manipulation | `src/content/index.ts` |
| Change permissions | `manifest.json` → permissions |

## Common Tasks

### Adding a New Storage Key

1. Edit `src/lib/storage.ts`:
```typescript
export interface StorageSchema {
  settings: { ... };
  userData: { ... };
  // Add your new key:
  myFeature: {
    enabled: boolean;
    data: string[];
  };
}
```

2. Use it anywhere:
```typescript
import { getStorage, setStorage } from "@/lib/storage";

const myData = await getStorage("myFeature");
await setStorage("myFeature", { enabled: true, data: ["item1"] });
```

### Adding a New Message Type

1. Edit `src/lib/messaging.ts`:
```typescript
export interface MessageTypes {
  // ... existing types ...

  FETCH_DATA: {
    request: { url: string };
    response: { data: unknown; success: boolean };
  };
}
```

2. Add handler in `src/background/index.ts`:
```typescript
createMessageHandler({
  // ... existing handlers ...

  FETCH_DATA: async (payload) => {
    const response = await fetch(payload.url);
    const data = await response.json();
    return { data, success: true };
  },
});
```

3. Call from popup/content:
```typescript
import { sendToBackground } from "@/lib/messaging";

const result = await sendToBackground("FETCH_DATA", { url: "https://api.example.com" });
if (result.success) {
  console.log(result.data);
}
```

### Adding a New Permission

Edit `manifest.json`:
```json
{
  "permissions": [
    "storage",
    "activeTab",
    "scripting",
    "tabs",          // Add for full tab access
    "notifications", // Add for notifications
    "alarms"         // Add for scheduled tasks
  ]
}
```

### Adding Keyboard Shortcuts

Edit `manifest.json`:
```json
{
  "commands": {
    "_execute_action": {
      "suggested_key": {
        "default": "Ctrl+Shift+Y",
        "mac": "Command+Shift+Y"
      },
      "description": "Open extension popup"
    },
    "toggle-feature": {
      "suggested_key": {
        "default": "Ctrl+Shift+U"
      },
      "description": "Toggle feature"
    }
  }
}
```

Handle in background:
```typescript
chrome.commands.onCommand.addListener((command) => {
  if (command === "toggle-feature") {
    // Handle command
  }
});
```

### Adding Context Menu Items

In `src/background/index.ts`:
```typescript
chrome.runtime.onInstalled.addListener(() => {
  chrome.contextMenus.create({
    id: "my-action",
    title: "Do Something",
    contexts: ["selection", "page"],
  });
});

chrome.contextMenus.onClicked.addListener((info, tab) => {
  if (info.menuItemId === "my-action") {
    // Handle click
    // info.selectionText contains selected text
  }
});
```

### Injecting UI into Web Pages

In `src/content/index.ts`:
```typescript
function injectUI() {
  // Create shadow root to isolate styles
  const host = document.createElement("div");
  host.id = "my-extension-root";
  const shadow = host.attachShadow({ mode: "closed" });

  // Add your UI
  shadow.innerHTML = `
    <style>
      .container { /* styles isolated from page */ }
    </style>
    <div class="container">
      <button id="my-btn">Click me</button>
    </div>
  `;

  document.body.appendChild(host);

  // Add event listeners
  shadow.getElementById("my-btn")?.addEventListener("click", () => {
    // Handle click
  });
}
```

### Making API Calls

Create `src/lib/api.ts`:
```typescript
const API_BASE = "https://api.example.com";

export async function fetchData<T>(endpoint: string): Promise<T> {
  const response = await fetch(`${API_BASE}${endpoint}`);
  if (!response.ok) {
    throw new Error(`API error: ${response.status}`);
  }
  return response.json();
}
```

Note: API calls should generally be made from the background script to avoid CORS issues.

### Adding Notifications

```typescript
// Requires "notifications" permission
chrome.notifications.create({
  type: "basic",
  iconUrl: chrome.runtime.getURL("icons/icon-128.png"),
  title: "Notification Title",
  message: "Notification message",
});
```

### Adding Alarms (Scheduled Tasks)

```typescript
// Requires "alarms" permission

// Create alarm
chrome.alarms.create("my-alarm", {
  delayInMinutes: 1,
  periodInMinutes: 60, // Repeat every hour
});

// Handle alarm
chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === "my-alarm") {
    // Do scheduled task
  }
});
```

## Build Commands

```bash
npm run dev      # Development with HMR
npm run build    # Production build
npm run lint     # Run ESLint
npm run typecheck # TypeScript check
```

## Loading the Extension

1. Run `npm run dev` or `npm run build`
2. Open `chrome://extensions/`
3. Enable "Developer mode"
4. Click "Load unpacked"
5. Select the `dist` folder

## Common Patterns

### Check if Extension is Enabled
```typescript
const settings = await getStorage("settings");
if (!settings?.enabled) return;
```

### Send Message to All Tabs
```typescript
const tabs = await chrome.tabs.query({});
for (const tab of tabs) {
  if (tab.id) {
    try {
      await chrome.tabs.sendMessage(tab.id, { type: "UPDATE" });
    } catch {
      // Tab might not have content script
    }
  }
}
```

### Get Current Tab
```typescript
const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
```

### Execute Script in Tab
```typescript
// Requires "scripting" permission
await chrome.scripting.executeScript({
  target: { tabId: tab.id },
  func: () => {
    // This runs in the page context
    document.body.style.backgroundColor = "red";
  },
});
```

## Debugging

- **Background script**: Go to `chrome://extensions/`, click "Service Worker" link
- **Popup**: Right-click extension icon → Inspect popup
- **Content script**: Regular page DevTools, check console for `[Content Script]` logs
- **Storage**: DevTools → Application → Local Storage → chrome-extension://...

## File Size Limits

- Total extension: 10MB recommended
- Individual files: No hard limit, but keep reasonable
- Icons: Keep small, PNG format

---
> Source: [KenKaiii/kens-chrome-extension](https://github.com/KenKaiii/kens-chrome-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
