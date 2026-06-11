## nexus-harness

> > 三端通用入口（Codex / Claude Code / Cursor）。所有 AI Agent 进入本仓库后，**第一件事是读这个文件**，然后按指引找 skill。

# nexus-harness · Agent 入口

> 三端通用入口（Codex / Claude Code / Cursor）。所有 AI Agent 进入本仓库后，**第一件事是读这个文件**，然后按指引找 skill。

---

## 启动入口引导

首次进入本仓库时，先判断用户首条输入是否以 `开发者模式` 或 `启动开发者模式` 开头。若命中，视为当前会话在维护 nexus-harness 框架本身：不展示 `[agent.md](./agent.md)` 启动入口，不做 `1 / 2 / 3 / 4` 路由，直接回答用户在该输入中的问题或按其维护目标处理。

**命中 `开发者模式` 时第一件事**：在你给用户的**首条**回复中包含一行 HTML 注释 `<!-- NEXUS:DEV_MODE -->`（用户看不见，但 `.cursor/hooks/agent_response_audit.py` 会捕获并写入 `.nexus/conversation/<conversation_id>/dev_mode`，让本 Cursor 实例的 stop / subagent / audit 钩子全部短路放行）。这个标记是**per-conversation** 的，只影响本 Cursor 实例，不会污染同时打开的其他 Cursor 实例。

新协议（per-conversation）和旧协议（仓库级 sentinel）共存：

- **新协议优先**：宿主 stdin 携带 `conversation_id` 时，stop hook 用 `.nexus/conversation/<id>/` 子目录的 `dev_mode` 标记 + `plan_dir` 记录决定是否干预；没记录的会话一律 ALLOW。
- **旧协议作 fallback**：宿主无 `conversation_id` 时，仍读 `.nexus/hook_paused` 全局 sentinel；当下游业务 active session 出现"被另一个 Cursor 维护会话误激活"时也可手动 `mkdir -p .nexus && touch .nexus/hook_paused` 紧急压制（重启会话即恢复）。
- 两份状态都不入 git；`.nexus/` 已在 `.gitignore`。

stop hook 引入 per-conversation `dev_mode` 标记 + 全局 `DEVELOPER_SESSION.json` 解析：默认按 workspace 中最近活跃的 `DEVELOPER_SESSION.json` 决定要不要拦；当前会话若有 `<!-- NEXUS:DEV_MODE -->` 标记（写到 `.nexus/conversation/<id>/dev_mode`），stop hook 直接放行——这是同时开两个 Cursor 实例时维护会话不被拉的唯一隔离手段。

未命中开发者模式时，展示 `[agent.md](./agent.md)` 的启动入口内容：`nexus harness` 字符横幅和 4 个一级选项。

非 CLI 宿主（如 Cursor 桌面版、VS Code AI 对话框、JetBrains/IDEA AI 对话框）无法像 Claude Code CLI 一样在空白启动页自动显示公告。因此在这些宿主里，**首次收到用户任何输入时**，除非命中开发者模式，否则必须先展示 `agent.md` 启动入口，再处理用户意图；如果用户输入或明确提到 `nexus` / `Agent` / `启动入口`，也必须重新展示该入口。

非 CLI 宿主展示入口时，只原样展示 `agent.md` 中的启动页代码块；禁止在代码块下方重复列出 `1 / 2 / 3 / 4`，禁止追加第二段“你想做什么”。`agent.md` 已经包含“下次输入 nexus 或 Agent 可回到此入口”的提示，不需要额外补充。

用户回复编号或自然语言意图后，按以下规则路由：

- `**下一步` / `继续` / `接下来` / `不知道怎么走` / `往下走` / `卡住了` / `然后呢` / `迷失了` 等恢复意图** →
 运行 `python3 scripts/what_next.py`（**会话隔离已内建**：脚本自动按 `.nexus/conversation/<id>/{plan_dir,workspace_dir}` 绑定锁定本会话项目，多 Cursor 实例并发时不会被另一个实例的 active session 吸走；conv_id 由 `.cursor/hooks/shell_plan_dir_bind.py` 通过 sidecar 自动注入，AI 无需手动传 `--conversation-id`）。按返回结果处理：
  - `mode=single_project`：
    - **若返回字段 `auto_advance=true`**（高置信单项目 + 当前 `next_required_action` 不属于 `user_review_gate / none / pause / handle_blocker`）：直接展示 `user_message`（一行进度）+ **立刻**按 `agent_instruction` 执行，**不等用户回复**。后续由对应 skill（典型为 Developer 的 `developer_autopilot.py`）接管。Cursor / Claude Code 若已挂 `.cursor/hooks.json` / `.claude/settings.json` 的 stop hook，Agent 试图擅自停顿时会被自动拽回循环；hook 不可用时仍以本规则为准。
    - **否则（`auto_advance=false`）**：把 `user_message` 原样展示给用户，等用户选 1/2/3；
    用户选 `1` → 按 `agent_instruction` 直接进入对应 skill，**禁止再走 `transition_confirmation` 或重新展示项目列表**；
    用户选 `2`/`3` → 按字面意思处理。
  - `mode=multi_project`：把 `user_message` 原样展示给用户，等用户选项目编号；
  用户选编号 N → 运行 `python3 scripts/what_next.py --select N`，得到 `mode=single_project` 后按上面规则处理，**禁止再走 `transition_confirmation`**。
  - `mode=no_projects`：把 `user_message` 原样展示给用户，等用户选 1/2/3。
  - **禁止把 `what_next.py` 的输出再走 `transition_confirmation` 或 `discover_workspace_stage.py`**。
