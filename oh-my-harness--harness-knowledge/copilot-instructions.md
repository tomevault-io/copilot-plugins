## harness-knowledge

> 这是一个专注于三大领域的 LLM 知识库（兼容 Obsidian）：

# LLM Wiki — Project Guide

## Overview
这是一个专注于三大领域的 LLM 知识库（兼容 Obsidian）：
1. **AI Coding Best Practices** — AI 辅助编程最佳实践
2. **RAG & Knowledge Engineering** — RAG、LLM Wiki、知识工程相关
3. **Agent Harness** — Agent 运行时/Harness 开发相关知识

## Structure
```
/
├── docs/
│   ├── ai-coding/             # AI 辅助编程
│   ├── rag-engineering/       # RAG & 知识工程
│   ├── agent-harness/         # Agent Harness
│   └── knowledge-base/        # 知识库维护规范
├── assets/
│   ├── papers/                # 论文、技术报告、白皮书
│   ├── references/            # 手册、规范、官方资料
│   └── images/                # 图片、截图、图表
├── scripts/                   # 辅助脚本
├── README.md                  # 首页导航
└── CLAUDE.md                  # 此文件
```

## Writing Conventions
- 每个文档须包含 frontmatter: `title`, `tags`, `created`, `updated`
- 使用 `[[wikilink]]` 格式进行 Obsidian 双向链接
- 使用 `#tag` 标签系统
- 新知识写在对应主题目录下，无法归类时建立新目录
- 每个目录须有 `_index.md` 作为该目录入口
- PDF 与附件放在 `assets/` 下，并遵循 [[pdf-assets|PDF 与附件维护规范]]
- 重要 PDF 需要配套 Markdown 摘要卡片，原始 PDF 不直接替代知识笔记

## Workflow
- 新增知识：在对应目录下创建 `.md` 文件
- 新增 PDF：放入 `assets/papers/` 或 `assets/references/`，再创建或更新对应 Markdown 摘要卡片
- 重构：发现跨领域内容时，在源文件留链接跳转
- 定期检查 `_index.md` 确保链接完整

---
> Source: [oh-my-harness/harness-knowledge](https://github.com/oh-my-harness/harness-knowledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
