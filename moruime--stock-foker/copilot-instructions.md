## stock-foker

> - 项目名称：Stock Foker（个人股票分析应用）

# AGENTS.md

## 项目信息

- 项目名称：Stock Foker（个人股票分析应用）
- 前端：React + Vite + TypeScript + Ant Design（端口 5173）
- 后端：Python FastAPI + SQLAlchemy + SQLite（端口 8000）
- 启动：`./start.sh`，终止：`./stop.sh`
- 日志目录：`logs/`

## 开发规范

### 代码风格

- 前端类型检查：`cd frontend && npx tsc --noEmit`
- 前端构建：`cd frontend && npm run build`
- 后端导入验证：`cd backend && source venv/bin/activate && python -c "from app.main import app"`
- A 股配色约定：红色 = 涨，绿色 = 跌
- 暗色主题（Ant Design dark token）

### Markdown 文档规范

生成或编辑任何 `.md` 文件时，必须调用 `markdown-lint` skill，确保输出内容符合 markdownlint 默认规则（MD001~MD060）。

### LLM 配置

- 配置文件：`backend/.env`（参考 `.env.example`）
- 当前模型：qwen3.6-plus（Thinking 模式已关闭）
- 修改配置后可通过前端 AI 设置页"重新加载配置"热生效，无需重启

## 迭代记录与待办列表

每次开发结束时，需更新迭代记录和待办列表文档。

### 文件位置与命名

- 迭代记录：`doc/迭代记录/YYYY-MM-DD-简短主题.md`
- 待办列表：`doc/待办列表/YYYY-MM-DD-简短主题.md`
- 两个文件夹下的文件**一一对应**，同日期同主题

### 日期规则

以天为单位，**天的更新时间截止为第二天早上 8:00**。即：

- 4 月 6 日 8:00 ~ 4 月 7 日 7:59 的开发工作 → 归入 `2026-04-06-xxx.md`
- 4 月 7 日 8:00 ~ 4 月 8 日 7:59 的开发工作 → 归入 `2026-04-07-xxx.md`

这是因为偶尔会熬夜开发，凌晨的工作仍属于前一天。

### 迭代记录格式

精简记录当日开发结果，包含以下部分：

1. **开发概述**：一句话总结
2. **新增功能**：表格（功能 | 说明）
3. **问题修复**：表格（问题 | 原因 | 解决方案），无则省略
4. **工程改进**：列表形式，无则省略
5. **关键文件变更**：分"新增文件"和"修改文件"两个表格

### 待办列表格式

记录当日开发后的遗留事项和后续计划：

1. **遗留问题**：当前已知但未解决的问题（表格：问题 | 优先级 | 说明）
2. **待开发事项**：按阶段分组列出
3. **优化建议**：可改进项，按类别分组
4. **后续计划**：带优先级的行动建议

### 更新时机

- 每次开发会话结束前更新
- 如果当天已有记录文件，在原文件上追加或修改
- 如果是新的一天（按 8:00 规则），创建新文件
- 新建待办列表时，参考前一天的待办列表，将已完成的项目移除，未完成的延续

### 更新前必做：确认当前日期

**在更新或创建任何文档之前，必须先执行 `date` 命令获取真实系统时间**，
再结合 8:00 规则判断目标文件名，禁止依赖对话上下文或假设来推断日期。

```bash
date
```

确认日期后，再决定是追加到已有文件还是创建新文件。

---
> Source: [Moruime/stock_foker](https://github.com/Moruime/stock_foker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
