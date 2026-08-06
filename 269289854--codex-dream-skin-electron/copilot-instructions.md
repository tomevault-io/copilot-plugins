## codex-dream-skin-electron

> 本文件适用于仓库根目录及所有子目录。除非更深层目录另有 `AGENTS.md`，所有自动化代理都必须遵守这里的约定。

# AGENTS.md

本文件适用于仓库根目录及所有子目录。除非更深层目录另有 `AGENTS.md`，所有自动化代理都必须遵守这里的约定。

## 项目定位

Codex Dream Skin Studio 是仅支持 Windows x64 的 Codex 桌面主题编辑器，技术栈为 Electron、React、Vite 和严格模式 TypeScript。应用负责管理本地主题、编译预览与运行时载荷，并通过 PowerShell 和本机 CDP 端点为 Microsoft Store 版 Codex 应用主题。

- 这是公开仓库，不是 OpenAI 官方产品。
- 不得修改 WindowsApps 中的安装文件、`app.asar`、官方签名或其他 Codex 程序文件。
- 主题应用必须保持可恢复：修改 Codex 配置前创建备份，写入失败时保留或恢复原内容。
- 只支持 Windows；不要为了跨平台表象削弱现有 Windows 路径、进程身份或 PowerShell 安全检查。

## 开始工作前

1. 先运行 `git status --short --branch`，识别并保留用户已有改动。不要回退、覆盖或格式化无关文件。
2. 若仓库根目录存在 `.codegraph/`，理解或定位代码时先使用 `codegraph explore "<问题或符号>"`，再按需使用 `rg` 和读取文件。
3. 阅读与改动直接相关的测试。跨进程或运行时改动还要检查相邻契约和调用方。
4. 依赖以 `package-lock.json` 为准，使用 `npm`。不要手工编辑锁文件。

## 目录与职责

- `src/main/`：Electron 主进程及所有特权操作。窗口生命周期、IPC 注册、本地主题存储、Codex 检测/启动/恢复、PowerShell 和 CDP 都属于这里。
- `src/preload/`：唯一的渲染进程桥接层。通过 `contextBridge` 暴露最小且有类型的 `window.studio` API。
- `src/renderer/`：React 编辑器与 Studio 内预览。不得直接使用 Node.js、Electron 主进程 API 或任意文件系统访问。
- `src/shared/`：跨进程的类型、Zod 模型、外观令牌、几何、排版、粒子和 IPC 契约。这里是主题数据结构的权威来源。
- `resources/windows/`：随安装包分发的 PowerShell、运行时注入模板、CSS、图标和字体。这些是源文件，不是可随意忽略的构建产物。
- `tests/`：Vitest、happy-dom 及 PowerShell 回归测试。
- `docs/`：用户指南、隐私规则和已有设计/验收记录。
- `dist/`、`dist-electron/`、`release/`、`output/`、`node_modules/`：生成或本地目录，不要手工修改或提交。

## 跨进程契约

- 保持 `BrowserWindow` 的 `contextIsolation: true`、`nodeIntegration: false` 和 `sandbox: true`。不要通过关闭这些选项解决功能问题。
- 新增或修改 IPC 时，同步更新 `src/shared/contracts.ts`、`src/main/index.ts`、`src/preload/index.ts`，以及需要的 `src/renderer/src/env.d.ts`、调用方和测试。
- 渲染进程传来的数据不可信。主进程边界必须验证标识符、主题对象、路径、素材类型、文件大小和布尔参数，不能只依赖 TypeScript 类型。
- Codex 操作继续使用 `captureIpcResult`/`unwrapIpcResult` 传递可序列化错误；不要让 Electron 把内部异常对象直接暴露给渲染进程。
- 订阅型 IPC 必须返回取消订阅函数，React 组件卸载时必须清理监听器。

## 主题模型与预览

- `src/shared/theme.ts` 中的 Zod schema、`ThemeProfile`、默认主题和迁移逻辑必须保持一致。持久化结构有破坏性变化时提升版本，并为旧版本提供显式迁移；不要静默丢弃已有字段。
- 新增主题字段时，逐项检查默认值、schema/迁移、编辑器状态与控件、Studio 预览、`src/main/theme-compiler.ts`、`src/main/codex-service.ts`、运行时注入载荷、复制/保存和相应测试。
- 外观、排版、图标、首页布局或粒子改动优先复用 `src/shared/` 中的令牌和生成函数，避免在 Studio 预览与 Codex 运行时各维护一套魔法值。
- `resources/windows/renderer-inject.js` 操作的是外部应用 DOM，必须可重复执行、可完整清理，并在 Codex DOM 不匹配预期时安全退出，不能留下半套布局。
- 尽量减少对用户整台电脑、鼠标、键盘和前台窗口的自动化控制。查看或验证效果时，优先通过 Electron、Playwright、CDP 或应用已有调试端点直接连接目标软件；只有直接连接无法完成验证时，才使用桌面级操作。
- 不确定 Codex 中某个元素、层级、属性或选择器时，优先连接实际运行中的 Codex 并读取其 DOM。不要根据截图、类名习惯或旧结构硬猜；无法读取实际 DOM 时，应明确说明假设并采用安全降级行为。
- 注入用户可编辑文案时使用文本语义，不得把文案拼成 HTML。载荷进入 JavaScript 或 CSS 前必须使用结构化序列化和现有转义逻辑。
- 修改运行时 CSS/DOM 时，同时核对 Studio 预览。尤其保持 `resources/windows/dream-particle-effects.css` 与 `src/renderer/src/particle-effects.css` 中共享动画的一致性。

