## deepwrite

> - 项目已首次建立全仓格式基线；`.editorconfig`、`prettier.config.mjs` 和 `eslint.config.js` 是统一格式与静态检查规则的唯一来源，后续新增及修改的适用文件必须遵守，未经明确讨论不得绕过、弱化或另建冲突规则。

# DeepWrite 项目协作规范

## 全仓格式基线

- 项目已首次建立全仓格式基线；`.editorconfig`、`prettier.config.mjs` 和 `eslint.config.js` 是统一格式与静态检查规则的唯一来源，后续新增及修改的适用文件必须遵守，未经明确讨论不得绕过、弱化或另建冲突规则。
- 格式化统一使用根目录脚本：`pnpm format` 用于写入格式，`pnpm format:check` 用于只读检查；ESLint 分别使用 `pnpm lint:eslint`、`pnpm lint:fix`，边界检查使用 `pnpm lint:boundary`。
- 提交前必须从仓库根目录运行 `pnpm verify`，确保格式检查、类型检查、ESLint、边界检查、测试和构建全部通过；CI 同样以 `pnpm verify` 为准。
- 日常改动只格式化任务范围内的文件，不得把无关文件的大面积格式变化混入功能修改；若确需调整全仓格式规则或重新建立基线，应单独说明影响范围并集中处理。

## GitHub 提交与署名

- 创建 Git commit、Pull Request 或执行合并时，不得擅自添加任何个人作为共同作者、协作者或附加署名；禁止自行加入 `Co-authored-by`、`Signed-off-by` 等署名尾注，也不得在提交信息、PR 描述或合并信息中额外列出个人姓名或邮箱。仅当用户明确要求添加某项署名时才可添加；正常使用仓库现有 Git 身份与 GitHub 自动生成的记录，不得擅自改写作者身份。
- 本仓库创建 Git commit 时固定使用 `carswj <wenjia@wenjiadeMacBook-Air.local>` 作为提交身份；如本地尚未配置，只能通过仓库级 `git config --local user.name carswj` 和 `git config --local user.email wenjia@wenjiadeMacBook-Air.local` 设置，不得修改全局 Git 身份。除非用户明确要求变更，否则后续提交均沿用该身份。

## 项目布局与基础结构设计

仓库是 pnpm workspace：`apps/*` 放可运行应用，`packages/*` 放共享库，`tools/` 放校验与打包编排。根目录 `package.json` 的脚本是开发、校验和打包入口；依赖版本以 `pnpm-workspace.yaml` 的 catalog 为准。新增能力必须先落入现有分层，不得另起平行应用、平行协议或平行打包入口。

```text
deepwrite/
├── apps/desktop/                 # 唯一桌面客户端（Electron + Vue）
│   ├── src/main/                 # 主进程：窗口、安全 IPC、配置、Utility 监管
│   ├── src/preload/              # 预加载：白名单暴露 window.deepwrite
│   ├── src/utilities/            # Utility 进程：core / agent / tool 入口与本地存储
│   ├── src/renderer/             # 渲染进程：工作台 UI
│   ├── src/extras/               # 主进程侧可选能力（如云备份），由 Main 注册 IPC
│   └── scripts/                  # 冒烟、安装包校验、electron-builder 钩子
├── packages/contracts/           # 命令、事件、领域模型的 Zod 契约
├── packages/pi-runtime-adapter/  # Agent Runtime 与受控工具适配
├── packages/shared/              # 无业务语义的 ID / 时间工具
├── tools/                        # 边界检查、Electron 运行时、测试包编排
└── docs/images/                  # README 配图
```

