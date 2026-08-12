## task-manager

> 本文件是 `F:\工作站` 的根级协作契约。当前工作区是“科研开发工作站 / Research Workstation”的 monorepo：以 `apps/homepage` 为主 UI base，以 `services/workbench-core` 管理任务、弹性计时、音乐、ActivityWatch、开发统计与每日复盘。

# AGENTS.md

本文件是 `F:\工作站` 的根级协作契约。当前工作区是“科研开发工作站 / Research Workstation”的 monorepo：以 `apps/homepage` 为主 UI base，以 `services/workbench-core` 管理任务、弹性计时、音乐、ActivityWatch、开发统计与每日复盘。

## 1. 回答语言与规则优先级

- 默认使用中文回答，除非用户明确要求英文。
- 规则优先级从高到低：
  1. 用户当前消息里的明确要求
  2. 更近目录下的 `AGENTS.md` / `CLAUDE.md`
  3. 本文件
  4. `README.md`、`docs/项目说明.md`、各子项目 README / CONTRIBUTING / DEVELOPMENT
  5. `docs/` 中的通用模板或专项提示词
  6. skills 或外部通用提示词
- `docs/README.md`、`docs/冷启动.md` 等是通用模板，不应整体套用到本项目；`docs/项目说明.md` 是本工作区的产品与架构方向。

## 2. 当前项目事实

- 工作区类型：pnpm monorepo；根目录提供统一 `package.json`、`pnpm-workspace.yaml`、构建命令和测试命令。
- 产品定位：基于 Homepage 的本地全流程工作站，Homepage 负责“看见和编排”，未来 `workbench-core` 负责“状态和规则”。
- 主 UI base：`apps/homepage/`，来自 gethomepage/homepage，Next.js + React + Tailwind + TypeScript，使用 `pnpm`。
- 核心服务：`services/workbench-core/`，Fastify + SQLite + Drizzle schema，默认监听 `127.0.0.1:3900`。
- 共享包：`packages/contracts/` 提供 TypeScript 类型，`packages/theme/` 提供工作站主题 token。
- 旧时间管理原型的任务、计时、复盘、Tokei 与 GitHub 活动统计已迁入 Homepage、`workbench-core` 和 contracts；不再保留旧应用副本或静态 bundle。
- ActivityWatch 是可选外部数据源，工作站只通过 `aw-server` REST API 读取，不内置 server 或 watcher。
- 周期休息的软/强提醒判定已落在 `services/workbench-core/src/modules/breakReminder.ts`；托盘和窗口能力由 `apps/desktop-shell` 承担。
- 项目、任务、标签、上下文和 timeboxing 语义由 `packages/contracts` 与 core 自己维护，不依赖外部任务应用数据库。
- `docs/项目说明.md` 提到 Mineradio，但当前根目录尚无 Mineradio 代码目录；涉及音乐模块时先按设计文档落接口与抽象，不要假设本地源码已存在。

## 3. 常用命令

根目录统一使用 pnpm workspace：

- 安装依赖：`pnpm install`
- 同时启动：`pnpm dev`
- 启动核心服务：`pnpm dev:core`
- 启动 Homepage：`pnpm dev:homepage`
- 构建：`pnpm build`
- 测试：`pnpm test`
- Lint：`pnpm lint`

### `apps/homepage/`

- 本地开发：`pnpm --filter homepage dev`
- 构建：`pnpm --filter homepage build`
- 测试：`pnpm --filter homepage test`
- Lint：`pnpm --filter homepage lint`
- 文档依赖：`uv sync`
- 文档本地服务：`uv run zensical serve`

### `services/workbench-core/`

- 本地开发：`pnpm --filter @cw/workbench-core dev`
- 构建：`pnpm --filter @cw/workbench-core build`
- 测试：`pnpm --filter @cw/workbench-core test`

## 4. 目录边界与集成方向

