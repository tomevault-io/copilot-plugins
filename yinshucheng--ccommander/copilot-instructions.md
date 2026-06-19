## ccommander

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 这是什么

Commander ⚡ —— AI 多实例实时指挥台,「像皇帝批阅奏章一样」管理多个 Claude Code 会话。机器上正在跑的 Claude Code 会话会自动汇聚成卡片,按「谁在等你」排序供逐条审阅/续话/调度。

## 命令

```bash
pnpm install                          # 用 pnpm，不是 npm

./start.sh                            # 首选：一键拉起（缺 dist 自动构建 + 装 hook + 起服务 + 开浏览器）
./start.sh --port 4000 --no-open      # 开关：--port/--build/--install-hooks/--open 及 --no-* 反向
./start.sh --help                     # 全部开关

pnpm build                            # vite 构建前端到 dist/（server 托管它）
pnpm dev                              # server(3890) + vite(5173) 并行，前端热更
pnpm dev:client                       # 只起 vite 开发服务器

# 也可绕过脚本直接调（start.sh 内部就是调它们）：
node bin/commander.js serve --port 3890   # 起服务（默认端口 3890）
node bin/commander.js install-hooks       # 装全局 Claude Code hook（追加，不覆盖）
node bin/commander.js status              # 终端速查队列
```

无 lint 配置。测试用 Node 内置 runner：`pnpm test`（= `node --test "test/**/*.test.mjs"`，测试文件后缀为 `.test.mjs`）。跑单个测试文件：`node --test test/scheduler.test.mjs`。验证靠：`pnpm test` 通过 + `pnpm build` 通过 + 浏览器实测 + `node -e` 直接 import server 模块跑函数。

### ⚠️ 改后端必须重启 node 进程

server 模块在进程启动时被缓存。**改 `src/server/*.js` 后只 `pnpm build` 不生效**,必须重启:

```bash
pkill -f "commander.js serve"; node bin/commander.js serve --port 3890
```

只改前端则 `pnpm build`（或 `pnpm dev` 热更）即可,无需重启 server。

> ⚠️ **并行开发时别用 `pkill -f`**——它会无差别杀掉所有 worktree 的 server。在 worktree 里改后端,用 `scripts/wt.sh restart`(只按本 worktree 的专属端口定位并重起)。见下「多特性并行开发」。

## 架构大图

> 📌 **文档同步是硬规则**:当你改动了架构、数据流、状态语义,或纠正了本文档里某个不准确的认知(比如「Notification 其实要按 matcher 精筛」这类),**必须在同一次改动里把本文件 / 对应 spec / 相关代码注释一起改齐**,别让文档落后于代码。下游每个 session 都靠这份文档建立认知,文档错一句,后面所有人跟着错。改完问自己:「按这份文档重建认知,会不会被误导?」

两层。**调度内核稳定,内容来源/渲染可插拔**——这条边界是设计基线,见 `specs/000-architecture.md`,新接入来源不得改内核。

### 数据怎么进来（两条源,scanner 兜底 hook）

```
Claude Code hook ──► ~/.commander/events.jsonl ──► events.js（tail 读新行）─┐
                                                                          ├─► upsertFromAgent()
~/.claude/projects/*.jsonl ──► scanner.js（周期扫描，末状态判定）──────────┘   (tasks.js)
```

- **events.js**：消费 hook 写的事件流(精确,waiting/completed ~95%)。默认只读启动后的新行。
- **scanner.js**：扫 `~/.claude/projects` 的会话 jsonl,靠 mtime 静默阈值(180s)+ 末条事件角色判定 waiting/idle(兜底,~70%,漏报为主)。
- **`upsertFromAgent`**（`tasks.js`）是两条源的汇合点：会话 upsert → 自动建/更新「隐式 task」→ 入队。**hook 数据优先级高于 scan**(`LIVE_RANK`),scan 不能把 hook 设的精确态覆盖回近似态。

### 任务 ↔ 会话模型（`tasks.js` + `scheduler.js`）

- 一个 **task** 聚合若干 **session**。会话发现时自动建 `implicit: true` 的隐式 task。
- **排序优先级**（`scheduler.js` `rank()`）：P0 置顶 → **liveState 权重 `waiting > completed > running > idle`**(让等你的先冒出来) → 优先级 P0-P3 → skipCount 降权 → queuedAt 升序。
- 操作:`done`/`skip`(重排到同档末尾)/`defer`(定时 `tickDefer` 到点复活)/`dismiss`(标记会话 `dismissed`,不再被扫描复活,除非来新的 waiting hook)。
- 任何改队列的操作走 `notifyChange()` → 持久化 + ws 广播 `queue_updated`,current 变了再推 `new_current`。

