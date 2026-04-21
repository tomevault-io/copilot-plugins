## research-writing-skill

> 本项目包含科研论文写作技能集。当用户提出与论文写作相关的任务时，请调用相应技能。

# 科研写作助手 (Research Writing Assistant)

本项目包含科研论文写作技能集。当用户提出与论文写作相关的任务时，请调用相应技能。

## 技能列表

| 技能名称 | 路径 | 用途 |
|----------|------|------|
| using-research-writing | skills/using-research-writing/SKILL.md | 入口技能，定义规则和路由 |
| brainstorming-research | skills/brainstorming-research/SKILL.md | 头脑风暴，确定论文类型和结构 |
| writing-chapters | skills/writing-chapters/SKILL.md | 章节写作 |
| latex-output | skills/latex-output/SKILL.md | LaTeX 输出 |
| literature-review | skills/literature-review/SKILL.md | 文献综述 |
| figures-python | skills/figures-python/SKILL.md | Python 图表 |
| peer-review | skills/peer-review/SKILL.md | 自审检查 |

## 使用方式

1. 用户提出论文写作任务
2. 调用 `using-research-writing` 技能确定流程
3. 按技能指引完成头脑风暴
4. 逐章写作并更新进度

## 核心规则

- 任何写作任务前必须完成头脑风暴
- 必须创建 plan/ 目录记录项目信息
- 每章写完必须让用户确认
- 绝不编造文献

---
> Source: [Norman-bury/research-writing-skill](https://github.com/Norman-bury/research-writing-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
