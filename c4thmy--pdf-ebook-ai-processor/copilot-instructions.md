## pdf-ebook-ai-processor

> 此文件为Claude Code (claude.ai/code)在处理此代码库时提供指导。

# CLAUDE.md

此文件为Claude Code (claude.ai/code)在处理此代码库时提供指导。

## 项目概述

这是一个PDF/HTML电子书自动处理系统，用于批量提取章节并生成AI结构化笔记。核心功能包括：

1. **多格式支持** - PDF电子书和HTML文档集
2. **智能主题分类** - 自动从文件路径和文件名识别书籍主题
3. **章节提取** - PDF书签提取、文本模式识别、HTML解析
4. **AI笔记生成** - 使用阿里云Qwen-Max模型生成6维度结构化笔记

## 构建和运行命令

### 环境配置
```bash
# 安装依赖
pip install -r requirements.txt

# 设置API密钥（必须用于生成笔记）
export DASHSCOPE_API_KEY="your-api-key"        # Linux/Mac
$env:DASHSCOPE_API_KEY="your-api-key"          # Windows PowerShell
```

### 测试系统
```bash
python test_system.py    # 验证环境配置和依赖
```

### 核心运行命令
```bash
# 完整处理：提取章节 + 生成AI笔记
python pdf_to_notes.py

# 仅提取章节（不需要API密钥）
python pdf_to_notes.py --no-notes

# 处理指定主题
python pdf_to_notes.py --theme "Python编程"

# 自定义输入输出路径
python pdf_to_notes.py --input ./my_books/ --output ./my_notes/
```

### 查看结果
```bash
# 查看处理摘要
cat output/processing_summary.json    # Linux/Mac
type output\processing_summary.json   # Windows

# 查看日志
cat output/pdf_to_notes.log
type output\pdf_to_notes.log

# 查看生成的章节和笔记
tree output/                           # 查看目录结构
```

## 核心架构

### 数据处理流水线

系统采用三阶段处理模式：

