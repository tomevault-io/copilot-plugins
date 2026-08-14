## berry

> > 灵感来源:Claude Code / Claw Code,做成飞书原生体验。

# Berry

> 接入飞书的编码 Agent 后端。
> 灵感来源:Claude Code / Claw Code,做成飞书原生体验。

---

## ⭐ Superpowers Skills 使用范围(对所有会话生效)

只允许使用以下两个 superpowers skill,其他一律不要 invoke:

- `superpowers:brainstorming` —— 设计前的方案探索与确认
- `superpowers:test-driven-development` —— 实现阶段写单测时使用

不要使用其他 superpowers skill(如 writing-plans / subagent-driven-development / systematic-debugging / verification-before-completion / using-git-worktrees / executing-plans / receiving-code-review / requesting-code-review 等),它们的精神可以体现在普通工作里,不需要走 skill 通道。

**写 plan**:直接在对话里写,或写成 markdown 文档,不 invoke `writing-plans`。
**实施**:在主会话里自己写代码、自己跑测试,需要时手动 spawn 一两个 subagent(用 `Agent` 工具),不 invoke `subagent-driven-development`。
**例外**:用户在新对话里显式要求用某个 skill,以用户当时的指令为准。

---

## ⭐ Git 安全:绝不提交 .env(对所有会话生效)

**千万不要 `git add` 任何带有真实凭证 / API key 的 `.env` 文件**,无论:
- 文件名变种(`.env` `.env.local` `.env.production` `.env.<provider>` 等)
- 路径(根目录、子目录、`config/.env`、`scripts/.env`)
- 看起来「文件很短没几行」

只有 `.env.example`(纯模板、占位值)允许进 git。

**操作规则**:
- 永远不用 `git add -A` / `git add .` 这种通配 add,改用具名 add(`git add path/to/specific.py`)
- 提交前检查 `git status` 输出,看到任何 `.env` 出现在 staged 列表立刻 `git restore --staged <file>`
- 如果一不小心已 commit,立刻告诉用户;不要自己 push,也不要试图用 `git filter-branch` / `git reset --hard` 等破坏性操作"补救"

`.gitignore` 已经把 `.env` 拉黑(`!.env.example` 例外),这条规则是双保险:`.gitignore` 是技术防线,这条是行为防线。

---

## ⭐ Git 操作:只改代码,不做提交(对所有会话生效)

在 berry 项目里,**只改代码,不做任何写入 git 的操作**。提交、推送、合并由用户本人执行。

**禁止的命令**:
- ❌ `git add` / `git commit` / `git push` / `git rebase` / `git reset` / `git tag` / `git merge` / `git checkout -b` 等任何修改仓库或工作树状态的 git 命令
- ❌ 不要主动建议「我帮你提交吧」「我帮你推一下」

**允许的命令**(只读,用来理解当前状态):
- ✅ `git status` / `git diff` / `git log` / `git show` / `git branch -v` 等只读查看类命令

**默认动作**:改完代码就停下,告诉用户「改完了,改了哪些文件、做了什么」,然后让用户自己决定怎么 commit、怎么分组、commit message 怎么写。即便是「顺手」的小改动也不行 —— 用户要保留对仓库历史的完全控制权。

例外:用户在当前对话里**明确**说「你帮我 commit 一下」,才可以执行 commit;即使如此,也要先 `git status` + `git diff` 给用户看,确认无误再提交,且不允许 push。

---

## ⭐ 部署前回归测试(对所有会话生效)

任何代码改动在 push 上线前**必须**先通过回归测试,生产可用性比开发速度优先。

### 回归测试范围(三层都要过)

1. **后端 unit**:`uv run pytest tests/unit -q` —— 必须全过(包括 baseline 已经红的)
2. **后端 integration**:`uv run pytest tests/integration -q` —— 需要本地 PG (`localhost:5432`,user `bbb`,password `berry`),conftest 自动 create/drop 临时库
3. **前端 TS 构建**:`cd web && npm run build` —— `tsc -b && vite build`,把类型错和构建错都拦下

### 强制点

- **GitHub Actions** 是**主拦截层**:每次 push 到 main / 任何 PR 都跑上面三层。CI 红 → main 视为不能上线
- **`scripts/deploy.sh`** 是**温柔提醒层**:在 build 之前跑同样的三层。任一失败 → 默认 abort,提示「跑测试失败,先修」
- **GitHub Actions 设为 required check**:从 GitHub branch protection 启用,push 不绕过(待用户在 GitHub 后台开)

### 热修逃生门

