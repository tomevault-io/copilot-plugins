## opendataloader-pdf-skill

> PDF解析提取转换工具，当用户提到解析pdf、转换pdf、提取pdf内容、pdf转markdown、pdf转json、pdf提取文本时使用，基于opendataloader-pdf实现，支持输出Markdown、JSON、HTML、Tagged PDF等多种格式，100%本地运行无需联网，处理速度快准确率高。

# opendataloader-pdf-skill 使用指南

## 触发时机
当用户的查询中包含以下关键词或相关意图时，必须使用本技能：
- 解析pdf
- 转换pdf
- 提取pdf
- pdf转markdown
- pdf转json
- pdf转html
- pdf提取文本
- pdf表格提取
- pdf OCR识别
- pdf accessibility处理

## 执行步骤
1. **确认输入路径**：首先确认用户提供的PDF文件路径或包含PDF的目录路径，如果用户未提供，明确询问用户要处理的PDF位置。
2. **自动初始化目录结构**：（首次使用自动创建，无需用户操作）
   - 检查并创建工作目录：`~/Documents/opendataloader-pdf/`
   - 检查并创建输出目录：`~/Documents/opendataloader-pdf/output/`
3. **切换工作目录**：执行命令时自动切换到 `~/Documents/opendataloader-pdf` 目录。
4. **自动检测并初始化环境**：（无需用户手动操作，自动完成）
   - 检测当前目录是否存在 `.venv` 虚拟环境目录
   - 如果不存在，自动执行初始化：
     ```bash
     uv venv
     source .venv/bin/activate
     uv pip install -U "opendataloader-pdf[hybrid]"
     ```
5. **激活环境并执行脚本**：
   - 激活venv命令：`source .venv/bin/activate`
   - 默认处理命令（技能根目录下相对路径）：`python ./scripts/convert_pdf.py [输入路径]`
6. **处理完成反馈**：处理完成后告知用户输出文件的位置，并根据用户需求展示部分提取内容。

## 高级场景处理
### 复杂/扫描版PDF处理
当用户需要处理扫描版PDF、复杂嵌套表格、数学公式、图表描述时，告知用户需要启用Hybrid模式，询问用户是否需要开启：
1. 启动Hybrid服务（在venv环境下执行）：`opendataloader-pdf-hybrid --port 5002 [额外参数]`
   - 扫描版OCR：添加 `--force-ocr` 参数，多语言OCR添加 `--ocr-lang "zh,en"` 等语言代码
   - 公式提取：添加 `--enrich-formula` 参数
   - 图表描述：添加 `--enrich-picture-description` 参数
2. 执行处理命令（技能根目录下相对路径）：`python ./scripts/convert_pdf.py [输入路径] --hybrid docling-fast`
   - 也可以直接添加对应参数：`--force-ocr`、`--ocr-lang`、`--enrich-formula`、`--enrich-picture-description`

### 自定义输出格式
需要输出其他格式时，添加 `--format` 参数，例如：
- 仅输出JSON：`--format json`
- 输出可访问性PDF：`--format tagged-pdf`
- 多种格式组合：`--format markdown,json,html`

### 输出格式说明
支持的输出格式及用途：
| 格式 | 用途 |
|------|------|
| markdown | 结构化文本，适合LLM/RAG使用 |
| json | 包含元素边界框、语义类型，适合二次开发 |
| html | 网页展示用 |
| text | 纯文本提取 |
| tagged-pdf | 可访问性Tagged PDF，适合屏幕阅读器 |

### 可访问性处理
当用户需要PDF无障碍处理时，使用 `--format tagged-pdf` 参数生成符合标准的Tagged PDF。

## 注意事项
- 所有处理均在本地运行，不会上传用户PDF文件到外部服务器
- 批量处理时建议一次传入所有PDF路径，避免重复启动JVM进程影响性能
- Java 11+是运行依赖，若运行失败先检查Java环境是否安装

---
> Source: [overstart/opendataloader-pdf-skill](https://github.com/overstart/opendataloader-pdf-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
