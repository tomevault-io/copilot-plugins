## tallya

> - 不要运行 `pnpm tauri dev`。

项目规则：

- 不要运行 `pnpm tauri dev`。
- Tallya 面向 Windows 和 macOS 双平台；项目脚本、文档示例和测试命令必须保持跨平台，不要写入 `cmd /c`、`powershell`、`.bat`、`.cmd`、`where.exe` 等只能在单一平台运行的命令，除非代码里有明确的平台条件分支。
- 开发前先阅读 `DESIGN.md`。
- 修改 UI 时必须遵守 `DESIGN.md`。
- 不要未经要求大改首页布局。
- 不要新增 Tauri 新窗口，除非用户明确要求。
- 版本号必须跟随 `package.json`，不要在 UI 中写死版本号。

产品方向：

- Tallya / 职迹 是本地优先的工作记忆工具，核心是"低摩擦记录 + 强检索"（见 `docs/PLAN.md`）。
- 记录是最高频动作，**不得**给它加 AI 网关或必填项；AI 是可选增强。
- 首页是"当天条目流"：顶部 composer 即记，下面按时间倒序展示当天 entry。
- 不要把它做成后台管理系统、SaaS 官网、普通日报生成器或项目管理系统。
- 首页必须保持聚焦：今天做了什么？

文案规则：

- 修改用户可见文案时，必须遵守 `DESIGN.md` 中的产品性格和文案语气规则。
- 默认提醒、空状态、错误提示、按钮文案都要保持克制、温和、清楚。
- 避免催促式、老板式、鸡血式、卖萌式文案。
- 不要使用“日报”作为核心产品心智，优先使用“工作记忆”“沉淀”“整理”等词。

UI 约束：

- 所有可点击元素必须有 `cursor-pointer`，disabled 状态使用 `cursor-not-allowed`。
- Tailwind 4 能用 canonical class 的地方优先不用任意 px class。
- TSX 文件不要堆太多代码；组件变长时按职责拆分为更小的组件文件。
- 不要做未经要求的大型布局重构。
- 不要随意重设计首页。
- 搜索使用 Spotlight / Command Palette 风格。
- 预览和确认使用轻量 Dialog。
- 没有可执行价值的按钮不要展示，不要用 disabled 按钮做占位。
- 设置页保持轻量分组：左侧分组菜单 + 右侧内容，不使用顶部横向 Tabs，不做后台 sidebar。

代码组织规则：

- 不要把多个大型 TSX 组件写进一个文件。
- Dialog、Panel、List、ListItem、Settings Section 应拆分为独立组件。
- UI 组件不要直接调用 `localStorage`、Tauri command、shell、AI provider。
- 本地存储走 repository。
- AI 调用走 service / provider。
- service / repository / provider 逻辑不得写死在 UI 组件中。
- 新增设置项时要考虑未来扩展。
- `components/ui` 只放 shadcn 基础组件，不放业务组件。
- 业务组件放到 `src/features/work-memory/components`。
- AI 逻辑放到 `src/features/work-memory/services/ai`。
- 设置存储逻辑放到 `app-settings-repository`。
- 工作记录存储走 `entry-repository`（条目）和 `clarification-repository`（补充）。
- 旧的 `daily-memory-repository` 已随 entry 模型移除；`daily_memories` 表与一次性迁移（`migrations.ts` / `daily-memory-entry-migration.ts`）仅为兼容旧数据保留，不要在新功能里使用。
- 检索走 `memory-search-service`（合并 entry 与 clarification 命中），不要在 UI 里直接拼检索逻辑。

测试组织规则：

- 业务测试放在所属模块的 `__tests__` 目录。
- `components` 测试放到 `src/features/work-memory/components/__tests__`。
- `components/settings` 测试放到 `src/features/work-memory/components/settings/__tests__`。
- `hooks` 测试放到 `src/features/work-memory/hooks/__tests__`。
- `services` 测试放到 `src/features/work-memory/services/__tests__`。
- AI 测试放到 `src/features/work-memory/services/ai/__tests__`。
- database 测试放到 `src/features/work-memory/services/database/__tests__`。
- feature 根级 view model / home 测试放到 `src/features/work-memory/__tests__`。
- 不要把大量 `.test.ts` / `.test.tsx` 文件平铺在业务目录。
- 不要把 feature 测试集中到项目根 `tests` 目录。
- 移动测试文件时必须修正 import，并运行测试。

