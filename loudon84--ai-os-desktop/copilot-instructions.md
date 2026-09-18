## 003-ipc-contract

> IPC channel definition order and hermesAPI event cleanup requirements


# IPC Contract Rules

Before writing UI that needs backend data:

1. Define TypeScript interface in `src/shared/`
2. Add `ipcMain.handle` in `src/main/index.ts` (or module registered from index)
3. Add `hermesAPI` wrapper in `src/preload/index.ts`
4. Add declaration in `src/preload/index.d.ts`
5. Use the wrapper in Renderer
6. Update `docs/API_CONTRACTS.md` when adding or changing channels

## Never

- Call `ipcRenderer` directly from Renderer.

## Event listeners

- All event listeners exposed by `hermesAPI` (and sibling preload APIs) must return unsubscribe functions.
- Renderer components must cleanup listeners on unmount.

```typescript
// ✅ GOOD
useEffect(() => {
  const unsubscribe = window.hermesAPI.onInstallProgress(handler);
  return unsubscribe;
}, []);
```

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