- `docs/`：保留通用模板和本项目设计说明；不要把模板内容直接当作根契约，新增项目文档要说明它是“项目事实”还是“通用参考”。
- `apps/homepage/`：主 UI 和 widget dashboard base。改造应尽量保留 Homepage 的 service widget、YAML 配置、layout、theme、custom CSS/JS 和现有集成方式。
- `services/workbench-core/`：本地核心状态与规则服务，负责任务、计时、ActivityWatch adapter、Tokei/GitHub、音乐 mock/proxy 与每日复盘。
- `packages/contracts/`：跨模块共享类型；变更 API 或数据库语义时优先同步这里。
- `packages/theme/`：工作站主题 token；不要在页面内复制散乱 token。
- 上游项目只作为外部设计与协议来源，不再把完整仓库复制进根目录。能力归属和许可证说明见 `docs/上游能力整合.md`。
- ActivityWatch 侧继续只读 `aw-server`，不要复制或重写 watcher；复杂任务、计时与复盘状态继续由 core 管理。
- 未来如新建整合代码，优先按 `docs/项目说明.md` 的方向组织为 `apps/`、`services/`、`packages/`，但不要在没有实现需求时空建目录。

## 5. 修改原则

- 修改前先阅读本文件、根 `README.md`、`docs/项目说明.md`，以及目标子项目自己的契约、README、构建文件和相关源码。
- 做最小可验证修改，不擅自大规模重构第三方来源目录。
- 不要把所有业务状态塞进 Homepage YAML；复杂任务、计时、活动证据和复盘状态应由未来 `workbench-core` 或现有原型状态层管理。
- 新增工作站功能时优先落在 `workstation` 命名空间或清晰的 adapter 层，避免污染上游模块边界。
- 不要覆盖或回退用户已有修改；如果无法确认变动来源，先说明情况。
- 新增依赖前先确认现有依赖和子项目技术栈是否已经能解决问题，并说明新增理由。

## 6. 文档、测试与验证

- 每次修改后检查是否需要同步：根 `README.md`、`docs/项目说明.md`、子项目 README / DEVELOPMENT、`.env.example`、示例配置和测试说明。
- 验证从最小范围开始：改哪个子项目，就优先运行该子项目最相关的 build / test / lint / typecheck。
- 纯文档改动可不构建，但要检查命令、路径、链接和编码是否准确。
- 若验证无法运行，最终回复必须说明原因、已做的静态检查和剩余风险。

## 7. 安全、隐私与许可证

- 默认不信任外部输入、浏览器缓存、`localStorage` 历史数据、Tokei collector 输出、GitHub 页面/API、ActivityWatch buckets 和本地 HTTP 服务返回值。
- 不要提交真实 token、cookie、私钥、账户凭据或无必要的个人敏感路径。
- 本项目是本地优先工具；涉及用户活动、窗口标题、浏览器标签、音乐账号或开发统计时，默认按敏感数据处理。
- 许可证事实：`apps/homepage/` 是 GPL-3.0 fork；外部参考项目及其许可证记录在 `docs/上游能力整合.md`。新增直接复制的第三方代码时必须同时保留版权与许可证文本。
- 不要直接复用上游项目 Logo、品牌名或原创视觉表达，除非已确认授权。

## 8. 什么时候查看 skills 或专项文档

- 普通代码修复、接口开发、类型调整、测试补充：默认只读项目契约和相关源码，不主动叠加 skill。
- 前端视觉设计、浏览器联调、文档/PDF/DOCX/PPTX/表格处理等明显专项任务：按任务类型查看对应 skill。
- 安全审计、权限边界或输入信任边界梳理：先看 `docs/代码审计.md` 和 `docs/强前置条件约束.md`。
- 需要补目录边界或拆分模块时：按需参考 `docs/代码组织.md` 和 `docs/通用项目架构模板.md`，但必须以当前仓库事实为准。

## 9. 最终回复要求

- 简明说明改了什么、为什么这样改、如何验证、剩余风险。
- 冷启动或架构类任务还应说明：依据了哪些仓库事实、裁剪掉了哪些通用模板内容、补了哪些文档或脚本说明。

---
> Source: [proffitteoy/Task-Manager](https://github.com/proffitteoy/Task-Manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