- **`./scripts/deploy.sh --skip-tests`** —— **只在生产已经挂了、必须立刻上**的时候用
- 用了 `--skip-tests` 部署后,**当天**必须把测试补上 / 修绿,下次 push 前不允许再用
- CI 没有逃生门(GitHub Actions 是 required check)。CI 不过 → push 就废,本地 deploy 也别想绕

### 逃生门用法的边界

| 情况 | 用 --skip-tests | 不用 |
|---|---|---|
| 生产 500、用户报障、要立刻补丁 | ✅ | |
| 我自己改了几行,本地懒得跑测试 | | ❌ — 跑测试 |
| 测试基线本来就有红的,跟本次无关 | | ❌ — 先把红的修绿,再 push 本次改动(否则永远修不完) |
| 紧急回滚 | ✅(用 `deploy.sh --rollback`,不走测试) | |

### 落实清单

- `scripts/deploy.sh` 新增 step 0:跑三层测试,任一失败 abort,`--skip-tests` 绕过
- `.github/workflows/test.yml`:push/PR 触发,起 PG 服务跑三层
- 默认要求 baseline 全绿 —— 当前 6 个 feishu approval card 单测红,**第一次部署前必须先修这 6 个或显式 --skip-tests**

---

## ⭐ 实现参考:claw-code(对所有会话生效)

berry 的目标是基本对齐 claw-code 的 agent 设计。**任何 agent 层的实现(工具、prompt、runtime、subagent、cache、history compaction、`<system-reminder>` 注入等)都应优先参考 `reference/claw-code_1/rust/crates/`** —— 不再单独发明 berry 自己的概念体系。

**默认对齐项**(只是常见入口,不是穷尽列表):

- **工具命名 / 形态**:用 claw-code 通用工具(`read_file` / `write_file` / `edit_file` / `bash` / `grep_search` 等),**不为业务场景单独开 `read_note` / `write_md` 这种命名**。业务边界靠 prompt 里的 `# Berry instructions` 表达,不靠工具名。
- **工具描述风格**:一句话 + 必要的「什么时候用」引导,不写多段教学。复杂行为约束放 system prompt 的 `# Doing tasks` / `# Learning together`,不放 tool description。
- **路径安全**:claw-code `path_scope.py` 那一套(`Path.resolve()` + `is_relative_to`),工具调用边界做一次,后续信任。
- **system prompt builder**:已对齐(`berry/skills/learning/prompts/system.md`)。
- **subagent / `<system-reminder>` / cache_break 跟踪 / auto_compaction**:见 `reference/claw-code_1/rust/crates/{tools,api,runtime}/src/`。

**例外**(不对齐 claw-code 的部分):

- **业务编排**(GoalTutor 等 assistants/<name>/)是 berry 自己的,因为 claw-code 是单一编程 agent,berry 是多场景平台
- **DB schema** 是 berry 自己的(claw-code 没数据库)
- **Channel 适配**(飞书等)是 berry 自己的

**红线**:发现自己写「先单独搞一个 berry 风格的 X」时,先去 reference/claw-code_1 看一下他们怎么做的,**沿用是默认值,自己造是例外**(造的话要写 ADR 解释为什么不沿用)。

---

## ⭐ 实现参考:openclaw(飞书相关功能,对所有会话生效)

**飞书 / 卡片 / 多渠道相关功能优先参考 openclaw**,不要先去翻 claw-code(claw-code 是单进程编程 agent,没 channel 层)。

具体范围:`berry/channels/feishu/` 下所有逻辑,以及任何「lark-oapi 调用 / 卡片 schema / card_action 回调 / 审批卡片 / streaming card / reply dispatcher / dedupe / mention gating」 之类话题。

**对齐路径**:`reference/openclaw_1/extensions/feishu/src/`。常见入口:

- `card-ux-approval.ts` —— 审批卡片 schema(orange header / Confirm + Cancel button / interaction envelope)
- `card-action.ts` —— `card.action.trigger` 事件 dispatcher
- `card-interaction.ts` / `card-ux-shared.ts` —— button value 用 envelope 编码,**不要把原始 args 塞 button.value**
- `approval-auth.ts` —— 操作人合法性校验(允许列表 normalize + sender 校验)
- `streaming-card.ts` / `reply-dispatcher.ts` —— streaming card / 回复编排
- `bot.ts` / `bot-content.ts` / `chat.ts` —— 入站消息处理
- `accounts.ts` / `app-registration.ts` —— 多账号 / app 配置

**红线**:写飞书相关代码前,先看 openclaw 对应文件;沿用 schema 命名 / event envelope 形态 / button value 结构 / 操作人校验思路。berry 自己造同款时要写 ADR 解释。