- ##### `1` / 开始新项目 / 新需求 / 我要开始 → 读取 `skills/project-start/SKILL.md`，进入 `project-start`。
- `2` / 继续项目 / 恢复项目 / 继续开发 → 先运行 `python3 scripts/discover_workspace_stage.py` 做 workspace 阶段识别；该脚本会静默把超过 30 天未更新的进行中 session 标记为 `stale`（不删除内容），再按识别结果进入对应 skill。
- `3` / 提取 AI 知识库 / 构建 docs / 更新 docs / build aiweave / sync aiweave → **立即** Read `skills/docs-build/SKILL.md`，由该 skill 接管后续全部交互。**禁止**先列 `skills/` 子目录做 skill 发现；旧版 `knowledge-extract` / `knowledge-extract-reviewer` / `knowledge-qa-reviewer` 已物理下沉到 `skills/_deprecated/`，**仅作 git 历史留档**，任何场景都不得读取或调用其中文件、脚本。收到 3 / 提取 AI 知识库意图后，下一个动作只能是 Read `skills/docs-build/SKILL.md`，无任何中间步骤。
- `4` / 归档项目 / 项目归档 / 项目收尾 / 项目完结 / finish project → 读取 `skills/finish-project/SKILL.md`，进入 `finish-project`。
- `5` / 问题定位 / 联调 / 排障 / 自测发现 / 联调发现 / 线上问题 / 线上故障 / diagnose → 读取 `skills/diagnose/SKILL.md`，进入 `diagnose`。
- `6` / 查看使用说明 / 打开文档 / 文档站 / 帮助 / docs / help → **宿主区分**：（**A**）**Cursor 桌面版 / Cursor Agent 聊天 / 其它无法在子 shell 可靠调用系统 `open` 的环境**：**禁止**执行 `bash .claude/hooks/start-docs.sh`。用工作区仓库根拼出 `site/public/index.html` 绝对路径，回复中**最多两行**：① 可点击的 Markdown 链接 `[file://绝对路径](file://绝对路径)`（**禁止**粗体包裹 URL）；② 一句「若无法点开，请复制 file:// 地址到浏览器地址栏」。**不要**把脚本整段输出粘贴进聊天。**（B）Claude Code CLI 等本机终端环境**：运行 `bash .claude/hooks/start-docs.sh`。脚本只做以 `file://` 打开本地 `index.html`，零运行时、无后台进程；stdout 原样展示。脚本退出后再用 Markdown 链接格式输出其打印的 `URL` 行。**禁止**用粗体包裹 URL。回到入口等待下一步，**不要追加**推荐下游 skill。

入口只负责一级选择；一旦命中具体 skill，必须立即读取该 skill 的 `SKILL.md` 并完全交给该 skill 接管输出格式、ASK 节点和后续状态机。入口层禁止二次包装下游结果，禁止自行模拟下游菜单，禁止把入口判断摘要和下游 skill 的项目选择/任务确认混在同一段里。

**全局返回入口（覆盖所有 skill 的 ASK / 等待输入）**：无论当前处在哪一个 skill 的状态机、是否在等待编号 / 是 / 否 / 确认文案，只要用户本条输入**仅为** `nexus` 或 `Agent`（单独一行；大小写不敏感；无其它实质指令），主 Agent 必须**立即退出当前 skill 的追问语境**，按 `.cursor/rules/agentflow.mdc` 与上文「非 CLI 宿主」规则**重新展示 1–6 启动入口**。**禁止**把 `nexus` / `Agent` 当作「无效选项 / 请重新回复 1」处理；**禁止**把该字符串当作 `selection` / `confirmed` 等参数继续调用当前 skill 的 wizard。用户在新入口重新选 `1`–`6`（或自然语言映射）后，再按路由进入对应 skill。

特别是选择 `2` 时：

