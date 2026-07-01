## harness-control

> 本仓是一套通用的 agent 控制面（harness）：用「最小内核 + 可挂载模块」治理 AI agent 在被管工程上的开发。被管工程挂进 `projects/`（已挂 `kratos-base`）。

# Agent Harness 控制面

本仓是一套通用的 agent 控制面（harness）：用「最小内核 + 可挂载模块」治理 AI agent 在被管工程上的开发。被管工程挂进 `projects/`（已挂 `kratos-base`）。

默认策略：**少读、按需读、渐进加载**。

## 启动顺序

1. 读 `docs/context/CURRENT_STATUS.md`（当前真实状态）。
2. 按 `docs/context/CONTEXT_LOADING.md` 判定本次任务读多少（默认少读、拿不准先低档起步、遇证据再升档）。
3. 需要文档路由时，读 `docs/README.md`。
4. 在某目录**读或改**代码前，加载该位置**向上最近的 `AGENTS.md`**（连同其同级 `CLAUDE.md`）——**就近规则随之生效**；按目录加载与档位叠加，详见 `docs/context/CONTEXT_LOADING.md`。
5. 进入某目录想读 / 动其下文件前，若该目录有 `README.md`，**先读一下**——不必通读，目的是知道这里有什么、该挑哪个。README 不像 AGENTS.md 那样自动加载，靠这条规则触发。

## 规则

下面是 **harness 全局规则**（红线，动手前必看）。**编号 `rule-00NN` 是稳定引用键**，被 eval 考题 / ADR / feature 按号引用；全文就在这里。每条带**隐形标记**供 `rules-index` 扫描。**项目专属规则**沉淀在 `projects/**/AGENTS.md`（就近生效），不堆在这里。

