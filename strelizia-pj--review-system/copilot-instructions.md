## review-system

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

芝士学爆（Gitee 仓库 review-system；package.json 与数据目录名仍为 forgetting-curve-reminder）— 基于 Electron + React + TypeScript 的 Windows 桌面应用。基于 FSRS 算法管理知识点复习提醒，集成番茄钟、每日计划、学习统计等功能。

## 常用命令

```bash
# 开发模式（Vite + Electron 热重载）
npm run dev

# 类型检查
npx tsc --noEmit

# ESLint 静态检查（0 错误，警告可存在）
npm run lint

# 单元测试（Vitest + Testing Library）
npm run test

# 生产构建
npm run build

# 一键构建+打包（需设 ELECTRON_MIRROR 镜像加速）
npm run package

# 同步 main+tags 到 GitHub（构建发布仓库）
npm run sync-github
```

**工作流程（必须遵守）**：用户提出需求 → 修改代码 → **自检** → 用户检查确认满足要求 → **此时才打包** Windows 安装包（输出到 `release/芝士学爆-Setup-<版本号>.exe`，版本号见 package.json）。未经用户确认，不执行 `npm run package`。

**自检分级**：

- **简单任务**（单文件小改、纯样式/文案调整、类型修复）：`npx tsc --noEmit` 零错误 + `npm run build` 通过即可
- **复杂任务**（涉及 3+ 文件、跨模块交互链路、弹窗/表单/导航等 UI 行为变更、数据层或迁移改动）：在上述基础上，**必须通过 Playwright（浏览器连接 dev server，可用 mock 注入 electronAPI）验证交互行为**

## CI/CD 与发布（GitHub Actions）

- **双仓库**：Gitee `review-system` 为主开发仓库；GitHub `Strelizia-PJ/review-system`（public）为构建发布仓库（remote 名 `github`）。日常推 Gitee，发布前 `npm run sync-github`
- **ci.yml**（push/PR main）：lint → tsc → vitest → vite build
- **release.yml**（tag `v*` 触发）：windows-latest 产出 NSIS 安装包（含 latest.yml/blockmap），macos-latest 产出 DMG（x64+arm64，ad-hoc 签名），全部自动上传 GitHub Release（内置 GITHUB_TOKEN，无需个人凭据）
- **发布 SOP**：`npm version <版本>` → `npm run sync-github`（tag 推送触发构建）→ 浏览器或匿名 API 确认 Actions 成功 → Gitee 网页手动建对应 Release
- **自动更新**：electron-updater，仅 Windows 打包环境启用（启动 30s 后静默检查 + 设置页手动检查/进度/重启安装）；更新源为 GitHub Releases。macOS 未签名不支持在线更新（设置页显示跳转下载页）。CI 状态可用匿名 API `https://api.github.com/repos/Strelizia-PJ/review-system/actions/runs` 查询
- **已知限制**：国内访问 GitHub 更新源可能慢（后续可加 OSS 镜像）；macOS 首次打开需右键→打开

## 工程化规范

- **ESLint**（flat config `eslint.config.mjs`）：typescript-eslint recommended + react-hooks；`no-explicit-any` 降为警告（迁移代码的历史字段访问）；空 catch 允许；提交前 husky+lint-staged 自动 `eslint --fix` + `prettier`
- **Prettier**：无分号、单引号、行宽 110
- **测试**（`tests/`）：Vitest + @testing-library/react + happy-dom；覆盖 constants 纯函数（FSRS 预览/保留率/封顶）与共享组件（Button/Badge/Bars/ConfirmDialog）；数据层单测待 IStorage 抽象后补充

## 技术栈

| 层          | 技术                                                         |
| ----------- | ------------------------------------------------------------ |
| 桌面框架    | Electron 42                                                  |
| 前端        | React 18 + TypeScript                                        |
| 构建        | Vite 8 + vite-plugin-electron                                |
| 样式        | Tailwind CSS 3（`darkMode: 'class'`）                        |
| 状态管理    | Zustand 5                                                    |
| 数据存储    | JSON 文件（`%APPDATA%/forgetting-curve-reminder/data.json`） |
| Markdown    | @uiw/react-md-editor + remark-math + rehype-katex            |
| 非标准 JSON | json5                                                        |
| 打包        | electron-builder (NSIS)                                      |

