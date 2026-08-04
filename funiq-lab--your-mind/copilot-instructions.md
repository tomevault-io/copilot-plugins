## your-mind

> This file provides guidance to Claude Code (claude.ai/code) when working in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working in this repository.

## 项目概述

**your-mind** 是一个开源的"数字大脑"框架：帮助你利用 Claude Code 构建个人知识管理和生产力系统，用于长期沉淀输入（阅读/资讯/灵感）、思考（结构化推演与复盘）与输出（文章、产品/设计规划、写作素材等）。

## 目录结构与导航

```
your-mind/
├── life-os/             # 人生规划系统（主工作区）
├── output/              # 内容输出系统（对外分享）
├── inspiration/         # 灵感捕捉（独立，仅供引用）
├── news/                # 资讯收集（按周归档）
├── tech/                # 技术学习记录
└── docs/references/     # 规范文档库
```

### 各目录定位

| 目录 | 定位 | 详细说明 |
|------|------|----------|
| `life-os/` | 人生规划系统 | 日常记录、目标追踪、项目管理 → 见 `life-os/CLAUDE.md` |
| `output/` | 内容输出系统 | 对外分享内容（通稿、各平台文章、视频脚本）→ 见 `output/CLAUDE.md` |
| `inspiration/` | 灵感库 | 瞬间想法，仅供引用不归档 → 见 `inspiration/CLAUDE.md` |
| `news/` | 资讯收集 | 有价值的网络内容，按周归档 → 见 `news/CLAUDE.md` |
| `tech/` | 技术知识库 | 问题卡片、知识卡片 → 见 `tech/CLAUDE.md` |
| `docs/references/` | 规范文档 | 各目录的操作规范 |

## 规范文档

操作不同目录时，请读取对应规范：

| 操作目标 | 规范路径 |
|----------|----------|
| `life-os/` 全部 | `docs/references/life-os/markdown-standards.md` |
| `life-os/Goals/`、`life-os/Daily Notes/` | `docs/references/life-os/productivity-workflow.md` |
| `life-os/Projects/` | `docs/references/life-os/project-management.md` |
| `output/` | `docs/references/output/content-standards.md` |
| `output/quaily/` | `docs/references/output/quaily-format.md` |
| `news/` | `docs/references/news/collection-format.md` |
| `tech/` | `docs/references/tech/documentation-format.md` |

## 可用命令

### 全局命令

| 命令 | 用途 |
|------|------|
| `/commit` | 暂存所有变更并生成提交（不推送） |
| `/catchup` | 同步最新状态并总结进展 |

### life-os/ 专用

| 命令 | 用途 |
|------|------|
| `/daily` | 创建今日每日笔记 |
| `/weekly` | 运行周回顾流程 |
| `/onboard` | 加载完整上下文 |

### inspiration/ 专用

| 命令 | 用途 |
|------|------|
| `/capture` | 快速捕捉灵感 |

### news/ 专用

| 命令 | 用途 |
|------|------|
| `/collect` | 收集资讯（提供网址+感想） |

### tech/ 专用

| 命令 | 用途 |
|------|------|
| `/card` | 创建技术卡片（问题/知识） |

## 可用技能

### life-os/ 专用

| 技能 | 用途 |
|------|------|
| `life-os-file-ops` | 文件读写规范（frontmatter、wiki-links、模板） |
| `daily-workflow` | 晨间/午间/晚间工作流 |
| `goal-tracking` | 目标进度追踪 |

### output/ 专用

| 技能 | 用途 |
|------|------|
| `blog-assistant` | 根据主题或灵感生成文章提纲 |

### tech/ 专用

| 技能 | 用途 |
|------|------|
| `tech-reviewer` | 审核技术卡片，验证准确性，指出错误 |

### 通用

| 技能 | 用途 |
|------|------|
| `skill-creator` | 创建新的 Claude Code skill |

## 可用代理

以下代理均用于 `life-os/` 目录：

| 代理 | 用途 |
|------|------|
| `goal-aligner` | 分析活动与目标的对齐情况 |
| `weekly-reviewer` | 引导周回顾流程 |
| `note-organizer` | 整理笔记、修复链接 |
| `inbox-processor` | GTD 式处理 Inbox |

## 输出风格

| 风格 | 用途 |
|------|------|
| `coach` | 生产力教练模式，挑战性问责 |

## 工作原则

- **默认保守修改**：避免大规模重命名/搬迁/批量改格式；如需整理，先说明方案并分批执行
- **尊重既有命名与语言**：仓库内中英混排属正常现象，保持原文件名不变
- **维护可追溯性**：重要结论/决策落到对应项目/目标/每日笔记中
- **读取规范后操作**：操作任何目录前，先读取对应的规范文档

## 个人覆盖

个人设置请创建 `CLAUDE.local.md`（已被 gitignore）。

---
> Source: [funiq-lab/your-mind](https://github.com/funiq-lab/your-mind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
