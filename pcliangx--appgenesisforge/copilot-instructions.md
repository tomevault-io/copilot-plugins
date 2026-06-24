## appgenesisforge

> > **Template Version: v6.8.0**（min Claude Code v2.1.154）。逐版本变更见 [CHANGELOG](CHANGELOG.md)。

# AppGenesisForge

> **Template Version: v6.8.0**（min Claude Code v2.1.154）。逐版本变更见 [CHANGELOG](CHANGELOG.md)。

## Project Overview

AI 应用开发项目，前端 Web、后端服务、AI Agent、Apple 原生（macOS / iOS）多方向并行。

**当前仓库形态**：AI 团队协作模板（脚手架）——`.claude/` 团队配置与 `docs/` 规范就绪，Tech Stack 已落地（见 ADR-000）；尚无应用代码，等待执行层在首个 feature 启动时创建 `backend/` 与 `frontend/`（Apple 轨为 `apple/`，按 ADR-007）。

## Tech Stack

技术栈**详尽决策**（备选、理由、版本查证）唯一来源是 [`docs/adr/000-system-architecture.md`](docs/adr/000-system-architecture.md)。本文件 `## Tech Stack` 仅放当前生效版本号摘要 + ADR 链接，不重复决策理由。

**当前摘要**：React + Vite、FastAPI + PostgreSQL、uv（Python 包管理 + `uv.lock` 依赖锁 + `.python-version` 解释器 pin）、SQLAlchemy + Alembic、orval（前后端 OpenAPI 契约同步，见 ADR-006）、DeepSeek/Doubao/Qwen/MiniMax 多 LLM、docker-compose 仅编排 Postgres；本地开发一键启动见根 `Makefile`（`make dev` / `make help`；UAT 仍走 deploy-engineer 的 `/agf-deploy-uat`）。详见 ADR-000。**Apple 原生轨**：Swift 6.2 + SwiftUI（[ADR-007](docs/adr/007-apple-native-stack.md)）、swift-openapi-generator（Apple 契约同步，[ADR-008](docs/adr/008-apple-backend-contract-sync.md)）、fastlane + notarytool 四渠道发布（[ADR-009](docs/adr/009-apple-release-pipeline.md)）。

**任何技术栈调整必须新开 ADR**（由 `tech-lead` 撰写）+ 同步更新本段摘要的版本号 / 链接。

## Project-Specific Rules