## 架构分层

### 数据流

```
React Component → Zustand Hook → preload.ts (contextBridge)
    → IPC (ipcMain.handle) → queries.ts → connection.ts → data.json
```

### 目录结构

```
electron/                  # Electron 主进程（Node.js 环境）
├── main.ts                # 窗口管理 + IPC 处理器注册 + 生命周期
├── preload.ts             # contextBridge，暴露 electronAPI 给渲染进程
├── tray.ts                # 系统托盘
├── notifications.ts       # Windows 原生通知
├── scheduler.ts           # 定时复习检查（每小时）
├── auto-start.ts          # 开机自启动
└── database/
    ├── connection.ts      # JSON 文件读写 + AppData 接口 + ID 计数器
    ├── migrations.ts      # schema_version 递增迁移
    ├── queries.ts         # 知识复习 + 每日计划 + 学习会话 + 统计查询
    ├── game-import.ts     # Chill with You 游戏存档解析导入
    └── images.ts          # 本地图片文件存储 + 孤儿图片清理

src/                       # React 渲染进程
├── main.tsx               # React 入口
├── App.tsx                # 根组件
├── vite-env.d.ts          # window.electronAPI 全局类型声明
├── types/index.ts         # 共享 TypeScript 类型
├── hooks/                 # Zustand stores
│   ├── useKnowledge.ts    # 知识点 CRUD + 选中状态
│   ├── useReview.ts       # 复习数据查询
│   ├── useTheme.ts        # 暗色模式（light/dark/system）
│   ├── useDailyPlans.ts   # 每日计划
│   ├── usePomodoro.ts     # 番茄钟计时核心逻辑
│   └── useStudyStats.ts   # 学习统计 + 月份导航
├── components/
│   ├── layout/            # AppLayout（导航框架）+ Sidebar
│   ├── knowledge/         # 知识点列表、卡片、添加表单、详情页(Markdown)
│   ├── review/            # 今日复习（含逾期）、统计面板
│   ├── mistakes/          # 易错点（计数器 + 排序）
│   ├── manage/            # 调度管理（全局/单点间隔上限、改期、提前复习）
│   ├── plans/             # 每日计划页面
│   ├── pomodoro/          # 番茄钟计时器页面
│   ├── stats/             # 学习统计（日历热力图 + 补登）
│   └── import/            # 游戏数据导入
├── constants.ts           # 共享常量（QUALITY_LABELS, DAY_LABELS_SUNDAY_FIRST, DAY_LABELS_MONDAY_FIRST）
└── styles/
    └── index.css           # 全局样式 + @uiw/react-md-editor 适配
```

### 导航系统

应用使用基于状态的页面切换（非路由）。`AppLayout` 维护 `currentPage: NavPage` 状态，`Sidebar` 渲染导航按钮。`NavPage` 类型定义在 `src/types/index.ts`：

```ts
;'knowledge' |
  'today' |
  'mistakes' |
  'manage' |
  'plans' |
  'pomodoro' |
  'study-stats' |
  'import' |
  'stats' |
  'settings'
```

逾期复习不再单独成页：今日复习面板包含全部待复习条目（含逾期，红色高亮标注）。

知识点详情页通过 `useKnowledge.selectedId` 单独处理，不占用 NavPage。

### 数据存储

`connection.ts` 中的 `AppData` 包含所有顶层集合：

- `knowledge_points` — 知识点（content + detail Markdown）
- `review_records` — 复习记录（每条知识点首次创建 1 条，每次复习后由 FSRS 算法动态生成下一条待复习记录）
- `daily_plans` + `daily_plan_completions` — 每日计划（一次性/每日任务）
- `study_sessions` — 学习时长记录（番茄钟产生）
- `mistake_points` — 易错点（带错误计数器，按次数降序展示）
- `settings` — 键值对（主题偏好、番茄钟设置等）

