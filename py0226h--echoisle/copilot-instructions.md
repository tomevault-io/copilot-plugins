## echoisle

> 1. If the task is `dev`, read `/Users/panyihang/Documents/EchoIsle/docs/harness/task-flows/dev.md`.

# AGENTS.md

## Task Flow Entry

1. If the task is `dev`, read `/Users/panyihang/Documents/EchoIsle/docs/harness/task-flows/dev.md`.
2. If the task is `refactor` or optimization, read `/Users/panyihang/Documents/EchoIsle/docs/harness/task-flows/refactor.md`.
3. If the task is `non-dev`, read `/Users/panyihang/Documents/EchoIsle/docs/harness/task-flows/non-dev.md`.
4. If the task is stage closure, read `/Users/panyihang/Documents/EchoIsle/docs/harness/task-flows/stage-closure.md`.
5. If no task type clearly matches, use `AGENTS.md`, the user's request, and relevant skill descriptions to decide whether a skill is needed.
6. Do not manually infer a full pre/post skill chain from `AGENTS.md`; use the matching task flow and lifecycle stage.

---

## Quick Rules

These are the rules that remain directly visible in `AGENTS.md`.
Detailed explanations now live under `docs/harness/`.

### Mandatory Python rule

1. For any turn that runs Python commands in this repository, run `python-venv-guard` first.
2. Never use global interpreters: `python`, `python3`, `pip`, `pip3`.
3. Always use `/Users/panyihang/Documents/EchoIsle/ai_judge_service/.venv/bin/python`.

### Mandatory PRD rule

1. For any development or refactor/optimization turn that changes project code, architecture, or module behavior, run `pre-module-prd-goal-guard` before coding.
2. The authority PRD remains `/Users/panyihang/Documents/EchoIsle/docs/PRD/在线辩论AI裁判平台完整PRD.md`.
3. The default fast path is now `/Users/panyihang/Documents/EchoIsle/docs/harness/product-goals.md`; high-risk modules must fall back to the authority PRD.

### Current code fact rule

1. 回答本项目的代码、行为、调用链、接口、测试、配置或架构现状问题时，必须优先基于当前工作区代码事实。
2. 文档只作为导航线索或背景，不得替代代码事实；若文档与代码不一致，以当前代码为准，并在回答中说明冲突。
3. 给出确定性结论前，应通过 `rg`、文件阅读、测试、类型定义、配置或实际运行结果核验。
4. 涉及具体实现结论时，优先给出文件路径、函数/类型/测试位置作为依据；无法核验时明确标注“不确定/未验证”。

### Architecture map freshness rule

1. 每次 dev 或 refactor/optimization 修改代码结构、模块边界、主入口、跨层调用路径、workspace/package 成员、主要页面/handler/service/domain 位置后，必须判断 `docs/architecture/README.md` 是否需要同步更新。
2. 只有影响“第一跳定位”的变化才更新代码地图；内部函数实现、局部算法、测试细节、临时脚本变化通常不更新。
3. 更新时保持代码地图轻量，只记录主线子系统、职责、需求反查入口和非优先目录，不展开成完整架构说明。

### Pre-release compatibility rule

1. EchoIsle is still in local development and has not been released to production users.
2. By default, do not preserve compatibility layers, gray rollout paths, legacy fallbacks, dual-write logic, adapter shims, or parallel old/new code paths for not-yet-released behavior.
3. Default to hard cutover and direct cleanup when implementing new functionality, refactors, or optimizations.
4. Only keep a temporary compatibility layer when at least one of the following is true:
   - the user explicitly asks to preserve compatibility
   - multiple active in-repo callers cannot be updated in the same turn
   - a migration, test baseline, or script cutover truly requires a short transition window
5. Any temporary compatibility layer must state its removal condition in code comments or plan notes; do not leave indefinite fallback paths in place.

### Comment style rule

1. EchoIsle 默认使用精简中文注释，但只在代码本身不够自解释时添加。
2. 注释优先说明“为什么这样做”、“这段逻辑在保护什么边界”或“这里在防什么风险”，不要逐行翻译代码表面行为。
3. 新增事务补偿、Redis/DB 一致性收敛、并发/锁语义、幂等保护、时序约束、复杂分支判定时，应补精简中文注释。
4. 简单赋值、普通 CRUD、显而易见的控制流不要为了“有注释”而加注释，避免制造注释噪音。
5. 注释默认控制在 1 到 2 行，除非用户明确要求更详细的说明。

### Test authoring rule

