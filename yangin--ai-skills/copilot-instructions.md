## ai-tech-review-committee

> 当用户要求软件方案评审、技术评审委员会、多角色专家分析、架构评审、PRD/RFC/ADR/MVP 评审、AI/数据系统评审、迁移/上线/重构方案评审，或希望从产品、架构、工程、安全、SRE、QA、数据/AI、UX、交付成本角度补强方案并给出 Go/Rework 判断时使用；不要用于明确 bugfix、直接编码、普通 API 解释或普通代码 review。


# AI 技术评审委员会入口规则

当本规则被 Cursor Agent 选中时，按 `ai-tech-review-committee/SKILL.md` 的委员会流程执行评审。

若当前上下文无法读取该 Skill 文件，则按以下最小流程执行：

1. `Intake`：复述目标、用户、当前方案、约束、假设和缺口。
2. `Role Selection`：选择主席、产品、架构、工程、安全、SRE，并按需加入 QA、Data/AI、UX、交付成本。
3. `Individual Expert Review`：各角色先独立给出 verdict、concerns、recommendations、questions。
4. `Cross-Examination`：指出角色之间的关键 trade-off。
5. `Chair Synthesis`：给出 `Go / Go with changes / Rework / Discovery needed`。
6. `Action Plan`：输出 Now / Next / Before Launch。

---
> Source: [yangin/ai-skills](https://github.com/yangin/ai-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
