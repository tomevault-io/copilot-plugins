## flutter-live-editor

> cat > .windsurf/rules/architecture_patterns.md << 'EOFMARKER'


cat > .windsurf/rules/architecture_patterns.md << 'EOFMARKER'
---
description: Architecture Patterns for Desktop & Mobile Apps
globs: **/*.{dart,ts,tsx}
alwaysApply: true
---

# 🏗️ ARCHITECTURE & COMMUNICATION PATTERNS

When building applications, follow these established patterns for robust, secure, and scalable architecture.

## 📱 FLUTTER (Mobile & Desktop)

**Pattern:** Repository + Riverpod/Bloc (Clean Architecture)

### 1. Data Layer (IPC/API Client)
- **Repository:** Acts as the single source of truth for data access.
  - *Desktop:* Uses `Process.run` or `FFI` to communicate with native backend if needed.
  - *Mobile:* Uses `http` or `dio` for API calls.
- **Error Handling:** MUST throw descriptive custom exceptions (e.g., `NetworkException`, `CacheException`) instead of returning `null` or `false`.

### 2. Domain Layer
- **Models:** Plain Dart objects (immutable, using `freezed` if possible).
- **Business Logic:** Pure Dart classes, independent of Flutter UI.

### 3. Presentation Layer (State Management)
- **Riverpod / Bloc:** Use for managing async state and caching (analogous to TanStack Query).
- **AsyncValue:** Handle `data`, `loading`, and `error` states explicitly in UI.
  - `loading`: Show skeleton or spinner.
  - `error`: Show user-friendly error message (Toast/Snackbar).
  - `data`: Render the UI.

**Example Flow:**
UI -> Provider (read) -> Repository (fetch) -> API/Native -> Result/Error -> Provider (update) -> UI (rebuild)

---

## 🖥️ ELECTRON / REACT (Desktop Web-based)

**Pattern:** IPC + TanStack Query (Client-Server Model)

### 1. IPC Layer (The Bridge)
- **ipc_client.ts (Renderer):** Singleton instance. Sends commands to Main process.
  - Usage: `await window.ipc.invoke('channel-name', payload)`
- **ipc_handlers.ts (Main):** Handles incoming commands.
  - **Registration:** `ipcMain.handle('channel-name', handler)`
  - **Error Handling:** MUST `throw new Error("Descriptive message")` on failure. NEVER return `{ success: false }`.

### 2. React Hooks (Data Fetching)
- **TanStack Query (useQuery/useMutation):**
  - **Query:** Fetching data.
    - `queryKey`: `['entity', id]`
    - `queryFn`: `() => ipcClient.getEntity(id)`
  - **Mutation:** Changing data.
    - `onSuccess`: `queryClient.invalidateQueries(['entity'])` (Auto-refetch)
    - `onError`: Show Toast notification.

**Example Flow:**
Component -> useQuery -> IpcClient -> IPC Invoke -> Main Handler -> DB/FS -> Result/Error -> useQuery (State Update) -> Component

---

## 🛡️ SECURITY & BEST PRACTICES (Universal)

1.  **Validation:** Validate all inputs at the boundary (before sending to API/Main process).
2.  **Error Propagation:** Errors must flow from the deepest layer up to the UI where they are handled gracefully.
3.  **Types:** strict typing everywhere (TypeScript / Dart). No `any`.
4.  **Locking:** Use mutex/locks for shared resources (files, db) to prevent race conditions.
EOFMARKER

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shark-777) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
