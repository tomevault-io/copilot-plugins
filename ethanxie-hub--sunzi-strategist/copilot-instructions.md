## sunzi-strategist

> 把《孙子兵法》维护成一个真实可用的东方战略诊断器 Skill。它服务工作、商业竞争、组织协作、AI 转型和高代价决策，目标是减少误判、降低消耗、提高行动质量。

# AGENTS.md

## 项目使命

把《孙子兵法》维护成一个真实可用的东方战略诊断器 Skill。它服务工作、商业竞争、组织协作、AI 转型和高代价决策，目标是减少误判、降低消耗、提高行动质量。

## 编辑原则

- `sunzi-strategist/SKILL.md` 是调度中枢，保持精炼；详细知识放入 `SYSTEM_PROMPT.md`、`references/`、`feedback/` 和 `examples/`。
- 新增框架先写清现代适用场景、诊断问题、输出建议和常见误用。
- 所有建议必须落到战场、行动、风险、止损线或复盘指标。
- 修改案例时优先使用真实工作和商业场景，必须脱敏。
- 保持中文为主，语气冷静、克制、专业、可执行。

## 禁止事项

- 不把 Skill 写成玄学预测、文化赏析、鸡汤或权谋教程。
- 不引入外部参考提示词里的虚构产品信息或平台说明。
- 不提供欺骗、构陷、报复、骚扰、违法规避或破坏信任关系的方案。
- 不承诺必胜、暴富、确定翻盘或精确胜率。
- 不为了显得完整而重复多个事实源；避免同一规则散落在多个文件。

## 测试要求

改动后至少运行：

```bash
python3 evals/static-checks.py
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py ./sunzi-strategist
```

如改了输出流程或安全边界，还要按 `evals/regression-checklist.md` 手测 B1、B2、B3 和一个右尺寸负例。

## 文档规范

- README 面向真实使用者，说明是什么、能做什么、不能做什么、怎么调用、如何评估。
- examples 必须包含用户输入、Skill 输出、为什么这样诊断。
- feedback 只记录抽象经验，不记录真实个人隐私、客户信息或敏感金额。
- CHANGELOG 只写对用户或维护者有意义的变化。

## 提交前检查清单

- [ ] `git status` 只包含本次任务相关文件。
- [ ] Markdown 链接有效。
- [ ] 没有误写外部参考提示词的产品名或虚构信息。
- [ ] 没有新增玄学化、神化、夸大承诺表达。
- [ ] `dist/sunzi-strategist.skill` 如需发布已重新构建。
- [ ] 安装副本如需本机使用已同步。

---
> Source: [EthanXie-Hub/sunzi-strategist](https://github.com/EthanXie-Hub/sunzi-strategist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