**例外**:openclaw 用 TypeScript + 自家 plugin SDK,berry 是 Python + lark-oapi,语言 / SDK 层面的差异属于「自然差异」,不算违反这条 —— 关键是**业务形态对齐**(卡片长什么样、value 怎么编、操作人怎么校验),不是逐字翻译。

**与 claw-code 规则的关系**:agent 内部(tools / prompt / runtime / subagent / compaction)优先 claw-code;channel 层(飞书等)优先 openclaw。两边不冲突,各自管各自的领域。

---

## ⭐ Berry Learning Loop:产品形态(对 learning assistant 生效)

berry 的 learning assistant 是「目标驱动的学习循环」,基于 `todo_write` 工具追踪进度。

### 核心设计:通用 Todo + Skill 业务特化

**Todo 是通用进度追踪工具,业务逻辑全在 Skill 里。**

- `todo_write` / `todo_read` — 对齐 claw-code TodoWrite,不包含任何业务字段
- `.berry/skills/learning/SKILL.md` — 教 LLM 用 todo 实现学习循环
- `.berry/skills/work/SKILL.md` — 教 LLM 用 todo 实现工作任务
- 新增业务场景 = 新增 Skill,不改 todo 工具

### 5 个阶段(由 Skill prompt 驱动)

```
Phase 1: Discovery        用户表达学习意图(纯对话,不调工具)
Phase 2: Planning         Berry 拆 10-20 项学习计划(纯对话)
Phase 3: Confirmation     用户「确认好了」→ 调 todo_write 落盘
Phase 4: Execution Loop   每项:teach → quiz(≥1简答) → score(≥8分才问OK?) → advance
Phase 5: Completion       全部 completed → 总结 + 复习/进阶/收工
```

### 飞书可视化

每次 `todo_write` 调用 → 飞书群内发一张进度卡片:
```
📋 任务进度 (3/13)
✅ ~~数据结构:SDS 设计与权衡~~
▸ **数据结构:ziplist 演进** ← 进行中
○ 数据结构:quicklist 双层结构
...
```

### 关键设计决策

- **Todo 工具不含业务逻辑** — 不知道什么是「学习」「工作」,只管 `[{content, activeForm, status}]`
- **业务流程靠 Skill prompt** — Learning/Work/Style 各有 SKILL.md
- **Nag reminder** — 连续 N 轮没调 todo_write → runtime 注入 `<reminder>` 提醒
- **verificationNudge** — 全完成且 ≥3 项且无 verif 关键字 → 提示验证
- **数据存储** — `.berry/todos.json`(JSON),飞书通过事件总线实时渲染

### 详细文档

- ADR: `docs/adrs/0007-learning-loop-with-todo.md`
- 旧方案归档: `docs/archive/learning-loop-v1/`

---

## ⭐ 架构设计语言:Modular Monolith(优先级最高,严格遵守)

> **本节是 Berry 的最高架构约束**。任何代码改动、目录调整、依赖添加都必须先通过本节的检查。
> 与本节冲突的写法 **一律拒绝**,即使其他章节、其他文档、其他工具说「这样也行」。
> 升级或修改本节需要在 `docs/adrs/` 留下 ADR,不能直接改本文件。

### 1. 我们用什么、不用什么

| | 用 | 不用 |
|---|---|---|
| **架构风格** | **Modular Monolith** —— 单进程多模块,模块按业务职责切,边界清晰 | ❌ 微服务 / SOA(MVP 没必要) |
| **包组织** | **Package by Feature** —— `feishu/` `agent/` `tools/` `sandbox/` 按业务能力切 | ❌ Package by Layer(`controllers/services/repositories/` 这种按技术分层) |
| **抽象哲学** | **抽象只在「会变的边界」做**(sandbox / channel / LLM provider / RAG client) | ❌ 全套 Clean Architecture(不建 entities/use_cases/adapters/frameworks 四层) |
| **业务建模** | **Ubiquitous Language**(`docs/glossary.md` 一次定终身) | ❌ 全套 DDD(MVP 不画 Bounded Context、不建 Aggregate、不搞 Event Storming) |
| **零件级借用** | ✅ Port + Adapter(给会变的边界加 Protocol)<br>✅ Anti-Corruption Layer(channel 的 mapper)<br>✅ ADR 制度(决策固化在 `docs/adrs/`) | |

### 2. 硬规则(违反则代码不合并)

