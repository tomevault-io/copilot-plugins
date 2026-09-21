## hermes-plugin-better-workspace

> Better Workspace is a web-native file manager and workspace plugin for Hermes Desktop (web port). It provides a full-featured file tree for the sidebar and full view (`/workspace`), code editing, drag-and-drop file uploads, directory zip downloads, and agent working directory synchronization.

# Better Workspace Plugin

## Overview
Better Workspace is a web-native file manager and workspace plugin for Hermes Desktop (web port). It provides a full-featured file tree for the sidebar and full view (`/workspace`), code editing, drag-and-drop file uploads, directory zip downloads, and agent working directory synchronization.

## Source Files
- `plugin.tsx` - Plugin entry point and area contributions (`SIDEBAR_NAV_AREA`, `panes`, `ROUTES_AREA`, `PALETTE_AREA`)
- `view.tsx` - Sidebar pane component (`BetterWorkspaceSidebar`) and full-page dual-pane view (`BetterWorkspacePage`)
- `tree.tsx` - Recursive directory tree with inline rename, drag-and-drop file upload, search filter, and parent navigation
- `toolbar.tsx` - Header bar with Back (`arrow-left`), Up (`arrow-up`), path editor, search toggle, and refresh
- `editor.tsx` - Embedded CodeMirror code editor with dirty indicator, save (`⌘S`/`Ctrl+S`), download, and copy path
- `context-menu.tsx` - Web-native context menu (Set working directory, download folder as zip, file actions)
- `modals.tsx` - Dialogs for file/folder creation, rename, permanent delete, and multi-file upload
- `store.ts` - Nanostores state management and session CWD / agent workspace synchronization
- `api.ts` - Client API for `/web-fs` endpoints (CRUD, list, upload, download, stat)
- `types.ts` - TypeScript interfaces and types

## Integration
In the Hermes Desktop Web Port, this plugin lives under `web/src/plugins/better-workspace/` and is discovered automatically by Vite glob discovery (`discoverBundledPlugins`).

---
> Source: [zedboii77/hermes-plugin-better-workspace](https://github.com/zedboii77/hermes-plugin-better-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
