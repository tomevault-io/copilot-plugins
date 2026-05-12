## md-audit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

MD Audit 是基于Python的Markdown SEO诊断Agent，结合规则引擎和AI语义分析，自动评估Markdown文件的SEO质量并提供可执行优化建议。

**核心特性**：
- 双引擎分析：规则引擎（75%权重）+ AI语义分析（25%权重）
- 原生Markdown支持：直接分析.md文件
- 智能关键词提取：自动从内容中提取Top关键词
- 批量分析：支持目录级批量分析和并发处理
- Web服务：提供REST API和Web界面

## 开发环境

### 虚拟环境管理

```bash
# 激活虚拟环境（必需）
source venv/bin/activate    # Linux/macOS
venv\Scripts\activate        # Windows

# 安装依赖
pip install -r requirements.txt

# 安装开发模式
pip install -e .
```

### 运行命令

**单文件分析**：
```bash
# 自动提取关键词
python -m md_audit.main analyze docs/article.md

# 指定关键词
python -m md_audit.main analyze docs/article.md -k "Python" "SEO"

# 保存报告
python -m md_audit.main analyze docs/article.md -o report.md

# 使用自定义配置
python -m md_audit.main analyze docs/article.md --config custom.json

# 禁用AI分析
python -m md_audit.main analyze docs/article.md --no-ai
```

**批量目录分析**：
```bash
# 分析整个目录（默认4并发）
python -m md_audit.main analyze docs/ -o reports/

# 自定义并发数
python -m md_audit.main analyze docs/ --workers 8 -o reports/
```

**Web服务**：
```bash
# 启动Web服务（默认http://127.0.0.1:8000）
python -m md_audit.main serve

# 自定义地址和端口
python -m md_audit.main serve --host 0.0.0.0 --port 8080

# 开发模式（自动重载）
python -m md_audit.main serve --reload

# API文档地址：http://127.0.0.1:8000/docs
```

### 测试命令

```bash
# 运行所有测试
pytest tests/ -v

# 运行单个测试文件
pytest tests/unit/test_parsers.py -v

# 查看测试覆盖率
pytest tests/ --cov=md_audit --cov-report=html
```

### 代码格式化

```bash
# 格式化代码（强制执行PEP8）
black md_audit/

# 代码检查
ruff check md_audit/

# 自动修复可修复的问题
ruff check --fix md_audit/
```

## 系统架构

### 核心流程

```
用户请求 → CLI/Web API → MarkdownSEOAnalyzer
    ↓
MarkdownParser (解析Frontmatter + Markdown)
    ↓
RulesEngine (规则检查：元数据、结构、关键词)
    ↓
AIEngine (LLM语义分析：内容深度、可读性) [可选]
    ↓
ScoreCalculator (加权评分：4维度)
    ↓
ReportGenerator (生成Markdown报告)
```

### 模块职责

| 模块 | 文件路径 | 核心职责 |
|------|---------|---------|
| **CLI入口** | `md_audit/main.py` | 命令行参数解析，单文件/批量分析，Web服务启动 |
| **核心分析器** | `md_audit/analyzer.py` | 协调解析、规则检查、AI分析、评分计算 |
| **Markdown解析** | `md_audit/parsers/markdown_parser.py` | Frontmatter解析、Markdown→HTML、关键词提取 |
| **规则引擎** | `md_audit/engines/rules_engine.py` | 执行META/STRUC/KEY规则检查 |
| **AI引擎** | `md_audit/engines/ai_engine.py` | OpenAI API调用、语义分析、降级处理 |
| **报告生成** | `md_audit/reporter.py` | 生成Markdown格式诊断报告 |
| **数据模型** | `md_audit/models/data_models.py` | Pydantic模型：SEOReport、Issue、Suggestion等 |
| **配置系统** | `md_audit/config.py` | 规则配置加载、环境变量支持 |
| **Web服务** | `web/main.py` | FastAPI应用、REST API、前端集成 |

### 评分体系

总分100分，4个维度加权评分：

| 维度 | 权重 | 检查项 |
|------|------|--------|
| **元数据** | 30% | Title长度（30-60字符）、Description长度（120-160字符） |
| **结构** | 25% | H1唯一性、图片Alt覆盖率（≥80%）、链接存在性 |
| **关键词** | 20% | 关键词密度（1%-2.5%）、关键词位置（Title/Desc/首段） |
| **AI语义** | 25% | 内容深度（15%）、可读性（10%） |

## 配置文件

**默认配置**：`config/default_config.json`

关键配置项：
```json
{
  "title_rules": {
    "min_length": 30,
    "max_length": 60
  },
  "description_rules": {
    "min_length": 120,
    "max_length": 160
  },
  "keyword_rules": {
    "min_density": 0.01,
    "max_density": 0.025
  },
  "llm_api_key": "",
  "llm_base_url": "https://newapi.deepwisdom.ai/v1",
  "llm_model": "gpt-4o",
  "enable_ai_analysis": true
}
```