- 入口层只允许运行 `scripts/discover_workspace_stage.py` 做轻量阶段识别与 session 过期静默标记；不要手写扫描逻辑、不要自行推断阶段。
- 如果脚本识别到 `stage=project_start_in_progress` / `route_skill=project-start`，立即读取 `skills/project-start/SKILL.md`，用脚本返回的 `session_path` 恢复 project-start。
- 如果脚本识别到 `stage=finish_ready` / `route_skill=finish-project`，说明已有 Plan/STATUS 且 finish 审计判断任务、TDD、阻塞项已满足收尾条件；先检查该项目 `transition_confirmation.required`，若为 true，必须先展示 `transition_confirmation.user_message` 等用户选 `1`；用户确认后读取 `skills/finish-project/SKILL.md`，把已选项目的 `workspace_path` 明确传给 finish-project。
- 如果脚本识别到 `stage=prd_review_required` / `route_skill=project-start`，说明已有 PRD 但 `prd_ready_for_trd=false` 或 PRD reviewer 报告仍有 blocking/major；读取 `skills/project-start/SKILL.md`，默认按 reviewer 报告自动修复 PRD。只有遇到业务取舍、结构性大改或用户明确要求跳过时，才停下问用户；不得直接进入 `prd-to-trd`。
- 如果脚本识别到 `stage=trd_review_required` / `route_skill=prd-to-trd`，说明已有 TRD 但 `trd_ready_for_plan=false` 或 TRD reviewer/hardcheck 报告仍有 blocking/major；读取 `skills/prd-to-trd/SKILL.md`，默认按 reviewer/hardcheck 报告自动定点修复 TRD。**修完运行 `lint_trd.py` 只是中间自检，不是终点；lint 通过后必须继续派 `prd-to-trd-reviewer` 或调用 `auto_converge` 并回流报告，直到状态机返回 `done` / `accepted_with_risk` / `unresolved_after_auto_rounds` / 其它 ask_user 节点。禁止对用户说“建议你下一步再派 reviewer / 或跑 auto_converge”后停住。**只有遇到业务取舍、结构性大改或用户明确回复「我了解风险，接受当前 TRD，继续生成 Plan」时，才允许跳过修复进入 `plan-from-trd`。
- 如果脚本识别到 `stage=plan_review_required` / `route_skill=plan-from-trd`，说明已有 Plan/STATUS 但 `plan_ready_for_development=false` 或 Plan reviewer/preflight 仍有 blocking/major；读取 `skills/plan-from-trd/SKILL.md`，默认按 reviewer/preflight 报告自动修复 Plan。只有遇到业务取舍、结构性大改或用户明确要求跳过时，才停下问用户；不得直接进入 developer。
- 如果脚本识别到 `stage=development_ready` / `route_skill=developer`，先检查该项目 `transition_confirmation.required`；若为 true，必须先展示 `transition_confirmation.user_message` 等用户选 `1`；用户确认后读取 `skills/developer/SKILL.md`，**必须把已选项目的 `plan_dir` 明确传给 developer**（即在交接语里注明 `plan_dir=<脚本返回的 plan_dir 值>`）；developer Step 1 收到 plan_dir 时直接跳到 Step 2，**禁止重新运行 `discover_tasks.py` 展示项目列表**。
- 如果脚本识别到 `stage=plan_required` / `route_skill=plan-from-trd`，先检查该项目 `transition_confirmation.required`；若为 true，必须先展示 `transition_confirmation.user_message` 等用户选 `1`，再读取 `skills/plan-from-trd/SKILL.md`，并把对应 `context_path` 作为 `project_context_path` 进入 plan 生成流程。
- 如果脚本识别到 `stage=trd_required` / `route_skill=prd-to-trd`，先检查该项目 `transition_confirmation.required`；若为 true，必须先展示 `transition_confirmation.user_message` 等用户选 `1`，再读取 `skills/prd-to-trd/SKILL.md`，按该 skill 的输入确认流程继续。
- **阶段切换确认**：用户选中项目后，若该项目 `transition_confirmation.required=true`，入口层必须先逐字展示 `transition_confirmation.user_message` 并停下等用户选择；用户回复 `1` 才进入对应下游 skill，回复 `2` 则暂停并提示可重新选择继续项目，回复 `3` 或自然语言则按用户新意图处理。禁止在用户刚选项目后直接调用下游 skill 抛出缓存、数据库、Plan 拆分或开发任务问题。
- **通用产物流转门禁**：PRD → TRD、TRD → Plan、Plan → Developer 三段都必须先看 readiness。优先读取 `context.json` 中的 `prd_ready_for_trd` / `trd_ready_for_plan` / `plan_ready_for_development`；若没有显式字段，再读取相邻 `*.review_report.json` / `*.semantic_review_report.json` / `*.hard_check_report.json` / `*.hardcheck.json` / `preflight_report.json`。显式 `false` 或报告含 blocking/major/fail 时，默认动作是回到对应 skill 自动修复；禁止把“修复”包装成用户必须手改的二选一。用户只有在明确接受风险跳过、或问题涉及业务拍板/结构性大改时才需要回复。
- 如果有多个可继续项目，入口层只展示 `scripts/discover_workspace_stage.py` 返回的 `presentation.user_message` 原文，让用户选一个；选中后若有 `transition_confirmation`，先走阶段切换确认，不要立刻交给下游 skill。`presentation.user_message` 是 Markdown 正文，必须直接输出，禁止包进 `text /` markdown / 引用块。禁止追加“运行脚本 / 已识别到 / 进入对应流程 / 正在运行 / 选定后我会 / 我将按 AGENTS.md / 加载某 skill / 把 context_path 设为...”等 Agent 内部动作说明，禁止把下游 skill 的项目选择/任务确认混在入口层。
- `developer/scripts/discover_tasks.py` 的任务列表、任务确认、分支 Gate、RED/GREEN/REFACTOR 提示，仍全部由 `developer` 规则接管；入口层不得越权模拟。

特别是选择 `5` 时：

读取 `skills/diagnose/SKILL.md`，由该 skill 的状态机接管后续全部交互（ASK 档位 → 扫盘 → 候选渲染 → 装载前体检 → ContextPack 装载 → 交接语 → 排障对话铁律）。入口层只负责触发，不再内联协议。

---

## 这个仓库是什么

AI Agent 研发工作流框架。把「PRD → 苏格拉底追问 → TRD → Plan → TDD 驱动开发 → 多维 Review → AI 知识库回写 → 归档」全流程 **skill 化**。

- **纯 skill 化**：不依赖任何 MCP，克隆即用
- **三端通用**：Codex / Claude Code / Cursor 任一都能直接加载
- **AI 知识库长期沉淀**：`knowledge/` 是活文档，跨项目持续生长

---

## 仓库目录速览

```
nexus-harness/
├── AGENTS.md                       # 本文件（Agent 入口）
├── README.md                       # 用户入口
├── agent.md                        # nexus / Cursor 启动入口横幅
├── nexus_harness/                  # CLI 实现（`nexus` 命令、auto-init、cheatsheet 等）
├── bin/nexus                       # 启动器（symlink 到 ~/.local/bin/nexus 即可全局用）
├── skills/                         # 所有 skill（每个 skill 一个子目录，含 SKILL.md + 脚本 + 模板）
│   └── aiweave-bridge/vendor/      # ⚠️ 只读 vendor：AIWeave 上游规范镜像（铁律 7.5）
├── workspace/                      # 进行中项目草稿（PRD/TRD/Plan/评审报告等，一次性产物）
└── archive/                        # 已完结项目归档（由 finish-project skill 从 workspace 迁入，只读）
```

> 注：旧版 `knowledge/` 目录在 AIWeave 整合后被取代，长期活文档由业务项目自身 `docs/` 承担（铁律 7.6）。`skills/knowledge-*` 三个 skill 已封存，菜单不再展示。