### transcript 渲染（结构化 parts）

- **`transcript.js` `getSessionContext`**：把会话 jsonl 解析成结构化消息,每条 `{seq, role, ts, parts[], text}`。`parts` 的 `kind ∈ text|thinking|tool_use|tool_result|todos`。
- 后端做 **`tool_use.id ↔ tool_result.tool_use_id` 配对**,结果挂在 `tool_use.result` 上,并吸收/丢弃独立的 tool_result 噪音消息。
- 保留顶层 `text`（parts 拼成）作兼容字段,供 LLM 分析 / firstMessage。
- 前端 **`parts.jsx`**：`MessagePart` 按 `kind` 分发到 DiffPart(Edit/Write,jsdiff 行级 diff)/BashPart(命令高亮+折叠输出)/FilePart(Read/Grep/Glob/LS 折叠)/ThinkingPart/TodoPart/GenericToolPart。highlight.js 按需注册 ~13 种语言。
- **`SourceView`**（`TaskCard.jsx`）按 `source.type` 分发：`claude`→结构化渲染；`codex`/`web`→占位(架构预留,见 specs 002/003)。

### 网页续话（`converse.js`）

面板里给某会话发消息 → 在该会话 `workingDir` 下起一个**长驻** `<launcher> --resume <sid> --input-format stream-json --output-format stream-json` 进程(spec 015,取代旧的 `-p` 短命模式),把消息编码成 NDJSON `{type:'user',...}` 写进它 stdin;解析 stream-json 增量经 ws `type:'converse'` 推前端。进程随会话存活(多轮共享上下文)、空闲超时回收。**launcher 从 `cmdTemplate` 派生**(取 `--resume` 之前的前缀):新装默认原生 `claude --dangerously-skip-permissions`,老配置(缺 `cmdTemplate`)沿用旧默认 `ccr code`(`config.js` `LEGACY_CMD_TEMPLATE`)。

**交互式权限审批 / 澄清 / 计划(L1+L2)**:放行与否**派生自 `cmdTemplate`**——含 `--dangerously-skip-permissions` → 全放行,不挂权限工具(实测 skip 模式下 `--permission-prompt-tool` 根本不被调用);**不含 skip** → 挂内置 perm MCP server(`perm-server.js`,独立 stdio 子进程)+ `--permission-mode default --permission-prompt-tool mcp__commander__approve`。Claude 想用工具/反问(`AskUserQuestion`/`ExitPlanMode`)时调 perm 工具 → perm-server 经回环 HTTP(`/internal/permission`,token 校验,绑 127.0.0.1)转交主进程 → `perm-registry.js` 按 `tool_use_id` 挂起 + ws 广播 `permission_request` → 前端弹审批/澄清/计划卡片 → 用户答复走 `POST /api/sessions/:sid/permission` 回灌 → resolve → perm 工具返回 `{behavior,updatedInput?,message?}`。**fail-closed**:缺 tool_use_id / 超时(5min)/ 会话回收一律 deny,绝不静默放行。决定校验是 `permission.js` 的纯函数(可测)。

**进程注入保护(长驻下重定义)**:`running` 态会话,若**我们自己没有持有它的长驻进程**(说明可能是真终端在跑)→ 禁止注入;我们持有的长驻进程不算「别处」,允许继续。并发单飞,5 分钟空闲回收。

> **L3(同步「上朝面奏」:逐字流 + 守着等回复 + 连续即时往返)未做**,但本特性已把长驻进程模型/stdin 回灌/权限通道这些地基铺好,L3 是其上的增量,不返工。见 spec 015 末尾。

### 前端

React + Vite,`App.jsx` 经 `api.js`(含 ws `onConverse`)连后端。`TaskCard` 展示 current 任务 + 关联会话面板 + transcript;`Queue`/`Overview`/`Settings`/`AddTask` 为侧栏视图。

## 状态与配置的存放（关键约束）

- **运行时状态** → 仓库内 `data/{tasks,sessions,history}.json`（`store.js`,原子写 tmp+rename）。**`data/` 已 gitignore**(含本机路径与会话内容,不入库)。
- **用户配置** → `~/.commander/config.json`（`config.js`）。含 `cmdTemplate`(续话命令模板,默认走 ccr)、LLM 分析的 `analyzeApiKey` 等。
- **密钥(SiliconFlow 等)只存 `~/.commander/config.json`,绝不入库、不硬编码、打印时脱敏。** `/api/config` 明文返回 key 仅在 localhost 可接受,若暴露到网络必须改。

