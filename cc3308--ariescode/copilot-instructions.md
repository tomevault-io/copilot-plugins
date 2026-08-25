## ariescode

> > 这是给 Claude Code（及其它编码 agent）读的项目上下文文件。

# CLAUDE.md

> 这是给 Claude Code（及其它编码 agent）读的项目上下文文件。
> 本文档已对照实际代码核实并定稿（2026-06-27，编码助手阶段 1–5 完成后）。
> 约定：若日后出现 `【待确认】` 标记，表示尚未核实的内容，**动手前请先读代码核实或向作者确认**。

---

## 1. 项目是什么

**Aries** —— 一个本地优先（local-first）的桌面 AI 助手。所有数据都在用户自己电脑上，不上传任何服务器。

技术栈：

| 层级 | 技术 |
| --- | --- |
| 桌面端 / 界面 | Electron + React |
| 语言 | TypeScript |
| 后端服务 | Node.js + Hono（本地 HTTP 服务） |
| 数据库 | better-sqlite3 `^12.11.1`（原生模块，开启 WAL 模式） |
| 对话模型 | DeepSeek（用户自带 key） |
| 视觉 / 看图 | 硅基流动 / SiliconFlow（用户自带 key） |
| 联网搜索 | Tavily（用户自带 key） |
| 许可证 | MIT |

后端以独立进程运行，桌面端通过它读写数据、调用 AI。

---

## 2. 常用命令

```bash
npm install      # 首次 / 换机器时安装依赖
npm run dev      # 同时启动后端服务 + Electron 界面（开发入口，用 concurrently 并行起两个）
```

下面这些脚本均在根 `package.json` / 各子包里**已确认存在**（除标「无」者）：

```bash
npm run typecheck   # ✅ 类型检查（根脚本：对所有子包跑 tsc --noEmit）
npm run build       # ✅ 构建（先 build server，再 build desktop）
npm test            # ✅ 测试（Node 内置 test runner + tsx，无 vitest/jest）；覆盖 server 的
                    #    sandbox / tools / agent / db（共 30+ 项），聊天界面与多数 desktop 仍未覆盖

# 只起其中一个进程时：
npm run dev:server  # 只起后端（tsx watch，端口 8787）
npm run dev:desktop # 只起前端（vite）

# 命令行编码助手（连同一个后端）：
npm run agent -- status                 # 后端状态 + 工作区路径
npm run agent -- sessions               # 列出会话
npm run agent -- run "<任务>" [-y] [--session <id> --save]   # 跑一个 agent 任务，流式 + 风险确认
```

> **改完代码后的自检要求**：每完成一块改动，至少跑一次 `npm run typecheck` 和 `npm test`；失败就先修好再继续，不要把破坏现有功能的改动留给作者去发现。注意测试覆盖到 server 的安全层/工具/agent/db，但聊天界面与大部分 desktop 仍需靠类型检查 + 手动跑 `npm run dev` 实测。

---

## 3. 目录与文件地图

采用 npm workspaces（monorepo），三个子包：`shared`、`server`、`desktop`。

```
desktop/          @aries/desktop —— Electron + React + Vite 前端（聊天框、侧边栏、设置等所有界面）
server/           server —— 后端服务（与 AI 通信、存聊天记录、读写文件），tsx 起、端口 8787
shared/           @aries/shared —— 前后端共用的 TypeScript 类型（纯类型，很小）
server/data/      🔒 用户私人数据，仅存在本机，绝不上传
server/workspace/ AI 被允许读写文件的专属目录
README.md         使用与安装说明（中文新手向）
LICENSE           MIT
package.json      根脚本与 workspaces 声明
tsconfig.base.json 全项目共享的 TS 编译配置
```

> 注：早期快照里出现过的 `petsona-extension/` 目录**现已不存在**，不再是项目的一部分。

`server/src/` 内关键文件（已对照实际代码核实）：

