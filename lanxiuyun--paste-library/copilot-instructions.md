## paste-library

> **paste-library**: A modern clipboard manager built with Tauri v2 and Vue 3. Features real-time clipboard monitoring, history persistence, search, and a clean card-based UI.

# AGENTS.md — Coding Guidelines for Tauri + Vue + TypeScript

## Project Overview

**paste-library**: A modern clipboard manager built with Tauri v2 and Vue 3. Features real-time clipboard monitoring, history persistence, search, and a clean card-based UI.

- **Frontend**: Vue 3 (Composition API with `<script setup>`), TypeScript, Vite
- **Backend**: Tauri v2 with Rust
- **Database**: SQLite (via rusqlite)
- **Plugins**:
  - tauri-plugin-clipboard-x (clipboard monitoring)
  - tauri-plugin-global-shortcut (global hotkey Alt+V)
  - tauri-plugin-tray (system tray integration)
  - tauri-plugin-autostart (auto start on boot)
- **Status**: In development — P0/P1 features complete (~98%), P3 optimization complete, tag system fully implemented

---

## Build, Lint, Test Commands

### Frontend (Vue/TypeScript)

```bash
# Development server (port 1422)
pnpm dev

# Type check (no emit)
pnpm run build  # Includes vue-tsc type check before bundling

# Production build
pnpm run build

# Preview production build
pnpm preview
```

### Tauri Desktop App

```bash
# Development (runs beforeDevCommand + Tauri dev)
pnpm tauri dev

# Production build (compiles Rust + frontend)
pnpm tauri build
```

### Running a Single Test

**Currently**: No test framework configured. If adding tests:
- Add `vitest` as devDep: `pnpm add -D vitest @vitest/ui`
- Create tests in `src/**/*.test.ts` or `src/**/*.test.vue`
- Run: `pnpm exec vitest run` or `pnpm exec vitest` (watch mode)

---

## Code Style Guidelines

### Imports & Modules

- **ES modules only** (`import`/`export`, not `require`)
- Order imports: Vue API → Tauri API → Local modules → Styles
  ```typescript
  import { ref, computed } from "vue";
  import { invoke } from "@tauri-apps/api/core";
  import { MyComponent } from "@/components/MyComponent";
  import "@/styles/global.css";
  ```
- Use explicit file extensions (`.ts`, `.vue`) in imports
- Organize local imports alphabetically within groups

### TypeScript & Types

- **Strict mode enabled** (`"strict": true` in tsconfig.json)
  - No `any` types without explicit justification
  - No `@ts-ignore`, `@ts-expect-error`, or `as any`
  - Unused variables/parameters forbidden
- Vue component types: Always use explicit generics for refs/computed
  ```typescript
  const count = ref<number>(0);
  const doubled = computed<number>(() => count.value * 2);
  ```
- Interface/type naming: PascalCase (e.g., `UserData`, `ApiResponse`)
- Avoid `interface` vs `type` debates — use `type` by default for consistency

### Naming Conventions

- **Variables/functions**: camelCase (e.g., `getUserData`, `isLoading`)
- **Components**: PascalCase in script, kebab-case in templates
  ```typescript
  // ✓ Correct
  import MyButton from "@/components/MyButton.vue";
  <MyButton />
  
  // ✗ Wrong
  import myButton from "@/components/myButton.vue";
  <my-button />
  ```
- **Constants**: UPPER_SNAKE_CASE for truly immutable values
  ```typescript
  const API_TIMEOUT = 5000;
  const MAX_RETRIES = 3;
  ```

### Vue Component Structure

- Use `<script setup>` (modern, preferred over `setup()` function)
- Keep components focused — max ~200 lines per file
- Props always typed and validated
  ```typescript
  interface Props {
    title: string;
    count?: number;
    disabled?: boolean;
  }
  withDefaults(defineProps<Props>(), { count: 0, disabled: false });
  ```
- Emit types explicitly
  ```typescript
  const emit = defineEmits<{ submit: [value: string] }>();
  ```

