## todo-web

> 这是一个专为计算机大三学生设计的极简 Web 待办与想法记录工具。其核心目标是提供如原生 App 般丝滑的交互体验，同时兼顾 PC 端的高效和移动端的便捷。

# Minimus - Simple & Beautiful Web Todo

## Project Overview
这是一个专为计算机大三学生设计的极简 Web 待办与想法记录工具。其核心目标是提供如原生 App 般丝滑的交互体验，同时兼顾 PC 端的高效和移动端的便捷。

### Main Technologies
- **Frontend:** React (TypeScript) + Vite
- **Styling:** Vanilla CSS (Apple-style, Glassmorphism)
- **Icons:** Lucide React
- **Architecture:** 响应式单页应用 (SPA)，预留 PWA 支持。

## Building and Running
由于网络环境限制，目前仅搭建了骨架，请按照以下步骤启动：

1. **安装依赖:**
   ```bash
   npm install
   ```
2. **本地开发:**
   ```bash
   npm run dev
   ```
3. **生产构建:**
   ```bash
   npm run build
   ```

## Development Conventions
- **UI/UX:** 严格遵循苹果设计规范（SF Pro 字体感、8px 倍数间距、柔和圆角）。
- **Component Style:** 优先使用 CSS 变量控制主题，确保深色模式易于扩展。
- **Code Quality:** 使用 TypeScript 接口定义数据模型，保持逻辑与样式的清晰分离。

---

## Design Blueprint (Original)
### 1. 视觉与交互
- **风格:** 苹果风/毛玻璃，大留白。
- **布局:** PC 端侧边栏，移动端底部 Tab Bar。
- **动效:** 缩放、位移等微交互。

### 2. 功能规划
- **待办 (Tasks):** 快速录入、分类管理、状态切换。
- **想法 (Notes):** 流式录入、标签聚合、一键转待办。
- **跨端:** PWA 离线支持，云端同步（待实现）。

### 3. 后续扩展计划 (TODO)
- [ ] 接入 SQLite/PostgreSQL 后端同步。
- [ ] 实现 PWA 配置文件 (manifest.json, service worker)。
- [ ] 添加热力图统计组件。
- [ ] AI 助手录入接口集成。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/buchuoer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