## Hook 安装的安全约束

`install-hooks.js` 把 `commander-emit.sh` 装成 Claude Code 全局 hook(Stop→completed / SessionStart→running / SessionEnd→closed)。**追加而非覆盖**,用 `commander-emit.sh` 标记识别自己装的条目以便幂等/卸载。

**`Notification` 必须按 matcher 精筛,不能无 matcher 全吞。** `Notification` 是个大伞事件,有 6 种子类型(`idle_prompt`/`permission_prompt`/`auth_success`/`elicitation_*`)。只有 `idle_prompt`(真·空闲等输入)和 `permission_prompt`(卡在权限审批、等你点允许)才 emit `waiting`,各装一条带 matcher 的 hook;其余子类型不接。**漏 matcher 会把 `permission_prompt`(此时模型还在 mid-turn 跑着)误判成 waiting**,面板显示「等你输入」但实际还在模型调用中——这是修过的坑(`buildHookGroups`/`ensureHook`,`ensureHook` 会自愈清除历史遗留的无 matcher 全吞条目)。

**改 hook 安装逻辑时务必保留用户已有的 hook**——尤其 Red Alert 提示音(`notify-waiting.sh`/`notify-done.sh`)和 vibe-island hook 不能丢。`~/.claude/settings.json` 会先备份(`.commander-bak`)。

## 通过 ccr 时的网络坑

本环境的 web 调用经 ccr(claude-code-router)代理。**`WebSearch` 工具会 400 报错**(`input_schema: Field required`)——改用 tavily MCP 工具(`mcp__tavily__tavily_search` / `tavily_extract`)。

## 开发流程

按改动大小走两条分支。

> 🧭 **新特性默认走「spec → `wt.sh impl` → worktree」,别在 main 工作区直接写实现代码。** 设计期在 main 写 `specs/NNN-<slug>.md`(纯文档不污染),定稿置 `accepted` 后用 `scripts/wt.sh impl NNN` 一键开隔离 worktree 实现。**判据:凡是会落地成代码改动的特性,实现都在 worktree 里做**——在 main 工作区堆多个特性的未提交改动,正是「`git status` 串台、分支散落、不知道谁合了没」这类混乱的根源(本仓库踩过)。只有「单文件 bugfix / 笔误 / 纯文档」这类轻改动才直接在 main 上做。详见下「多特性并行开发」。

> **改了架构或认知,必须同步文档——和代码一起改,别留到「之后」。** 凡是改动会让现有文字变得不准确(架构边界/数据流/状态语义/某个「坑」的结论/默认值/命令),就在同一次改动里把对应文档一并更新:代码里的概念注释、`CLAUDE.md`、相关 `specs/`。判据是「读者照着旧描述会被误导吗?」——会,就得改。本文件就被「`Notification→waiting` 其实要按 matcher 精筛」这种认知漂移坑过,见上文 Hook 安装小节。

### 大特性 / 起点干净 → spec 驱动

动手前先在 `specs/NNN-<slug>.md` 写规格(背景/目标/验收标准/技术方案/任务拆解),定稿确认再实现。`specs/README.md` 是索引 + 工作流;`specs/000-architecture.md` 是不可违反的架构基线;`.plans/` 放实现期临时笔记(可丢)。接到需求时**先读 `specs/README.md`**。

### Bugfix / 小改动 → 观测驱动 + 回归测试

**主线**：复现 → 定位根因 → 在根因层写一个会失败的测试(red) → 修复 → 测试通过(green) → 留存防回归。

硬规则(每次都遵守,治「反复横跳/方向跑偏」):

1. **先观测后动手**：用一条证据(`node -e` 跑函数 / `evaluate_script` 查 DOM / `curl` 打接口)钉死现象,**确认 bug 真的存在**,再改代码。没有观测不动手。
2. **追到根因层**：显式回答「这是症状还是根因?」,沿数据/控制流往上游读。测试要写在**根因层,不是症状层**——否则会「修好测试但没修好 bug」。
   - 反例:markdown 不换行,症状在 `<p>` 标签,根因在三层外 `renderContent` 折叠了 `\n`。该断言 `getSessionContext` 的 `text` 保留 `\n`,而非断言某个 `<p>`。