### Window UI Guidelines

**Settings Window** (Main window):
- Normal window with system title bar (`decorations: true`)
- Size: 600x700, resizable
- Left sidebar navigation + right content area

**Clipboard Window** (Popup window):
- Frameless window (`decorations: false`)
- Size: 800x600, resizable
- Uses `skip_taskbar(true)` to hide from taskbar
- Uses `always_on_top(true)` for floating behavior
- Auto-hide on blur in standard mode
- Pin mode only disables blur-triggered auto-hide; manual hide via hotkey, `Esc`, or explicit hide command still works

**Key points**:
- Use `data-tauri-drag-region` attribute for draggable areas
- Use `-webkit-app-region: drag` / `app-region: drag` in CSS
- Interactive elements (buttons, inputs) must have `app-region: no-drag`
- Top drag bar height: 36px with a simple line indicator

### Formatting & Spacing

- Use Vue/TypeScript defaults (no explicit prettier config needed — follow existing patterns)
- 2-space indentation (default in templates)
- Single quotes for strings (following existing code)
- Semicolons: Optional but consistent (currently omitted in existing files)

### Error Handling

- **Always use try-catch** for async operations, especially Tauri invokes
  ```typescript
  async function greet() {
    try {
      greetMsg.value = await invoke("greet", { name: name.value });
    } catch (err) {
      console.error("Greet failed:", err);
      // Handle error gracefully
    }
  }
  ```
- Log errors meaningfully (include context)
- No silent failures — notify user or log clearly
- Tauri errors: Expect `string` payloads, validate before using

### State Management

- Use Vue reactivity (`ref`, `computed`, `watch`) for component-level state
- For shared state across components → Consider composables or Tauri backend
- Use `reactive()` for form data in settings panel
- Avoid global state until project needs it
- Reactive refs for form data, UI state; computed for derived values

### CSS & Styling

- Scoped styles preferred (`<style scoped>`)
- Global styles in `src/` root CSS file (currently in App.vue as fallback)
- CSS variables for theming (already used: light/dark mode)
- Avoid inline styles unless truly dynamic
- Media queries for responsive design (already present for dark mode)
- **Card-based design**: Rounded corners (8px), subtle shadows, clean typography
- **Settings panel**: Left sidebar (220px) + right content area with grouped settings

### Comments & Documentation

- Explain *why*, not *what* (code should be self-documenting)
- Use JSDoc for public functions:
  ```typescript
  /**
   * Invokes Tauri command to greet user
   * @param name - User's name
   * @returns Greeting message
   */
  async function greet(name: string): Promise<string> {
    return await invoke("greet", { name });
  }
  ```
- No trailing TODOs; open GitHub issues instead

---

## File Structure

