## life-management

> - **永不删除数据库**: 无论进行何种修改或重构，都不要删除或清空 tasks_db 数据

# 生活管理系统项目规范

## 重要指令

### 数据库管理
- **永不删除数据库**: 无论进行何种修改或重构，都不要删除或清空 tasks_db 数据
- **数据持久化说明**: 使用 JSON 文件存储 (data/tasks.json)，服务重启后数据保留
- **数据位置**: `data/tasks.json` 文件，自动创建和维护

## 项目结构
- 前后端分离架构
- 前端: 原生 HTML/CSS/JavaScript
- 后端: FastAPI + Python
- AI集成: DeepSeek API
- 数据存储: JSON 文件 (data/tasks.json)

## 版本管理
- 当前版本: v5.1
- 所有版本更新记录在 claude_changelog.md
- 所有版本更新，需要 git commit、注意修改所有文件对应版本号（如在主页文件index.html上和在这个文档中）

## 部署说明
- 前端: GitHub Pages
- 后端: Vercel
- API密钥通过环境变量管理

## 开发规范
1. 修改代码前先理解现有逻辑
2. 保持代码简洁，避免过度复杂化
3. 所有 API 调用使用 /api 前缀
4. 保持前后端响应格式一致性

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chenzhan4321)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chenzhan4321)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