| 文件 | 职责 |
| --- | --- |
| `index.ts` | 总调度 / 服务入口：Hono 路由 + WebSocket + 加载 `.env` |
| `db.ts` | 数据库读写（better-sqlite3，表：agents / messages / session_titles） |
| `deepseek.ts` | DeepSeek 文字对话 client（流式 + function calling） |
| `vision.ts` | 看图（SiliconFlow Qwen3-VL） |
| `search.ts` | 联网搜索（Tavily） |
| `memory.ts` | 长期记忆（读写 `memory.md`，带 .bak 备份） |
| `personality.ts` | 人格设定（读写 `personality.md`，带 .bak 备份） |
| `prompt.ts` | 拼装系统提示词（记忆 / 人格 / 工具说明等） |
| `runtimeConfig.ts` | 运行时模型配置（UI 值 > .env > 默认值，逐字段回退） |
| `sandbox.ts` | 🔒 安全层：路径守卫（写删限 workspace、防 .. 与符号链接）+ 命令守卫（cwd 锁定、超时、无 key 环境、危险命令拦截） |
| `tools.ts` | 工具层：read_file / write_file / edit_file / delete_file / list_files / grep_files / run_command（+ remember_fact / search_web）；文件操作走 `sandbox.ts`，run_command 由开关 `ARIES_ENABLE_RUN_COMMAND` 控制、默认关 |
| `agent.ts` | 编码 agent 主循环（plan→act→observe→自纠）：安全阀（步数/token/超时）、确认钩子、可读执行轨迹；复用 `streamCompletion`，与聊天循环隔离 |
| `cli.ts` | 命令行入口（server-first）：连同一个后端，`status` / `sessions` / `run`，流式显示 + 风险动作终端确认 |
| `extract.ts` | 从 PDF / docx 抽取文字（mammoth / pdfjs-dist） |

> server 的 WebSocket 有两个：`/ws`（聊天，原有）与 `/agent`（编码助手任务，阶段 4 新增）；两者协议独立、互不影响。

---

## 4. 数据与存储（极其敏感，谨慎对待）

全部位于 `server/data/`，**只在用户本机，删掉该目录 = 清空所有数据**：

- `aries.sqlite`（+ `-wal` / `-shm` 临时文件）—— 聊天记录，逐句保存。
- `memory.md` —— 跨会话生效的长期记忆（关于用户的事实：名字、偏好、在做的项目等）。
- `personality.md` —— AI 的人格设定。

规则：
- **不要删除、清空或重写 `server/data/` 下的任何文件**，除非作者在当次对话里明确要求。
- 涉及数据库 schema 变更时，先说明影响、给出迁移方案，**经作者确认后再执行**，不要直接改表。

---

## 5. 外部服务

所有 key 由用户自带，从配置/环境变量读取，**绝不要硬编码 key，也不要把 key 写进日志或提交进 git**。

- DeepSeek → 文字对话（`DEEPSEEK_API_KEY`，可选 `DEEPSEEK_BASE_URL` / `DEEPSEEK_MODEL`）
- 硅基流动 / SiliconFlow → 视觉（`SILICONFLOW_API_KEY` / `SILICONFLOW_BASE_URL` / `SILICONFLOW_VISION_MODEL`）
- Tavily → 联网搜索（`TAVILY_API_KEY`）

**key / 配置实际怎么读（已核实）**：有两个来源，按字段逐个回退，优先级 **UI 设置 > `server/.env` > 内置默认值**：

1. **`server/.env`**：后端启动时用 Node 原生的 `process.loadEnvFile()` 读取（`index.ts`，包了 try/catch，没有 .env 也能起、只是不能聊天）。可参考 `server/.env.example` 复制成 `.env` 填写。
2. **软件「设置 → 模型与密钥」**：Electron 主进程把 key 用 `safeStorage`（系统钥匙串）**加密**后存到 `<userData>/aries-config.json`（在用户目录，**不在仓库内**），连上后端时经 localhost HTTP 推给 `runtimeConfig.ts`，只存在内存、后端从不落盘也不打印 key。