## 产物归档约定（一次性产物 vs 长期活文档）


| 产物                                                                                            | 位置（开发中）                      | 归档位置                                      | 备注                                                              |
| --------------------------------------------------------------------------------------------- | ---------------------------- | ----------------------------------------- | --------------------------------------------------------------- |
| `prd.md` / `trd-qa.md`                                                                        | `workspace/YYYY-MM-DD-<项目>/` | `archive/YYYY-MM-DD-<项目>/`                | finish-project 统一 mv                                            |
| `trd.md` / `api-contract.md`                                                                  | 同上                           | 同上                                        | 同上                                                              |
| `plans/{backend,frontend}/`                                                                   | 同上                           | 同上                                        | 同上                                                              |
| `context.json`                                                                                | 同上                           | 同上                                        | 项目级上下文，记录 PRD/TRD/Plan/业务代码根目录，供下游 skill 和后续会话读取                |
| `***.review_report.json**`（reviewer 评审报告）                                                     | 产生在 TRD 旁                    | `archive/YYYY-MM-DD-<项目>/review-history/` | **不入 git**（.gitignore 已排除）；finish-project 搬到 review-history/ 留痕 |
| `***.hard_check_report.json`**（硬质检报告）                                                         | 同上                           | 同上                                        | 同上                                                              |
| KB（`knowledge/<topic>/<module>/<name>-knowledge-base/00-index.md` ~ `06-code-conventions.md`） | 仓库根 `knowledge/`             | 永久保留                                      | 长期活文档，finish-project 只写回确认项，不移动 KB 目录                           |


---

## Agent 工作铁律

1. **找 skill 再动手**：动手前先扫 `skills/` 下各 SKILL.md 顶部的 `description` 与 `when_to_use`，匹配到合适 skill 才执行。
2. **逢 ASK 必停**：每个 skill 内标注的「向用户提问」节点，必须真的把问题抛出来等用户回，禁止替用户填默认值。
3. **skill 链式衔接**：一个 skill 完成且下一步是明确的下一个 skill 时，主动问用户「是否继续走 X」，用户点头后**自动调起**下一个 skill，不要求用户手动输提示词。
4. **AI 知识库改动走规范**：写盘前必跑 skill 自带的 preflight 脚本，frontmatter 必填项缺一不写盘。
5. **结构化产出必须「索引 + 章节文件」**：KB / TRD / Plan 等长期/复杂产出**禁止单文件**，采用 `INDEX.md + <NN>-<slug>.md` 或各 skill 规定的固定文件结构；PRD / 苏格拉底追问等过程性产出超过 200 行时同样拆分。AI 读取时**先读 INDEX.md 再按需读单章节**，禁止一次性吞整目录。
6. **不擅自跑测试用例 / 不做 git 写操作**：仅在用户明确要求时执行。全仓所有 skill 还必须遵守 Git 安全铁律：**禁止执行或建议自动执行 `git push`**；**禁止自动 merge `dev` / `master` / `main` 分支**（无论作为 merge 源还是 merge 目标）。如确需发布、推送、合入受保护分支，只能停止并让用户人工处理或在独立确认后由用户自行执行。
7. **禁止在"使用 skill"过程中修改 skill / 框架自身**（铁律，下称 **框架不可变铁律**）：
  - 本仓库（`skills/`**、`scripts/**`、`nexus_harness/**`、`AGENTS.md` 及一切非 `knowledge/` 的框架文件）是**给用户用的产品**，不是当前任务的可改素材。
  - 当你在执行某个 skill 的工作流（生成 TRD / 提炼 KB / 走 project-start 等）时，**只能**读取 skill 脚本/模板、并向用户业务产出目标目录写盘；**不得**编辑、新增、删除 skill 本身的任何文件（含 `SKILL.md`、`scripts/`、`templates/`、`tests/`、YAML 配置等）。
  - 如发现 skill 有 bug / 参数漏传 / 模板缺漏 / 质检误判等问题：**立刻停下当前 skill 工作流**，向用户明确报告「skill X 的 Y 处有问题，建议在独立会话中以『维护 nexus-harness 框架』为目标修复」，由用户决定是否开新会话修。绝不能一边用一边改。
  - 唯一例外：用户在**当前会话**明确说「现在就是来改这个 skill / 框架的」，此时才允许写框架文件；仍然禁止在"正跑某个业务 skill 流程"的同一轮里夹带框架修改。
  - 适用于**所有现存 skill 和未来新增 skill**，新 skill 接入时 reviewer 必须对照本条验收。
- **7.5 vendor/aiweave 不可改铁律**（AIWeave 整合后新增）：
  - `skills/aiweave-bridge/vendor/aiweave/**` 是 AIWeave 上游规范的整目录镜像（commit/sync 信息见 `vendor/README.md`）。
  - 任何 nexus-harness 的 skill / reviewer / CI / 主 Agent **不得**修改、删除、覆盖 vendor 内任何文件；升级方式只能是「整目录覆盖」，由维护者在「开发者模式」会话里执行。
  - 所有 bridge 脚本（`detect.py / compile_context.py / bootstrap.py / invoke_skill.py / update_build_status.py`）只读 vendor、写盘业务项目；写盘前必先经 `assert_write_target.py` 校验。
  - 违反此条 = AIWeave 跨项目可分发性立即失守。
