## product-manual-agent

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

跨境电商多语言产品说明书生成器。支持两种工作模式：
- **模式 A（亚马逊链接）**：输入亚马逊产品 URL → 自动抓取 → 多语言翻译 → 生成 PDF
- **模式 B（素材上传 + AI Agent）**：上传图片/文本文件 + 自然语言 Prompt → LangGraph Agent 智能分析 → 生成 PDF/Word

## Commands

```bash
# 安装依赖
pip install -e ".[dev]"
playwright install chromium

# 启动服务
uvicorn src.api.main:app --host 0.0.0.0 --port 8000 --reload

# 运行全部测试
pytest tests/ -v

# 运行单个测试
pytest tests/test_basic.py::test_scraper_mock -v
```

## Architecture

```
src/
├── models.py              # Pydantic 数据模型（ProductData, ManualRequest, MaterialRequest 等）
├── api/main.py            # FastAPI 应用，两个核心路由：/api/generate（模式A）和 /api/generate-material（模式B）
├── scraper/               # 亚马逊页面抓取（Playwright），支持 Mock 模式
├── translator/            # 多语言翻译（DeepL/OpenAI/Mock），含家具行业术语表
├── pdf_generator/         # 传统 PDF 生成（fpdf2），用于模式A
├── doc_generator/         # 通用文档生成器：HTML渲染(Jinja2) + PDF渲染(WeasyPrint) + Word渲染(mammoth/python-docx)
└── agent/                 # AI Agent 模块（LangGraph）
    ├── graph.py           # LangGraph 状态图：analyze_input → plan_layout → generate_content → render_document
    ├── tools.py           # Agent 工具集：文件提取、图片分析、HTML生成、PDF/Word渲染、术语查询
    ├── prompts.py         # System Prompt 和各阶段 Prompt 模板
    ├── schemas.py         # AgentState TypedDict 定义
    └── llm.py             # LLM 连接封装（OpenAI 兼容接口，模型 mimo-v2.5）
```

### Key Design Patterns

- **模式 A 流程**：`AmazonScraper.scrape()` → `Translator.translate_product()` → `PDFGenerator.generate_manual()`
- **模式 B 流程**：`run_agent()` → LangGraph 状态图依次执行 analyze → plan → generate → render
- **LLM 回退机制**：当 `LLM_API_KEY` 未配置时，Agent 自动降级为回退模式（拼接素材内容为简单 HTML），不阻断流程
- **Mock 模式**：`AmazonScraper(use_mock=True)` 和 `TranslationProvider.MOCK` 用于本地开发和测试，无需真实 API

### Environment Variables (.env)

```env
LLM_BASE_URL=https://token-plan-sgp.xiaomimimo.com/v1
LLM_API_KEY=your_key       # 模式B必需，模式A可选
LLM_MODEL=mimo-v2.5
DEEPL_API_KEY=              # 模式A翻译可选
PORT=8000
```

## Testing Notes

- 测试使用 Mock 模式运行，不需要真实 API Key
- API 集成测试（`test_api_generate` 等）需要服务运行在 `localhost:8001`
- `test_pdf_generator` 验证实际 PDF 输出（文件头 `%PDF-`）
- Windows 环境下 PDF 中文字体路径为 `C:/Windows/Fonts/msyh.ttc`

## Deployment

Railway 部署，配置见 `railway.toml`。构建方式为 Dockerfile，启动命令：`uvicorn src.api.main:app --host 0.0.0.0 --port $PORT`。

---
> Source: [Delight0628/product-manual-agent](https://github.com/Delight0628/product-manual-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