默认值（用户没配时）：DeepSeek `https://api.deepseek.com` + `deepseek-chat`；SiliconFlow `https://api.siliconflow.cn/v1` + `Qwen/Qwen3-VL-32B-Instruct`。
另有数据路径类环境变量可覆盖默认位置：`ARIES_DB_PATH` / `ARIES_WORKDIR` / `ARIES_MEMORY_PATH` / `ARIES_PERSONALITY_PATH`，端口 `PORT`（默认 8787）。

新增功能需要调用模型时，**复用现有的 client**（`deepseek.ts` / `vision.ts` / `search.ts` 里已有的封装），并通过 `runtimeConfig.ts` 取配置，不要另起一套新的请求逻辑、也不要直接读 `process.env` 绕过回退机制。

---

## 6. 编码规范

（以下已对照 `tsconfig.base.json` 与现有源码核实，请遵循）

- **TypeScript 严格模式**：`strict: true`，且开了 `noUnusedLocals` / `noUnusedParameters`（有未用变量/参数会报错）。避免 `any`，类型尽量显式。
- **ESM 模块**：`module: ESNext` + `moduleResolution: Bundler`，源码里 import **本地文件要带 `.js` 后缀**（如 `from "./runtimeConfig.js"`，即便文件是 `.ts`）。
- **注释风格**：现有文件普遍带较详细的「为什么这么做」文档注释（见 `vision.ts` / `runtimeConfig.ts` 顶部），新代码请保持同等说明密度，而非只写「做了什么」。
- **错误处理**：对外部调用（模型 / 网络）惯例是**不抛异常**，而是返回 `{ ok, text }` 这类结构、把失败转成对用户友好的中文说明（见 `vision.ts` 的 `VisionResult`）。沿用这一模式。
- **面向用户的文案用中文**，代码标识符/注释可中英混用，沿用周边文件习惯。
- 沿用项目已有的命名、文件组织和错误处理方式，不要引入与现有风格冲突的新模式。
- 保持本地优先原则：默认不引入任何会把用户数据外发的依赖或调用。

---

## 7. 硬性规则 / 护栏（最重要）

1. **不要破坏现有聊天功能。** 现有的对话、记忆、人格、看图、搜索是项目核心，任何改动都不能让它们回归。
2. **AI 的文件写入与删除只允许发生在 `server/workspace/` 内**；其余路径默认只读。已由 `sandbox.ts` 的 `assertWritable` / `assertReadable` 强制——新增任何文件操作都必须走这两个守卫，不要绕过自己拼路径判断。
3. **跑命令 / 执行代码必须走 `sandbox.ts` 的 `runGuardedCommand`**（cwd 锁 workspace、超时、无 key 环境、危险命令拦截）；run_command 工具本身还由开关 `ARIES_ENABLE_RUN_COMMAND` 默认关闭，不允许 agent 无约束执行任意命令。
4. **不碰 `server/data/`（用户私人数据）**，除非作者明确点名。
5. **不提交 key、不外发用户数据、不开启任何公网入口**，除非作者明确要求。
6. 破坏性或不可逆操作（删文件、改 schema、批量重构）前，**先说明 + 等确认**。

---

## 8. 命令行编码助手（阶段 1–5 已完成）

目标已达成：一个在终端干活的编码助手 —— 能读代码、改文件、(可选)跑命令、规划多步任务、失败时自我纠正，并与现有聊天/记忆/人格共存。

五个阶段都已实现、各配单元测试、逐阶段 git 存档：

