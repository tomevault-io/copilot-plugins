## paper-compass-skill

> This repository contains two Claude Code skills:

# CLAUDE.md

This repository contains two Claude Code skills:

- `paper-compass-learnpath`
- `paper-compass-score`

## 项目定位

### `paper-compass-learnpath`

用于在“读论文前”生成先修知识路线图，重点是：

1. 找出必须先懂的概念
2. 给出概念在论文中的证据锚点（章节 + 短引文）
3. 按依赖顺序、学习难度、时间成本组织学习路径
4. 结合 `memory.md` 做个性化剪枝

### `paper-compass-score`

用于生成“论文价值分析报告”，重点是：

1. 对目标论文做 10.0 分制的严格评分
2. 固定比较 5 篇 arXiv 同领域论文
3. 从期刊/会议、作者、引用量、被引质量、技术增量、业界贡献、奠基潜力等维度给出证据化判断
4. 帮用户从一批论文中优先筛出更值得读的论文

## Skill 路径

- Learnpath skill: `skills/paper-compass-learnpath/SKILL.md`
- Learnpath references: `skills/paper-compass-learnpath/references/`
- Score skill: `skills/paper-compass-score/SKILL.md`
- Score references: `skills/paper-compass-score/references/`

## 运行入口

```text
/paper-compass-learnpath <paper-input> [memory=<path/to/memory.md>] [lang=zh|en]
/paper-compass-score <paper-input> [lang=zh|en]
```

## 设计原则

- Evidence first: 无证据不下结论
- Structured output: 报告结构固定，便于横向比较
- Honest first: 信息不足时明确标注，不编造
- Comparison first: 评分必须结合相似论文做横向参照

---
> Source: [cenzihan/paper-compass-skill](https://github.com/cenzihan/paper-compass-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
