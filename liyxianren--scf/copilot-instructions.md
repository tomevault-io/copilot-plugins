## scf

> - SCF Hub 当前已经不是泛化校务系统，而是围绕 `auth + enrollment + workflow + oa + feedback + chat` 的内部交付平台。

# AGENTS.md

这是本仓库的 Codex 入口文件。

## 当前事实

- SCF Hub 当前已经不是泛化校务系统，而是围绕 `auth + enrollment + workflow + oa + feedback + chat` 的内部交付平台。
- 网站业务层和数据库是运行期业务真相源。
- 项目交付管理仍然是 `V2` 方向，不属于当前已落地的 `V1` 能力。

## 工作区规则

当当前工作区是 `scf-main` 时，不要把这个仓库当作通用学校系统或通用 SaaS CRM 处理。
先按 SCF 的内部交付业务去理解问题，再决定实现和文档路径。
在当前会话第一次处理 `scf-main` 任务时，先调用一次 `scf-platform-context` 作为项目级 grounding skill，再按任务叠加其他 skills。

## 沟通规则

- 如果当前对话对象是架构师、产品负责人或流程 owner，默认先用业务语言沟通：
  - 角色分工
  - 流程节点
  - 状态迁移
  - 系统边界
  - 风险与约束
- 不要默认用“某个函数 / 某段代码 / 某个文件影响了什么”来解释问题，除非对方明确要求进入实现层。
- 在说明修复方案时，优先说明“业务流程如何调整”，其次才是“系统如何承载这个流程”。

## 阅读顺序

在对产品、架构、流程做判断前，先按顺序阅读：

1. `docs/README.md`
2. `docs/versions/V1-status.md`
3. `docs/handoff/current-focus.md`
4. `docs/versions/V1.md`
5. `docs/architecture/current-system.md`

然后按任务类型继续：

- 当前 OA、auth、排课、反馈、聊天、intake、权限、workflow 相关问题：
  - 先读 `skills/scf-platform-context/SKILL.md`
- 当前 V2 主线问题（短信提醒、销售前端接入、腾讯会议课后反馈）：
  - 先读 `skills/scf-platform-context/SKILL.md`
  - 再读 `docs/versions/V2.md`
- `ProjectTrack / SubProject / Milestone / Risk Board / 2+1 / 1+1+1` 这类 V2 交付模型问题：
  - 先读 `skills/scf-platform-context/SKILL.md`
  - 再读 `skills/scf-delivery-workflow/SKILL.md`
  - 再读 `docs/versions/V2.md`
- 外部产品化、未来多端集成、长期平台方向：
  - 读 `docs/versions/V3.md`
- 文档、handoff、入口文件、版本事实更新：
  - 先读 `skills/scf-docs-governance/SKILL.md`
  - 再读 `docs/governance/docs-governance.md`

## Skill 触发规则

- 只要当前工作区是 `scf-main`，开始处理该项目任务时默认先调用一次 `scf-platform-context`，用于加载 SCF 的业务 framing、系统边界和当前 V2 主线语境。
- 只要任务涉及 `oa`、`auth`、排课、反馈、chat、workflow 边界、短信提醒、销售前端接入、腾讯会议课后反馈、Mini Program、WeChat、Feishu、OpenClaw、通知架构或内部优先产品判断，优先使用 `scf-platform-context`。
- 只要任务进入 V2 的交付对象、里程碑、风险面板、模板化结构设计，使用 `scf-delivery-workflow`。
- 只要任务涉及 `docs/`、`AGENTS.md`、`CLAUDE.md` 的更新，使用 `scf-docs-governance`。
- OpenAI API、模型、平台问题才使用 `openai-docs`。

## Repo Local Skills 规则

- 仓库内 skill 位于 `skills/`，属于 source-controlled guidance assets。
- 如果这些 skills 还没有安装到 `$CODEX_HOME/skills`，就必须先手动阅读对应的本地 `SKILL.md`，再继续工作。
- skills 负责提供阅读顺序、业务 framing 和设计约束，但 `docs/versions/*.md` 仍然是 `V1 / V2 / V3` 的版本事实层。

## V1 / V2 / V3 协议

- 把 `V1-status.md`、`current-focus.md`、`V1.md`、`current-system.md` 当作当前事实包。
- 把 `V2.md` 当作下一阶段交付管理规划，不当作已上线行为。
- 把 `V3.md` 当作远期方向，不当作运行期事实。
- 除非代码和测试已经证明存在，否则不要把 `V2` 或 `V3` 内容说成已经实现。

## 最小验证

```bash
pip install -r requirements.txt
python app.py
pytest -q tests/integration
pytest -q tests/integration/test_enrollment_flow.py tests/integration/test_oa_p1_regressions.py
```

## 维护规则

保持这个文件简短。
详细产品事实、架构事实、路线判断和 handoff 细节应写在 `docs/` 与 repo-local skills 中，而不是继续堆回入口文件。

---
> Source: [liyxianren/scf](https://github.com/liyxianren/scf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