```
依赖方向(三层模型 + 横切底座):
                  entrypoints  ← 组装层,可 import 任何东西

   channels  ←平级→  gateway        ← 对外触手 + 控制平面
       \              /
        ▼            ▼
                    core                   ← 引擎(agent/llm/tools/db/sandbox/rag)

              ▼ (任何层均可 import,它们不反向 import)

  domain  observability  security  utils  config.py  ← 横切 + 顶层底座

  skills/<name>/ ← markdown 资源,通过 SkillTool 加载,不是「层」(ADR-0008)
```

1. **`core/` 不能 import `channels/`** —— 换 channel 时 core 不动
2. **`core/` 不能 import `gateway/`** —— 主动触发与被动入站都不污染 core
3. **`core/` 不能 import `skills/`** —— core 是引擎,不知道业务(ADR-0008,取代旧 ADR-0003 中关于 assistants 的同名规则)
4. **`channels/` 与 `gateway/` 平级互不 import** —— 共享逻辑下沉到 core/domain
5. **`channels/` 不能 import `skills/`** —— channel 不直接调业务模块(ADR-0008)
6. **`gateway/` 不能 import `skills/`** —— 同上
7. **`skills/<a>/` 不能 import `skills/<b>/`** —— 业务能力之间平级互不依赖
8. **`core/agent/` 不能 import `core/sandbox/{local,docker,e2b}` 实现,只能 import `core.sandbox.base` Protocol** —— 沙箱可替换(软规则:CR 兜底,不进 CI)
9. **`core/tools/` 不能 import `core/agent/`** —— 工具是叶子节点,不反向依赖
10. **`core/db/` 不能 import 任何业务模块** —— 只暴露仓库接口
11. **`domain/` 不能 import 任何 Berry 内部模块** —— 它是最底层,纯模型 / 事件 / 异常

`import-linter` 必须强制执行 1-7 / 9 / 10 / 11(规则 8 是软规则,CR 兜底)。

> **历史**:旧版本的「四层模型」中存在 `assistants/` 业务层(ADR-0003)。
> ADR-0008 取消了该层,业务能力改以 `skills/<name>/SKILL.md` 形式承载,由 SkillTool 按需加载。

### 3. 「会变的边界」必须有 Port + Adapter

只有这些边界做抽象,其他地方直接写,**不要无原则抽象**:

| 边界 | Port(Protocol) | Adapter(实现) |
|---|---|---|
| 沙箱 | `sandbox.base.Sandbox` | `LocalSandbox` / `DockerSandbox` / `E2BSandbox` |
| LLM Provider | `agent.client.LlmClient` | `AnthropicClient` / `OpenAIClient` / ... |
| Channel | `<channel>.base.Channel` | `FeishuChannel` / `TelegramChannel`(未来) |
| RAG | `rag.client.RagClient` | `HttpRagClient` / `InMemoryRagClient`(测试) |

**不该做 Port 的地方**:agent 内部节点、具体工具实现、业务模型 —— 直接写。

### 4. 加新功能时,文件放哪(决策树)

```
新业务能力(learning / work / style / diet / ...)?
  └─ 新建 berry/skills/<name>/SKILL.md(prompt 主体,这就是业务定义)
  └─ 可选:berry/skills/<name>/init_workspace.py(workspace 初始化 helper)
  ※ 业务流程靠 prompt + 通用工具组合表达,不写 Python 业务编排
  ※ 业务专属表(若需要)的 SQLModel 加到 core/db/models.py
  ※ 详见 ADR-0008

新通用工具(任何 skill 都能复用)?
  └─ core/tools/core/ | core/tools/web/ | core/tools/<新分类>/

新主动任务(定时)?
  └─ core/skills/builtin/<name>/(配 manifest.yaml,跟会话型 skill 区分)

新 Channel?
  └─ 新建 berry/channels/<channel>/ 平级 channels/feishu/ / channels/web/,绝不侵入既有 channel

新 LLM Provider?
  └─ core/llm/adapters/ 新加 Adapter,实现 Adapter Protocol

新通用数据表(user/session/message 这种)?
  └─ core/db/models.py + core/db/repos/<name>_repo.py + alembic 迁移

新 channel 内部的渲染/click handler?
  └─ channels/<channel>/<feature>.py(不要跨层放进 skills/)

新 Sandbox 后端?
  └─ core/sandbox/<backend>.py 实现 Sandbox Protocol

横切关注点(日志/指标/trace)?
  └─ observability/

不知道放哪?
  └─ 先放 domain/ 或问 ADR
```

### 5. 通用语言(Ubiquitous Language)

核心名词的定义在 `docs/glossary.md`,**任何代码、文档、commit message 必须使用这套术语**:

- `Session` = 一次 agent 对话的完整生命周期(不是 HTTP / DB session)
- `Task` = 用户的待办事项(不是 LangGraph task_packet)
- `Workspace` = Workbench 的隔离工作区(不是 Git workspace)
- `Approval` = 危险工具调用前的用户确认
- `Skill` = 由 manifest 驱动的高层任务包(不是 LLM 工具)
- `Tool` = LLM 可调用的具体能力(Read/Write/Bash 等)
- `Intent` = Inbox 路由后的消息分类
- `Channel` = 对外的消息通道(目前只有飞书)
- `Assistant` = 业务层一个具体助手(learning / work / style / ...),由 system prompt + 专属工具集 + 专属编排类组成,跑在通用 ConversationRuntime 之上

新增名词必须先加 glossary,再写代码。

### 6. 「现在不做」清单(过度设计警戒线)

下面这些事 **MVP 严禁做**,做之前需要 ADR:

- ❌ 建 `entities/use_cases/adapters/frameworks/` 四层目录
- ❌ 划 Bounded Context、建 Aggregate Root
- ❌ 引依赖注入容器(wireup / dependency-injector)—— FastAPI Depends 够用
- ❌ 引 CQRS / Event Sourcing
- ❌ 在每个边界都加 DTO + Mapper(只在 channel 边界做)
- ❌ 把单进程拆成多个微服务(RAG service 是唯一例外,因为索引重)

### 7. 升级触发条件(什么时候该重新审视本节)

下面任意一条触发,就开 ADR 讨论是否升级架构:

| 信号 | 可能要升级到 |
|---|---|
| `agent/` 文件超过 30 个 | 抽 `use_cases/` 层(Clean Arch 风格) |
| Channel 超过 3 个 | 正式化 DTO + Anti-Corruption Layer |
| 团队 + 第 3 个开发者 | import-linter 严格规则 + ADR 制度强制 |
| 业务概念互相侵蚀(`coding session` 字段污染 `personal session`) | 划 Bounded Context |
| 单进程性能/可用性瓶颈 | 拆服务(按已有目录边界拆,代码不大动) |

**没触发就别动**。架构升级是有成本的,过早升级 = 自残。

### 8. 最高优先级原则

> **当本节与项目其他章节、其他文档、技术选型、个人偏好冲突时,以本节为准。**
>
> 唯一可以覆盖本节的,是用户的明确要求(写在新一轮对话里),并且要在该次会话结束前补一份 ADR 解释为什么破例。

---

## ⭐ 企业级开发规范(优先级仅次于架构设计语言)

> 目标:**可维护、可扩展、可演进**。架构设计语言定边界,这一节定具体写法。
> 与本节冲突的写法需要给出理由(在 PR 描述或 ADR 里说明),**默认拒绝**。

### 1. 配置管理(12-Factor App)

- **运行时配置**:走 `.env` + `pydantic-settings` 的 `Settings` 类,**不要硬编码**(等价 Java 的 application.yml + `@Value`)
- **`.env`** 不入 git;**`.env.example`** 必须维护,任何新加的 env 变量同步加到 example
- **不同环境**(dev / staging / prod)通过 env 变量切换,**不允许写 `if env == 'prod'` 之类分支**
- **敏感信息**(token / password / API key)只走 env 或 secret manager,**绝对不进 git、不进日志、不进异常消息**
- **配置只读**:`Settings` 实例化一次,运行时不可变

### 2. 模块边界与可测试性

- 每个模块只暴露**最小必要的 public 接口**(顶部 `__init__.py` 控制 export)
- **依赖注入优先**:函数 / 类接受依赖作为参数,而不是模块级 import 全局对象。便于测试时 mock
- 业务逻辑**不直接依赖具体框架对象**:不在 service 层 import FastAPI Request,不在 agent 节点 import lark client
- **副作用集中**:DB 写、HTTP 调用、文件 IO 集中在专门的 adapter / repo 层,纯逻辑保持纯函数

### 3. 类型注解 + 文档

- **所有函数签名必须有类型注解**(参数 + 返回值),mypy strict 模式跑通
- 类型注解用 Python 3.12 语法(`list[X]` 而不是 `List[X]`,`X | None` 而不是 `Optional[X]`)
- public 函数 / 类必须有 docstring,**讲清「做什么」「为什么这么做」「调用注意事项」**;私有函数(下划线开头)除非复杂,否则不强制
- docstring 风格统一(Google / numpy 二选一,本项目用 Google 风格)
- 复杂业务函数加**示例代码块**(docstring 里 ```python ... ```)