`schema_version` 当前为 13（定义在 `connection.ts` 的 `CURRENT_SCHEMA_VERSION`，新装用户默认数据直接使用该版本）。每次新增集合通过 `migrations.ts` 递增版本号进行迁移；新增集合同时在 `loadData()` 做缺失字段规范化兜底（避免旧数据文件被严格校验误判损坏）。

### 暗色模式

Tailwind 使用 `darkMode: 'class'`。`useTheme` hook 管理三种模式（light/dark/system），通过 `document.documentElement.classList.toggle('dark')` 切换。偏好持久化到 `settings.theme_mode`。

### IPC 通道命名规范

所有通道使用 `domain:action` 格式：`knowledge:add`、`knowledge:set-max-interval`、`knowledge:reschedule`、`review:get-today`、`review:rate`、`review:rollback`、`review:forget`、`mistake:add`、`mistake:increment`、`plans:toggle`、`study:get-month-stats`、`import:scan`、`settings:get`、`app:minimize-to-tray`。渲染进程通过 `window.electronAPI.xxx` 调用，guard 子句 `if (!api()) return` 确保非 Electron 环境下优雅降级。

### 复习间隔上限与调度

- **全局上限**：settings 键 `max_review_interval_days`（默认 28，范围 1-365），在「调度管理」页配置。
- **单点上限**：知识点 `max_interval_days`（null = 跟随全局）；**有效上限 = min(全局, 单点)**，超出会自动封顶 pending 记录与 card.due。
- **评分自定义时间**：`rateReview` 第三参数 `customDays`（≥1），FSRS 记忆状态照常更新，仅覆盖本次间隔，不受上限约束。
- **纯改期/提前复习**：`reschedulePendingReview` 只移动 pending 记录日期与 card.due，不改 FSRS 状态。

### 注意事项

- **kcimg:// 协议**: 本地图片通过自定义协议加载，路径有严格校验（防目录穿越）。图片文件存储在 `%APPDATA%/images/{kpId}/`。
- **图片清理**: 更新知识点（`knowledge:update`）时自动通过 `deleteOrphanImages()` 清理 markdown 中不再引用的图片文件，删除知识点时通过 `deleteImages()` 删除整个目录。
- **循环依赖避免**: `notifications.ts` 不能直接导入 `main.ts`，通过 `setMainWindowGetter()` 注入主窗口引用。
- **@uiw/react-md-editor**: 基于 ProseMirror 的 WYSIWYG Markdown 编辑器，原生支持 Ctrl+Z/Y 撤销重做、KaTeX 数学公式（$...$ 行内 +

  $$
  ...
  $$

  块级）、自定义图片上传。内容通过 `markdownUpdated` 事件 2s 防抖自动保存。

- **托盘退出**: 使用 `app.quit()` 而非 `mainWindow.destroy()` 确保进程完全结束。
- **开机自启动**: 仅在 `settings['auto_start'] !== 'false'` 时启用，尊重用户偏好。
- **数据校验**: 加载 data.json 时进行顶级字段结构校验 + ID 计数器 NaN 过滤。
- **图片上传限制**: 单文件最大 10MB，每知识点最多 50 张图片。
- **图片引用格式**: markdown 中使用 `![alt](kcimg://{kpId}/{filename})` 引用本地图片。

## 测试方式

### 自动化测试（通过 Playwright）

**仅复杂任务需要执行**（3+ 文件、跨模块交互链路、弹窗/表单/导航等 UI 行为变更、数据层改动）；简单任务只需 TypeScript 检查与构建通过。浏览器无 `electronAPI` 时会优雅降级为空数据，验证数据相关功能需通过 `page.addInitScript` 注入 mock（参照既有 mock 模式）。

**流程：**

1. **`npm run dev` 启动应用** → Electron 窗口自动打开
2. **`browser_navigate`** 连接到 Electron 窗口（或 Vite dev server URL）
3. **`browser_snapshot`** 抓取页面结构，验证 UI 正常渲染
4. **针对修改的模块执行操作验证**（见下方示例）
5. **`browser_console_messages`** 检查无错误日志
6. **关闭浏览器页面**（`browser_close`），保持 dev server 运行