## 文件、进程与恢复安全

- 主题和素材路径必须约束在主题目录内。保留 UUID 校验、绝对路径拒绝、路径穿越检查、素材大小限制、格式/文件头验证和 SVG 安全检查。
- 持久化配置和运行时载荷使用临时文件、同步落盘、原子替换及失败回滚。不要改回直接覆盖写入。
- 调用 PowerShell 时使用参数数组和 `-File`，不要拼接命令字符串。保留非交互模式、超时、输出大小限制和结构化 JSON 结果。
- `resources/windows/config-utf8.ps1` 必须继续严格校验 UTF-8，并逐字节保留无关 TOML 内容、换行及嵌套 `[desktop.*]` 子表。
- Codex 启停和 CDP 连接必须继续验证 Microsoft Store 安装身份、进程归属、浏览器 ID 和 `127.0.0.1` 端点。不得连接任意调试端口或终止身份未验证的进程。
- 保留操作互斥、会话恢复和配置备份归档。任何“停止”与“恢复”行为变更都要覆盖成功、失败和重启场景。

## 隐私与示例数据

严格遵守 `docs/PRIVACY.md`：

- 源码、测试、文档、日志、截图和可提交图片中不得出现真实项目、任务、分支、工作区、账号、团队、邮箱、用户目录、令牌或调试会话信息。
- 预览和夹具只使用当前公开仓库名或明显虚构的数据；不得从运行中的 Codex 同步真实内容。
- 修改 `src/renderer/src/preview-home.ts` 的固定快照时，同步更新完整允许列表，并运行隐私文档指定的预览布局测试。
- 截图或视觉验收产物在提交前必须人工检查；临时产物留在已忽略目录中。

## 代码风格

- 遵循现有 TypeScript：ES modules、2 空格缩进、单引号、无分号、严格类型和 `noUncheckedIndexedAccess`。
- 优先使用小而明确的函数、判别联合、Zod schema 和已有共享帮助函数。避免 `any`、非必要类型断言和复制共享常量。
- React 使用函数组件和 hooks；副作用应可取消，异步状态必须处理失败和组件卸载。
- 面向用户的 Studio 文案保持简体中文；协议字段、类型和内部符号保持现有英文命名。
- 不要进行与任务无关的重构、全仓格式化或依赖升级。新增依赖必须有明确必要性，并同步提交 `package.json` 与 `package-lock.json`。
- 注释只解释不明显的约束或安全原因，不复述代码。

## 验证

按改动范围先运行目标测试，再运行必要的完整检查。常用命令：

```powershell
npm run typecheck
npm test
npm run test:config
npm run build
```

- 纯文档改动可不运行构建，但要检查链接、命令和 `git diff`。
- 主题 schema、编译、外观、排版或几何改动：运行对应的 `tests/theme.test.ts`、`tests/runtime-theme.test.ts`、`tests/appearance.test.ts`、`tests/typography.test.ts` 或 `tests/geometry.test.ts`。
- 素材或持久化改动：运行 `tests/profile-store.test.ts`。
- IPC、Codex 生命周期、CDP 或 PowerShell 改动：运行相关 Vitest，并执行 `npm run test:config`。
- 注入模板、Codex DOM 或运行时样式改动：运行 `tests/renderer-payload.test.ts`、`tests/renderer-home-dom.test.ts`、`tests/codex-service.test.ts` 及相关预览测试。
- Studio UI 改动：运行对应 happy-dom 测试并做实际 Electron 视觉/交互检查；至少覆盖窗口最小尺寸 `1120x720`，确认无重叠、溢出和失效控件。
- 只有打包或安装器相关改动才需要 `npm run package:dir` 或 `npm run package:win`。产物位于 `release/`，不要提交生成文件。

运行单个 Vitest 文件可使用：

```powershell
npm test -- tests/theme.test.ts
```

修改预览快照或示例数据时还必须运行：

```powershell
npm test -- --run tests/preview-layout.test.ts
```

## 完成标准

- 行为、类型、跨进程契约、预览和运行时实现保持一致。
- 安全、恢复、隐私和 UTF-8 不变量未被削弱。
- 已执行与风险匹配的测试，并如实报告未执行或受环境限制的验证。
- 最终检查 `git diff --check` 和 `git status --short`；除非用户明确要求，不要创建提交、打标签、发布或打包。
- 用户要求提交时，沿用仓库格式，例如 `feat(preview): ...` 或 `fix(runtime): ...`，摘要简洁且可用中文。

---
> Source: [269289854/Codex-Dream-Skin-electron](https://github.com/269289854/Codex-Dream-Skin-electron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