### 4. 错误处理

- **不静默吞异常**:`except` 必须要么记日志重抛、要么转成业务异常
- 所有自定义异常继承 `berry/domain/errors.py` 里的基类
- 用户输入 / 外部系统调用 / 文件 IO 这些**外部边界**必须 try/except;内部纯函数不写防御性 try
- HTTP 错误响应:统一异常处理器(FastAPI exception_handler),不在每个路由手写 try
- **绝不使用空的 `except:`** 或 `except Exception` 无理由放过
- **catch 的日志必须带 `error_type` + `error` + `exc_info=True`**:health 检查、外部调用这种「失败也要继续」的场景,日志缺了类型名就难以定位(踩过坑:greenlet 缺失被吞成「postgres down」,排查浪费时间)

### 5. 日志(structlog)

- **绝不 `print`**(除非 CLI 工具的 user-facing output)
- `structlog` 必须带 context:`logger.info("user_login", user_id=..., channel=...)`,不要拼字符串
- 日志级别:`DEBUG` 调试细节 / `INFO` 业务关键路径 / `WARNING` 可恢复异常 / `ERROR` 不可恢复
- 敏感字段(token / password / 凭证)**绝不进日志**,统一用 `[REDACTED]` 替代
- 生产环境 JSON 输出,本地可切 console 格式(`LOG_FORMAT=console`)

### 6. 测试

- **每个新功能必须有测试**;测试在 `tests/` 跟 `berry/` 平行
- 单元测试用 fake / mock,**不连真实 DB**(快速 + 隔离)
- 集成测试用真 PG(testcontainers 或 docker-compose),命名 `*_integration_test.py`
- 测试函数命名:`test_<被测函数>_<场景>_<期望>`(例:`test_create_session_when_user_not_exist_raises`)
- **覆盖率不是硬指标**(过度追求会鼓励垃圾测试),**关键路径必须覆盖**:权限决策、Sandbox 边界、外部凭证使用、审批流转
- 测试代码本身也要符合本节规范

### 7. 数据库

- **DB 设计先尽可能简单**:MVP 阶段只建必要的表 + 字段 + FK + UNIQUE 约束,**不主动加索引、不加 CHECK 约束、不加触发器、不分区、不物化视图、不部分索引、不 deferrable 约束**。理由:索引/约束/触发器一旦加进迁移,改起来要写新迁移迁数据,过早优化的代价大于收益。等真出现性能问题或数据完整性事故,再加,加之前要 ADR。**主键唯一索引 / FK 自带索引除外**(那是 PG 自动加的,不算「主动加」)。
- **schema 变更走 alembic**:`uv run alembic revision --autogenerate -m "..."` → review 生成的迁移 → upgrade
- **autogenerate 的迁移必须人工 review**,alembic 不能完美检测所有变化(jsonb default、约束改名、`use_alter=True` 的循环 FK 等)
- **特别注意:循环 FK 必须手写 `op.create_foreign_key`**:autogenerate 把 `use_alter=True` 写进 `ForeignKeyConstraint(...)` 时,SQLAlchemy 编译 `CreateTable` 会**静默丢弃**这条 FK,导致字段建出来但 FK 不存在。修法:在迁移文件里用 `op.create_foreign_key(...)`(放在 `op.create_table` 后面)+ `op.drop_constraint(...)`(在 `downgrade` 反向 drop)。踩过坑,详见 `docs/pitfalls/0007-use-alter-fk-needs-manual-op.md`(如有)。
- **生产迁移可逆**:写 `upgrade()` 同时写 `downgrade()`,即使你认为不会回滚
- 不在应用代码里 `CREATE TABLE` / `ALTER TABLE` —— 只能通过迁移
- 数据修复脚本(data migration)也走 alembic,跟 schema migration 区分文件命名
- **默认值必须两层都加**:Python `default_factory`(ORM 兜底)+ SQL `server_default`(所有写入来源兜底)。只有一层 = 绕过 ORM 的写入会炸(psql / PyCharm / 别语言客户端 / 数据修复脚本)
- **NOT NULL 字段必须有 server_default,或者写入时强制提供** —— 不留空

### 8. 命名

- 模块 / 文件:小写 + 下划线 (`session_repo.py`)
- 类:`PascalCase` (`SessionRepo`)
- 函数 / 变量:小写 + 下划线 (`create_session`)
- 常量:全大写 (`MAX_RETRY = 3`)
- 私有:单下划线开头 (`_internal_helper`)
- 业务名词遵守 [`docs/glossary.md`](./docs/glossary.md)
- **不允许缩写**(`usr` / `msg` / `cfg` 都不行,除非是行业普遍缩写如 `id`)