```
src/
  ├── main.ts                    # App entry point
  ├── App.vue                    # Root component (Settings window)
  ├── ClipboardView.vue          # Clipboard window entry
  ├── assets/                    # Static images/SVGs
  ├── components/                # Reusable Vue components
  │   ├── clipboard/             # Clipboard-specific components
  │   ├── settings/             # Settings panel components
  │   │   ├── components/       # Reusable setting widgets
  │   │   └── sections/         # Setting categories
  │   ├── ClipboardItem.vue      # Card component for single clipboard item
  │   ├── ClipboardList.vue      # Main list with tabs and search
  │   ├── ContextMenu.vue        # Right-click context menu
  │   ├── DragHandle.vue         # Window drag capsule (for clipboard window)
  │   ├── SettingsPanel.vue      # Settings panel with left navigation
  │   ├── PasteQueuePanel.vue    # Paste queue panel for batch operations
  │   ├── DrawerEditor.vue       # Drawer-based editor for text/image preview
  │   ├── SmartSearch.vue        # Search input component
  │   └── TagManager.vue         # Tag management popup dialog
  ├── composables/               # Reusable logic (hooks)
  │   ├── useClipboard.ts        # Clipboard monitoring & CRUD (text/image/files)
  │   ├── useContentType.ts      # Content type detection utilities
  │   ├── useFileOperations.ts   # File open/reveal in folder operations
  │   ├── useImageLoader.ts      # Image loading with retry mechanism (5 retries)
  │   ├── usePasteQueue.ts       # Paste queue state management
  │   ├── usePinMode.ts          # Pin mode state (disables blur auto-hide, keeps paste open)
  │   ├── useSettings.ts         # Settings read/write via Tauri
  │   └── useSmartSearch.ts      # Smart search with @tag/@type syntax
  ├── utils/                      # Utility functions
  ├── types/                     # TypeScript type definitions
  │   ├── index.ts               # Shared types (ClipboardItem, AppSettings, etc.)
  │   ├── components.ts          # Component-specific types
  │   └── window.ts              # Window state types
  └── styles/                    # Global CSS (if needed)

src-tauri/
  ├── src/                       # Rust backend
  │   ├── lib.rs                 # Main entry + Tauri commands + global shortcut
  │   ├── clipboard.rs           # Clipboard manager logic
  │   ├── models.rs              # Data structures (ClipboardItem, AppSettings, etc.)
  │   ├── storage.rs             # SQLite database operations
  │   ├── fuzzy_search.rs        # Fuzzy search with pinyin support
  │   ├── window_manager.rs      # Window management (create/hide/show clipboard window)
  │   └── tray_manager.rs        # System tray integration
  ├── tauri.conf.json            # Tauri config
  ├── capabilities/              # Permission definitions
  └── Cargo.toml                 # Rust dependencies
```

---

## Tauri-Specific Guidelines

### Window Configuration

**Settings Window (Main)**:
```json
{
  "label": "main",
  "title": "Paste Library - 设置",
  "width": 600,
  "height": 700,
  "decorations": true,
  "center": true,
  "resizable": true
}
```

**Clipboard Window (Popup)**:
```rust
WebviewWindowBuilder::new(app, "clipboard", WebviewUrl::App("/clipboard".into()))
    .title("剪贴板历史")
    .inner_size(width, height)
    .decorations(false)
    .skip_taskbar(true)
    .always_on_top(true)
    .build()
```

### Required Permissions

```json
{
  "permissions": [
    "core:default",
    "clipboard-x:default",
    "core:window:allow-minimize",
    "core:window:allow-hide",
    "core:window:allow-show",
    "core:window:allow-is-visible",
    "core:window:allow-set-focus",
    "core:window:allow-create",
    "core:window:allow-is-focused",
    "global-shortcut:allow-is-registered",
    "global-shortcut:allow-register",
    "global-shortcut:allow-unregister",
    "global-shortcut:allow-unregister-all",
    "core:tray:default",
    "core:menu:default"
  ]
}
```

### Image Asset Protocol

For loading local images in Tauri v2, configure CSP and asset protocol:

```json
{
  "app": {
    "security": {
      "csp": "default-src 'self'; img-src 'self' asset: http://asset.localhost; media-src 'self' asset: http://asset.localhost",
      "assetProtocol": {
        "enable": true,
        "scope": ["$APPLOCALDATA/**", "$APPDATA/**", "**"]
      }
    }
  }
}
```

Then use `convertFileSrc` to load local images:

```typescript
import { convertFileSrc } from '@tauri-apps/api/core';
const imageSrc = computed(() => {
  if (!item.thumbnail_path) return '';
  return convertFileSrc(item.thumbnail_path);
});
```

### Commands