**按修改模块的测试清单：**

| 修改涉及模块                               | 必须验证的操作                                                                              |
| ------------------------------------------ | ------------------------------------------------------------------------------------------- |
| `electron/database/*`                      | 启动后检查控制台无数据加载错误                                                              |
| `useKnowledge.ts` / knowledge 组件         | 添加知识点 → 搜索 → 编辑标题 → 删除                                                         |
| `useReview.ts` / review 组件               | 查看今日复习列表 → 完成一条复习                                                             |
| `useDailyPlans.ts` / plans 组件            | 添加计划 → 切换完成状态 → 删除                                                              |
| `usePomodoro.ts` / pomodoro 组件           | 启动番茄钟 → 等待 5s 确认倒计时 → 暂停 → 重置                                               |
| `DetailPanel.tsx` / `@uiw/react-md-editor` | 打开知识点详情 → 编辑 markdown → 输入$...$ 公式确认渲染 → Ctrl+Z 撤销 → 保存 → 重新打开确认 |
| `images.ts` / kcimg 协议                   | 上传图片后确认渲染 → 删除图片引用后保存 → 确认孤儿文件已清理                                |
| `tray.ts` / `notifications.ts`             | 检查托盘图标可见（通过 snapshot 确认不报错）                                                |
| `GameImportPage.tsx`                       | 输入路径 → 扫描 → 选中日期 → 导入                                                           |
| `Sidebar.tsx` / 导航                       | 逐个点击导航按钮确认页面切换正常                                                            |
| `useTheme.ts` / 暗色模式                   | 切换主题确认无报错                                                                          |

**验证通过标准：**

- 浏览器控制台无红色错误（error）
- 页面快照中关键 UI 元素存在（按钮、列表、输入框）
- 操作后状态正确变更（loading 消失、数据更新）

### 手动测试（Playwright 不可用时备选）

1. `npm run dev` 启动应用
2. Electron 窗口内实际操作验证修改过的功能路径
3. Node.js 脚本独立验证后端逻辑（如 `electron/database/game-import.ts` 的解析功能）

## Claude Code 自动化

本项目 `.claude/` 目录下配置了以下专属自动化工具：

| 类型  | 名称                         | 触发方式       | 用途                                     |
| ----- | ---------------------------- | -------------- | ---------------------------------------- |
| Skill | `package-app`                | `/package-app` | 一键 TypeScript 检查 → 构建 → 打包安装包 |
| Skill | `verify-app`                 | `/verify-app`  | 启动应用 → Playwright 自动化验证 UI      |
| Agent | `electron-security-reviewer` | 自动/手动派遣  | Electron 安全审计（IPC、协议、沙箱）     |
| Agent | `code-reviewer`              | 自动/手动派遣  | 通用代码审查（类型、错误处理、规范）     |
| Hook  | `PreToolUse`                 | 自动（拦截）   | 阻止直接编辑 `package-lock.json`         |

**Skills** 通过 `/skill-name` 调用。**Agents** 可被 Claude 自动派遣或在对话中指定。**Hooks** 在对应事件时自动触发。

## 必须遵守的编码准则

1. **先思考再编码** — 不确定时开口问；存在多种解读时列出所有选项；有更简单方案直接说
2. **简单优先** — 只写最少代码；不做未被要求的抽象或灵活性；如果 200 行能 50 行完成就重写
3. **外科手术式修改** — 只动必须改的代码；不重组相邻代码；匹配现有风格；只清理自己引入的孤立引用
4. **工作流程** — 用户提需求 → 修改 → 自检（TypeScript 零错误 + 构建通过；复杂任务加 Playwright 交互验证）→ 用户确认后才打包。**自检不通过不交付，用户未确认不打包。**

---
> Source: [Strelizia-PJ/review-system](https://github.com/Strelizia-PJ/review-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
