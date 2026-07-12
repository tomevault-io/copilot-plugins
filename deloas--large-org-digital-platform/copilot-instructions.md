## large-org-digital-platform

> 1. **对话、规划、报告、验证结果、错误分析**：默认使用简体中文。

# CLAUDE.md — 项目级工作规范

## 语言要求

1. **对话、规划、报告、验证结果、错误分析**：默认使用简体中文。
2. **除非我明确要求英文**，否则不要使用英文回复。
3. **代码标识符**：类名、方法名、变量名、接口路径、数据库字段名、文件名、包名保持英文，不要翻译。
4. **技术术语**：可保留英文，例如 Spring Boot、Vue、TypeScript、RBAC、RAG、AOP、DTO、VO、Mapper、Service。
5. **Git commit message**：继续使用 Conventional Commits 英文格式。
   - `feat: add audit log foundation`
   - `fix: correct menu permission rendering`
   - `docs: update deployment guide`
6. **前端 UI 文案**：默认使用简体中文。
7. **README、项目文档、验证报告**：默认使用简体中文。
8. **命令、代码、SQL、配置文件**：引用时保持原格式，不要为了中文化破坏语法。
9. **确认项**：需要我确认时，用简体中文列出清晰的确认项。

## 开发流程

10. 每个阶段严格按以下顺序执行：
    - **先规划**：输出完整实施方案。
    - **等我确认**：获得明确批准后再动手。
    - **再实现**：严格按确认的方案写代码。
    - **再自检**：编译 / type-check / build 必须通过。
    - **再提交**：使用 Conventional Commits 英文格式提交。

## 禁止事项

11. 未经我确认，不得：
    - 开发新功能。
    - 引入新的 UI 组件库。
    - 引入 Tailwind CSS。
    - 引入复杂动画库。
    - 修改 docs 目录。
    - 破坏已有功能。
    - 实现采购审批、知识库 RAG、异常登录检测、告警管理、IP 黑名单、大模型 API 接入。
    - 做多 agent 拆分。

---
> Source: [Deloas/large-org-digital-platform](https://github.com/Deloas/large-org-digital-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
