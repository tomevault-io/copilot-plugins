## agent

> 1. **架构分析** — 分析项目类型、现有架构、涉及的文件

# Route Planning Agent - 项目规范

## 代码编写原则

当要求编写代码时，必须按以下流程执行：
1. **架构分析** — 分析项目类型、现有架构、涉及的文件
2. **设计方案** — 给出实现方案，确认后再动手
3. **分析目录结构** — 明确文件放在哪里、命名规则
4. **推荐技术选型** — 对新增功能给出合理的技术选型建议
5. **建立规范** — 代码风格、模块划分、错误处理等

## 项目概览

- 后端: Python + FastAPI + LangGraph 多智能体
- 前端: Vue 3 + Naive UI
- 向量库: ChromaDB
- LLM: 阿里云 DashScope (通义千问)

## Docker 部署

- Dockerfile: 多阶段构建（Node 20 → Python 3.13）
- docker-compose.yml: 单服务 + 数据卷持久化
- 基础镜像走 `docker.m.daocloud.io/library/` 国内镜像
- npm registry: https://registry.npmmirror.com
- PyPI mirror: https://pypi.tuna.tsinghua.edu.cn/simple

---
> Source: [YIGUAREN/-agent](https://github.com/YIGUAREN/-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
