## skills-2026

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个通用 AI Agent Skill 集合，包含 **81 个专业 Skill**，为各种 AI 智能体（Claude Code、Cursor、Windsurf 等）提供特定领域的专业指导、工作流程和工具集成能力。技能是平台无关的——提供的是领域知识和方法论，可被任何支持 Skill 机制的 AI 助手调用。

## 技能结构

每个技能是一个独立目录：
```
skill-name/
├── SKILL.md              # 主文档（必需）
├── scripts/              # 可执行工具（可选）
├── references/           # 加载的参考文档（可选）
└── assets/               # 模板、图标等（可选）
```

### SKILL.md 格式

```markdown
---
name: skill-name-with-hyphens
description: Use when [触发条件] - 描述何时使用，不总结工作流
---

# 技能名称
概述、何时使用、核心模式、快速参考、实现细节...
```

**关键规则：**
- `name`: 只用字母、数字、连字符（无特殊字符）
- `description`: 第三人称，仅描述触发条件（**绝不总结工作流**），以 "Use when..." 开头
- 保持在 500 行以内；超出内容移到 `references/`

## Claude 搜索优化（CSO）

描述字段决定技能是否被触发：
- 描述工作流的描述会导致智能体跳过阅读完整技能
- 只写触发条件，让智能体阅读正文获取工作流

```yaml
# 错误：总结工作流
description: Use for TDD - write test, fail, code, refactor

# 正确：仅触发条件
description: Use when implementing any feature or bugfix, before writing code
```

## 创建/测试技能

使用 `skill-creator` 技能创建新技能，遵循 TDD：

1. **红** - 无技能运行压力场景，记录失败行为
2. **绿** - 编写最小技能解决具体问题
3. **重构** - 堵住合理化借口，重新测试

**测试技能：**
```bash
# 运行技能评估循环
python -m skill-creator/scripts/run_loop --eval-set evals.json --skill-path ./skill-name

# 打包技能
python -m skill-creator/scripts/package_skill ./skill-name
```

## 核心技能分类

- **工作流程类**: using-superpowers（技能调用机制）、brainstorming（设计前使用）、writing-plans
- **开发方法论类**: test-driven-development、systematic-debugging、verification-before-completion
- **前端设计类**: impeccable（消除 AI 味 UI 设计）、frontend-dev、ui-ux-pro-max
- **中国特色类**: chinese-code-review、chinese-documentation、chinese-git-workflow
- **飞书集成类**: lark-* 系列（使用 lark-cli 命令行工具）
- **Obsidian 集成类**: obsidian-markdown、obsidian-bases、json-canvas、obsidian-cli、defuddle

## 前端设计技能使用指南

| 技能 | 定位 | 何时使用 |
|------|------|----------|
| **impeccable** | 设计质量把关 | 任何前端 UI 工作时自动生效 |
| **frontend-dev** | 端到端构建 | 从零构建完整页面时 |
| **ui-ux-pro-max** | 设计参考 | 需要查配色/字体/布局时 |

## 技能调用优先级

1. 流程技能优先（brainstorming、systematic-debugging）
2. 实现技能其次（frontend-design、mcp-builder）

"构建 X" → 先 brainstorming，再实现技能
"修复 bug" → 先 systematic-debugging，再领域技能

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/466852675) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