3. **一次只改一个变量**,改完立即观测。同时改多处 = 修好了不知道是哪处起作用,反复的温床。
4. **验证打在真实运行路径上**：build 成功 ≠ 验证通过;改后端记得 `pkill + 重启 node`(见上文模块缓存坑)。
5. **每个 bugfix 配一条回归断言**,加进 `test/`。

例外(别教条)：复现不出来时,第一优先级是「想办法让它能复现」(加日志/观测),不是硬写测试;一次性笔误等不值得搭测试的,当场手动验一次即可,但「先观测、确认存在、改完真验证」任何情况不能省。

## 多特性并行开发

多个**基本独立**的特性同时推进时,用 `git worktree` 强隔离:每个特性一个独立目录 + 分支 + 端口,代码互不污染、进度互不丢失。完整设计见 `specs/014-parallel-feature-workflow.md`。

### 标准流程:一条特性的一生(spec 驱动 + worktree 隔离)

**默认每个新特性都走这条路,别在 main 工作区直接写实现代码**(那正是「多特性堆在一个 `git status` 里、互相串台」的根源)。

```
设计期:  在 main 直接写 specs/NNN-<slug>.md(纯文档,不碰代码,不污染) → 与用户定稿 → 状态置 accepted
实现期:  scripts/wt.sh impl NNN     # 按 spec 编号一键开实现 worktree:
                                    #   从文件名派生 slug → 建 .worktrees/<slug> + 分支 feat/<slug> + 专属端口
                                    #   + pnpm install + 把 worktree 内那份 spec 置 in-progress
         cd .worktrees/<slug> && 专心实现,与 main 及其他 worktree 互不干扰
合并期:  git rebase origin/main → pnpm test + pnpm build 通过 → 合回 main → spec 置 done
收尾:    scripts/wt.sh rm <slug>     # 移除 worktree(脏则拒绝)
```

**为什么设计期留在 main**:spec 是 `.md` 文档,不碰代码、不会串台,且放 main 才能被所有会话读到、好讨论。真正会互相污染的是**实现代码**,所以 worktree 在 `impl` 时才开。

### 命令速查

```bash
scripts/wt.sh impl 015             # 【首选】按 spec 015 一键开实现 worktree(见上「标准流程」)
scripts/wt.sh new codex-source     # 无 spec 的临时/探索性特性:直接按 slug 开 worktree
scripts/wt.sh list                 # 看所有 worktree:分支 / 分配端口 / 该端口是否在跑
cd .worktrees/<slug>
../../scripts/wt.sh serve           # 在专属端口起服务(缺 dist 自动 build),可与主目录 3890 并存
../../scripts/wt.sh restart         # 改后端后只重启「这个」worktree 的 server(不误杀别的)
../../scripts/wt.sh rm <slug>       # 干完移除(有未提交/未合并改动会拒绝,需 --force)
```

**为什么这套能行(项目特有约束)**:

- `data/{tasks,sessions,history}.json` 是**仓库内相对路径**(`store.js`),每个 worktree 自带独立 `data/`,队列状态天然隔离。
- 端口靠 `COMMANDER_PORT` env 注入(`bin/commander.js` 已支持)。**主目录恒用 3890,worktree 从 3891 递增**,端口号记在各 worktree 的 `.commander-port`(gitignore)。
- `~/.commander/config.json`(续话模板/API key)和 hook 事件源是**全局共享的**——同一个用户、同一套真实 Claude 会话,本就该看到同样的 live 数据,不隔离。

**硬规则**:

1. **改 worktree 的后端,用 `wt.sh restart` 不用 `pkill -f "commander.js serve"`**——后者会杀掉所有 worktree 的 server。
2. **每个 worktree 开工前、合并前各 `git rebase origin/main` 一次**,减小漂移。
3. **合并前在该 worktree 跑 `pnpm test` + `pnpm build` 通过**。
4. **新建 spec 前先 `ls specs/` 看最大编号**——并行时编号易撞(本 spec 就从 012 顺延到 014);撞了往后顺延,别覆盖别人的。
5. 若两特性确实碰了同一核心文件(scheduler/tasks/App.jsx),**先合冲突小的,后者 rebase 吸收前者**。

### 测试

无第三方测试框架,用 Node 内置 runner：`node --test test/`(`package.json` 的 `pnpm test`)。后端纯函数最好测(`node:assert` 直接断言);前端行为用留存的 `evaluate_script` 断言脚本。**只为「修过的 bug」和「核心不变量」写测试**,不追覆盖率——半年下来 `test/` 就是这个项目踩过的坑的可执行清单。

---
> Source: [yinshucheng/ccommander](https://github.com/yinshucheng/ccommander) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