- 本文件只放项目特有规则，团队通用规则统一放在 `.claude/standards/`，结构性指引在 `.claude/rules/`。
- 项目涉及外部合规、部署约束、第三方集成限制时，在此处追加。
- 设计交付物路径约定：项目级 `docs/design/DESIGN.md`（**设计 token SSOT**，维护者 uiux-designer，各 feature 只引用其 token，禁内联重声明色板/间距）+ 每 feature `docs/design/[feature]/spec.md`（设计规范）+ `docs/design/[feature]/index.html`（自包含静态原型，Tailwind CDN 或内联样式，可直接 `open` 打开），资源放 `assets/`。设计 token 纪律见 `.claude/standards/coding.md` 设计 token 纪律，reviewer 审查项见 `.claude/agents/code-reviewer.md` 设计 token 审查项。
- 修改 `.claude/agents/*.md` 的职责或产出时，必须同步检查 `docs/team-capability-map.md`（"主要输出"列）与 `.claude/standards/team-roles.md`（仅当工具/skill/推荐 mode 变更时）。
- 前后端对接（前端调后端 API；防"按钮没事件 / 契约对不上"两类下游高频缺陷）→ 契约纪律见 `.claude/standards/coding.md` 前后端契约纪律 + [ADR-006](docs/adr/006-frontend-backend-contract-sync.md)（OpenAPI=SSOT，前端走 orval 生成，禁手写 fetch/类型/mock）；测试侧强制覆盖（含交互完整性 + E2E 控件遍历）见 `.claude/standards/testing.md` 前后端对接强制覆盖项。
- 涉及多 LLM SDK 接入（DeepSeek/Doubao/Qwen/MiniMax 切换、fallback、env 变量）→ 必须先看 skill `.claude/skills/agf-wiring-multi-llm-sdk/`。
- **Apple 原生轨（macOS / iOS）**：平台 target 是 task 参数（`macos` / `ios` / `universal`，派工必声明，见 `.claude/standards/apple-native.md` §2）；Apple 客户端契约纪律（swift-openapi-generator 生成 client，禁手写 URLSession / DTO / mock）见 `coding.md` Apple 契约纪律 + ADR-008；Apple SIT 自跑走 skill `agf-running-apple-sit`；app 内接 LLM 先看 skill `agf-wiring-apple-llm`（密钥永不进客户端）。
- Apple feature 合并到 main 后（apple code review 含 SIT Audit 通过 + PL 合并），product-lead 必须**提示用户**是否构建签名分发包；确认后派 `apple-release-engineer` 按 skill `agf-releasing-apple` 构建（fastlane match 签名 + 公证 + 渠道 lane）并冒烟自检，二元 gate（`✅ 构建成功（冒烟通过）` / `❌ 构建失败`）；通过后 apple-qa-engineer 对该分发包跑 E2E / UAT。渠道 → lane 映射权威源见 `.claude/standards/deployment.md` §7；手动触发 `/agf-apple-release`。apple-release-engineer Pool=1（禁 pool，唯一签名身份）。
- SIT 测试由 execution-layer dev 自跑（不是独立 phase）→ 必须按 skill `.claude/skills/agf-running-sit-tests/` 流程；证据落到 `progress/<role>.md` 的 `**SIT 证据**` 段，由 `code-reviewer` 在 code review 时 audit（参见 `.claude/agents/code-reviewer.md` `## SIT Audit` 节）。dev 报告前 + reviewer audit **step 0** 都先跑 `bash .claude/scripts/agf-sit-precheck.sh progress/<role>.md` 机筛证据（placeholder / 漏证据 / 标记矛盾，advisory 不阻断，[ADR-011](docs/adr/011-delivery-pipeline-efficiency.md) 决策 2）。
- 写 E2E / UAT 报告 → skill `.claude/skills/agf-writing-qa-report/`（SIT 不在此 skill 范围，证据走 dev 自跑流程）。
- **UAT 执行前必有用例文档 + 用户审核**（MAJOR / MINOR 强制）：qa-engineer 先按模板 `docs/qa/uat-cases-_TEMPLATE.md` 生成 `docs/qa/<feature>-uat-cases-<date>.md`（每条 AC ≥1 用例、6 字段 + AC 覆盖矩阵 + 界面渲染核查矩阵），**用户审核确认（frontmatter `status: Approved`）后才开测**（用例可在 **dev 实现期并行起草**、审批移出关键路径，[ADR-011](docs/adr/011-delivery-pipeline-efficiency.md) 决策 1）；证据执行时回填用例文档（SSOT），UAT 报告只引用用例 ID；**UAT 每个用户可见界面必须 chrome-devtools 真渲染 + 截图 + 读图四查（导航 / 裁切 / 控件可点 / 视觉达标——截图必须用 Read 视觉读回、对照 design spec 核是否达到可交付用户标准），纯 API 断言不构成含界面用例的 Pass**。PATCH 级 hotfix 可由 product-lead 显式豁免。细则见 `.claude/standards/testing.md`「UAT 用例文档」+「UAT 界面渲染核查」节。
- **交付 lane（full / fast，[ADR-011](docs/adr/011-delivery-pipeline-efficiency.md) 决策 3）**：PL 派单按规模 + 风险**显式选**——**full**（默认；Medium/Large、MINOR/MAJOR、任何高风险）走全套尾部门；**fast**（仅 Small + PATCH + 非高风险，显式选 + 记录风险接受）**只减不跳**（仍部署 + 冒烟 + P0 pass² + 受影响界面渲染核查，E2E 缩到改动面目标 AC）。高风险（auth / schema migration / LLM 切换 / cross-cutting）一律 full、不得 fast。SSOT 见 `.claude/standards/workflow.md` §交付 lane。
- 写 PRD → skill `agf-writing-prd`；写 ADR → skill `agf-writing-adr`。
- 程序化生成中文 docx 报告（决议书 / 评审 / 投标书等高密度文档）→ skill `agf-writing-docx-reports`（docx-js）；程序化生成中文 pptx（制度 / 党政 / 宣贯 deck）→ skill `agf-writing-pptx-reports`（python-pptx）。两者依赖 `.claude/skills/docx/` 与 `.claude/skills/pptx/`（Anthropic 第三方低层 skill，附带 `scripts/office/soffice.py` 做 PDF 预览闭环）。
- 在仓库提 GitHub issue（手工创建 / 报 bug / dev 在 SIT 中发现 P0/P1 自动 path / qa-engineer 在 E2E/UAT 中发现 P0/P1 自动 path）→ skill `agf-writing-github-issue`（含标签锁定 + 最小输入模式）。
- **Multi-instance Worker Pool**（dev / reviewer / qa 同 type ≥ 2 task 自动 fan-out）→ 见 [ADR-001](docs/adr/001-multi-instance-worker-pool.md) + `workflow.md` §Multi-instance Worker Pool；实例命名 `<type>-<N>`，pool 上限按 `team-roles.md` `Pool 上限` 列；PL 用 `bash .claude/scripts/agf-matrix.sh --type=progress|review|qa` fan-in 决策。
- Release 推 tag + `gh release create` 完成后（MAJOR / MINOR）→ **提醒用户**可跑 `/agf-release-retro vX.Y.Z` 做复盘（skill `.claude/skills/agf-running-release-retro/` 产出 `docs/reviews/retro-vX.Y.Z-YYYY-MM-DD.md`）；**非强制**——有 team 交付周期数据时价值高，maintainer-direct session 可跳过；PATCH 不提醒。
- 代码 merge 到 main 后（code review 含 SIT Audit 通过 + PL 合并），product-lead 必须**提示用户**是否拉取合并代码部署 UAT；确认后派 `deploy-engineer` 按 skill `agf-deploying-uat` 部署隔离栈（独立 compose project + 端口偏移）并冒烟自检，二元 gate（`✅ 部署成功（冒烟通过）` / `❌ 部署失败`，不发明新 verdict）；通过后 qa-engineer 对该 UAT 栈跑 E2E / UAT。隔离契约（端口字面值权威源）见 `.claude/standards/deployment.md`「UAT 环境部署」节；手动触发 `/agf-deploy-uat`。deploy-engineer 进 Pool=1（禁 pool，唯一 UAT 环境）。
- 实施计划（派工前用 `superpowers:writing-plans` 生成）必须轻量化 —— 参见 `.claude/standards/plans-format.md`（无 inline 代码、无逐 step commit、每 Task ≤ 3 步、整个 Plan ≤ 500 字）。
- 仓库目录约定见 `.claude/rules/repo-layout.md`；Team Mode 启动协议见 `.claude/rules/team-mode.md`（按 path 自动加载）。