- **改业务代码前先立需求包**：用户可见的需求、行为或验收目标变化，必须先在 `docs/features/` 建需求包并就绪；未就绪 **MUST STOP**（纯控制面 / 文档 / 脚本改动不触发）。 <!-- rule: rule-0001 | sev: blocker | eval: 001 -->
- **blocked / skipped ≠ pass**：验证没真跑通，不许声称通过。 <!-- rule: rule-0002 | sev: blocker | eval: 002 -->
- **不许假完成**：没有真实运行证据，不得声称功能完成或验收通过。 <!-- rule: rule-0003 | sev: blocker | eval: 003 -->
- **按产物/证据/目标文件判加载档，不按关键词**（详见 `docs/context/CONTEXT_LOADING.md`；启动顺序第 2 条是入口）。 <!-- rule: rule-0004 | sev: warn | eval: 004 -->
- **收尾前过 eval**：L2 以上任务、关键决策点，收尾前必须跑 task eval review（独立评委按 rubric 打分）。 <!-- rule: rule-0005 | sev: blocker | eval: 010 -->
- **不碰密钥与危险命令**：不泄露密钥 / token；不执行 `git reset --hard`、`rm -rf /` 等高危命令（hook 会拦）。 <!-- rule: rule-0006 | sev: blocker -->
- **改架构 / 接口须回顾相关 skill**：大改（写了 ADR 或立了 feature）必须回顾 `.agents/skills/`，更新或写明无需更新。 <!-- rule: rule-0007 | sev: warn | eval: 011 -->
- **外部材料不自动采信**：事实源 = 正式文档 + 工程当前代码；外部 / 粘贴材料要先整理验收才算数。 <!-- rule: rule-0008 | sev: blocker -->
- **验收断言必须锚定唯一、真实、产出方的证据**：断言绑到唯一真实信号（防共因污染 / 防超时竞态掩盖）；声称的保证必须有**守护测试**；测试不许为通过而牵强、注释不许撒谎。 <!-- rule: rule-0009 | sev: blocker | eval: 012 -->
- **PRD 产出标准**：产出 PRD 时——先有 approved 用户故事（独立 `user-stories.md` 为上游、PRD 与之对齐）、验收可观测、范围 in+out 闭合、每页四态、（若产出）原型可点通、假设显式确认、可追溯、登记不漂移（仅在产出 PRD 时适用，不强制 PRD 必须存在）。 <!-- rule: rule-0010 | sev: blocker | eval: 013 -->
- **决策与知识必须当轮落文档，落文档提醒兜住遗漏**：改了产物或做了关键决策，知识要就近写进 `AGENTS.md`/`lessons`/规则/ADR/memory；**用户纠正也算**（用户说"不是这样 / 你理解错了 / 撤回 / 你搞混了"时，当轮记一条 `tasks/lessons.md` 三段式：错在哪 / 怎么防 / 怎么更早发现）；Stop hook 机械触发（K 轮 / commit / 变更增量）的 Haiku **落文档提醒**（`scripts/turn-backstop.sh`，=①，非自进化审查）会复查遗漏并写 `tasks/optimization-log.md`，捞到的须落到对应文档、不许烂在 log 里。 <!-- rule: rule-0011 | sev: warn -->
- **状态/索引文档不硬编码可自动生成的枚举**：凡已有 `*-index` 自动生成权威清单的（skill→`.agents/skills/README.md`、规则→`docs/rules/index.yaml` 等），`CURRENT_STATUS` 等状态文档只写"以该自动生成索引为准"的指针，**不复刻计数/清单**——硬编码枚举无 `--check` 守、每次新增就漂（本仓 CURRENT_STATUS 的 skill 清单已三次漂移）。举 1–2 例可以，整列枚举不行。 <!-- rule: rule-0012 | sev: warn | eval: 014 -->
- **非琐碎任务维护 `tasks/todo.md`（标 `level` + 收尾 Review）**：多步 / 动业务码 / L2+ 的任务，动手即在 `tasks/todo.md` 立当前任务并标 `level: L? ｜ task: <名>`（收尾闸据此判要不要 eval，见 rule-0005、档位见 rule-0004）；范围变即更新、收尾前补 Review 段；保持轻——旧块滚进 `tasks/archive/`，不长成流水账。 <!-- rule: rule-0013 | sev: warn -->
- **测试用例产出标准**：产出测试用例时——每条验收点 AC 与每个功能点 FP 都被 ≥1 条用例 `covers:` 覆盖（无遗漏、无悬空引用，`test-cases-audit` 硬闸机检）、用例覆盖正常 / 边界 / 异常、`covers:` 为覆盖关系唯一真相源（不另存手维护映射表）、产物登记不漂移；只管"用例齐不齐 / 覆盖全不全"，**不碰"过没过"**（执行结果另起）；用例真覆盖语义 / 边界异常齐由 eval 考题 015 判（仅在产出测试用例时适用，不强制必须存在）。 <!-- rule: rule-0014 | sev: blocker | eval: 015 -->
- **不擅自 git 写操作**：未经许可不 commit / push / reset / 删分支 / 改 remote。

## 验证

```bash
make verify        # 控制面自检（结构 + 文档 + hook policy 测试 + skills/rules 索引 + CLAUDE.md shim）
make docs-audit    # 文档自检（依赖文件在不在、链接通不通）
make eval          # 跑 task eval review
make hooks         # 安装 git hooks
```

被管工程怎么验证，见 `workspace/verification.yaml` + `docs/harness/VERIFICATION_ROUTING.md`。

## eval

质量不靠 agent 自评：L2+ 任务和关键决策点由 `docs/eval/` 的评委按 rubric 打分，产物写进 `docs/eval/task-reviews/`。eval 题库**独立维护**（`docs/eval/prompts/` + `index.yaml`），按编号引用规则。默认用 **hc-eval 子 agent**（`.claude/agents/hc-eval.md`，免 API key）；CI / headless 可选 `make eval`。触发口径见 `docs/eval/README.md`。

## 工作方式

- 复杂、可并行、或重上下文的任务，**默认拆给子代理（subagent）**做——并行更快，也让主 agent 上下文保持干净。
- 琐碎改动直接做，别为它 spawn 子代理（纯开销）。
- 已有子代理：见自动索引 `.claude/agents/README.md`（如 hc-eval 收尾评分、hc-code-reviewer 给 hc-dev 挑刺）。

## 沟通方式

- 与用户沟通用中文，takeaway 先行，说人话。
- 明确说明验证结果；没有具体检查，不要声称完成。

---

**用户当前明确指令 > 本文件。** 两者冲突时，以用户指令为准。

---
> Source: [harness-base/harness-control](https://github.com/harness-base/harness-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
