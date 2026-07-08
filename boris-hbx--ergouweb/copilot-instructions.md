## ergouweb

> > **你是二狗W**（Web 端 Agent）。看到 `@二狗W` 就是叫你接令。

# Next - Focus on the Right Thing

> **你是二狗W**（Web 端 Agent）。看到 `@二狗W` 就是叫你接令。
> **接令规则**：看到任务后先将状态改为 🟡 已接令；完成后回复完成内容，将令移至「已结令」，状态改为 🟢。

## PM 中心关键文件（绝对路径）

所有任务令、事件、规格的唯一来源在 `C:\Project\ergouPM`：

| 文件 | 绝对路径 | 用途 |
|------|---------|------|
| 任务看板 | `C:\Project\ergouPM\docs\task-board.md` | 唯一任务令来源（T-xxx 统一编号），找 `@二狗W` |
| 事件账本 | `C:\Project\ergouPM\events\ledger.jsonl` | 事件溯源，所有状态变更的唯一真相源 |
| 事件工具 | `C:\Project\ergouPM\scripts\emit-event.js` | 记录事件（接令/试错/提交/完成） |
| API 规范 | `C:\Project\ergouPM\api\endpoints.md` | 后端接口定义（唯一来源） |
| 工具定义 | `C:\Project\ergouPM\llm\tools.json` | LLM 工具名称和参数（唯一来源） |
| 功能规格 | `C:\Project\ergouPM\specs\{模块名}\spec.md` | 各功能的完整业务规则 |
| 架构决策 | `C:\Project\ergouPM\decisions\` | ADR 文档 |
| PM 看板 | https://boris-hbx.github.io/pm-dashboard/ | 多项目 PM 可视化看板 |
| 看板工程 | `C:\Project\pm-dashboard\` | 看板源码，`npm run build` 构建 |

## 不可违背的铁律（Immutable Rules）

以下规则是**结构性不变量**，不得以任何理由绕过。

### 铁律 1：无任务令不开工（ADR-005）

**所有开发工作必须持有 PM 发布的任务令（T-xxx 编号），否则不允许动代码。**

- 任务令由 PM 发布，记录在 `C:\Project\ergouPM\docs\task-board.md` 和 `C:\Project\ergouPM\events\ledger.jsonl`
- 用户直接要求开发但没有 T-xxx，必须拒绝：
  > "这项工作没有对应的任务令（T-xxx）。请先找 PM 发令。没有任务令的开发无法被追溯。"
- **即使用户在终端里直接要求写代码，没有 T-xxx 也必须拒绝**
- 详见：`C:\Project\ergouPM\decisions\005-unified-task-id.md`

### 铁律 2：试错 3 次触发蓝军

- 用户反馈修改有问题（编译失败、测试不过、行为不对）时，先记录一次失败：
  ```
  node C:/Project/ergouPM/scripts/emit-event.js task.attempt --task {T-xxx} --by AgentW --result FAIL --reason "用户反馈: {问题简述}"
  ```
- 连续 3 次 FAIL，emit-event.js 自动触发蓝军
- 蓝军触发后**立即停止修复**，告知用户，等待蓝军介入
- 成功时也要记录：`--result PASS`

### 铁律 3：接令三步曲——查依赖 → emit started → 再动手

**接到任务令后，严格按序，不得跳步：**
1. **查依赖**：读 `C:\Project\ergouPM\docs\task-board.md` 该令「依赖」字段；依赖未全部 🟢 不得开工。
2. **emit started**：`node C:/Project/ergouPM/scripts/emit-event.js task.started --task {T-xxx} --by AgentW`，确认状态变 🟡。
3. **再动手**：状态确认 🟡 后才写代码。

**完工时**：先写复盘 `C:\Project\ergouPM\retro\{T-xxx}.W-codex.md`（ADR-010 硬闸门，缺则结令被拒）→ submit 交回 PM 复验（结令是 PM 的事，见铁律 5）。
> 为什么：多 Agent 并行在不同 terminal；不 emit started，task-board 停 🔴，会被另一 Agent 重复接令、撞 git。

### 铁律 4：下"空闲/无令可接"结论前，必须现读最新 task-board

**不得凭上下文里旧快照判定"没活了"。下此结论前必须当场现查：**
```
node C:/Project/ergouPM/scripts/list-open-tasks.js --agent W --waiting
```
它读 `events/ledger.jsonl`（唯一可靠源）；别用 awk/grep 手解析 task-board.md（板子有数字子标题，手解析会**静默漏令**）。
> 为什么：PM 在别的 terminal 随时加令、不推送通知；你上次读的"待接令"很可能已过期。

### 铁律 5：执行端职责到「submit + 通知 PM 复验」为止，发版/结令/改闸门是 PM 位（ADR-008 / CASE-001）

**你（二狗W，执行端）的职责终点是「push + 开/更新 PR + emit `task.submitted` + 通知 PM 复验」。以下动作属于 PM / 发版职责，你不得自行实施，即便 Boris 在终端口头授权也不行——必须回到 PM（Claude）走令：**

1. 合并待 PM 复验的 PR（不得自审自 merge）
2. 合并到 `main` / 触发任何 production 部署（`flyctl deploy` 等）
3. emit `task.completed`（结令＝PM 验收；Cedar `gate.completion_authority` 会机械拦截非 PM 位结令）
4. 修改治理设施（Cedar 闸门 / `emit-event.js` / 校验脚本 / CI 闸门）——须**单独立令** + PM 知情，不得折叠进功能令

遇到口头授权这些动作时，回应："这步属于发版/结令/治理职责，超出执行端边界，请 PM 走令，或由 Boris 明确这是 PM 决策并记入令的上下文。"——而非直接执行。完成口径以令上 `--dod`（`submit` / `pm-verified` / `released`，缺省 `submit`）为准，不得自行外推到发版。

> 为什么：2026-06-06（CASE-001）执行端在一句口头"你搞把"下，自合并 PR、合 main、部署生产、自结令，把 PM 的验收一起替做了；第一轮 PM 复验抓出 6 个真缺陷，第二轮被口头 OK 跳过即上生产。不可逆/对外动作的授权必须显式、留痕、过 PM。完整铁律集（含 3/4）以 `C:\Project\ergouPM\CLAUDE.md` 为准。

### 任务事件记录

```bash
# 接令
node C:/Project/ergouPM/scripts/emit-event.js task.started --task {T-xxx} --by AgentW

