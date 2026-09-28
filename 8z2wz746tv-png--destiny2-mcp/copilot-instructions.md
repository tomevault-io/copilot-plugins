## destiny2-mcp

> This repository contains a local Destiny 2 MCP server with Bungie OAuth authentication.

# Agent Instructions

This repository contains a local Destiny 2 MCP server with Bungie OAuth authentication.

For any agent, not only Codex:

- Read `README.md` first for the platform-neutral installation and verification flow.
- Read `skills/destiny2-mcp/SKILL.md` for tool routing and evidence boundaries.
- Read `skills/destiny-mcp-setup/SKILL.md` only for installation, OAuth, registration, or setup troubleshooting.
- Run `scripts/verify_mcp.py` for the real MCP handshake; do not treat registration alone as proof of readiness.

## 遇到 Bungie API 问题怎么查

官方文档一页全包：<https://bungie-net.github.io/> —— OAuth scope 表、每个端点的请求/响应、AWA 三段流程、DestinyComponentType 组件枚举，都在这一页。

要深查（离线读原文、确认字段名或端点路径）就把它拉到仓库外：

    python scripts/fetch_bungie_api_docs.py            # 抓到 ~/.destiny_mcp/reference/，带日期与 manifest.json（版本号/字节数/sha256）
    python scripts/fetch_bungie_api_docs.py --check    # 只比在线文档版本号与上次记录，看文档换没换版本（--exit-code 时不一致退出 1）

那份快照**不进 git**：它是上游产物、不是我们的口径；我们依赖的是自己的实测事实。

**实测优先于文档**：`docs/reference/bungie_api.md` 记着我们踩过的坑与真机结论（scope、AWA、free vs 付费插槽接口、组件 305 必须带清单类、写入后的同步窗口、神器三个 hash 家族…）。与官方文档冲突时，以那里的「实测」为准，并去更新那一条。

## Skill maintenance

`skills/destiny2-mcp/` is the single source for the agent-facing guide. Nothing in it may assume one host: the same folder is loaded by hosts with a skills directory, and hosts without one get a pointer block plus the public URL advertised in the MCP handshake.