- **7.7 AIWeave 硬 gate 不可绕过铁律**（v0.x · S3 配套）：
  - **Gate A · 进入 RED 必须先证明读了 docs**：当业务项目 `aiweave_status ∈ {T1, T2}` 且任务 `aiweave_docs_refs` 非空时，`developer_tdd_gate.py enter_red` 会强制要求 `<plan_dir>/<task_id>.aiweave_docs_evidence.json` 已覆盖每条 ref。该 JSON 的 `key_tokens` 字段会被 `record_docs_read.py` 反向 grep 到对应 docs 章节，token 不存在则拒登记，杜绝 AI 凭印象虚填证据。
  - **Gate B · `verify_refactor` 必须先翻 BUILD_STATUS**：T1/T2 任务必须先调 `update_build_status.py` 把 `build_status_module_path` 对应模块翻到 `build_status_target`（默认 🟢），否则 `verify_refactor` 拒绝通过；旧 plan 缺该字段时降级为 warning（plan-from-trd-reviewer 责任在生成阶段强制）。
  - **Gate C · `user_accept` 必须固化 docs 同步决策**：`developer.per_task_aiweave_sync=true` → 每任务必须有 `<task_id>.docs_sync_diff.json`；默认 `false` → 自动把 `task_id` 追加到 `context.json.pending_docs_sync[]`，由 finish-project 兜底统一同步。两条路径都不能"沉默忽略"。
  - **audit 快进路径**：会话恢复时若代码+测试已落地且 PASS，`developer_step_runner` 会先 `audit_phase.py` 探测，并提供"快进 / 重跑 / 暂停"三选一菜单。选快进时，`audit_fast_forward` 写入的 RED/GREEN/REFACTOR 证据带 `evidence_source="audit"` + `gate_token="audit:<sha256>"`，**不是绕过 TDD 的旁路**，而是对真实状态的诚实记账。developer-reviewer 看到 `evidence_source="audit"` 时会做加权抽查（实地读测试 + 复跑命令），抽查失败按 blocking 登记。
  - **任何借助删码 → 跑红 → 改回的"重建 RED"操作均视为反 TDD 作弊**：发现这类操作的会话，reviewer 必须登记 blocking 并要求回滚。
  - **Gate D · TDD 阶段（RED + GREEN）反 Explore 子 agent**（v0.x · S4 配套）：当业务项目 `aiweave_status ∈ {T1, T2}` 且任务 `aiweave_docs_refs` 非空时，`developer/scripts/prevent_explore_during_tdd.py` 强制要求主 AI 按 `task_context.aiweave.docs_quick_index` + `related_code_excerpts` 写 RED 测试与 GREEN 实现，**不得**派 Explore / general / search 子 agent 翻业务源码（典型反例：GREEN 阶段「先查项目里 DB/Redis 使用模式」），**不得**单 turn 累计 Read/Grep/Glob > 8 次仍不调 `record_docs_read.py record` 或写代码。命中拦截后渲染**极简双选项菜单**：[1] ✅ 继续（TRD + excerpts + 受限 Explore；docs 留 finish 回写，推荐默认） / [2] 暂停回 TRD 重新 build。菜单内部差异（占位 / 真缺口 / AI 自作主张）仅体现在诊断行，不再分三套选项。`prevent_explore_during_red.py` 保留为 thin shim 透传至 `prevent_explore_during_tdd.py --phase red`。
  - **Gate F · developer 阶段零 docs 写盘铁律**（v0.x · S4 配套）：AIWeave docs 回写只在两个合法时机——① TRD 阶段首次 bootstrap（`aiweave-bridge/bootstrap.py`）；② finish-project 阶段统一 sync-feature-to-docs。developer 全流程（RED / GREEN / REFACTOR / repair / audit）任何路径试图写 `<project>/docs/`，一律视为反 AIWeave 作弊，reviewer 登记 blocking 强制回滚。`rebuild_task_docs.py` CLI 强制 `--called-from finish-project`，developer 阶段误用直接报错。Gate A token 反向 grep 在占位 doc_ref（备注含「待开发 / 占位」）时自动回退到 TRD 校验，evidence 标 `evidence_source=trd_fallback`；token 在 TRD 也不存在才拒绝（保留反虚填）。Gate D 选 [1] 后自动调 `register_pending_docs_sync.py` 把 task_id + docs_refs 登记到 `context.json.pending_docs_sync_notes`，finish-project 批量消费。
  - **资源不在 dev 阶段范围**（2026-05-27 修正）：dev 阶段不强制连接真实 DB / ES / Redis / API。资源真值由用户在联调期通过改 `conf/**` yaml、环境变量、docker compose 等方式自行处理。Developer 写代码时按 TRD / aiweave docs 上记录的接口语义、表名、Redis key、API 路径等正常实现；找不到自定义符号或具体配置 key 时按合理占位实现。Reviewer 仅评判测试是否覆盖关键逻辑分支与错误路径、断言强度是否到位、声明的 mock 是否真在测试中替换；不评判是否使用真实库连接。
  - **Gate E · rebuild_task_docs 严格受 vendor 约束**（v0.x · S4 配套）：用户选「按 AIWeave 模板补 docs」时，`skills/aiweave-bridge/scripts/rebuild_task_docs.py` 按任务 `aiweave_skill` 反查 vendor `templates/skills/<skill>/SKILL.md` 整段加载进 next_action。主 AI **必须**严格按 vendor 字段顺序与 schema 写 docs：禁止新增字段、禁止改字段顺序、禁止套其他 skill 模板、禁止在 docs 写无代码依据的内容。三道闸门（assert_write_target → lint_aiweave_invariants → vendor /doc-sync-check）+ token 反向 grep 五道防线锁死随意发挥。vendor 字段在代码里找不到对应实现 → 登记 `vendor_template_mismatch` 到任务 notes，docs 顶部插 `<!-- aiweave:gap:<field> -->`，**严禁**主 AI 自行调整模板。**任何偏离 vendor 模板的 rebuild 操作视为反 AIWeave 作弊**，reviewer 必须登记 blocking 并要求回滚。
