## str-download

> - 本文件适用于 `D:\code\str_download` 仓库内的所有自动化编码代理。

# AGENTS 指南（str-download）

## 1. 适用范围
- 本文件适用于 `D:\code\str_download` 仓库内的所有自动化编码代理。
- 目标：在最小改动前提下，保持可读性、类型安全、可测试性。
- 若用户指令与本文件冲突，优先执行用户指令。

## 2. 仓库事实（必须遵守）
- 语言与运行时：Python `>=3.11`（见 `pyproject.toml`）。
- 项目入口（CLI）：`python -m src.cli --root-path "D:\\movies"`。
- 项目入口（GUI）：`python -m src.gui`。
- 测试框架：`pytest`。
- 测试目录：`tests/`（由 `tool.pytest.ini_options.testpaths` 指定）。
- Python 路径：项目根目录（`pythonpath = ["."]`）。
- 目前仓库未发现专门 lint/type 配置文件（如 `ruff.toml`、`mypy.ini`、`pyrightconfig.json`）。

## 3. Cursor / Copilot 规则同步
- 已检查 `.cursor/rules/`：未发现规则文件。
- 已检查 `.cursorrules`：未发现该文件。
- 已检查 `.github/copilot-instructions.md`：未发现该文件。
- 若后续新增上述规则文件，代理在实现前必须先读取并合并规则。

## 4. 环境初始化与依赖
- 安装基础与开发依赖：
  - `python -m ensurepip --upgrade`
  - `python -m pip install -e .[dev]`
- 若仅运行应用可使用最小依赖：`python -m pip install -e .`
- 建议在虚拟环境中执行所有命令。

## 5. 常用命令（Build / Lint / Test）
- 运行 CLI：`python -m src.cli --root-path "D:\\movies"`
- 运行 GUI：`python -m src.gui`
- 打包 Windows GUI（onedir）：
  - `pyinstaller --noconfirm --clean --windowed --name str-download-gui src/gui.py`

### 5.1 测试命令
- 全量测试：`python -m pytest -q`
- 指定文件：`python -m pytest tests/test_pipeline.py -q`
- 单个测试（重点）：
  - `python -m pytest tests/test_pipeline.py::test_runPipeline_organizeAndDownload -q`
- 关键字过滤：`python -m pytest -k organize -q`

### 5.2 Lint / 格式化命令（仓库未强制，但建议）
- 建议 lint：`python -m ruff check src tests`
- 建议格式化检查：`python -m ruff format --check src tests`
- 自动格式化：`python -m ruff format src tests`
- 说明：当前仓库未提供 ruff 配置，默认规则可能与历史代码不完全一致。

## 6. 代码组织与模块边界
- 源码目录：`src/`，测试目录：`tests/`。
- 主要模块职责：
  - `src/organizer.py`：扫描并整理根目录 mp4。
  - `src/pipeline.py`：主流程编排、日志、manifest 去重、下载调度。
  - `src/subtitle_client.py`：迅雷字幕 API、限流、重试、缓存、预算控制。
  - `src/scorer.py`：候选字幕打分与 Top3 排序。
  - `src/gui.py`：PyQt6 队列执行 UI。
  - `src/logger.py`：JSONL 日志输出。
  - `src/models.py`：Pydantic 数据模型。
- 新逻辑优先放入现有模块，不随意新增“杂项 util 文件”。

## 7. 导入规范（Imports）
- 优先使用绝对导入：`from src.xxx import yyy`。
- 标准库、第三方、本地模块分组，组间空一行。
- 建议保留 `from __future__ import annotations`（现有文件普遍使用）。
- 避免未使用导入；提交前清理。

## 8. 命名与风格规范
- 遵循仓库现状：
  - 函数/变量：`camelCase`（如 `runPipeline`, `rootPath`）。
  - 类名：`PascalCase`（如 `XunleiSubtitleProvider`）。
  - 常量：`UPPER_SNAKE_CASE`（如 `MOVIE_INTERVAL_SEC`）。
- 新增代码保持与所在文件一致，不跨文件强行改风格。
- 避免单字符变量名（循环计数除外）。

## 9. 类型与数据建模
- 必须写显式类型注解（参数、返回值、关键局部变量）。
- 禁止使用 `any` 逃避类型约束。
- 数据实体优先使用 `pydantic.BaseModel`（参考 `src/models.py`）。
- 对不稳定外部输入先做 `isinstance` 校验再使用。
- 路径统一使用 `pathlib.Path`。

## 10. 格式与可读性
- 缩进使用 2 空格（新增代码遵守团队约定）。
- 单函数保持单一职责；过长时拆分私有函数。
- 仅在“非显而易见”逻辑处添加简短注释。
- 不保留注释掉的废弃代码。
- 文本文件读写显式指定编码（优先 `utf-8`）。

## 11. 错误处理与稳定性
- 输入校验失败应尽早抛错（例如目录不存在时 `ValueError`）。
- 对外部 I/O（网络、文件、JSON 解析）做异常处理并降级。
- 捕获异常时：
  - 优先捕获具体异常（如 `json.JSONDecodeError`）。
  - 必要时记录上下文（`fileId`、目标路径、URL、状态码）。
  - 不要静默吞错；至少日志可追踪。
- 抛出新异常时保留链路：`raise RuntimeError(...) from error`。

## 12. 日志与可观测性
- 流程事件通过 `JsonlLogger.logEvent` 记录。
- 日志字段保持兼容：`ts/runId/fileId/stage/status/msg/data`。
- 新阶段命名简短稳定（如 `scan/search/score/download`）。
- 面向 UI 的日志内容应简洁可读。

## 13. 测试策略
- 每次改动至少补充或更新一个相关测试。
- 优先单元测试纯函数（`scorer`、辅助函数）。
- 涉及流程改动时，至少覆盖一条 `runPipeline` 路径。
- 避免真实网络请求：使用 `SubtitleProvider` mock/stub。
- 对文件系统行为使用 `tmp_path` 隔离。
- 新增测试命名格式：`test_<module>_<behavior>()`。

## 14. 变更实施流程（代理执行）
- 第一步：阅读将修改的文件与相关测试。
- 第二步：做最小必要改动，不顺手重构无关代码。
- 第三步：运行受影响测试；可再跑全量测试。
- 第四步：若引入新命令/参数，同步更新 `README.md`。

## 15. 禁止事项
- 不要无依据修改公共行为阈值（如评分阈值、限流、预算）。
- 不要引入重量级新依赖，除非用户明确要求。
- 不要把临时调试输出留在提交中。
- 不要破坏 `subtitle_manifest.json` 的兼容结构（`downloads` 列表）。

## 16. 提交前检查清单
- `python -m pytest -q` 通过。
- 关键路径可运行（至少验证 CLI 或相关测试）。
- 新增/变更代码有类型注解。
- 未引入未使用导入和死代码。
- 文档与实现一致（命令、参数、行为描述）。

## 17. 代理输出建议
- 向用户汇报时先给结果，再给关键文件路径。
- 命令输出只提炼关键信息，不粘贴无关长日志。
- 若存在不确定点，明确标注“事实”与“假设”。

## 18. 快速命令抄录
- 安装开发依赖：`python -m pip install -e .[dev]`
- 全量测试：`python -m pytest -q`
- 单测示例：`python -m pytest tests/test_pipeline.py::test_runPipeline_organizeAndDownload -q`
- 启动 CLI：`python -m src.cli --root-path "D:\\movies"`
- 启动 GUI：`python -m src.gui`

---
> Source: [lulalulaluobo/str_download](https://github.com/lulalulaluobo/str_download) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