**第零阶段：文件扫描与主题识别** ([pdf_to_notes.py:87-181](pdf_to_notes.py#L87-L181))
- `scan_input_directory()` - 递归扫描`in_files/`目录
- `extract_theme_from_path()` - 多策略主题识别（目录名 > 文件名 > 默认）
- `create_output_structure()` - 按主题创建输出目录结构

**第一阶段：章节提取** ([pdf_to_notes.py:217-551](pdf_to_notes.py#L217-L551))
- **PDF处理**:
  - `extract_chapters_from_pdf()` - 主入口
  - `extract_chapters_from_toc()` - 基于PDF书签（优先）
  - `extract_chapters_from_text_patterns()` - 基于正则表达式（备选）
  - 降级策略：无章节时将整本书作为单章节
- **HTML处理**:
  - `parse_html_to_markdown()` - HTML解析和Markdown转换
  - `detect_file_encoding()` - 使用chardet自动检测编码
  - `clean_html_content()` - 移除导航、页脚等无关内容

**第二阶段：AI笔记生成** ([pdf_to_notes.py:593-783](pdf_to_notes.py#L593-L783))
- `generate_note_prompt()` - 构建6维度结构化提示词
  - 核心观点、关键论据、重要术语、实践应用、章节总结、思考问题
- `call_dashscope_api()` - 调用Qwen-Max API（含重试机制）
- `split_long_text()` - 超长文本分段处理（>8000字）
- `process_theme_notes()` - 批量生成主题下所有笔记

### 关键数据结构

**BookMetadata** ([pdf_to_notes.py:54-80](pdf_to_notes.py#L54-L80))
```python
class BookMetadata:
    id: str              # 书籍唯一标识
    theme: str           # 主题名称
    source_path: str     # 源文件路径
    file_type: str       # "pdf" 或 "html"
    file_list: List[str] # HTML文档集的多个文件
    output_dir: Dict     # 输出目录结构
    status: str          # 处理状态
    chapter_files: List  # 已提取的章节文件列表
    notes_count: int     # 生成的笔记数量
```

### 目录结构规范

```
项目根目录/
├── in_files/              # 输入目录（用户放置书籍文件）
│   ├── 主题A/             # 支持子目录分类
│   │   ├── book1.pdf
│   │   └── chapter1.html
│   └── book2.pdf          # 也支持根目录直接放置
├── output/                # 输出目录（自动生成）
│   ├── 主题A/
│   │   ├── chapters/      # Markdown章节文件
│   │   └── notes/         # AI生成的笔记
│   ├── processing_summary.json
│   └── pdf_to_notes.log
├── pdf_to_notes.py        # 主程序（1056行）
├── test_system.py         # 测试脚本
└── requirements.txt       # 依赖清单
```

## 重要技术细节

### 主题识别策略
1. **目录结构优先**: `in_files/Python编程/book.pdf` → 主题: "Python编程"
2. **文件名解析**: `in_files/《机器学习》.pdf` → 主题: "机器学习"（自动清理书名号）
3. **规范化处理**: 移除非法字符、限制长度（50字符）

### PDF章节提取策略
采用级联降级策略以提高成功率：
1. **策略A**: PDF TOC书签（最准确）
2. **策略B**: 正则表达式文本模式（`第X章`、`Chapter N`）
3. **策略C**: 全文降级（将整本书作为单章节）

### HTML处理特性
- 自动编码检测（支持UTF-8、GBK、GB2312）
- 智能内容提取（优先级：`<article>` > `<main>` > `.content` > `<body>`）
- 清理无关标签（nav、footer、header、script、style）
- 标题提取（优先级：`<title>` > `<h1>` > 文件名）

### API调用优化
- 重试机制：最多3次，指数退避（2秒 → 4秒 → 8秒）
- 限流处理：检测HTTP 429，自动延迟重试
- 长文本分段：>8000字自动分段处理，避免API限制

### 日志系统
- 双层日志：控制台（INFO级别）+ 文件（DEBUG级别）
- 日志文件：`output/pdf_to_notes.log`（UTF-8编码）
- 进度条：使用tqdm显示批量处理进度

## 常见问题处理

### Q: PDF没有识别到章节？
**原因**: PDF无书签且章节标题格式不匹配正则表达式
**解决**: 系统会自动降级为全文处理（见[pdf_to_notes.py:247-254](pdf_to_notes.py#L247-L254)）

### Q: HTML编码错误？
**原因**: 编码检测失败
**解决**: `detect_file_encoding()`会尝试多种编码（见[pdf_to_notes.py:367-390](pdf_to_notes.py#L367-L390)）

### Q: API调用失败？
**检查项**:
1. 环境变量 `DASHSCOPE_API_KEY` 是否设置
2. 网络连接是否正常
3. API额度是否充足
4. 查看 `output/pdf_to_notes.log` 获取详细错误

### Q: 处理时间太长？
**优化方案**:
1. 先使用 `--no-notes` 提取章节（快速验证）
2. 使用 `--theme "主题名"` 逐个主题处理
3. 检查章节是否过长（>50000字需手动拆分）

## 性能指标

- **PDF章节提取**: 10-30秒/本（取决于页数和章节数）
- **HTML处理**: 1-5秒/文件
- **AI笔记生成**: 10-60秒/章节（取决于内容长度和API响应速度）
- **内存占用**: 通常 <500MB（流式处理）
- **API费用**: 约 ¥0.01-0.05/章节（基于Qwen-Max定价）

## 依赖库说明

| 库 | 用途 | 版本要求 |
|---|---|---|
| pymupdf | PDF解析和文本提取 | >=1.23.0 |
| beautifulsoup4 | HTML解析 | >=4.12.0 |
| markdownify | HTML到Markdown转换 | >=0.11.0 |
| dashscope | 阿里云AI API客户端 | >=1.14.0 |
| chardet | 文件编码检测 | >=5.2.0 |
| tqdm | 进度条显示 | >=4.66.0 |

## 开发注意事项

### 修改代码时
- 主程序逻辑在 [pdf_to_notes.py](pdf_to_notes.py) 中，采用模块化设计
- 修改提示词模板: 编辑 `generate_note_prompt()` 函数（第622-646行）
- 调整章节识别规则: 修改 `extract_chapters_from_text_patterns()` 正则表达式（第300行）
- 更改AI模型: 修改 `call_dashscope_api()` 中的 `model` 参数（第660行）

### 测试流程
1. 运行 `python test_system.py` 验证环境
2. 使用小样本测试（1-2个文件）
3. 先用 `--no-notes` 测试章节提取
4. 再测试单个主题的笔记生成
5. 最后进行全量处理

### Windows平台特殊处理
- 控制台编码: test_system.py使用UTF-8包装器解决中文显示问题
- 路径分隔符: 代码使用 `os.path.join()` 确保跨平台兼容
- 文件名非法字符: 自动移除 `/\:*?"<>|` 字符

---
> Source: [c4thmy/pdf-ebook-ai-processor](https://github.com/c4thmy/pdf-ebook-ai-processor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