Provider 规则：

- 不要向用户暴露 Mock Provider。
- Mock Provider 只用于测试和开发。
- 当前用户可见 AI 服务是 Codex CLI、OpenAI Compatible 和 Claude / Anthropic。
- Provider 结构要继续为后续服务（如本地模型）预留扩展。
- 不要把未来 Provider 假设写死进 UI。

目录边界：

- `src/features/work-memory/components`：放业务 UI 组件。
- `src/features/work-memory/components/settings`：放设置页相关组件。
- `src/features/work-memory/hooks`：放页面状态和交互 hooks。
- `src/features/work-memory/services`：放业务服务、repository、AI provider、窗口能力。
- `src/features/work-memory/services/ai`：放 AI Provider、AI Service、Codex CLI Provider、Mock Provider、相关测试。
- `src/components/ui`：只放 shadcn 基础组件，不放业务组件。

验证约束：

- 不启动新的 dev server；如需查看网页或做浏览器验证，访问用户已运行的 `http://localhost:1420`。
- 常规验证只运行 `cargo check`、前端 lint 和 type-check。
- 修改前端逻辑后至少跑 type-check 和 lint。

发版流程（用户要求"发版"时按此走）：

- 先写 CHANGELOG：在 `CHANGELOG.md` 顶部加一段 `## <版本号>`，写**面向用户**的要点（新增/优化/修复，温和清楚，不要照搬 commit 信息）。发布工作流会把这段写进 `latest.json` 的 `notes`，应用内"检查更新"会展示。
- 再升版本号：运行 `pnpm bump <版本号>`，它会同步更新 `package.json`、`src-tauri/tauri.conf.json`、`src-tauri/Cargo.toml`、`src-tauri/Cargo.lock` 四处（发布工作流校验三处版本一致，缺一不可）。
- 提交（`chore: release v<版本号>`）并推送 master，然后打 tag `v<版本号>` 推送即触发 `.github/workflows/release.yml`。
- 工作流默认建 **draft** release；发版后需在 GitHub 手动 Publish，并确保该版本是 "Latest"（更新器 endpoint 用的是 `releases/latest/download/latest.json`，草稿/非 latest 都拉不到）。
- 自动更新依赖签名：CI 必须配好 `TAURI_SIGNING_PRIVATE_KEY`（及可选 `TAURI_SIGNING_PRIVATE_KEY_PASSWORD`）secret，否则 `createUpdaterArtifacts` 打包会失败。
- 自动更新只验证"能检测+下载+安装"，**不**保证无 SmartScreen（安装包未做 OS 代码签名）。`latest.json` 当前只含 `windows-x86_64`。

网络与代理：

- 所有原生 HTTP（更新器、OpenAI Compatible / Anthropic 请求、网关探测）统一走系统代理——见 `system_proxy.rs`，逻辑：有系统代理走代理、没有走默认网络（并回退环境变量），**loopback（本地网关）永远直连**。新增对外 HTTP 请求时沿用这条，不要绕开。
- Codex CLI 是子进程，不走我们的 reqwest，代理需它自己/继承环境变量。
- 时序坑：0.2.6 的更新器读不到系统代理，**代理环境下 0.2.6 无法自动更新**；0.2.7 起修复。因此 0.2.7 需手动安装，自动更新首次端到端验证是 0.2.7 → 0.2.8。
- 若给更新加 GitHub 加速镜像：**必须多 endpoint 兜底**（镜像在前、直连 GitHub 在后），绝不硬编码单一镜像（免费镜像会挂，会焊死更新通道）。镜像只对无代理用户有意义。

---
> Source: [c-jayden/tallya](https://github.com/c-jayden/tallya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
