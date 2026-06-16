## a-stock-monitor

> **A股盯盘助手** — Tauri 2 桌面应用，双窗口（小窗 Widget + 大窗 Main），实时 A 股行情与规则化风险/机会分析。

# AGENTS.md

## 项目概况

**A股盯盘助手** — Tauri 2 桌面应用，双窗口（小窗 Widget + 大窗 Main），实时 A 股行情与规则化风险/机会分析。

## 技术栈

| 层 | 技术 |
|----|------|
| 桌面壳 | Tauri 2 (Rust) |
| 前端 | Vue 3 + TypeScript + Pinia + UnoCSS |
| 行情 | Rust `reqwest` → 东方财富 / 腾讯备用 |
| 存储 | `tauri-plugin-store` |

## Cursor Cloud specific instructions

### 必须运行的服务

| 服务 | 命令 | 说明 |
|------|------|------|
| 开发模式 | `npm run tauri:dev` | Vite :1420 + 双 Tauri 窗口 |
| 仅前端 | `npm run dev` | 无 Tauri IPC，行情命令不可用 |

### 常用命令

```bash
npm install          # 安装 Node 依赖
npm run lint         # vue-tsc 类型检查
npm test             # Vitest（风险/机会规则）
npm run build        # 前端生产构建
npm run tauri:build  # 完整桌面打包（需平台依赖）
cd src-tauri && cargo test  # Rust 单元测试
```

### Linux Cloud VM 注意事项

- 需要 **Rust ≥ 1.85**：`rustup default stable`
- Tauri Linux 构建依赖：`libwebkit2gtk-4.1-dev` `libgtk-3-dev` `libayatana-appindicator3-dev` `librsvg2-dev`
- `reqwest` 使用 **rustls**，无需 OpenSSL 开发包
- macOS / Windows 打包请在对应 OS 上执行 `npm run tauri:build`

### 双窗口标签

- `widget` — 小窗，置顶
- `main` — 大窗，配置与详情

前端通过 `getCurrentWindow().label` 切换 `WidgetView` / `MainView`。

### 架构要点

- 行情拉取在 **Rust**（`src-tauri/src/quote.rs`），避免 WebView CORS
- 自选股变更通过 `watchlist-changed` 事件双窗同步
- AI：`src/services/analysis.ts` 预留接口，MVP 返回占位文案

### Git

- 功能分支命名：`cursor/<name>-654d`
- 远程：`github.com/MasterWWW/a-stock-monitor`

---
> Source: [MasterWWW/a-stock-monitor](https://github.com/MasterWWW/a-stock-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
