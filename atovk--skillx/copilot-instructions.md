## skillx

> 这是一个帮助学习者快速上手 Claude Code Skills 能力的学习资源库。

# SkillX - Claude Code 项目说明

这是一个帮助学习者快速上手 Claude Code Skills 能力的学习资源库。

## 项目概述

SkillX 提供从入门到精通的完整学习路径，包含：

- 系统性的文档和教程
- 实用的示例 Skills
- 循序渐进的练习题
- 可复用的模板

## Skills 组织结构

项目中的 Skills 按类别和难度组织：

```
skills/
├── 01-basics/          # 基础技能（入门）
├── 02-file-operations/ # 文件操作（初级）
├── 03-code-generation/ # 代码生成（初级）
├── 04-data-processing/ # 数据处理（中级）
├── 05-web-development/ # Web 开发（中级）
└── 06-automation/      # 自动化（高级）
```

每个 Skill 目录包含：

- `SKILL.md` - 必需的主文件
- `README.md` - 使用说明
- `examples.md` - 使用示例

## 开发原则

### 1. 示例 Skills

所有示例 Skills 应该：

- **可以独立运行** - 不依赖其他 Skills
- **有清晰的使用说明** - README 或 examples.md
- **遵循最佳实践** - 参考 [docs/03-best-practices.md](docs/03-best-practices.md)
- **简洁专注** - 每个Skill只做一件事并做好

### 2. 文档编写

文档应该：

- **面向学习者** - 使用清晰的语言和示例
- **循序渐进** - 从简单到复杂
- **实用性强** - 提供可运行的代码示例

### 3. 练习题设计

练习应该：

- **有明确目标** - 说明学习目标和验收标准
- **有足够提示** - 提供思路和参考
- **有解答参考** - 提供 example/ 或 hints/

## 工作流

### 创建新 Skill

1. 选择合适的类别目录
2. 复制相应的模板（`templates/basic-skill-template/`）
3. 修改 SKILL.md 的 frontmatter
4. 编写核心指令内容
5. 添加 README.md 和 examples.md
6. 测试 Skill 可用性

### 修改现有 Skill

1. 先理解 Skill 的设计意图
2. 保持向后兼容
3. 更新相关文档
4. 测试修改后的功能

### 添加文档

1. 确定文档类型（教程/参考/指南）
2. 使用清晰的标题结构
3. 添加代码示例
4. 链接到相关资源

## 目录导航

| 路径 | 描述 |
|------|------|
| `docs/` | 核心文档和教程 |
| `skills/` | 示例 Skills |
| `exercises/` | 练习题（按难度分级） |
| `templates/` | Skill 模板 |
| `resources/` | 图表、检查清单等辅助资源 |

## 关键文件

- `README.md` - 项目主页（面向用户）
- `CLAUDE.md` - 本文件（面向 Claude Code）
- `docs/00-getting-started.md` - 快速入门
- `docs/03-best-practices.md` - 最佳实践指南

## 测试指南

测试 Skill 时应该：

1. 验证 frontmatter 格式正确
2. 测试核心功能可用
3. 检查输出格式
4. 验证错误处理

## 相关资源

- 官方最佳实践: [docs/03-best-practices.md](docs/03-best-practices.md)
- 示例参考: `skills/` 目录
- 模板: `templates/` 目录

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atovk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