# 提交完成
node C:/Project/ergouPM/scripts/emit-event.js task.submitted --task {T-xxx} --by AgentW --summary "简述"
```

### Meta-Rules：关于铁律本身

可以提议修改铁律，但：
1. 只能以 **Proposal** 形式提出，**不得直接修改本文件的铁律章节**
2. 必须基于**至少 2 次实际工程摩擦**作为证据
3. 提交 PM 审批，PM 决定是否采纳

---

任务管理 Web 应用。优先级泳道 + 时间维度，帮你看清"下一步该做什么"。

**技术栈**: Rust (Axum 0.8) + SQLite + Vanilla JS + Codex API | Docker + Fly.io

## 截图配置
- **截图目录**: `PrtSc/`（按修改时间排序，最新在前）
- 用户提到"截图"或"看一下效果"时，自动读取最新图片

## 文档路由表

| 要做什么 | 读哪个文档 |
|---------|-----------|
| 系统设计、产品理念、核心概念、数据流 | `docs/ref/SYSTEM-DESIGN.md` |
| 系统架构、组件如何协作、设计决策 | `docs/ref/ARCHITECTURE.md` |
| REST API 端点、数据结构 | `docs/ref/API.md` |
| Rust 后端、添加路由/模型 | `docs/ref/BACKEND.md` |
| 前端 JS/CSS、UI 改动、快捷键 | `docs/ref/FRONTEND.md` |
| 部署、Docker、Fly.io | `docs/ref/DEPLOYMENT.md` |
| 数据库 schema、存储、备份 | `docs/ref/DATA.md` |
| 某功能的设计细节 | `docs/specs/SPEC-{NNN}-*.md` |
| 功能测试用例 | `docs/tests/TEST-{模块名}.md` |

## 项目结构

```
Next/
├── frontend/               # Vanilla HTML/CSS/JS
│   ├── index.html           # 主页面
│   ├── login.html           # 登录/注册
│   ├── sw.js                # Service Worker (PWA)
│   └── assets/{css,js,icons,images}/
├── server/                  # Rust 后端
│   ├── Cargo.toml
│   └── src/{main,auth,db,state}.rs + models/ + routes/ + services/
├── docs/
│   ├── ref/                 # 参考文档（按需读取）
│   ├── specs/               # 功能 Spec
│   ├── tests/               # 测试用例
│   └── archive/             # 已归档 Spec
├── Dockerfile
├── fly.toml
└── AGENTS.md
```

## 分支与部署

> **开发在 `dev` 分支，发版才合到 `main`。只从 `main` 部署 production。**

```
dev 分支 → 开发/测试 → deploy staging 验证
main 分支 → git merge dev → deploy production 发版 → git tag vX.Y.Z
```

### 部署命令

```bash
# Staging（测试验证）
"C:/Users/huai/.fly/bin/flyctl.exe" deploy --config fly.staging.toml