## Verified Facts

> AGF 模板本体硬事实（Pool 上限 / progress 5 段格式 / Verdict 词表 / Worktree baseRef / scan-secrets 厂商数 / 角色能力 SSOT 等）已下沉为 path-scoped 规则 [`.claude/rules/verified-facts.md`](.claude/rules/verified-facts.md)，仅在编辑模板内部（`.claude/agents|standards|hooks|scripts|commands` / `docs/adr`）时**自动加载**，不再常驻本文件 context（官方建议 CLAUDE.md 精简、path-scoped 规则按需加载省 context，见 [memory](https://code.claude.com/docs/en/memory)）。组件清单 / 计数一律以实际目录为准（`ls .claude/...`），不在文档里重复声明。

## Tool Boundaries

四层 hook 防御（注册位置：`.claude/settings.json` + `.git/hooks/pre-commit`，文档：`.claude/standards/security.md`）：

1. **`PreToolUse` (Bash) — `block-dangerous-bash.sh`**：硬阻断 `rm -rf` / `DROP TABLE` / `git push --force` / `git reset --hard` / `curl|sh` 下载即执行。
2. **`UserPromptSubmit` — `scan-secrets.sh`**：硬阻断 AWS/GitHub/OpenAI/Anthropic/Google/Slack/DeepSeek/Doubao/Qwen/MiniMax 密钥 + Apple 签名材料（ASC API key / match 密码）+ PEM/SSH/PuTTY 私钥 + BIP39 助记词。
3. **`PostToolUse` (WebFetch/WebSearch/Read/Bash/mcp__*) — `sanitize-tool-output.sh`**：软告警外部内容里的 prompt-injection 指令（含所有 MCP 工具输出）。
4. **`pre-commit` (git) — `scan-commit.sh`**：commit 前对 staged diff 跑同套 secret 正则，防 Edit/Write 绕过 prompt 扫描。安装：`ln -sf ../../.claude/hooks/scan-commit.sh .git/hooks/pre-commit`。

**第 5 层（推荐叠加，非 AGF 自有）**：`security-guidance` plugin（Anthropic 原生 · 免费 · `PostToolUse` hook，编辑落地后扫描告警、**不阻断写入/提交**）补**代码级危险模式**（`eval`/XSS/`pickle`/`os.system`/`child_process` 等，AGF 四层不覆盖）。安装 `/plugin install security-guidance@claude-plugins-official`；优雅降级（未装不影响四层），详 [`security.md`](.claude/standards/security.md) "第 5 层"。

`.claude/settings.json` 的 `permissions.deny` 已禁读 `.env*`、`~/.ssh/**`、`~/.aws/**`、`~/.gnupg/**` 等敏感路径 + Apple 签名材料二进制（`*.p12` / `*.mobileprovision` / `*.p8`，文本扫描抓不到 → 按扩展名拦读），并禁 `eval` 等远程执行链路（`curl|sh` 下载即执行改由第 1 层 `block-dangerous-bash.sh` 可靠拦截——官方判定约束参数的权限 glob 脆弱，详 [`security.md`](.claude/standards/security.md)）。

附加 workflow hooks（**不属于安全防御**，仅维护团队工作流）：

5. **`TeammateIdle` — `teammate-keepalive.sh`**：task list 还有 pending 时阻止 teammate 提前 idle。
6. **`SubagentStop` / `TeammateIdle` — `check-progress-file.sh`**：执行层 role 退出时若 `progress/<role>.md` 缺 SIT 证据段则阻断。
7. **`TaskCreated` — `validate-task-schema.sh`**：任务创建时校验 task 6 段齐全（description / 上下游产物 / AC 等），漏字段 exit 2 阻止创建（官方专用 `TaskCreated` 事件，替代旧 `PreToolUse(TaskCreate)` matcher）。
8. **`SessionStart` — `session-start-context.sh`**：`resume`/`compact` 时注入团队态恢复提示（重读 `progress/` + 跑 `agf-tasks.sh`），`startup`/`resume` 时设 session 标题；不阻断。
9. **`SubagentStop` / `TeammateIdle` — `validate-verdict.sh`**：reviewer（code-reviewer / miniapp-code-reviewer / apple-code-reviewer）+ qa（qa-engineer / miniapp-qa-engineer / apple-qa-engineer）共 6 role 退出时，从**报告 frontmatter**（唯一 SSOT，无注释块）重算 verdict 守门——薄 bash 包装委托 `agf-verdict.py`：code（`critical_count>0→block` 等）+ SIT Audit（`sit_checks` 含 fail→Redo SIT）硬拦、QA 客观底线（`critical_defect_count>0` 非 Block / P0 未全 pass² 却 Promote）拦不可能组合，声明≠推导 exit 2 打回。堵"有 Critical 却 approve"。极保守 fail-open（非目标 role / 缺字段 / 坏 yaml / 异常一律放行）。详 [ADR-003](docs/adr/003-verdict-from-findings.md) → [ADR-010](docs/adr/010-structured-verdict-contract.md)。

撞到硬阻断时按 `.claude/standards/security.md` "No Equivalent Bypass" 处理（不得寻找等价绕过）。

## Team Runtime Contract

本项目复用 `.claude/` 中的 AI 团队模板。权威来源：
- 角色与协作边界：`.claude/standards/team-roles.md`
- 工作流（含 Parallel Dispatch + worktree 强制）：`.claude/standards/workflow.md`
- 文档与单一来源原则：`.claude/standards/document-rules.md`
- Superpowers 使用规范：`.claude/standards/superpowers.md`
- 测试与验收：`.claude/standards/testing.md`、`.claude/standards/ac-lifecycle.md`
- 编码与安全：`.claude/standards/coding.md`、`.claude/standards/security.md`
- 观测与成本：`.claude/standards/observability.md`、`.claude/standards/cost-budget.md`
- 版本与发布：`.claude/standards/versioning.md`（SemVer 应用细则 + release 流程）
- 对话回复风格：`.claude/standards/communication.md`（所有 agent 面向人的回复精简，docs 产出物不受影响）

运行时关键约束（摘要，细则以权威来源为准）：

- 角色能力（额外工具 + 预加载 skills）的唯一 SSOT 是 `.claude/agents/roles.yaml`；`.claude/standards/team-roles.md` 两张能力表与各 agent `.md` frontmatter 均为其**生成物**（`.claude/scripts/gen-roles.py` 产出，drift 由 `lint-all.sh` 硬阻断），改能力编辑 roles.yaml 后重跑生成器、勿手改生成物。表中 `Permission` 列是**团队约定的"推荐运行模式"**，并非 Claude Code 官方 sub-agent frontmatter 字段。
- `product-lead` 是唯一流程编排者；`tech-lead` 仅在缺少基线、新技术选型或架构风险升级时强制介入。
- 会话入口判断（直接执行 vs 派给 `product-lead`）见 `.claude/standards/workflow.md` "Session Entry" 节。
- Team Mode 启动条件与协议见 `.claude/rules/team-mode.md`（path 命中 `.claude/agents/**` 等时自动加载）。
- 交付链路固定为 `代码实现 + Unit + SIT 自跑 → code review (含 SIT Audit) → UAT 部署 → E2E → UAT`；任一阶段失败后，由 `product-lead` 重新分派执行层修复。
- `code-reviewer` 为 review-only 角色；`qa-engineer` 负责执行 E2E / UAT 并提交报告（集成层 SIT 由 dev 自跑、code-reviewer 审证据），最终业务签字由 `product-lead` 完成。
- `deploy-engineer` 为 deploy-only 角色：code review 通过 + PL 合并到 main 后，按 skill `agf-deploying-uat` 部署隔离 UAT 栈（独立 compose project + 端口偏移）并冒烟自检，供 qa-engineer 对该共享栈跑 E2E / UAT；不修源码（代码层失败退回 PL → dev）。
- `apple-release-engineer` 为 Apple 轨的 deploy-only 对应角色：合并到 main 后按 skill `agf-releasing-apple` 构建签名分发包（match 签名 / 公证 / 渠道 lane）并冒烟自检，供 apple-qa-engineer 对分发包跑 E2E / UAT；不修业务源码（`apple/fastlane/` 配置是其可写域）。
- `backend-dev` / `ai-agent-dev` 在高风险变更（schema 迁移、认证逻辑、LLM 提供商切换、生产 prompt 变更等）必须先进 Plan Mode 拿 product-lead 授权。
- token 消耗与成本按 `.claude/standards/cost-budget.md` 分级（含会话规模上限、cache 利用率 ≥ 60% 与全 agent 默认 model 路由）；如需查看本次会话用量，跑 Claude Code 内置 `/usage`。
- 对话回复一律按 `.claude/standards/communication.md` 执行；写入 `docs/` 的产出物不压缩。

若本文件中的项目规则与团队规范冲突，以本文件为准。

---
> Source: [pcliangx/AppGenesisForge](https://github.com/pcliangx/AppGenesisForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