1. ✅ **安全沙盒 / 访问控制** → `sandbox.ts`：写删限 `server/workspace/`（防 .. 与符号链接），命令守卫（cwd 锁定、超时、无 key 环境、危险命令拦截）。`sandbox.test.ts`。
2. ✅ **工具层** → `tools.ts`：read_file / write_file / edit_file / delete_file / list_files / grep_files / run_command，全部走守卫。`tools.test.ts`。
3. ✅ **agent 循环** → `agent.ts`：plan→act→observe→自纠；安全阀 maxSteps=12 / maxTokens≈120k / 超时 5 分钟；可读执行轨迹；复用 `streamCompletion`。`agent.test.ts`。
4. ✅ **CLI 入口** → `cli.ts`：server-first，连同一后端，`status` / `sessions` / `run`，流式显示 + 风险动作终端 y/N/a 确认（`-y` 自动批准）。后端 `/agent` WebSocket 端点。
5. ✅ **接入现有 server**：`/agent` 端点复用 DeepSeek client；可选持久化（`run --session <id> --save`）用现有 schema 把任务/结果写入会话、桌面端可见；默认不写库。回归验证通过（健康、历史会话、人格、记忆、schema 均不受影响）。

关键开关与默认值：
- `ARIES_ENABLE_RUN_COMMAND=1` 才启用 run_command（默认关；关闭时既不广告也不执行）。
- 灰区命令（联网 / 装包，如 npm install / curl / git push）当前一律拒绝，等做了"确认 UI"再放开。
- CLI 的 agent 任务默认**不写数据库**，只有显式 `--save` 才写。

设计参考（**只参考模式，不照搬代码**，历史记录）：CodeWhale（Rust，MIT）的终端 agent 形态；openhanako / HanaAgent（TS + Electron + Hono + better-sqlite3，Apache-2.0）的 server-first 结构。

后续可选（尚未做）：灰区命令的"确认 UI"、OS 级沙盒加固（Seatbelt / bwrap / 受限令牌）、桌面端的「agent 模式」入口、把 `aries` 做成全局 `bin` 命令。

---

## 9. 与 Claude Code 的协作约定

- **先理解，再动手**：被指派新任务时，先读相关代码、给出实现计划和将改动的文件清单，**等作者确认后再写代码**（除非作者说"直接做"）。
- **分阶段交付**：按上面的顺序逐块推进，不要一次性大改。
- **每步自检**：改完跑类型检查 / 测试（若存在），失败自行修复。
- **不确定就问**：遇到本文件标了【待确认】、或代码与描述不符的地方，停下来说明并询问，不要猜着往下做。

---

## 10. 待作者补全 / 确认清单

已核实回填（2026-06-27，逐项对照实际代码）：

- [x] `package.json` 脚本（根 + 三个子包全部核实）：
  - 根：`dev` / `dev:server` / `dev:desktop` / `build` / `typecheck` / `test`
  - server：`dev`(tsx watch) / `build`(tsc) / `start` / `typecheck` / `test`(`node --import tsx --test`)
  - desktop：`dev`(vite) / `build` / `preview` / `typecheck`
  - shared：仅 `typecheck`
  - → 详见第 2 节
- [x] SQLite 具体库：**better-sqlite3 `^12.11.1`**（`server/src/db.ts` 实际 import；WAL 模式） → 见第 1 节
- [x] `petsona-extension/`：**确认已不存在**（磁盘与 git 工作区均无此目录），相关引用已从本文档移除
- [x] 第 6 节编码规范：已据 `tsconfig.base.json` + 现有源码改写为真实风格
- [x] 配置 / key 读取方式：`server/.env`（`process.loadEnvFile`）+ 软件设置（safeStorage 加密存 userData，推给 `runtimeConfig.ts`） → 见第 5 节

仍待作者决定：

- [ ] 测试覆盖：server 的 sandbox / tools / agent / db 已有测试；聊天界面、desktop 端、memory/personality、HTTP 路由等仍无，是否补齐。

---
> Source: [cc3308/Ariescode](https://github.com/cc3308/Ariescode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