**环境变量**：
- `MD_AUDIT_LLM_API_KEY`: OpenAI API密钥
- `MD_AUDIT_LLM_MODEL`: 覆盖默认模型
- `SEO_RULES_CONFIG`: 指定配置文件路径

## 关键实现细节

### 1. 关键词自动提取

参考 `/mnt/d/SEO_develop/SEO-AutoPilot/pyseoanalyzer/analyzer.py:16-94`

**过滤规则**：
- 拒绝URL片段（http://, www., .com等）
- 拒绝HTML/CSS代码（<, >, {, class=等）
- 拒绝纯数字或特殊字符
- 拒绝停用词和过短/过长单词

实现位置：`md_audit/parsers/markdown_parser.py:is_quality_keyword()`

### 2. 规则引擎评分逻辑

**Title检查（META_01）**：
- 缺失：0分，Critical
- <30字符：50%得分，High
- >60字符：50%得分，High
- 30-60字符：100%得分

**关键词密度检查（KEY_01）**：
- <1%：30%得分，建议增加
- 1%-2.5%：100%得分
- >2.5%：50%得分，警告堆砌

实现位置：`md_audit/engines/rules_engine.py`

### 3. AI引擎降级机制

**重试策略**：
- 3次重试，指数退避（2^attempt秒）
- 超时设置：30秒
- 失败后自动降级到纯规则分析

实现位置：`md_audit/engines/ai_engine.py:analyze_semantics()`

### 4. 批量分析并发

使用ThreadPoolExecutor实现并发分析：
- 默认4个工作线程
- 支持通过`--workers`参数调整
- 生成汇总报告（SUMMARY.md）

实现位置：`md_audit/analyzer.py:analyze_directory()`

## 开发约定

### 代码规范

1. **PEP8遵守**：强制使用`black`和`ruff`格式化
2. **中文注释**：复杂逻辑、关键算法使用中文注释
3. **Docstrings**：公共API必须有文档字符串（中文）
4. **类型注解**：所有函数参数和返回值使用类型注解

### 文件组织

**禁止在以下目录外创建文件**：
- 源代码：`md_audit/`
- 测试：`tests/`
- 配置：`config/`
- 文档：`docs/`
- Web前端：`frontend/`
- Web后端：`web/`

**不要创建**：
- 根目录下的Markdown文件（除非更新README.md）
- 临时测试文件
- 个人笔记或草稿

### 错误处理

**LLM调用失败**：
- 自动降级到规则引擎
- 日志记录详细错误信息
- 不应导致整体分析失败

**Frontmatter缺失**：
- 返回空字典，不抛异常
- 记录警告日志
- 继续分析Markdown正文

### 测试要求

**测试fixtures**：
- `tests/fixtures/high_quality.md`：高分示例（>85分）
- `tests/fixtures/medium_quality.md`：中等示例（50-85分）
- `tests/fixtures/low_quality.md`：低分示例（<50分）

**测试覆盖**：
- 所有规则检查必须有单元测试
- AI引擎需Mock OpenAI API
- 批量分析需集成测试

## 参考文档

**产品需求**：`docs/PRD.md`
**技术设计**：`docs/TECH_DESIGN.md`
**测试计划**：`docs/TEST_PLAN.md`
**开发指令**：`docs/CODEX_PROMPT.md`

**参考项目**：
- `/mnt/d/SEO_develop/SEO-AutoPilot/pyseoanalyzer` - 配置系统、关键词提取、规则检查逻辑

## 常见问题

### Q: 如何调试AI引擎？
```bash
# 禁用AI分析，快速迭代规则引擎
python -m md_audit.main analyze test.md --no-ai

# 查看详细日志
python -m md_audit.main analyze test.md --config config/debug_config.json
```

### Q: 如何添加新的规则检查？
1. 在`md_audit/engines/rules_engine.py`中添加检查方法
2. 在`md_audit/models/data_models.py`中更新RuleCheckResult
3. 在`tests/unit/test_rules_engine.py`中添加测试
4. 更新`config/default_config.json`中的阈值

### Q: Web服务如何集成前端？
- 前端位于`frontend/`目录（Vite + React）
- 后端API位于`web/main.py`（FastAPI）
- 静态文件通过FastAPI的StaticFiles挂载
- 开发时前端和后端分别启动

## 项目状态

✅ **已完成**：
- 核心分析引擎（单文件、批量）
- 规则引擎（4维度检查）
- AI引擎（OpenAI集成）
- CLI工具
- Web服务MVP
- 前端UI（React）

🔧 **待优化**：
- 更多语言支持（当前仅英文关键词提取）
- 性能优化（大文件处理）
- 更多AI Provider支持（Claude、Gemini等）
- 更丰富的报告格式（HTML、PDF）

---
> Source: [calderbuild/MD_Audit](https://github.com/calderbuild/MD_Audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