- **7.6 AIWeave 业务产物归属铁律**（AIWeave 整合后新增）：
  - AIWeave 的 build 产物（业务项目下 `docs/`、`.claude/skills/`、`BUILD_STATUS.md`、`CLAUDE.md`）**必须**写入目标业务项目 A 的仓库根目录下，并随 A 的 git 一起提交。
  - vendor 是规范源码，**只在 nexus-harness 仓库**；nexus-harness 的 `workspace/<日期-项目>/` 仍只放 PRD / TRD / Plan / 评审报告等过程产物，**不得把 A 项目的 docs/ 写进 workspace 或 vendor**。
  - 所有 bridge 脚本（`bootstrap.py / invoke_skill.py / update_build_status.py / sync-feature-to-docs / doc-sync-check`）的写盘目标必须是 `project_path/docs/...` 或 `project_path/.claude/skills/...`，**禁止**写到 nexus-harness 仓库或 workspace。`assert_write_target.py` 在写盘前批量校验。
  - 非 Go 项目（`<project_path>/go.mod` 不存在）一律降级，跳过整条 AIWeave 流程。
8. **编辑受 guard block 保护的文档必须先读守护声明、改完必须跑 lint**：
  - `prd-to-trd` 产出的 TRD 会在 H1 下注入 `<!-- prd-to-trd:guard-block:begin/end -->` 守护块（人类可见的引用块，明写「给后续编辑本文档的 AI 阅读」）。
  - 任何对该文件的后续编辑（包括普通对话里的 "帮我改一下 X"），**先完整读完守护块**再动笔；禁止以索引式引用 / HTML 注释塞过程信息 / `PRD X.Y.Z` 做来源列 / 节选占位 等方式偷懒。
  - 编辑完成后**必须**运行 `python3 skills/prd-to-trd/scripts/lint_trd.py <文件路径>`；存在 blocking fail 时不得对用户声明"已完成"，必须逐条修复。
  - 本条适用于所有带 guard block 锚点的文档，未来其他 skill 产出长文档时同样按此范式加守护块。
9. **nexus-harness 工作模式（仓库级触发 + skill 强制路由）**：
  - **仓库识别**：当 AI 加载到本文件（`AGENTS.md` 位于 `nexus-harness` 仓库根目录）时，无论 AI 是哪家（Claude / GPT / Gemini / Cursor Composer / Codex / 其他），都进入 **nexus-harness 工作模式**。本模式下的一切文档产出与编辑，都必须走 `skills/` 下对应 skill 的约束，不得绕过。
  - **skill 强制路由**：任何对 `prd-to-trd` 产出过的 TRD（即文件头含 `<!-- prd-to-trd:guard-block:begin -->` 锚点）的编辑请求，即便用户只是对话式追问 "把 X 改成 Y / 评估可行性 / 帮我优化一下"，AI **必须**在动笔前先 Read `skills/prd-to-trd/SKILL.md` 把完整 HARD-GATE 清单加载进上下文，然后按 skill 的硬约束（而不是自己的写作偏好）执行。**禁止以"对话追问不在 skill 范围内"为由跳过约束**——这正是最容易出事故的场景。
  - **约束一致性**：普通对话编辑 / skill 自动产出 / 后续会话补丁，三种场景的输出标准**必须一致**，都由 guard block + lint_trd.py 兜底。
  - **唯一例外**：用户在当前会话明确说明"现在是维护 nexus-harness 框架"，此时才允许修改 `skills/`** 本身；即便如此，该会话不得同时夹带业务 TRD 的编辑。

---

## 当前已就绪的 skill

> 完整规划共 18 个 skill，分三梯队建设。本节只列**当前可用**的部分，新增后请同步更新此处。


| skill                        | 状态                                          | 入口                                           |
| ---------------------------- | ------------------------------------------- | -------------------------------------------- |
| `aiweave-bridge`             | ✅ 可用（utility · vendor AIWeave 适配层）          | `skills/aiweave-bridge/SKILL.md`             |
| `project-start`              | ✅ 可用（仅收集 project_path，不再做 KB 选择）            | `skills/project-start/SKILL.md`              |
| `project-start-reviewer`     | ✅ 可用（PRD 草稿 subagent 质检）                    | `skills/project-start-reviewer/SKILL.md`     |
| `prd-to-trd`                 | ✅ 可用（AIWeave 入口探测 + 选项式追问 + docs_increment_plan） | `skills/prd-to-trd/SKILL.md`            |
| `prd-to-trd-reviewer`        | ✅ 可用（含 docs/ 蓝图一致性维度）                       | `skills/prd-to-trd-reviewer/SKILL.md`        |
| `plan-from-trd`              | ✅ 可用（含 aiweave_skill / docs_refs / 4 类 red_tests） | `skills/plan-from-trd/SKILL.md`         |
| `plan-from-trd-reviewer`     | ✅ 可用（含 aiweave_skill / 4 类测试维度）             | `skills/plan-from-trd-reviewer/SKILL.md`     |
| `developer`                  | ✅ MVP 可用（含 BUILD_STATUS 翻状态 + 四角对照可选评审）     | `skills/developer/SKILL.md`                  |
| `developer-reviewer`         | ✅ MVP 可用（扩 AIWeave 四角对照维度）                  | `skills/developer-reviewer/SKILL.md`         |
| `qa-from-trd`                | ✅ 可用（TRD 完成后自动触发，4 层 Case 零交互生成）           | `skills/qa-from-trd/SKILL.md`                |
| `finish-project`             | ✅ MVP 可用（最终 sync-feature-to-docs + doc-sync-check） | `skills/finish-project/SKILL.md`        |
| `finish-project-reviewer`    | ✅ MVP 可用（评审 docs/ 同步 diff）                  | `skills/finish-project-reviewer/SKILL.md`    |
| `diagnose`（问题定位）              | ✅ MVP 可用（L3 改读 docs/INDEX.md；非 Go 降级 dev_notes 优先） | `skills/diagnose/SKILL.md`             |
| `docs-build`（提取 AI 知识库）       | ✅ 可用（T0 单子 Agent 一次性冷启动 / T1 补缺 / T2 检查并按需同步；末尾 lint + reviewer 兜底） | `skills/docs-build/SKILL.md`           |
| `docs-build-reviewer`        | ✅ 可用（评审 AIWeave docs 裸占位、gap 密度、INDEX/BUILD_STATUS 自洽） | `skills/docs-build-reviewer/SKILL.md`        |

