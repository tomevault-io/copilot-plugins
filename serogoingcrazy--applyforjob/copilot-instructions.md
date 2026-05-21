## applyforjob

> 基于 CrewAI + FastAPI + Next.js 的全栈智能求职系统。正在从 CLI/Streamlit 工具升级为交互式 Web 应用。

# Apply4Job - 智能求职助手

## 项目概述
基于 CrewAI + FastAPI + Next.js 的全栈智能求职系统。正在从 CLI/Streamlit 工具升级为交互式 Web 应用。

## 实施计划
**开发时务必参考 [`implementation.md`](./implementation.md)**，包含完整的分阶段实施计划、API 设计、架构图和关键复用文件说明。每完成一个 Phase 的任务项后，在 `implementation.md` 中勾选对应的 checkbox。

### 当前状态
- ✅ Phase 0: 现有 CLI + CrewAI 多 Agent 流水线已完成 (端到端 dry_run 测试通过)
- ✅ Phase 1: 基础搭建 (FastAPI 后端 + Next.js 前端脚手架) — 完成
- ✅ Phase 2: PDF 简历上传 — 完成
- ✅ Phase 3: 岗位搜索 + JD 爬取 — 完成
- ✅ Phase 4: LLM 简历优化建议 — 完成
- ✅ Phase 5: 投递追踪看板 — 完成 (含数据库重构)
- 🔄 Phase 6: CopilotKit 集成 — 待开始

## 原有功能 (CLI 模式)
基于 CrewAI Flow 的多 Agent 自动化求职系统，涵盖岗位搜索、简历优化、自动投递三大功能。

## 技术栈
### 后端 (Python 3.12)
- CrewAI >= 0.80 (Flow + Crew + Tools) — 现有 Agent 编排
- FastAPI + uvicorn — 新增 Web API 层
- python-jobspy >= 1.1.82 — 多平台岗位搜索
- Pydantic v2 — 数据模型
- SQLite (aiosqlite + WAL) — 投递追踪
- PyMuPDF (pymupdf) — PDF 简历解析
- httpx + BeautifulSoup4 — JD 网页爬取
- WeasyPrint / python-docx / Jinja2 — 简历渲染
- copilotkit (Python SDK) — CopilotKit 后端运行时
- LLM 默认 gpt-4o-mini，可在 `data/config.yaml` 中切换

### 前端
- Next.js (App Router) + TypeScript + Tailwind CSS
- CopilotKit (`@copilotkit/react-core`, `@copilotkit/react-ui`) — AI 对话侧边栏
- recharts — 数据可视化
- @hello-pangea/dnd — 看板拖拽

## 架构

```
main.py → JobHuntFlow (flow.py)
             │
             ├── @start: load_config        加载配置+简历
             ├── @listen: search_jobs        SearchCrew 搜索岗位
             ├── @router: check_jobs         有岗位? → 继续 / 无 → 结束
             ├── @listen: optimize_resume    ResumeCrew 定制简历+求职信
             ├── @listen: apply_to_job       ApplyCrew 投递/记录
             ├── @router: next_or_done       还有下一个? → 循环 / 完成 → 报告
             ├── @listen: generate_report    输出汇总
             └── @listen: handle_no_jobs     无匹配岗位处理
```

### 三个 Crew

| Crew | Agents | Tools | 职责 |
|------|--------|-------|------|
| **SearchCrew** | JobSearcher, JDAnalyzer | JobSearchTool (python-jobspy) | 多平台搜索 + JD 分析评分 |
| **ResumeCrew** | ResumeAnalyst, ResumeWriter, CoverLetterWriter | ResumeReadTool, ResumeWriteTool, CoverLetterWriteTool | 匹配分析 + 简历改写 + 求职信 |
| **ApplyCrew** | ApplicationStrategist, Applicant | CheckDuplicateTool, EmailApplyTool, RecordApplicationTool | 查重 + 投递 + 记录 |

## 项目结构
```
apply4job/
├── pyproject.toml
├── .env                         # OPENAI_API_KEY (必需)
├── .env.example
├── data/
│   ├── config.yaml              # 求职偏好 (目标岗位/城市/模型/dry_run)
│   ├── resume_zh.yaml           # 中文简历
│   └── resume_en.yaml           # 英文简历
├── output/
│   ├── resumes/                 # 生成的定制简历 YAML
│   └── cover_letters/           # 生成的求职信 Markdown
├── db/
│   └── applications.db          # SQLite 投递追踪
└── src/apply4job/
    ├── main.py                  # CLI 入口 (--dry-run / --no-dry-run / --plot)
    ├── flow.py                  # JobHuntFlow 顶层编排
    ├── models/                  # Pydantic: Job, Resume, Application
    ├── crews/
    │   ├── search_crew/         # agents.yaml + tasks.yaml + crew.py
    │   ├── resume_crew/
    │   └── apply_crew/
    ├── tools/
    │   ├── search_tools.py      # JobSearchTool (封装 python-jobspy)
    │   ├── resume_tools.py      # 简历/求职信读写
    │   ├── apply_tools.py       # EmailApplyTool (SMTP, dry_run 支持)
    │   └── db_tools.py          # SQLite CRUD + 查重
    └── utils/
        ├── config_loader.py     # 加载 config/resume YAML + get_model()
        ├── resume_renderer.py   # YAML → HTML/PDF/DOCX
        └── templates/
            └── resume.html      # Jinja2 简历模板
```

## 运行方式

### Web 应用 (推荐)
```bash
./dev.sh    # 同时启动后端 (:8000) 和前端 (:3000)，Ctrl+C 停止
```

### CLI 模式 (原有)
```bash
source .venv/bin/activate
PYTHONPATH=src python -m apply4job.main          # dry_run 模式 (默认)
PYTHONPATH=src python -m apply4job.main --plot    # 生成流程图
PYTHONPATH=src python -m apply4job.main --no-dry-run  # 真实投递 (需确认)
```

## 配置说明
- `data/config.yaml` — 目标岗位、城市、平台、模型、dry_run 开关
- `data/resume_zh.yaml` / `resume_en.yaml` — 简历 (YAML 为 single source of truth)
- `.env` — API keys: `OPENAI_API_KEY` (必需), `EMAIL_USER` + `EMAIL_PASS` (投递时需要)

## 开发规范
- 每个 Crew 可独立测试: `SearchCrew().crew().kickoff(inputs={...})`
- 投递功能默认 `dry_run=True`，首次运行不会真发邮件
- YAML tasks 中避免用 `{xxx}` 写非 input 变量，会被 CrewAI 模板引擎解析报错
- 支持中英文双语简历
- venv 用 Python 3.12 (3.13 与 jobspy 依赖不兼容)
- 依赖安装用 `uv` (pip 在构建 numpy 时有 setuptools 兼容问题)

## 开发流程
1. 开发前阅读 `implementation.md` 了解当前 Phase 和待办任务
2. 开发时优先复用 `src/apply4job/` 现有代码 (直接 import，不要复制)
3. 后端 CRUD 操作直接调用工具类函数 (如 `JobSearchTool._run()`)，不启动 CrewAI Crew
4. 完成任务后在 `implementation.md` 中勾选对应 checkbox `[x]`

## 已验证
- 端到端 dry_run 测试通过: 搜索 4 岗位 → 生成 5 份定制简历 + 5 封求职信 → 4 条 DB 投递记录

---
> Source: [SeroGoingCrazy/ApplyForJob](https://github.com/SeroGoingCrazy/ApplyForJob) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
