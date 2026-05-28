# Agentic-RAG-Knowledge-Assistant

> Source: [dyj-naj/Agentic-RAG-Knowledge-Assistant](https://github.com/dyj-naj/Agentic-RAG-Knowledge-Assistant) — distributed by [TomeVault](https://tomevault.io)

 本项目是一个基于 Agentic RAG 架构的智能知识问答系统，采用前后端分离设计。后端基于 FastAPI 构建，前端使用 React + Vite，集成 Qdrant 向量数据库、BM25 关键词检索、MySQL 结构化查询和 Serper 网络搜索四大知识源。系统核心为一条 7  阶段流水线：查询优化、意图识别、任务拆解、ReAct 代理检索、相关性检查、答案生成及 Self-RAG 质量评估，通过闭环纠错机制保障回答质量。支持多用户注册登录，每位用户拥有独立的个人知识库，可上传 PDF、Word、Excel  等多种格式文件并自动向量化。系统还实现了三级压缩对话记忆、用户画像自动提取和 SSE 流式响应，提供实时、准确、可追溯的问答体验。

## GitHub Copilot Config

The `copilot-instructions.md` file in this directory is the project config converted for GitHub Copilot.
Original source: `CLAUDE.md` in [dyj-naj/Agentic-RAG-Knowledge-Assistant](https://github.com/dyj-naj/Agentic-RAG-Knowledge-Assistant).

## Also available for

- **Codex** — `AGENTS.md`
- **GitHub Copilot** — `copilot-instructions.md`
- **Cursor** — `project-config.mdc`
- **Gemini CLI** — `GEMINI.md`
- **Windsurf** — `project-config.md`

From [dyj-naj/Agentic-RAG-Knowledge-Assistant](https://github.com/dyj-naj/Agentic-RAG-Knowledge-Assistant) — a repo with 9+ stars on GitHub.

---

Install this config instantly:
```
npx tomevault install dyj-naj/Agentic-RAG-Knowledge-Assistant
```
Source: [github.com/dyj-naj/Agentic-RAG-Knowledge-Assistant](https://github.com/dyj-naj/Agentic-RAG-Knowledge-Assistant).

<!-- genome:d-i-s -->