> 旧版 `knowledge-extract` / `knowledge-extract-reviewer` / `knowledge-qa-reviewer` 已物理下沉到 `skills/_deprecated/`，仅作 git 历史留档；详见 `skills/_deprecated/README.md`。


> 想验证整套链路是否能跑通 → 在一个干净的业务项目下跑 `nexus`，按横幅选 `1` 走"开始新项目"，从 PRD → TRD → Plan → Develop → Finish 完整 e2e 一次即可。无需单 skill 测试入口。

---

## 找 skill 的标准做法

1. 列 `skills/` 子目录，挑可能相关的几个
2. 读它们的 SKILL.md 顶部 frontmatter（`description` / `when_to_use` / `inputs` / `outputs`）
3. 命中即用；都不命中就告诉用户「当前仓库没有匹配的 skill」，**不要硬凑**

---

---

## 给用户的提示

- 如果你是第一次用本仓库，先按 `README.md` 装好 `nexus` 命令，然后在一个干净业务项目目录下跑 `nexus` 选 `1`，走一次完整 e2e 即可确认环境就绪。
- 想新增 skill：在 `skills/<name>/` 下放 `SKILL.md`（顶部带 frontmatter），需要的脚本放 `scripts/`，模板放 `templates/`。然后回到本文件「当前已就绪的 skill」表格补一行。
- **新 skill 接入要求**：`SKILL.md` 在 frontmatter 与正文之间**必须**复述一遍「框架不可变铁律」通知块（可从现有 skill 如 `skills/prd-to-trd/SKILL.md` 顶部直接复制）。这是第 7 条铁律的第二道文字防线，reviewer 合并前必须核对。

---

## §Subagent 派发适配（v2 双 Agent 模式·宿主无关协议）

本仓库的双 Agent 评审能力（已落地场景：`prd-to-trd` + `prd-to-trd-reviewer`、`plan-from-trd` + `plan-from-trd-reviewer`、`finish-project` + `finish-project-reviewer` 等）**不绑定任何宿主的 subagent API**。skill 只产出一份宿主无关的 `action="dispatch_reviewer"` 指令包；怎么把 reviewer 跑起来，由各宿主按本节协议自行适配。

> **铁律补强**：`dispatch_reviewer` 节点的默认行为是「真子 Agent 派发」。在 Cursor / Claude Code 等**有原生 Task / subagent 能力**的宿主上，主 Agent **不得**未尝试 Task 就直接走脚本降级（`run_review.py`）。一旦走了降级，给用户的汇报里必须显式标注「⚠️ reviewer 已降级」。

各 reviewer skill 的报告 schema 见 `skills/<x>-reviewer/schema/*.schema.json`。

### Subagent 注册文件清单（**勿删**）

下列文件是各宿主识别 reviewer subagent 的**唯一锚点**。删任意一份 = 对应宿主的双 Agent 评审通道立刻失效（会静默退化为 Bash 跑 `run_review.py`，或 generalPurpose 通用代理，铁律失守）：


| 路径                                             | 服务宿主        | 删了会怎样                                                                  |
| ---------------------------------------------- | ----------- | ---------------------------------------------------------------------- |
| `.claude/agents/prd-to-trd-reviewer.md`        | Claude Code | `Task(subagent_type="prd-to-trd-reviewer", ...)` 找不到名字，主 Agent 退化 Bash |
| `.claude/agents/project-start-reviewer.md`     | Claude Code | `project-start` 的 PRD 草稿语义评审无法用 Task 启动                                |
| `.claude/agents/plan-from-trd-reviewer.md`     | Claude Code | Plan 评审同上，`plan-from-trd` 退化 Bash                                      |
| `.claude/agents/finish-project-reviewer.md`    | Claude Code | Finish docs 同步 diff 评审无法用 Task 启动                                       |
| `.claude/agents/docs-build-reviewer.md`        | Claude Code | docs-build / aiweave-snapshot 产物无法用独立 reviewer 复核，AIWeave docs 质量闸门退化 |


新增宿主或新增 reviewer skill 时，必须**同步**补对应注册文件，并把这张表更新。

### 通用流程（所有宿主都一样）

1. 主 Agent 调 `skills/prd-to-trd/scripts/dispatch_reviewer.py --trd_path=... --prd_path=...` 取得派发指令包；
2. 按本节对应宿主的适配方式，派出一个**独立上下文**的 Agent，加载 `prd-to-trd-reviewer` skill，输入 = 指令包的 `reviewer_inputs`；
3. reviewer Agent 工作完成后，把 `review_report.json` 写到 `reviewer_output_path`；
4. 主 Agent 再次调同一脚本并附 `--reviewer_report_path=<上一步路径>` 做结果回流；
5. 按返回的 `decision`：`repair` → 走 `ai_repair_trd`；`accept`* → 交付用户。

**铁律**：