- DeepWrite 采用 Electron 多进程架构，职责不可混用：Renderer 只负责界面与会话编排；Preload 只做 `window.deepwrite` 白名单与双向 Zod 校验；Main 管理窗口、密钥、配置存储，并通过 `UtilitySupervisor` 监管 Utility；Core Utility 是本地项目与路径注册表的唯一写入者；Agent Utility 运行模型与智能体；Tool Utility 只提供受控工具执行边界。跨进程通信走 `@deepwrite/contracts` 中的 Envelope 命令/事件，不得另开未校验通道。
- 进程依赖单向、收口明确。Renderer 只能从 `@deepwrite/contracts/renderer` 取运行时值，禁止引入 `electron`、`node:`、Pi SDK 或 `@deepwrite/pi-runtime-adapter`；该边界由 `pnpm lint:boundary` 强制检查。模型密钥与 Provider 凭据只存在于 Main / Agent，不得下发到 Renderer。Agent 需要读本地作品时，只能经 Main 授权的内部命令桥访问 Core 的只查询令（当前限于长篇 `long.getWorkspaceIndex` / `long.readDocument` / `long.search`），不得让 Agent 直接写盘。
- `packages/contracts` 是协议与领域模型的唯一来源。新增命令、事件、清单字段或 Preload API 时，先改契约与对应 Schema，再改 Preload、Main 路由和 Utility 实现；Renderer 新增从 `@deepwrite/contracts` 导入的运行时值，必须同步导出到 `packages/contracts/src/renderer.ts`。`packages/shared` 只放 `createId` 一类无业务语义的工具，不得把领域逻辑塞进去。Pi / 工具 schema / 子智能体运行时留在 `packages/pi-runtime-adapter`；桌面进程编排留在 `apps/desktop`。
- 本地作品以文件夹为单位，清单文件固定为 `deepwrite.json`，正文与设定使用 UTF-8 Markdown。短篇/剧本/素材库/技能库走 `folder-catalog-store`；长篇走 `long-project-store` 与 `long-workspace-service`。Core 必须原子写入，智能体对文稿的修改先以 proposal 事件展示差异，用户接受后才由 Core 落盘；不得让 Agent 或 Renderer 静默覆盖较新版本。
- Renderer 以 `WorkspaceShell.vue` 为工作台壳：默认三栏写作面留在入口 chunk，设置、长篇、市场、云备份等用 `lazyAppComponents` 按需加载。界面状态放 `stores/`，跨组件编排放 `composables/`，领域页面可放 `features/` 或 `extras/`。主进程可选能力（如云备份）放 `apps/desktop/src/extras/`，对应界面放 `renderer/src/extras/`，由 Main 注册专用 IPC，并复用同一套 contracts。新增功能先扩现有 coordinator / store / Utility handler；只有新的可选能力才新增 extras，不得把业务写入逻辑放进组件或 Preload。
- electron-vite 的打包入口固定为 Main `src/main/index.ts`、三个 Utility（`core-entry` / `agent-entry` / `tool-entry`）、Preload 与 Renderer。改进程边界、新增 Utility 或调整 `files` 时，必须同步 `apps/desktop/electron.vite.config.ts`、supervisor 启动路径、冒烟脚本和边界检查；不得把运行时依赖改回塞进 ASAR 的 pnpm 树，除非同时更新 before-build 钩子并补安装包内验证。

## 代码体量、拆分与耦合

新增和修改代码必须保持可维护：文件精简、职责单一、依赖方向清晰，禁止继续堆成难以阅读和拆改的大文件。