- After editing anything under `skills/destiny2-mcp/`, run `scripts/install_skill.py` so the installed copy keeps up. It installs into **the host that is running you** by default (DSH: `~/.dsh/skills/`, Claude Code: `~/.claude/skills/`, Codex: `~/.codex/skills/`, detected from the environment); `--list` shows the current host, `--host <name>` picks one, `--all` installs into every detected host, `--pointer` refreshes the instruction-file pointers, and `--mcp` registers the MCP server (writes the DSH profile patch; prints the paste-ready command for the others).
- A new user installs this for the agent they are talking to, not for every agent on the machine: do not spread copies into Codex and Claude "just in case".
- Never write into a directory the host manages itself (for example Cursor's `skills-cursor`); use `--target` or `--pointer <file>` for unlisted hosts.
- `tests/test_skill_contracts.py` checks `references/routing.md` against the code: intent coverage, parameter ownership, write intents, community categories. A red test there means the document is stale, not that the check is too strict.
- `tests/test_skill_install.py` checks the delivery layer: pointer idempotence, mirror sync, and that `config.ROUTING_GUIDE_URL` matches the installer's URL.
- `destiny_mcp/error_codes.py` is the only place `error.code` may come from: literal codes live in `ErrorCode`, exception classes are turned into codes by `code_for_exception()` (the class name *is* the contract), and write failures use `write_failed(intent)`. `tests/test_error_codes.py` rejects bare string codes and pins the exception→code mapping.
- `destiny_mcp/vocabulary.py` is the only place Chinese labels live (six stats, classes, locations, elements, legacy stat names). Labels are the Manifest's official Chinese strings; input aliases may be many but must resolve to one key. `tests/test_vocabulary.py` pins the official names, checks aliases round-trip (legacy names must point at *existing* labels), and rejects re-copied tables.
- `tests/test_conclusion_paths.py` guards diagnosis/conclusion output: a conclusion module may fall back to weaker wording, but never silently — the `except` must re-raise or leave a trace (append / reason / log). Add new conclusion modules to `_CONCLUSION_MODULES` there.
- `tests/test_architecture_layers.py` guards the module graph: imports may only go downwards (`_LAYERS`), no two modules may import each other, and every `svc["…_svc"]` key used in `tools/` must be declared in `service_context.ServiceContext`. A new top-level module must be registered there, otherwise the test fails on purpose — decide its layer instead of letting the rule silently skip it.

## Setup-related tasks

For installation, reinstallation, OAuth login, Codex MCP registration, or setup troubleshooting:

1. Read `skills/destiny-mcp-setup/SKILL.md` completely before taking action.
2. Follow that Skill through a real MCP handshake and verification; registration alone is not sufficient.
3. Run `skills/destiny-mcp-setup/scripts/verify_mcp.py` and require `BUNGIE_PROFILE_CHECK=ok` plus all eight tools in the normal profile.
4. Preserve unrelated worktree changes and inspect existing `.env`, OAuth tokens, and Codex MCP entries before changing them.

Never print or request secrets in chat. This includes `.env` contents, Bungie API keys, OAuth client secrets, authorization codes, access tokens, refresh tokens, and callback URLs containing authorization codes. Ask users to enter credentials locally in `.env`.

After registering or changing the MCP server, tell the user to restart Codex or open a new task so the new server is discovered.

## 写代码的习惯

这些不是风格偏好，是踩过的坑换来的。能自动化的都配了守门测试——改代码前**先看它守什么**。

### 单一出处：一个事实只写一次

| 事实 | 唯一出处 | 守门 |
| --- | --- | --- |
| 错误码 | `destiny_mcp/error_codes.py` | `tests/test_error_codes.py`（禁止裸字符串；异常类名即契约） |
| 中文词表（六维/职业/位置/元素/旧名） | `destiny_mcp/vocabulary.py` | `tests/test_vocabulary.py`（官方名钉住、旧名必须指向存在的规范名、禁止再抄） |
| 活动统计形状 | `destiny_mcp/activity_stats.py` | `tests/test_activity_stats.py`（上游键清单当夹具，拼错或新增会红） |
| intent 取值 / 写作清单 | `tools/_requests.py` | `tests/test_skill_contracts.py`、`tests/test_ignored_parameters.py` |
| 参数归属 | `tools/_param_contracts.py` | 同上（认领了必须真读、没认领必须拒收） |
| 响应信封 | `tools/_responses.py` | 语料 `sweep` 组的信封规则（110 个 intent 全查） |
| 服务容器 | `service_context.py` | `tests/test_architecture_layers.py` |

不要复制一张表、一份映射、一段正则到第二个文件。要共享就抽模块，并加一条"禁止再抄"的扫描测试。

### 不许静默降级，也不许替调用方下结论

- **没算就不能说**：`analyze` 曾在没验证过的情况下断言"没有合法组合"，而同一组约束 `recommend` 能配出 100% 达标的方案（0.1.13 修）。诊断类输出只说自己算过的东西。
- **"没探成" ≠ "试过没有"**：探测失败记 `ok: null` + `not_probed` + `reason`，不能记成 `ok: false`（`_armor_ladder` 踩过）。
- **结论路径的 `except` 必须留痕**：重抛、写进返回值、或打日志；`tests/test_conclusion_paths.py` 扫。
- **可选数据可以降级，但要带 `warnings`**；坏的可选资料不能弄坏主结果。
- **缺值给 `None`，不编 `0`**；"没查到"和"没有"是两件事（`coverage_complete`、`unverifiable_reason`、`selectable_plug_status`）。

### 禁止兼容、不做补丁

- 改响应形状**旧键一个不留**（不双写、不留过渡分支）：0.2.0 活动统计从手写 8 键改行式，`activitiesEntered` 直接消失。
- 别名要么是**永久**（中文说法，如 `概况`/`重复武器`），要么**登记待删**（英文近义）；一律写进 `docs/COMPATIBILITY.md`，并保证同组别名走同一段分派。
- 旧实现整段删掉，不保留"失败就退回旧路径"的分支。

### 分层与体量

- 依赖只能向下、禁止环、`svc["…_svc"]` 必须在 `ServiceContext` 里声明；新顶层模块要登记层号（`tests/test_architecture_layers.py`）。
- 模块超限先抽代码，**不抬上限**；上限只能下调（`tests/test_module_size_ratchet.py`）。
- 纯形状工厂与词表放低层；工具层只做分派、参数守卫与话术。

### 失败、写入与证据

- **失败必须是失败**：`ok=false` + `error.code` + 中文说明"缺什么/下一步"；裸抛异常算 bug（schema 层拒收是例外，见 `docs/testing/TESTING_CORPUS.md` 第十二章）。
- **改账号的操作**：服务端签发候选 + 用户明确 `confirmed=true` 才写；写完回读核对。
- **数据分三档不许混**：Manifest 定义 / 你的账号 / 社区资料（不可信参考，带 `source_ref` 与更新时间；出处与致谢见 README 的 Starside 段）。
- 上游故障如实说是上游，不编、不循环重试。

### 测试与验证

- 每个新守门都要**注入一次违规、确认变红、再恢复**——不验证就不知道它会不会咬人。
- 改响应形状：跑 `pytest` + 对应语料 runner + 基线 diff。
- 单测用替身、任何机器能跑（干净 `HOME` 下也要过）；真机脚本单独放，分工见 `docs/testing/TESTING_CORPUS.md` 第一张表。
- **CI 跑 ruff 的规则集是写死的**（`pyproject.toml` 的 `[tool.ruff.lint] select`，版本钉在 `ruff>=0.15,<0.16`）：ruff 的默认集随版本变（0.16 把 UP/SIM/FURB 都算默认），不写死就是「本地绿、CI 红」。想收规则要显式加进 select 并一次修完。
- **CI 现在跑 ruff**（`python -m ruff check`，紧挨在 pytest 前面）：它抓「未定义的 name / 未使用的 import」这类，比测试更早——真机上它抓到过 `profile_components` 用了没导入（build 取账号护甲那条路直接 NameError）。mypy 存量 411 个错，**不进 CI**；路线是「新代码先干净」：先把某个模块修到 mypy 干净，再加进 ci.yml 里那行注释的文件列表。
- **声称之前先量**：性能改动给前后对比，文档里的数字必须来自实跑。

**验证成本与省时约定**（2026-09-21 实测，写在这是为了让每个 agent 都少烧时间）：

| 动作 | 实测耗时 | 什么时候才跑 |
| --- | --- | --- |
| `pytest -q`（1652 条） | ≈ 2.5 分钟 | 代码改完、提交前跑一次；**文档改动不要跑全量** |
| `pytest -q tests/test_agent_docs.py` | 5 秒 | 只动 `docs/**` 时 |
| `+ tests/test_skill_contracts.py` | 3 秒 | 动了 `skills/**` 或 intent/参数契约时 |
| `scripts/run_corpus_all_rows.py`（260 行，打真机） | ≈ 4–5 分钟 | 一个功能**只在提交前跑一次**；不要每改一处就跑 |
| `scripts/install_skill.py` | < 1 秒 | 动过 `skills/**` 之后 |
| `scripts/install_skill.py --mcp`（重连标记改成**代码版本**，DSH 随即重组重连） | < 1 秒 + 重连 ≈ 40 秒 | 要让宿主/GUI 用上新代码时；同一版本重复跑是幂等的 |

- **别把重活并发跑**：实测与其它任务并发时 pytest 从 100 秒涨到 155 秒，还会让
  `tests/test_build_compute.py::test_queue_wait_is_not_charged_to_the_budget` 假失败。
- 要游戏内才能确认的事实（入口是常驻还是轮换、某个名字到底叫啥）**先问用户**，
  不要围着未知先造一套表再返工。

### 注释、文档与提交

- 注释写**为什么**（根因、踩过的坑、不能改的理由），不写"做了什么"：例 `a_p_i_error` 为什么长这样、`or ""` 防的是什么。
- 口径写进可执行断言，不靠人记；计划文档（`*_FORMAT_PLAN.md`）记实机证据与取舍。
- 提交信息见下面的「提交信息格式」：一行标题、一次只做一件事。
- 密钥永不进日志、提交与截图（`.env`、token、client secret）。

## 提交信息格式

一条提交 = **一行标题**，不写正文（细节在 CHANGELOG、计划文档与代码注释里）。

    <类型>：<做了什么>

- 类型用中文，现有取值：`版本号`（如 `0.2.0：…`）、`修复`、`重构`、`文档`、`语料`、`CI`、`错误码`、`词表`、`信封`、`兼容面`、`结论路径`，以及计划阶段的 `护甲 P3` / `武器 P4`。
- 标题写"做了什么"，不写「抓出的 XXX」这类腔调；不用 emoji、结尾不加句号。
- **一次提交只做一件事**：跨主题的改动拆成多条；发布提交也一样——内容照发，标题只留一行。
- 破坏性变更在标题里点明（例：`活动统计改为行式（破坏性）`），影响面写进 CHANGELOG。
- **推到哪**：工作分支是 `codex/bundle-starside-markdown`，`origin/main` 与它同源。发布用
  `git push origin HEAD:main HEAD:codex/bundle-starside-markdown --tags` —— 本地那个 `main` 是
  2026-09 的旧分支，`git push origin main` 会被非快进拒绝（2026-09-24 踩过）。
  GitHub 直连常超时（SSL / 443），加 `-c http.lowSpeedLimit=1000 -c http.lowSpeedTime=20` 重试几轮即可。

## 代码地图

顶层模块按分层表分组列在下面。**清单那半由脚本生成，不要手改** —— 手改会被
`tests/test_agent_docs.py` 判红（它拿脚本现算的结果逐字比）。层号来自
`tests/test_architecture_layers.py` 的 `_LAYERS`：新增顶层模块先在那边登记层号，再重新生成：

    .venv/bin/python scripts/gen_code_map.py

<!-- code-map:begin -->
**第 0 层：纯基础：不依赖项目里任何东西（除了彼此）**
- `destiny_mcp/__init__.py`
- `destiny_mcp/bungie_errors.py`
- `destiny_mcp/config.py`
- `destiny_mcp/error_codes.py`
- `destiny_mcp/exceptions.py`
- `destiny_mcp/logging_config.py`
- `destiny_mcp/models/`
- `destiny_mcp/utils/`
- `destiny_mcp/vocabulary.py`
**第 1 层：基础设施：Manifest / Bungie 客户端 / 账号解析 / 类型容器 / 实测事实表**
- `destiny_mcp/activity_stats.py`
- `destiny_mcp/audit.py`
- `destiny_mcp/build_contracts.py`
- `destiny_mcp/bungie_client.py`
- `destiny_mcp/bungie_stats.py`
- `destiny_mcp/data/`
- `destiny_mcp/manifest.py`
- `destiny_mcp/manifest_armor.py`
- `destiny_mcp/manifest_artifacts.py`
- `destiny_mcp/manifest_catalog.py`
- `destiny_mcp/manifest_data.py`
- `destiny_mcp/manifest_definitions.py`
- `destiny_mcp/manifest_fingerprint.py`
- `destiny_mcp/manifest_item_queries.py`
- `destiny_mcp/manifest_lookup.py`
- `destiny_mcp/manifest_names.py`
- `destiny_mcp/manifest_plugs.py`
- `destiny_mcp/manifest_search.py`
- `destiny_mcp/oauth_setup.py`
- `destiny_mcp/player_resolver.py`
- `destiny_mcp/service_context.py`
- `destiny_mcp/wishlist_data.py`
**第 2 层：领域层（纯计算，可被服务和工具复用）**
- `destiny_mcp/build/`
- `destiny_mcp/rag/`
**第 3 层：服务层：账号读写、外部数据、形状工厂**
- `destiny_mcp/services/`
**第 4 层：工具层：MCP 门面（只做分派、守卫与话术）**
- `destiny_mcp/tools/`
**第 5 层：装配层**
- `destiny_mcp/__main__.py`
- `destiny_mcp/server.py`
<!-- code-map:end -->

`legacy/`（仓库根目录）不在这张清单里，也**不进包、不参与测试与 lint**：它是 2026-09-20 剥离的
历史工具面存档（67 个低层工具 + 配装导入整块），只作查阅用。要复活里面的东西，见 `legacy/README.md`。

### 各模块一句话职责

这半**手写**、不生成：上面说"有哪些"，这里说"往哪儿放、别往哪儿堆"。新增模块必须在下面补一行，
漏写会被测试判红；表里出现清单外的模块同样红（模块删了或名字写错）。可以写"贴着上限""别往里堆"这类经验。

| 模块 | 做什么 |
| --- | --- |
| `destiny_mcp/__init__.py` | 包标记与版本来源，别往里放逻辑。 |
| `destiny_mcp/config.py` | 配置唯一入口（env + dotenv）；密钥只在这里读、永不打印。 |
| `destiny_mcp/error_codes.py` | `error.code` 的唯一出处；禁止裸字符串。 |
| `destiny_mcp/exceptions.py` | 异常类定义，类名即 `error.code` 契约。 |
| `destiny_mcp/logging_config.py` | 日志配置；日志里不许出现密钥。 |
| `destiny_mcp/models/` | 响应的 Pydantic 形状定义，放低层。 |
| `destiny_mcp/utils/` | 纯工具（无服务/API 依赖）；带领域逻辑就该挪去 `services/`。 |
| `destiny_mcp/vocabulary.py` | 中文词表唯一出处（六维/职业/位置/元素/旧名）。 |
| `destiny_mcp/activity_stats.py` | 活动统计形状唯一样式：上游 `statId` → 行式；改口径先看 `tests/test_activity_stats.py`。 |
| `destiny_mcp/audit.py` | 每次 MCP 工具调用落盘审计，调用日志只在这里写。 |
| `destiny_mcp/build_contracts.py` | 配装契约类型（`BuildRecipe`/`CanonicalBuild`/`ExecutableBuild`），见 ADR-001；只有最后一个能执行。 |
| `destiny_mcp/bungie_errors.py` | 上游 HTTP 错误 → 领域错误的唯一翻译层（404 单独成码、503 归上游不可用）；客户端与取数模块共用，禁止在别处再写一套 except。 |
| `destiny_mcp/bungie_client.py` | Bungie API 客户端门面：token 生命周期 + 各端点；活动统计端点已拆到 `bungie_stats.py`，错误映射在 `bungie_errors.py`。 |
| `destiny_mcp/bungie_stats.py` | 活动/战绩取数域：历史统计的按角色/账号级两条路；`modes` 只在按角色端点上生效、`periodType` 没有 Season（实测）。 |
| `destiny_mcp/manifest.py` | Manifest 管理器门面（查名/搜索），其余按域拆到 `manifest_*.py`；贴着 262 上限，只做聚合。 |
| `destiny_mcp/manifest_armor.py` | 护甲模组与套装加成域。 |
| `destiny_mcp/manifest_artifacts.py` | 赛季神器：列表、按名/按 hash 查询、层级与模组解析。 |
| `destiny_mcp/manifest_catalog.py` | 武器目录：按类型/稀有度/职业/属性/弹药筛条目。 |
| `destiny_mcp/manifest_data.py` | Manifest 静态常量（基址、物品类型名、社区别名）。 |
| `destiny_mcp/manifest_definitions.py` | 物品定义查询：hash 索引、原始 JSON、中英连接回退。 |
| `destiny_mcp/manifest_fingerprint.py` | Manifest 指纹：给"生成物跟不跟得上 Manifest"一个便宜判定。 |
| `destiny_mcp/manifest_item_queries.py` | 单侧连接或按 JSON 子串扫描的物品查询。 |
| `destiny_mcp/manifest_lookup.py` | 通用定义查询与命名解析（白名单表、占位、按名搜、收藏品反查）。 |
| `destiny_mcp/manifest_names.py` | 伤害/弹药/破盾/属性的中文名唯一定义处。 |
| `destiny_mcp/manifest_plugs.py` | Plug 池与 Perk 描述：插槽类别、可插条目、Perk 文本。 |
| `destiny_mcp/manifest_search.py` | 中英双语名称索引与三层匹配/模糊匹配。 |
| `destiny_mcp/oauth_setup.py` | 个人 OAuth 助手：授权码流程与令牌落盘。 |
| `destiny_mcp/player_resolver.py` | 玩家/角色解析共享逻辑（BungieName → membership）。 |
| `destiny_mcp/service_context.py` | 服务容器；`svc["…_svc"]` 的 key 必须在这里声明。 |
| `destiny_mcp/wishlist_data.py` | DIM 愿单数据获取（个人安装用）。 |
| `destiny_mcp/data/` | 纯数据表（静态常量/映射）：`activity_modes.py` 是**模式词与 `modeType` 的唯一出处**（中文名去 Manifest 取，别再抄标签表）；`pvp_counters.py` 是「计数器 → 模式/周期」对照表。每条带实测证据，改表先看 `tests/test_activity_modes.py` / `tests/test_pvp_counters_table.py`。 |
| `destiny_mcp/build/` | 配装求解引擎（护甲优化）：纯计算、不碰账号；`farm_target.py` 贴着 1296 上限。 |
| `destiny_mcp/rag/` | 本地社区资料检索（Phase 3）。 |
| `destiny_mcp/services/` | 服务层：账号读写、外部数据、形状工厂；判断逻辑落这里，别落工具层。**PvP 武器榜**（逐场 PGCR 聚合）在 `services/pvp_weapon_service.py`，它的结算缓存单独在 `services/pgcr_cache.py`（两者都登记了体积上限）；**游戏内生涯计数器**（profile 组件 1100）在 `services/activity_counters_service.py`：它与统计接口是两个来源，读抖动要重试、空要报 `unavailable`（不许当 0）；**锻造武器模式/红框**（图鉴「模式和催化」，进度在 profile 组件 900）在 `services/pattern_service.py`，社区来源的按名索引在 `services/starside_crafting_sources.py`（见 ADR-009）；口径见 `docs/reference/bungie_api.md`。 |
| `destiny_mcp/tools/` | 工具层：只做分派、参数守卫与话术；intent 取值/参数归属/响应信封三张契约表都在这层。分支响应按域放 `_*_branches.py`（如 `_counters_branches.py` = `activity_assistant(intent="counters")`）。 |
| `destiny_mcp/__main__.py` | `python -m destiny_mcp` 入口；写法要 spawn 安全（构建求解 worker 会重跑它）。 |
| `destiny_mcp/server.py` | MCP 门面装配：注册工具与握手；别塞业务逻辑。 |

## 文档索引

`docs/**/*.md` 全部登记在这里，每条一句话说"什么时候该看它"。新增文档要补一行，删文档要同步删行 ——
`tests/test_agent_docs.py` 双向核对（索引里的路径必须存在，`docs/adr/` 之外的文档必须出现在索引里）。
ADR 单独一张台账，见 `docs/adr/README.md`。

- `docs/COMPATIBILITY.md` — 改响应形状、加/删别名、删旧键之前：哪些入口必须留、哪些能删、删之前先做什么。
- `docs/benchmarks/README.md` — 引用 `docs/benchmarks/*.json` 里的性能数字之前看它：怎么读、哪些不能当结论、P0 的实测结论是什么。
- `docs/community/COMMUNITY_DATA_NOTICE.md` — 引用或再分发社区资料（Starside）之前看授权与边界。
- `docs/community/小黑盒_功能总览.md` — 面向中文玩家的八工具功能总览；写对外说明或话术时对齐口径。
- `docs/community/小黑盒_更新公告_2026-09-18.md` — 上次发文以来的**新增与修复**发布稿（计数器/生涯三档/纯 PvP 武器榜 + 一串口径修复）；要发更新文章或对齐对外口径时看它。
- `docs/plans/ARMOR_FORMAT_PLAN.md` — 动护甲载荷格式（体积口径、字段取舍）之前看实机证据与取舍。
- `docs/plans/EQUIP_FLOW_PLAN.md` — 动装备流程（候选签发 → `confirmed` → 回读）之前看它为什么长这样。
- `docs/plans/HOST_COMPAT_PLAN.md` — 要支持「只发标量」的宿主（豆包 connector 这类）时看它：哪些参数收文本写法、为什么 `equip_build` 改成也能收 `execution_id`、守门在哪。
- `docs/plans/PVP_STATS_PLAN.md` — 动生涯/赛季战绩（计数器 vs 统计接口、模式与角色范围标注）之前看它为什么分三档。
- `docs/plans/LEGACY_SURFACE_REMOVAL_PLAN.md` — 要动历史工具面（`full`/`expert` profile、`ENABLE_LEGACY_TOOLS`）或配装导入入口之前看它：删什么、导入怎么搬、怎么验都在里面。
- `docs/plans/PATTERN_QUERY_PLAN.md` — 要做锻造武器模式（玩家说的红框、「模式和催化」那一页、游戏里的「模式进度 4/5」）查询之前看它：进度只在组件 900、183 vs 219 两个口径的实测证据都在里面。
- `docs/plans/ROTATION_PLAN.md` — 要做周常轮换（夜幕/宗师词缀、遗失区域、上维挑战、泉源、异域任务、突袭/地牢特色）之前看它：官方接口给了哪半、哪半只能维护周期表、锚点与诚实口径都在里面。
- `docs/plans/RESPONSE_PROJECTION_PLAN.md` — 要动「响应投影」（默认出口给行还是给整包）之前看它：两个大响应的实测基线、这轮只投影 `find`/`recommend` 与 `duplicates` 的边界、`execution_id` 作为引用协议的口径，以及冻结清单。
- `docs/plans/PERFORMANCE_PLAN.md` — 要动性能（搜索 N+1、武器目录全量展开、载荷体积、启动、并发与缓存）之前看它：六项的实测基线与验收口径，外加**第七项（装备流程的往返次数）的基线、四步方案与实测口径**。
- `docs/plans/SOLVER_OPTIMALITY_PLAN.md` — 要动护甲求解器的**目标函数与排序**（"刚好达标"、上限、偏好、调谐进搜索、可达区间）之前看它：根因的代码位置、五条待拍板口径与 P0–P6 阶段划分都在里面。
- `docs/plans/PVP_WEAPON_BOARD_PLAN.md` — 动 PvP 武器榜（PGCR 窗口聚合、成本与并发实测、模式名出处）之前看它为什么只能给"最近 N 场"。
- `docs/plans/STARSIDE_ENTITY_PLAN.md` — 要把 Starside 作者给的新归档（按 hash 的武器推荐/评语/神器关联/帧级 DPS）
  接进工具面之前看它：全量字段与覆盖、hash 归一纪律、标记解析表、八条硬口径与 P0–P5 阶段都在里面。
- `docs/plans/TUNING_WRITE_PLAN.md` — 要把「代写调谐」做成正式功能（`equip_build`/`equip_mod` 写调谐）之前看它：
  三条硬口径（只写组件 310 清单里的、并进 `mods` 走同一执行器、写不进去不回退）与守门/验收清单都在里面。
- `docs/plans/SUBCLASS_ARTIFACT_PLAN.md` — 动子职业/神器读写之前看计划与取舍。
- `docs/plans/WEAPON_FORMAT_PLAN.md` — 动武器载荷格式之前看计划与实机证据。
- `docs/reference/bungie_api.md` — 遇到 Bungie API 问题（scope、端点、AWA、组件号、上游错误码）时的实测事实清单；与官方文档冲突时以它为准。
- `docs/testing/TESTING_CORPUS.md` — 改响应形状、加 intent、发版之前必读：测试分层与语料总纲。
- `docs/testing/TESTING_CORPUS_FULL.md` — 要跑逐 intent 的全量回归、或查某个字段该长什么样时看它。
- `docs/testing/TESTING_PHRASES.md` — 要**手动**试功能（照着念的口语清单：一句话一个功能、组合句、必须拒绝的句子）时看它；和代码行的语料分工不同。

## 决策记录（ADR）

架构级决定写在 `docs/adr/`：索引与编号规矩在 `docs/adr/README.md`，格式照 `docs/adr/TEMPLATE.md`。

- **什么时候必须写一条**：改对外契约（响应形状、工具与 intent 面）、破坏性变更、推翻既有决定，
  以及"上游就长这样"的结论（例：护甲模组必须游戏内装）。
- **写在哪**：`docs/adr/NNN-kebab-title.md`，标题行 `# ADR-NNN: 标题`；编号连续、不复用，
  写完在 `docs/adr/README.md` 补一行。
- **分工**：ADR 只记**决定了什么和为什么**（连同被否掉的选项与代价）；进度、实测数据、逐次取舍
  仍写 `docs/plans/`，别把两件事混进一个文件。

`tests/test_agent_docs.py` 核对编号连续无重复、每条 ADR 都有 Status/Date、README 与实际文件双向一致。

---
> Source: [8z2wz746tv-png/destiny2-mcp](https://github.com/8z2wz746tv-png/destiny2-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