- 主 Agent 禁止"我来扮演 reviewer"绕过派发。单 Agent 自评失效是本架构演进的根本原因。
- **宿主有原生 subagent 能力时，禁止并行跑 `run_review.py` 骨架**。二者会抢写同一 `reviewer_output_path`，导致最终回流到 `auto_converge` 的报告来源不可控。规则：  
  - **有原生 subagent（Claude Code / Cursor / Codex / Gemini）** → **只**走原生派发，不跑骨架。  
  - **无原生 subagent** → **只**走降级协议（见下），骨架在降级协议内部按顺序跑。
- 违反此条的表现："先跑骨架兜底再做上下文重置评审 / 先跑骨架再派 subagent"——日志里一旦出现这类自述，立即叫停。

### 各宿主适配


| 宿主                     | 能力                 | 派发方式                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ---------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Claude Code**        | 原生 subagent        | 仓库已预置 `.claude/agents/prd-to-trd-reviewer.md`、`.claude/agents/project-start-reviewer.md`、`.claude/agents/plan-from-trd-reviewer.md`、`.claude/agents/finish-project-reviewer.md`、`.claude/agents/docs-build-reviewer.md`（**跟仓库走，clone 即可用，无需安装**）。读到 dispatch 包后，下一步必须是对应 reviewer 的 `Task(subagent_type="<xxx-reviewer>", prompt=<dispatch 包里的 task_prompt_template>)`。**禁止** `subagent_type="generalPurpose"` 兜底；禁止当前会话自评；禁止直接 Bash 跑 `run_review.py`。若没有 Task 工具，停止并报告"未启动 reviewer 子 Agent，本轮 reviewer 流程作废" |
| **Cursor**             | 原生 subagent        | 用 Cursor 原生 Task / subagent，`subagent_type="generalPurpose"` + `prompt=<task_prompt_template>` 即可；这仍是独立上下文真子 Agent，**不算降级**。Cursor 当前不依赖仓库内注册自定义 subagent type；只有未启动 Task、改跑 `run_review.py` 才算降级                                                                                                                                                                                                                                                                           |
| **Codex CLI**          | 暂不支持（v2 路线推迟）      | 走"降级协议"（见下）                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| **Gemini CLI**         | skill 激活           | `activate_skill("prd-to-trd-reviewer")` 后直接把 `reviewer_inputs` 丢给它                                                                                                                                                                                                                                                                                                                                                                                                          |
| **任何无 subagent 能力的宿主** | 无                  | 走降级协议（见下）                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |


### 降级协议：上下文重置评审

> 可复制提示词模板见：`skills/prd-to-trd-reviewer/references/degraded-review-prompt.md`（模板 A/B/C）。

宿主无法派独立 subagent 时，主 Agent **必须**按以下顺序降级：

1. **优先**：调 `skills/prd-to-trd-reviewer/scripts/run_review.py --dispatch=<指令包 JSON 路径>` 脚本骨架评审，先抓硬闸门残留 + 外部引用变种；
2. **补位**：在当前会话内声明"**上下文重置评审**"：
  - 明文说出："现在我扮演 reviewer Agent，忘记之前所有生成 TRD 的语境。我只看 TRD 产物 + HARD-GATE 摘要，按 reviewer skill 的 SKILL.md 执行评审。"
  - 然后 Read `skills/prd-to-trd-reviewer/SKILL.md` + `skills/prd-to-trd/references/hard-gates-digest.md` + TRD 产物 + 硬质检报告；
  - 产出符合 `review_report.schema.json` 的 JSON，写到 `reviewer_output_path`；
3. 效果比真 subagent 弱（上下文无法真正清零），但**必定强于单 Agent 自动自评**——至少强制了"只看产物、对账摘要"这一步仪式。

### 自测与接线验证

任何宿主接入后，建议跑两条验证线：

- **真实 e2e**：在干净业务项目目录下 `nexus` → 选 `1` → 走 PRD → TRD → Plan → Develop → Finish 全链；过程中观察是否真的把 reviewer subagent 派出来（而不是降级骨架）。
- **单脚本骨架**：`python3 skills/prd-to-trd-reviewer/scripts/run_review.py --trd <fixture-trd.md>`，确认 `prd-to-trd-reviewer` 的硬质检骨架能抓出故意违规（fixture 见 `skills/prd-to-trd-reviewer/scripts/`）。

### CI 化（pre-commit + GitHub Actions）

`run_review.py` 提供两种 CLI 模式，支撑 subagent 与 CI 两类场景：


| 模式                      | 触发            | 命令                                                                            |
| ----------------------- | ------------- | ----------------------------------------------------------------------------- |
| **A · subagent**        | 双 Agent 主流程   | `python3 run_review.py --dispatch=<dispatch.json>`                            |
| **B · CI / pre-commit** | 仓库钩子、PR check | `python3 run_review.py --trd <TRD.md> [--prd <PRD.md>] [--out <report.json>]` |


落地入口：

- **pre-commit**：`.pre-commit-hooks.yaml` 暴露 `trd-review` hook（消费者仓库按 README 示例引用即可）；本仓库自用 `.pre-commit-config.yaml` + 包装脚本 `skills/prd-to-trd-reviewer/scripts/precommit_trd_review.sh`（逐文件调用，支持一次提交多 TRD）。
- **GitHub Actions**：`.github/workflows/trd-review.yml`，PR 里命中 `*技术方案*.md`/`*TRD*.md`/`.Agent/**.md` 自动触发，报告落 artifact + PR Summary。
- CI 只覆盖"机械评审"层（硬闸门 + heuristic + PRD 覆盖率打分，若 `--prd` 给出），**深度语义判读仍需本地/PR 作者手动触发 reviewer subagent**。CI 红灯不代表通过 = 合格，只代表连"机械不该错的地方"都有错，属于零容忍拦截。

---
> Source: [xurb-nexus/nexus-harness](https://github.com/xurb-nexus/nexus-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
