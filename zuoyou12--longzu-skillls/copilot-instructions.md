## longzu-skillls

> 本仓库是**作者文风仿写技能库**（江南《龙族》/ 烽火戏诸侯《剑来》），供任何编码/写作 agent 使用。当用户要求以这两位作者的风格创作、续写、润色中文小说时，按以下流程执行。

# AGENTS.md

本仓库是**作者文风仿写技能库**（江南《龙族》/ 烽火戏诸侯《剑来》），供任何编码/写作 agent 使用。当用户要求以这两位作者的风格创作、续写、润色中文小说时，按以下流程执行。

## 工作流程

1. **路由**：现代都市/校园/青春题材 → 江南模式；古典仙侠/江湖/庙堂题材 → 烽火模式；用户点名"杂交"（如"江南的情绪弧线写仙侠"）→ 读 `style-writer/SKILL.md` 的杂交规则
2. **读取规则并严格遵循**：
   - 融合入口（含路由与两作者共享的 7 条叙事骨架）：`style-writer/SKILL.md`
   - 江南皮肤层：`style-writer/references/jiangnan.md`；写龙族同人时另读 `style-writer/references/longzu-characters.md`（角色 OOC 红线）
   - 烽火皮肤层：`style-writer/references/fenghuo.md`
   - 规则的统计学证据（可选深读）：`style-writer/references/corpus-notes-*.md`
3. **成文后审计**：`python style-writer/scripts/audit_style.py --author jiangnan|fenghuo <文件>`（需 Python 3.8+，无第三方依赖）。FAIL 必须修复后重跑；WARN 逐条判断。退出码非零 = 有 FAIL
4. **遵守边界**：各规则文件中的"使用边界"（非商业同人定位）与"语料边界"（江南仅保真龙族期、烽火仅保真剑来期）声明对所有 agent 生效

## 两条铁律（最常见的穿帮点）

- 江南模式：对话动词用"说"体系，**绝不用"道"**（原著 220 万字"笑道"仅 4 次）
- 烽火模式：对话动词用"X道"体系并把神态复合进前缀（笑眯眯道/没好气道），**绝不用"人名+说"**；时间用古法（片刻/一炷香），不出现"几分钟"

## 无文件读取能力的环境

把 `standalone/` 下的单文件版直接粘贴为系统提示或规则：

- `standalone/style-writer-standalone.md` —— 融合版全量（推荐）
- `standalone/jiangnan-standalone.md` / `standalone/fenghuo-standalone.md` —— 单作者版

---
> Source: [zuoyou12/longzu.skillls](https://github.com/zuoyou12/longzu.skillls) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