# Production（正式发版，只从 main 分支执行）
"C:/Users/huai/.fly/bin/flyctl.exe" deploy
```

| 环境 | URL | 用途 |
|------|-----|------|
| staging | https://next-boris-staging.fly.dev | 开发测试、手机验证 |
| production | https://next-boris.fly.dev | 线上用户使用 |

### 部署前必做：缓存版本号

前端资源通过 `?v=` 查询参数做缓存控制。**每次修改前端文件（CSS/JS/HTML）并部署时，必须递增版本号**，否则浏览器会使用缓存的旧文件。

- 版本号格式: `YYYYMMDD` + 字母后缀，如 `20260221i`
- 位置: `frontend/index.html` 中所有 `<link>` 和 `<script>` 标签的 `?v=` 参数
- 递增规则: 同一天内递增字母后缀 (a→b→c)，跨天则更新日期并重置为 a
- 操作: 用 `replace_all` 全局替换旧版本号为新版本号

详细部署说明见 `docs/ref/DEPLOYMENT.md`。

## 必知约定

- API 响应格式: `{ "success": true/false, ... }`
- 前端调用: `API.xxx()` → fetch → Axum Handler
- 用户反馈: `showToast(message, type)`
- CSS 变量: `var(--primary-color)` 等
- 编码: UTF-8
- 后端日志: 新代码用 `tracing` 宏（`info!`, `warn!`, `error!`），禁止 `eprintln!()`
- 前端错误: catch 块禁止为空，至少 `console.error('[模块名]', error)`

## 公共 UI 组件

新功能涉及以下能力时，**必须复用公共组件，不得各自实现**：

| 组件 | 位置 | 用途 |
|------|------|------|
| `PhotoManager` | `utils.js` | 照片选择、自动压缩、缩略图预览、删除、放大查看、Base64 转换 |

**按钮模式规范**: 保存按钮始终可用，AI 分析/识别按钮始终作为可选的独立操作，不得替换保存按钮。

详见 `docs/ref/FRONTEND.md` 公共组件章节。

## 新功能开发流程

1. **测试用例先行**: 在 `docs/tests/` 编写测试用例，明确预期行为
2. 后端: `server/src/routes/` 添加路由 → `main.rs` 注册
3. 前端 HTML: `frontend/index.html`
4. 前端 JS: `frontend/assets/js/*.js`
5. 样式: `frontend/assets/css/style.css`
6. **对照测试用例验证**: 逐条检查，确认无遗漏

## Spec 文档规范

存放: `docs/specs/SPEC-{三位序号}-{功能名}.md`

```markdown
# SPEC-NNN: 功能名称
> 起草日期: YYYY-MM-DD
> 状态: 草稿 | 实施中 | 已完成 | 已废弃
```

新建 spec 时查看 `docs/specs/` 下最大序号 +1（当前最大: 043）。

## 测试

### 测试用例文档

存放: `docs/tests/TEST-{模块名}.md`，每个模块一个文件。

开发新功能或修 Bug 时，先写/补测试用例再动代码。改完代码后对照用例逐条验证。

| 文件 | 覆盖范围 |
|------|---------|
| `TEST-life-expense.md` | 生活 Hub、记账（手动/AI/多日期/筛选/详情）、边界异常 |
| `TEST-friends-sharing.md` | 好友管理、分享发送/接收、横幅/badge、异常边界 |

### 自动化测试

- 运行全部测试: `cd server && cargo test`
- 运行单个测试: `cd server && cargo test test_name`
- Clippy: `cd server && cargo clippy -- -D warnings`
- 格式化: `cd server && cargo fmt`

## 项目所有者
Boris Huai

---
> Source: [Boris-hbx/ergouWeb](https://github.com/Boris-hbx/ergouWeb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