- 普通实现文件（`.ts` / `.js`，含 store、composable、service、handler）尽量不超过 300 行，超过 400 行必须按职责拆分后再继续改。Vue 单文件组件尽量不超过 400 行，超过 500 行必须把模板、脚本或样式拆到独立子组件 / composable，不得继续往同一 `.vue` 追加功能。测试文件允许稍长，但尽量不超过 600 行；超过时应按场景拆成多个测试文件，而不是在同一文件无限追加用例。
- 拆分必须提高内聚、降低耦合，禁止为凑行数做无职责的机械切分，也禁止把互不相关的逻辑堆进同一文件形成上帝模块、上帝组件或上帝 store。一个文件只承载一个明确职责；页面编排、状态、领域写入、IPC 路由不得混写。
- 新逻辑优先落入现有分层：界面状态放 `stores/`，跨组件编排放 `composables/`，领域页面放 `features/` 或 `extras/`，业务写入走 coordinator / Utility handler。不得把可复用逻辑继续塞进已偏大的 `App.vue`、`WorkspaceShell.vue`、巨型 store 或 Preload。
- 模块之间只通过明确的公共接口协作，禁止循环依赖，禁止互相直读内部状态或私有实现。共享代码只放真正跨领域且无业务语义的工具，或已由 `@deepwrite/contracts` 定义的协议；不得为了图省事抽出过宽的“万能工具”或让无关领域互相引用。
- 修改已超限的文件时，本次改动必须顺带把触及的职责拆出去，而不是在超大文件上继续打补丁。拆出的新文件同样遵守上述体量与分层约束。

## 重要：测试代码敏感信息保护

- 所有测试代码、测试夹具、快照、示例请求、Mock 数据和测试日志中，严禁出现真实接口地址、服务器 IP、生产域名、明文 Token、API Key、管理员口令或其他可用凭据。
- 测试需要构造地址或凭据时，只能使用 `example.test` 等保留测试域名、明显无效的占位值，或从已被 Git 忽略的本地环境文件读取；不得把本地环境变量的真实值写入快照、错误信息或测试输出。
- 提交前必须检查新增及修改的测试文件，确认没有真实接口信息或密钥；一旦发现泄露，应先移除明文并立即轮换已经暴露的凭据。

## 前端反馈与布局

- 表单校验警告、操作错误、成功和普通提示，统一使用不参与页面布局的浮层反馈，例如 toast、message、notification；无需用户处理的提示应自动消失。
- 禁止把临时警告或错误块插入表单、弹窗或操作按钮之间，以免改变容器高度、挤压按钮、造成页面跳动。
- 只有必须由用户确认后才能继续的风险操作才使用模态确认弹窗；普通校验和操作结果优先使用短暂浮层提示。
- 字段说明可以作为稳定的辅助文案保留，但不得用它承载临时警告或错误状态。

## 前端视觉、主题与控件一致性

- 新增或调整页面、弹窗时必须沿用现有全局视觉体系，优先复用 `--surface-main`、`--surface-raised`、`--surface-muted`、`--surface-hover`、`--surface-selected`、`--theme-line`、`--theme-line-soft`、`--text-primary`、`--text-secondary`、`--text-tertiary`、`--accent`、`--accent-soft` 等主题变量；不得为普通容器另建一套与主界面割裂的固定色板。
- 页面必须与“设置 → 外观”联动，主题模式、强调色、背景色、前景色、UI 字号和相关可读性设置更新后应即时生效。使用 `Teleport` 的弹层同样必须读取根节点主题变量，并同时验证浅色、深色和自定义强调色下的对比度。
- 列表选择框统一复用 Renderer 的 `PopupSelect` 组件及其交互、尺寸、圆角、焦点和禁用状态，不得在同类业务表单中混用浏览器原生 `<select>`。弹窗内使用时必须保证弹出菜单层级高于所属弹窗且不会被遮挡。
- 普通保存、创建、确认等主操作按钮沿用主界面的中性深色实心样式；只有删除持久化数据、不可恢复覆盖等真正危险操作才使用红色危险按钮。“新建学习”只清空当前临时样本、预览和聊天记录且不影响已落盘资料，因此“确认新建”必须使用中性深色主按钮，不得使用危险红色。
- 学习仿写等复杂页面要与设置页保持一致的容器层级、圆角、边框、间距和控件风格，并兼容紧凑窗口及设置中允许的 UI 字号范围；避免依赖会在字号增大后造成裁切的固定高度。

---
> Source: [swjybky/deepwrite](https://github.com/swjybky/deepwrite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