1. 测试必须验证真实业务语义、真实代码路径和真实契约；优先通过公开 API、handler、service、domain 包或既有测试工具触达被测逻辑，不要在测试里重写一套实现逻辑来证明自己正确。
2. 期望值应来自 PRD/业务规则、协议契约、固定 fixture、可独立推导的不变量或明确的输入输出样例；不要复制生产代码算法、私有分支或当前 bug 行为来生成 expected。
3. 禁止为了让测试通过而修改生产语义：不得加入 test-only 分支、静默兜底、宽松解析、假成功返回、吞错、无边界重试、隐式降级或只服务测试的兼容层。确需测试接缝时，优先使用既有依赖注入、fixture、mock server 或测试配置，并保持用户可见语义不变。
4. Mock / stub 只能用于外部不可控边界（网络、时间、随机数、第三方服务、系统 IO 等），不得 mock 掉被测主体本身；使用 mock 时要断言关键入参、权限、状态迁移、错误语义或外发契约，避免只断言“被调用过”。
5. 不得通过降低断言、删除断言、无理由更新 snapshot/golden、标记 skip/ignore/flaky、放宽 matcher、硬拉 timeout、依赖本机残留状态或调用无关外部工具来制造绿灯。确需调整时，必须在最终说明中解释业务依据和风险。
6. 失败测试的处理顺序是：先判断产品/契约预期，再修真实实现或修正错误测试假设；不要把失败当成要绕过的障碍。若发现现有测试与真实业务冲突，应说明冲突来源并按当前 PRD/契约收敛。
7. 新增或修改测试时，应覆盖本次改动最可能破坏的负向路径、边界条件、权限边界、幂等/并发语义、Redis/DB 一致性或跨层契约；如果某类风险不适用，可以不补，但不要只写快乐路径。
8. 最终回复必须说明新增/修改的测试保护了什么真实行为，以及实际运行了哪些测试或因何受阻；不能把“未运行”“环境阻塞”表述成“已通过”。
9. 编写测试时，优先比较整个对象是否相等，而不是逐字段比较。

### Backend / chat rule

1. 修改 `chat/` Rust 主线时，优先保护事务边界、幂等语义、Redis/DB 一致性、事件/outbox 收敛和权限边界，不要只看接口表面行为。
2. 新增或修改后端接口时，应同步检查 route、handler、model、RBAC / middleware、OpenAPI、错误语义与测试是否一起更新。
3. 对尚未发布的能力，默认直接切主链并清理旧路径，不为“未来可能兼容”预留长期双轨逻辑。
4. 对复杂一致性保护、补偿、时序或防重逻辑，补精简中文注释，优先解释边界和风险。

### API contract & cross-layer sync rule

1. 只要改动 API、DTO、错误码、分页字段、状态字段或 WS payload，就必须同步检查跨层调用方。
2. 后端契约变更时，至少同步检查 `openapi.rs`、前端相关 domain / SDK（如 `auth-sdk`、`realtime-sdk`、`ops-domain`）、必要测试，以及需要留痕的学习或计划文档。
3. 默认保持单一主语义字段，不为未发布能力长期保留 alias 字段、双字段并存或旧新 payload 双写。
4. 如果一个回合内无法同步所有调用方，才允许保留短期兼容层，并在计划或注释里写清移除条件。

### Frontend / journey rule

1. 修改 `frontend/` 时，优先把共享业务逻辑放在 `packages/*`，应用壳 `apps/web` 与 `apps/desktop` 只保留平台装配和入口差异。
2. 改动登录、绑手机、Lobby、Room、Wallet、Ops 等主流程时，应同时检查页面层、domain / SDK 层、路由守卫和异常提示是否一致。
3. 已有 Web/Desktop 共用语义时，优先收敛到共享包，不要在双端重复堆逻辑。
4. 前端主流程改动完成后，优先补 smoke 或专项运行态证据，而不只停留在静态 typecheck。

---

## Harness Docs

Use these files as the detailed source of truth:

1. [docs/harness/task-flows/README.md](/Users/panyihang/Documents/EchoIsle/docs/harness/task-flows/README.md): task-specific lifecycle flows for dev/refactor/non-dev/stage-closure
2. [docs/harness/product-goals.md](/Users/panyihang/Documents/EchoIsle/docs/harness/product-goals.md): summary-first product constraints for everyday module work
3. [docs/harness/runtime-verify.md](/Users/panyihang/Documents/EchoIsle/docs/harness/runtime-verify.md): current verification model and the gap before unified runtime verify lands
4. [docs/harness/doc-governance.md](/Users/panyihang/Documents/EchoIsle/docs/harness/doc-governance.md): plan docs, evidence docs, explanation/interview docs, and current document ownership rules
5. [docs/harness/quality-gates.md](/Users/panyihang/Documents/EchoIsle/docs/harness/quality-gates.md): current quality gates, guards, and CI responsibilities
6. [docs/harness/usage-tutorial.md](/Users/panyihang/Documents/EchoIsle/docs/harness/usage-tutorial.md): current day-to-day usage tutorial

---

## Project Map

Use this file when you need a lightweight codebase map before opening implementation files:

1. [docs/architecture/README.md](/Users/panyihang/Documents/EchoIsle/docs/architecture/README.md): current code map for backend, AI service, frontend, harness, and "where to look first"

---

## Agent Surface Sync

1. `AGENTS.md` and `skills/*` are the project-level sources of truth for agent instructions and project skills.
2. `CLAUDE.md` and `.claude/skills/*` are Claude Code mirrors generated from those sources; do not edit them manually.
3. After changing `AGENTS.md` or `skills/*`, run `bash scripts/harness/sync_claude_surface.sh`.
4. Use `bash scripts/harness/sync_claude_surface.sh --check` to verify the Claude mirrors are current.

---

## Working Guidance

1. Read only the harness doc section needed for the current task.
2. Prefer concise summaries over loading long documents wholesale.
3. Treat `docs/harness/` as the detailed rules layer, and `AGENTS.md` as the navigation layer.
4. Do not assume future-phase behavior is already active unless a harness doc explicitly marks it as current.
5. Do not use or recreate `module-turn-harness`; it has been retired after adding too much workflow friction.
6. 阶段收口时，`completed.md` 只记录主体完成快照，`todo.md` 只记录延后技术债；不要把活动计划正文原样复制进长期文档。

---
> Source: [PY0226H/EchoIsle](https://github.com/PY0226H/EchoIsle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
