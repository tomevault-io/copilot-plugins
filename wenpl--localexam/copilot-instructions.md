## localexam

> 本项目把 `解析版/` 中的天津初中数学 DOCX 试卷拆分为题目，转换 Word

# CLAUDE.md

## 项目概述

本项目把 `解析版/` 中的天津初中数学 DOCX 试卷拆分为题目，转换 Word
公式、提取图片、调用 DeepSeek 标注知识点，最后输出可迁移的题库 JSON。

当前输入和 `output/exam_questions.json` 均为18份七、八、九年级试卷，
共435道题。历史418题产物中的8道答题说明伪题已清除，第二南开学校
25道题已补入；当前发布校验为0错误、0警告。

## 目录

- `解析版/`：源 DOCX。
- `规范/`：JSON 字段定义和 302 条四级知识树。
- `pipeline/docx_parser.py`：切题、OMML 转 LaTeX、图片提取。
- `pipeline/llm_tagger.py`：LLM 标注、路径规范化、综合标签和去重。
- `pipeline/builder.py`：来源解析、题型内容组装、全量构建。
- `pipeline/validator.py`：发布前 JSON、知识路径和图片校验。
- `pipeline/main.py`：命令行构建入口。
- `pipeline/gui.py`：双击即用的图形界面入口（tkinter，可打包成 exe）。
- `pipeline/generate_browser.py`：生成题库浏览页面。
- `pipeline/generate_label_tool.py`：生成标注校对页面。
- `pipeline/test_pipeline.py`：固定试卷端到端解析测试。
- `tests/`：行为和回归测试。
- `output/`：JSON、图片和生成后的 HTML 产物。

## 运行

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m unittest discover -s tests -v
.\.venv\Scripts\python.exe pipeline\test_pipeline.py
```

全量构建会调用真实 API：

```powershell
.\.venv\Scripts\python.exe pipeline\main.py
```

新试卷建议使用独立输入目录和输出文件，先限制少量题目测试：

```powershell
.\.venv\Scripts\python.exe pipeline\main.py `
  --input-dir "D:\新试卷" `
  --output "output\new_exam_questions.json" `
  --limit 10
```

确认结果后从检查点继续。成功题会跳过，失败题会重新调用 API：

```powershell
.\.venv\Scripts\python.exe pipeline\main.py `
  --input-dir "D:\新试卷" `
  --output "output\new_exam_questions.json" `
  --resume
```

每道题完成后都会原子写入检查点。API 对超时、429 和 5xx 最多自动尝试
3 次并指数退避。未使用 `--resume` 且输出已存在时，会先生成同路径
`.bak` 备份。

默认会从现有 `output/exam_questions.json` 复用相同来源文件、题号和题型的
UUID v4。只有明确需要全部刷新 ID 时才使用：

```powershell
.\.venv\Scripts\python.exe pipeline\main.py --fresh-ids
```

生成前端页面不会调用 API：

```powershell
.\.venv\Scripts\python.exe pipeline\generate_browser.py
.\.venv\Scripts\python.exe pipeline\generate_label_tool.py
```

## 数据规则

- `question_id` 使用 UUID v4。重复构建默认复用旧ID，新题才生成新ID。
- 旧ID匹配键为来源文件名、正式题号和题型；历史重复键使用规范化题干消歧。
- API Key 从环境变量 `DEEPSEEK_API_KEY` 或本地 `pipeline/.env` 读取（两者皆无则跳过 LLM 标注）；`.env` 已被 gitignore，勿提交。
- 题型为选择题、填空题或解答题。
- 成功题目必须至少包含一条合法知识路径；标注失败时 `status=failed`。
- 选择题必须有 `content.options`；多小问解答题使用
  `content.sub_questions`，单问解答题允许只使用 `content.stem`。
- 最终 JSON 不存 `answer` 和 `solution`。
- 图片使用 `images/...` 相对路径。
- `build_all()` 会写入输入文件数、失败文件、完整性和验证报告。

## 维护约束

- `规范/知识点层级结构.md` 是知识树的权威来源。
- 标签去重只能删除完全相同的路径，不得按领域优先级丢弃不同标签。
- 未选择四级标签时必须保留空数组，不得从同一三级的多个候选中猜测。
- DOCX 答案、解析和详解中的编号不得被识别成新题。
- 修改解析器后必须验证当前 18 份试卷均非空且合计 435 题。
- 不要在未经确认时运行全量 API 标注。

---
> Source: [wenpl/LocalExam](https://github.com/wenpl/LocalExam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
