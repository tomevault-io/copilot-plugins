## ai-basic-browser-v1

> - 应用位于 `ai-browser/`。

# 仓库指南

## 项目结构与模块组织
- 应用位于 `ai-browser/`。
  - `src/main/` Electron 主进程（窗口、IPC、AI 编排）。
  - `src/preload/` 预加载安全桥，暴露为 `window.api`。
  - `src/renderer/` React 界面（`src/renderer/src/*.jsx`，样式在 `style.css`）。
  - 构建产物：`out/{main,preload,renderer}`。
- 根目录文档：`README.md`、`CLAUDE.md`、`技术设计.md`；日志在 `logs/`。

## 构建、测试与开发命令
- 环境（Node.js 18+）：
  - `cd ai-browser && npm install`
- 启动开发（Vite+Electron，热重载）：
  - `npm run dev`
- 构建（输出到 `out/`）：
  - `npm run build`
- 预览生产构建：
  - `npm start`
- 环境变量：在 `ai-browser/.env` 写入 `GOOGLE_GENERATIVE_AI_API_KEY=...`（勿提交）。

## 编码风格与命名约定
- 主/渲染进程使用 JavaScript ESM；预加载使用 CommonJS（受 Electron 约束）。
- 缩进 2 空格；单引号；保留分号；模块职责单一。
- React 组件用 PascalCase（如 `App.jsx`）；函数/变量用 camelCase；文件名用短横线小写或 `index.js(x)`。
- 保持 `contextIsolation: true`、`nodeIntegration: false`；仅通过 `window.api` 暴露最小 IPC。

## 测试指南
- 暂无正式测试框架，优先手动冒烟测试：
  - 启动开发，访问若干网站，确认导航/搜索与 AI 响应正常。
  - 验证 `window.api.sendQuery(...)` 能从 `<webview>` 取内容并返回结果。
  - 麦克风/语音权限被拒时应优雅降级。
- 若新增测试：单测与源码同目录或放在 `src/**/__tests__`；端到端建议 Playwright（Electron），渲染层建议 Vitest/React Testing Library。

## 提交与 Pull Request 指南
- 采用约定式提交：`feat:`、`fix:`、`docs:`、`chore:` …（与现有历史一致）。
- PR 需包含：
  - 变更目的与范围、关联 Issue（如有）、UI 变更的截图/GIF。
  - 复现步骤与风险说明；涉及环境变量或迁移请标注。
  - 变更应小而聚焦；行为变化需同步更新文档。

## 安全与配置提示
- 切勿提交 `.env` 或任何密钥；优先使用系统钥匙串/CI Secrets。
- 导航前校验/清洗 URL；生产环境避免设置 `webSecurity: false`。
- 收敛 IPC 暴露面；一律将网页内容视为不可信。

---
> Source: [MikeBets/ai-basic-browser-v1](https://github.com/MikeBets/ai-basic-browser-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
