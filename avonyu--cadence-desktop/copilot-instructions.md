## cadence-desktop

> `./TODO.md` 文档为当前的任务清单，不要修改，我会随时更新

# 项目指导

## 文件指定

`./TODO.md` 文档为当前的任务清单，不要修改，我会随时更新

## 技术栈

相应的技术文档使用 Context 7 MCP 来获取最新的内容

- Vite
- Bun
- Tauri v2
- React
- React Router
- Video.js
- Zustand: global status manager
- Tailwind CSS V4+: CSS framework
- Shadcn: UI Component
- Lucide: Icon Library
- react-i18next

## Git 分支命名规范

主分支：main
开发分支：dev
功能分支：feat/<feature-name>
修复分支：fix/<fix-name>
发布分支：release/<release-name>
热修复分支：hotfix/<hotfix-name>
支持分支：support/<support-name>

## Tauri 注意事项

### CSP 限制

- **dev 模式**：Vite dev server 不应用 Tauri CSP 策略，前端可以直接发起外部 HTTP 请求
- **build 模式**：打包后的应用会严格执行 `tauri.conf.json` 中的 CSP 配置，阻止前端直接访问外部 API
- **解决方案**：通过 Rust 后端代理外部 API 请求（使用 `tauri-plugin-http`），而不是在前端直接调用

### Shadcn/Radix ScrollArea 与 CSP

- Radix `ScrollArea` 会在运行时插入内联 `<style>` 来隐藏原生滚动条，例如 `[data-radix-scroll-area-viewport]{scrollbar-width:none;...}`
- Tauri dev 模式不会应用生产 CSP，因此该内联样式通常能生效；build 模式会应用 `tauri.conf.json` 中的 CSP，若未允许 inline style，Radix 注入的隐藏规则可能被阻止，导致原生滚动条出现
- 不要为了这个问题放宽 CSP 使用 `style-src 'unsafe-inline'`
- 项目内应将隐藏规则写入打包 CSS，例如在 `src/App.css` 中维护 `[data-radix-scroll-area-viewport]` 和 `::-webkit-scrollbar` 规则
- Shadcn `ScrollArea.Root` 应保持 `relative overflow-hidden`，确保 viewport 和自定义 scrollbar 按预期被裁剪

### 开发与生产样式一致性

- 第三方组件的基础样式必须显式静态导入，使 Vite 将其写入生产 CSS；不要依赖组件在运行时插入内联 `<style>`
- Sonner 必须导入 `sonner/dist/styles.css`，否则生产 CSP 会阻止其运行时样式，导致 Toast 进入普通文档流
- 发现第三方组件依赖运行时注入样式时，应优先导入其静态 CSS，或将必要规则维护在 `src/App.css`，不要通过 `style-src 'unsafe-inline'` 放宽 CSP
- Tailwind 类名必须以完整字符串出现在源码中；不要使用 ``bg-${color}-500`` 等动态拼接，应通过映射表返回完整类名，避免生产构建遗漏样式
- `bun run preview` 只能检查 Vite 生产资源、CSS 打包和 Tailwind 类名扫描，不能完整复现 Tauri 协议与 CSP
- 涉及布局、弹窗、Toast、滚动区域或第三方 UI 组件的修改，完成前应按以下层级验证：
  1. `bun tauri dev`：快速交互和 HMR 验证
  2. `bun run build` 和 `bun run preview`：验证 Vite 生产构建
  3. `bun tauri build --debug --no-bundle`：使用真实 Tauri WebView、`frontendDist` 和生产 CSP 验证
- 在 Tauri debug build 中检查 WebView 控制台，不得忽略 `Refused to apply inline style`、`Refused to load` 或 CSP directive violation 等错误
- 发布前至少执行 `bun test`、`bun run build` 和 `bun tauri build --debug --no-bundle`；如果存在与当前修改无关的测试失败，需要明确记录

### 开发与生产内存/性能差异

- **`bun tauri dev` 的内存/性能指标不能代表生产表现**，不要据此判断是否存在内存泄漏或性能问题
- dev 与 release 加载前端的方式不同：
  - dev：Vite dev server（`http://localhost:1420`），React 处于 development 模式，开启 HMR / Fast Refresh，代码未压缩并带 source map，WebView 启用 debug 特性
  - release：内嵌静态资源（`frontendDist`），React production 模式，无 HMR/source map，无 debug WebView
- dev 专属层（React dev 记账、HMR/Fast Refresh 运行时保留引用并关闭优化、source map 等）会随运行时间持续累积内存，**这些在打包后全部被剥离**
- 已知现象：视频播放时 `bun tauri dev` 下 WebView2 渲染进程内存随播放进度线性增长（≈码率），而**打包安装版完全没有此问题**——根因是 dev 工具链层累积，而非解码媒体或应用代码（视频解码在两种模式下是同一个 WebView2 引擎，行为一致）
- 高频回调（如 `requestVideoFrameCallback` 每帧驱动的 `onTimeUpdate`/重渲染）会放大 dev 模式的内存累积，进一步误导判断
- 排查内存/性能问题时，必须用 `bun tauri build`（或 `bun tauri build --debug --no-bundle`）的真实 release WebView 复现后再下结论；不要在 dev 模式下为 dev-only 的假象设计修复方案

---
> Source: [avonyu/cadence-desktop](https://github.com/avonyu/cadence-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
