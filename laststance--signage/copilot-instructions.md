## signage

> alwaysApply: Electron Architecture & IPC

# Electron Architecture & IPC

This application follows Electron's multi-process architecture with secure IPC communication.

## 🏗️ Process Architecture

### Main Process
- **Entry Point**: [lib/main/main.ts](mdc:lib/main/main.ts)
- **Purpose**: Controls application lifecycle, creates renderer processes, handles native APIs
- **Responsibilities**: Window management, menu creation, file system access, OS integration

### Renderer Process
- **Entry Point**: [app/renderer.tsx](mdc:app/renderer.tsx)
- **Purpose**: Runs the React application in a Chromium web environment
- **Restrictions**: Limited access to Node.js APIs for security

### Preload Scripts
- **Entry Point**: [lib/preload/preload.ts](mdc:lib/preload/preload.ts)
- **Purpose**: Secure bridge between main and renderer processes
- **API Exposure**: [lib/preload/api.ts](mdc:lib/preload/api.ts)

## 🔐 IPC Communication

The application uses a secure IPC system defined in the preload scripts:

### Send Message (Renderer → Main)
```typescript
// In renderer process
window.api.send('channel-name', ...args)
```

### Receive Message (Main → Renderer)
```typescript
// In renderer process
window.api.receive('channel-name', (data) => {
  console.log(data)
})
```

### Invoke Method (Renderer ↔ Main)
```typescript
// In renderer process
const result = await window.api.invoke('channel-name', ...args)
```

## 🪟 Custom Window Implementation

The application features a custom window system located in [lib/window/](mdc:lib/window/):

### Key Features
- Custom titlebar with app branding
- Native macOS window controls
- Built-in menu system with keyboard shortcuts
- Dark/light mode toggle
- Menu visibility toggle (Option key)

### Menu Configuration
- **Menu Items**: Defined in [lib/window/titlebarMenus.ts](mdc:lib/window/titlebarMenus.ts)
- **Toggle Key**: macOS: `Option (⌥)` key

## 🔒 Security Considerations

- **Context Isolation**: Enabled by default
- **Node Integration**: Disabled in renderer for security
- **Preload Scripts**: Used for safe API exposure
- **CSP**: Content Security Policy configured for production builds

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laststance) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