### 9. 提交规范(Conventional Commits)

```
<type>(<scope>): <subject>

[optional body]
```

- type:`feat` / `fix` / `refactor` / `docs` / `test` / `chore` / `perf` / `style`
- scope:模块名(`agent` / `db` / `feishu` / `tools` 等)
- subject:动词开头,小写,不超过 50 字
- 一次提交只做**一件事**;不要混 feat 和 refactor

### 10. 代码评审(Code Review)

- 所有非紧急修复进 main 前必须 review(自审 / agent 审 / 他人审)
- review 关注:**业务正确性 > 架构合规 > 可读性 > 性能**
- 性能问题除非有数据,否则不 block 合并(YAGNI)
- review 找的问题必须给出**怎么改**,不能只说「不好」

### 11. 文档

- 设计文档放 `docs/`(产品 / 结构 / db schema / glossary 等)
- 决策固化用 ADR(`docs/adrs/NNNN-<topic>.md`),**变更架构必须有 ADR**
- README 永远保持「5 分钟跑起来」可达
- 注释:**讲清「为什么」,不解释「做什么」**(代码本身能讲清楚做什么)

### 12. 依赖管理

- 添加新依赖必须**说明理由**(commit message 或 PR 描述)
- 审查依赖时考虑:维护活跃度、license、安全公告、bundle 大小
- 不引入「为了某一个小特性」的重型依赖
- 锁文件 (`uv.lock`) 必须入 git
- 依赖升级:小版本随手升,大版本走 ADR
- **隐式依赖必须显式声明**:`sqlalchemy[asyncio]`(带 greenlet)、`psycopg[binary]`(免编译) —— 按 extras 名声明,不要赌上游会自动装

### 13. 安全

