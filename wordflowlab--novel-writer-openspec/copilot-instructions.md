## novel-writer-openspec

> <!-- OPENSPEC:START -->

<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

---

# 小说创作工作流程

本项目使用 OpenSpec 方法论进行小说创作管理。

**完整的小说创作工作流程指导**：请参见 `@novelspec/AGENTS.md`

## 快速链接

- 📖 [小说创作工作流程指导](novelspec/AGENTS.md)
- 🎯 [澄清规格（批量澄清模式）](novelspec/AGENTS.md#澄清规格clarification--并行路径展示模式)
- 📋 [项目约定](novelspec/project.md)
- 📝 [规格文件](novelspec/specs/)
- 🔄 [变更提案](novelspec/changes/)

## 核心命令

```bash
# 初始化项目
novelspec init <project-name>

# 澄清规格中的模糊决策点（批量澄清模式）
novelspec clarify

# 验证规格
novelspec validate

# 查看帮助
novelspec --help
```

## 工作流程概览

```
规格定义 (specify) → 澄清决策 (clarify) → 规划创作 (plan) → 执行写作 (write) → 分析验证 (analyze)
```

详见 `@novelspec/AGENTS.md` 获取完整指导。

---
> Source: [wordflowlab/novel-writer-openspec](https://github.com/wordflowlab/novel-writer-openspec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