- Commands in `src-tauri/src/lib.rs` prefixed with `#[tauri::command]`
- Frontend calls via `invoke("command_name", { param: value })`
- Always handle Tauri errors in try-catch (they're `string` payloads)
- Use `tokio::sync::Mutex` instead of `std::sync::Mutex` for async commands

### Global Shortcut

- Registered in `lib.rs` setup with `tauri-plugin-global-shortcut`
- Default hotkey: `Alt+V`
- Toggles clipboard window visibility
- Window auto-hides on blur via `on_window_event` when not pinned
- Pin shortcut is managed separately and only changes blur auto-hide behavior

---

## Project Features

### Implemented ✅
- Real-time clipboard monitoring (text + HTML + image + files)
- SQLite persistence with automatic deduplication (SHA256 hash)
- Card-based UI with tabs (All/Text/Image/File/Favorite)
- **Search functionality** (fuzzy search with pinyin, initials, fault tolerance, `@tag` / `@type` mixed query)
- **Global hotkey** (configurable via key recording) to show/hide clipboard window
- **Settings panel** with left sidebar navigation
- **Window management** (frameless clipboard, normal settings)
- **Pin mode**:
  - Disables blur-triggered auto-hide
  - In `Pinned`, frontend copy/paste actions do not proactively hide or reset the clipboard panel
  - For continuous paste, backend `simulate_paste` temporarily hides the window, pastes to the previously focused app, then restores and refocuses the clipboard window
  - Does not disable manual hide actions like hotkey toggle or `Esc`
- **Image/File clipboard support**:
  - Image thumbnails with dimensions display
  - File/folder icons with names
  - Multi-file count display
- **Context menu** (right-click):
  - Copy/Paste/Copy as plain text
  - Open file / Show in folder
  - Favorite/Unfavorite
  - Delete
- **Interaction enhancements**:
  - Single click: copy to clipboard (configurable: copy/paste)
  - Double click: copy and paste
  - Right click: context menu
  - Hover detail button: open drawer editor
- **Cross-application drag and drop** (drag items to other apps)
- **Favorite system** with database persistence
- **System tray integration**:
  - Double-click tray icon to open settings
  - Right-click menu (Open settings, Show clipboard, Quit)
- **Data backup** (export/import JSON)
- **Storage path display** (data/log directories with copy/open)
- Copy/delete clipboard items
- Data persistence with comprehensive settings
- **Variable height Item design** (text 3 lines, adaptive image height)
  - **Tag system** replaces favorite (data model ✅, UI display ✅, tag manager popup ✅)
- **Hover quick action buttons** (detail/queue/copy/tag/delete)
- **Keyboard navigation** (↑/↓, Enter, 1-9, Esc)
- **Paste queue** (shopping cart mode, batch paste)
- **Drawer editor** (text editing, image preview)
- Local image loading via `convertFileSrc`
- **Application Lifecycle**:
  - First-run detection (show settings window on first launch only)
  - Window close-to-tray (hide instead of exit on close button)
  - Graceful autostart error handling (Windows registry errors)
- **Click Actions**:
  - **Copy**: Copy data to clipboard only
  - **Paste**: Copy data → hide window → simulate paste shortcut at previous focus
  - **None**: Only select the item, no copy/paste action
  - Configurable single/double click actions in settings (copy/paste/none)
  - **Paste Shortcut Mode**: User-selectable paste shortcut (Ctrl+V or Shift+Insert)
  - **Hide Window After Copy**: Auto-hide clipboard window after copy action
- **Interaction Experience Enhancements**:
  - **focus_search_on_activate**: Auto-focus search box on window activation
  - **Unified search state**: Search input is the single source of truth; fixed tabs and pinned tabs both toggle search conditions instead of maintaining a separate active tab state
  - **Smart Activate Optimization**: Distinguish system clipboard vs internal copy
  - **Search auto-scroll to top**: Auto scroll to top on search text change
  - **Right-click item highlight**: Show selected state on right-click context menu
  - **Keyboard navigation auto-scroll**: Auto scroll to keep selected item visible
  - **Keyboard selection restore on reopen**: Restore the last selected item and scroll position when the clipboard window is reopened, unless the panel state is explicitly reset
- **UI Polish**:
  - Hidden unfinished features (paste queue) for cleaner UI
  - Simplified quick action buttons (detail/delete only)
  - Image loading retry mechanism (5 retries with loading states)
  - Fixed hover/scroll issues in floating window

### In Progress ⏳
*当前无进行中的功能*

### Planned 📋
- Cross-device sync architecture
- Dark theme (currently light only)
- Advanced search filters (by date range)
- Multi-language support

---

## Settings Panel Structure

### Navigation Items
1. **剪贴板** - Window settings, sound effects, search settings, content settings
2. **历史记录** - Max history count, auto cleanup
3. **通用设置** - Auto start, blacklist apps
4. **快捷键** - Hotkey display, window size
5. **数据备份** - Export/import data, storage path display
6. **关于** - App info, open clipboard button

### Settings Categories

**窗口设置**:
- 窗口位置 (remember/center/cursor) ✅ 已实现
- 智能激活 (5秒内复制则清空搜索/回到顶部/切换全部/聚焦搜索) ✅ 已实现

**音效设置**:
- 复制音效 (+ preview button) ⏳ 待实现（需音效文件资源）

**搜索设置**:
- 搜索框位置 (top/bottom) ✅ 已实现
- 默认聚焦 ✅ 已实现（合并到智能激活）
- **focus_search_on_activate** ✅ 已实现（激活窗口时自动聚焦搜索框）

**内容设置**:
- 单击动作 (copy/paste/none) ✅ 已实现
- 双击动作 (copy/paste/none) ✅ 已实现
- 粘贴快捷键 (Ctrl+V / Shift+Insert) ✅ 已实现
- 复制后隐藏窗口 ✅ 已实现
- 图片OCR ⏳ 待实现（需OCR库）
- 复制为纯文本 ✅ 已实现
- 粘贴为纯文本 ⏳ 待实现
- 删除确认 ✅ 已实现
- 自动排序 ✅ 已实现（系统剪贴板复制置顶，内部复制保持位置）

**历史记录设置**:
- 最大历史记录数 (100-10000) ✅ 后端已实现
- 自动清理 (0/7/30/90 days) ✅ 后端已实现

**通用设置**:
- 开机自启 ✅ 已实现

**快捷键设置**:
- 唤醒快捷键 (按键录制, 如: Alt+V, Win+Shift+C) ✅ 已实现
- 钉住快捷键 (默认 `Ctrl+Shift+P`) ✅ 已实现

---

## Notes for Agents

### Build & CI

- **Versioning**: Date-based (YY.MM.DD) - update package.json + tauri.conf.json + Cargo.toml
- **Security**: `dangerousInsecureTransportProtocol: true` in tauri.conf.json — disable for production
- **Platforms**: Windows + macOS (ARM) only — Linux disabled in release workflow

### Code

- **No linter/prettier config**: Follow existing code patterns
- **No test framework yet**: Run type checks with `pnpm run build`
- **Type strictness**: `strict: true`, no `any` types
- **src/utils/**: Exists for utility functions (not documented in original structure)

### Commands

- **Never run tauri dev**: User will run by themselves
- **Never run cargo check**: User will run by themselves
- **Never read src-tauri/Cargo.toml**: Auto-generated by Tauri

- **No linter/prettier config**: Follow existing code patterns (Vue template formatting, semicolons-optional style)
- **Never run tauri dev**: I will run by my self
- **Never run cargo check**: I will run by my self
- **Never read src-tauri/Cargo.toml **: It's a auto generated file by tauri, you should not read it
- **No test framework yet**: Run type checks with `pnpm run build` (includes `vue-tsc`)
- **Type strictness is critical**: The project has `strict: true` and `noUnusedLocals`; zero tolerance for `any` types
- **Settings panel**: Normal window with title bar (decorations: true)
- **Clipboard window**: Frameless, skip taskbar, always on top (decorations: false)
- **Pin mode semantics**: only affects blur-triggered auto-hide; do not treat pin mode as a separate positioning or visibility mode
- **Clipboard action semantics**:
  - In non-`Pinned` mode, `paste` always hides/resets the clipboard window before invoking the paste shortcut
  - In non-`Pinned` mode, `copy` only hides/resets when `hide_window_after_copy` is enabled
  - In `Pinned` mode, frontend should not hide/reset after `copy` or `paste`
  - Frontend must execute `simulatePaste()` as the final step so focus can return to the target input before the native paste shortcut fires
  - Reopening the clipboard window after a normal hide/blur should restore the last keyboard-selected item and scroll position; only explicit panel reset flows should clear that memory
- **Search / Tab semantics**:
  - Do not reintroduce a separate `activeTab` source of truth for clipboard filtering
  - `searchQuery` is the only filter state; UI highlights must be derived from `parseSearchQuery(searchQuery)`
  - Fixed tabs (`全部/文本/图片/文件`) are type-filter toggles that add/remove the corresponding `@type`
  - Pinned tabs are saved search-condition toggles; clicking can merge or remove that saved query from the current search instead of blindly replacing everything
  - A highlighted tab should always mean "this condition is active in the current search"
- **Windows image paste caveat**:
 - `tauri-plugin-clipboard-x` uses `clipboard-rs` underneath; on Windows 11, writing PNG/image content can intermittently fail with `OSError(1418): 线程没有打开的剪贴板`
 - When restoring an image item to the system clipboard, prefer short retry/backoff instead of assuming the first `writeImage()` call will succeed
 - Treat this as a clipboard handle timing/competition issue, not as a bad image path by default
- **Greenfield project**: Modern best practices take priority over legacy patterns
- **Desktop-first UX**: Consider Windows/macOS/Linux platform differences in UI
- **Global shortcut**: Configurable via key recording in settings (restart required to apply changes)

### 📦 Release Version Bump

When releasing a new version, update ALL three files:

```bash
# 1. package.json
"version": "0.2.0"

# 2. src-tauri/tauri.conf.json
"version": "0.2.0"

# 3. src-tauri/Cargo.toml
version = "0.2.0"
```

Then commit and push to `release` branch to trigger the release workflow.

## Project Documentation

### 📁 文档说明

项目已移除独立的 docs/ 目录文档，功能规格和技术细节直接通过源码和本文件维护。

### 📖 如何开始新任务

**步骤1**: 阅读本文件 (AGENTS.md)
了解项目整体结构、已实现功能清单和代码规范。

**步骤2**: 查看相关源码
- **实现前端功能** → 查看 `src/components/` 和 `src/composables/`
- **实现后端功能** → 查看 `src-tauri/src/`
- **类型定义** → 查看 `src/types/index.ts`

**步骤3**: 参考相关目录的 AGENTS.md
- `src/components/AGENTS.md` - 组件开发指南
- `src/composables/AGENTS.md` - Composables 开发指南
- `src-tauri/src/AGENTS.md` - Rust 后端开发指南

### 🎯 协作原则

> **我负责定义问题和用户体验，但你是技术的完全所有者。你要挑战我，不要做老好人。**
>
> 这意味着：
> - 当我的需求有技术风险时，你有义务提出质疑
> - 当我忽视了更好的技术方案时，你应该主动建议
> - 当我的设计可能导致性能/安全/维护问题时，你必须明确指出
> - 技术决策权在你手中，用你的专业判断来推动最佳实现

### ⚠️ 重要约束

1. **从不使用 `any` 类型** - 项目启用 strict mode
2. **不提交代码** - 用户会自己运行和测试
3. **遵循现有代码风格** - 特别是 Vue Composition API 模式
4. **Rust 后端命令** - 所有新功能都需要对应的 Tauri 命令
5. **类型安全** - 更新类型定义后再实现功能

### 🔄 开发工作流程

```
1. 阅读相关文档了解需求
2. 更新 TypeScript 类型定义 (src/types/index.ts)
3. 更新 Rust 模型 (src-tauri/src/models.rs)
4. 实现 Rust 后端命令
5. 实现 Vue 前端组件
6. 运行类型检查: pnpm run build
7. 确认无 TypeScript 错误
```

---
> Source: [lanxiuyun/paste-library](https://github.com/lanxiuyun/paste-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