- **凭证加密落库**(Fernet,master key 走 env)
- 用户输入**永远不信任**,业务边界做校验
- 外部命令执行必经 Sandbox,不直接 `subprocess.run`
- 错误消息不泄露内部路径 / 堆栈到客户端
- 第三方库的安全公告:在 `uv.lock` 升级前查 [GitHub Security Advisories](https://github.com/advisories)

### 14. 性能

- **先写对,再写快**。MVP 不预优化
- 真出现慢:先 profile(`cProfile` / `py-spy`),再优化
- DB 查询:N+1 问题第一时间消除(SQLAlchemy `selectinload`)
- 缓存:不预加(过早缓存 = bug 工厂),真热点再加,且必须有失效策略

### 15. 可观测性(Day 1 就要)

- 关键业务路径有 metric(Prometheus,V1 接入)
- LLM 调用有 trace(Langfuse,V1 接入)
- 错误有上下文(structlog 带 session_id / user_id 等业务标识)
- **能从日志定位问题** = 可观测性合格

---

## 产品定位

- **形态**:后端服务 + 飞书机器人(无 Web 前端、无 IDE 插件)
- **入口**:飞书单聊 / 群聊 @机器人
- **核心交互**:
  - 用户在飞书提需求 → 流式回复
  - 危险操作(Bash / Write)用飞书卡片按钮做审批
  - 跨设备共享会话(电脑发,手机看)
- **目标**:**高并发 + 高可用**,production 级

---

## 技术选型(已定)

### Web 框架与协议层
- **FastAPI + uvicorn** —— ASGI 异步原生,LLM 流式 + WebSocket 长连刚需
- **飞书官方 SDK(lark-oapi)** —— WebSocket 长连模式,免公网回调;支持卡片交互、流式更新、文件上传

### LLM Provider
- **各家模型官方 SDK** —— `anthropic` / `openai` / `deepseek` 等,**直接对接**
- **不引 LiteLLM** —— 多一层抽象,新模型支持滞后
- **不引 LangChain `init_chat_model`** —— 抽象重

### Agent 编排
- **LangGraph** —— 为「高并发 + 高可用」目标选择
  - `Checkpointer` 挂 PG → agent 状态持久化,节点挂了能恢复
  - `interrupt()` 机制 → 飞书审批卡片 human-in-the-loop 天然适配
  - 生产级用户:Klarna(客服 multi-agent)、Replit(Ghostwriter)

### 数据持久化
- **PostgreSQL**(主存储)
  - 会话 / 消息 / 工具调用历史(JSONB 字段)
  - 审批状态 / 任务状态
  - 用户配置 / API key(加密)
  - LangGraph checkpointer 的后端
- **Redis**(V1 接入,MVP 不做)
  - 分布式锁 / 限流 / 任务队列 / session 路由

### 工具沙箱
- **MVP:暂用 subprocess + cwd 隔离**
- **V1+:已记入待办(Task #1)** —— 推荐路径:Docker per session → e2b.dev → 自建 Firecracker
- **关键约束**:第 1 周就把 `sandbox/` 抽象成 Protocol,业务层不感知底层实现

### 可观测性
- **Langfuse** —— LLM 专用 trace(每轮 token / latency / tool 调用链路)
- **Grafana + Prometheus** —— 系统级指标(QPS / 延迟 / 资源)
- **Sentry**(可选)—— 错误监控
- **结构化日志**:`structlog` JSON 格式

### 部署
- **MVP**:Docker Compose(FastAPI + Postgres + Redis + Langfuse)
- **V2+**:K8s + Helm(沙箱用 Job / Pod-per-session)

---

## 有意不引的库

- ❌ LangChain Core —— 抽象重
- ❌ LiteLLM —— 多模型时直接加 SDK 客户端
- ❌ Django / Flask —— 同步框架,流式弱
- ❌ Celery / RQ —— asyncio + LangGraph 已够
- ❌ Pydantic v1 —— 全部用 v2
- ❌ requests —— 用 httpx

---

## 项目结构

完整目录树、模块职责、依赖规则、决策树见:**[`docs/berry-project-structure.md`](./docs/berry-project-structure.md)**

产品定位、场景、架构原理见:**[`docs/berry-product-design.md`](./docs/berry-product-design.md)**

> **单一真相源**:本 CLAUDE.md 只保留约束 / 选型 / 路线图;具体目录树以 `docs/berry-project-structure.md` 为准。两处冲突时以 docs/ 下文档为准并同步更新。

---

## 阶段路线图

### MVP(3 周)
| 周 | 目标 |
|---|---|
| 第 1 周 | lark-oapi WebSocket 通 + Anthropic 流式 + 卡片流式更新 + sandbox Protocol 占位 |
| 第 2 周 | 5 个工具(Read/Write/Edit/Bash/Grep)+ 审批卡片 + Postgres session |
| 第 3 周 | LangGraph checkpointer + 多用户隔离 + 错误处理 + 上线 |

### V1(MVP+1 个月)
- [ ] 上下文压缩(防 token 爆)
- [ ] Cost / Usage 跟踪 + 配额限制
- [ ] Todo Tracking 工具(进度可视化)
- [ ] Web Search / Web Fetch 工具
- [ ] Langfuse 接入
- [ ] Redis 分布式锁 / 限流
- [ ] Docker per session 沙箱

### V2(V1+2 个月)
- [ ] 多 Provider(GPT / DeepSeek / Kimi)
- [ ] Sub-Agent(主 agent 派生子 agent)
- [ ] 项目记忆(类似 CLAUDE.md)
- [ ] Skills 系统(预定义能力包)
- [ ] 文件上传(飞书发文件 → agent 处理)
- [ ] 沙箱切到 e2b.dev

### V3(看反馈)
- [ ] MCP 客户端(接 GitHub / Linear / Notion)
- [ ] 团队空间(多人共享 agent 工作区)
- [ ] Web 前端(管理后台)
- [ ] K8s 化 + 自建 Firecracker 沙箱

---

## 待办事项(已记)
- **Task #1** —— 设计并接入 Tool 执行沙箱(详见 task 描述)

---

## 决策原则

> 架构层面的硬约束已写在 **「⭐ 架构设计语言」** 章节(本文件顶部)。本节只列产品 / 工程层面的软原则。

1. **不要追求一次到位** —— 按 dogfood 反馈决定下一步加什么
2. **每周自己用** —— 用 berry 修自己的代码 / 干自己的事,真实使用决定优先级
3. **生产级目标** —— 高并发、高可用从 day 1 就要考虑(LangGraph checkpointer、PG、Langfuse)
4. **过度设计警戒** —— 见架构章节 §6「现在不做」清单

---

## 参考

- 飞书文档「Claw-Feishu 技术选型方案」https://www.feishu.cn/docx/IPBOdCaq9oujbux7M3tcNTVYn6c
- 灵感:[Claw Code](https://github.com/ultraworkers/claw-code) / Claude Code
- LangGraph:https://github.com/langchain-ai/langgraph
- lark-oapi:https://github.com/larksuite/oapi-sdk-python
- Langfuse:https://github.com/langfuse/langfuse

---
> Source: [DdGnoybab/berry](https://github.com/DdGnoybab/berry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
