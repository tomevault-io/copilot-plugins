## electron

> Electron main process and IPC security


# Electron Security Model

## IPC Architecture

```
Renderer ──Promise──> window.electronAPI ──IPC──> Main Process
Renderer <──Event─── window.electronAPI <──IPC─── Main Process
```

All IPC goes through `window.electronAPI` (defined in `src/electron.d.ts`).

## Security Invariants

- Node.js APIs: NEVER exposed to renderer
- Context isolation: always enabled
- File paths: sanitize against traversal (`../`)
- IPC inputs: validate in main process

## Adding New IPC

1. Define type in `src/electron.d.ts`
2. Implement handler in `electron/handlers/`
3. Expose via `preload.ts` contextBridge

## Test Mode

```bash
npm run dev -- -- --test-mode
# or
BLUEPLM_TEST=1 npm run dev
```

---
> Source: [bluerobotics/bluePLM](https://github.com/bluerobotics/bluePLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
