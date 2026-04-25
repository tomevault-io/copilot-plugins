## darkeye

> 面向在本仓库内协助开发的 AI / 自动化助手：以下为默认应遵守的上下文；细节以链接文档为准。

# Agent 说明

面向在本仓库内协助开发的 AI / 自动化助手：以下为默认应遵守的上下文；细节以链接文档为准。

## 项目概览

- **Darkeye**：桌面端影片收藏与管理（PySide6），含本地服务（FastAPI / Uvicorn）、爬虫与数据库等模块。
- **Python**：`>=3.13,<3.14`（见 `pyproject.toml`）。
- **UI**：业务界面多在 `ui/`；可复用组件与主题在 `darkeye_ui/`。

## 目录速览

| 路径 | 说明 |
|------|------|
| `main.py` | 应用入口 |
| `ui/` | 页面与窗口级 UI |
| `darkeye_ui/` | 共享 UI 组件与布局 |
| `core/` | 业务核心（数据库、爬虫、图布局、DVD 等） |
| `controller/` | 应用壳层：主题上下文、全局信号、快捷键、`MessageService` 等 |
| `server/` | 本地 HTTP 服务相关 |
| `utils/` | 通用工具（含日志配置 `log_config`、启动分析 `profiler` 等） |
| `config.py` | 全局配置入口 |
| `tests/`、`manual_tests/` | 自动化测试与手工/demo 脚本 |
| `cpp_bindings/` | Qt/C++ 扩展；非必要不改动 |
| `docs/` | 开发与变更说明 |

## 环境与运行

首次/setup、资源复制、浏览器插件加载等：**`docs/development.md`**（开发环境准备、运行、打包）。

常见本地运行：`pip install -e ".[docs]"`（或按文档选用 conda 环境）后执行 `python main.py`，或在 IDE 中选择解释器后 F5。

## 代码与风格

- 编写或修改 Python/Qt 代码时遵守 **`.cursor/rules/python-code-style.mdc`**。
- 格式化：**Black**（行宽等见 `pyproject.toml` 中 `[tool.black]`）。
- 规范条文全文：**`docs/development.md`**「代码规范」一节。

## 测试

- 需要验证逻辑时，可运行：`pytest`（项目已依赖 pytest）；专注套件见 `tests/`。
- `manual_tests/` 多为交互或临时脚本，按需单独运行，不必默认全跑。

## 修改范围与禁区
- 不要删除用户的注释
- **聚焦需求**：只改与当前任务相关的文件；避免顺带大重构或无关格式化。
- **勿提交密钥**：路径、令牌、个人数据不要写入仓库。
- **打包与精简排除**：`build-pyinstaller.ps1` / `build-nuitka.ps1` 等涉及激进裁剪；无明确需求不要改打包配置。
- **`cpp_bindings/`**：仅在任务明确要求修改 C++/绑定时再动，并需对应 Qt/VS 环境（见 `docs/development.md`）。

## 文档索引

- **`docs/development.md`**：环境、运行、代码规范、打包。
- **`docs/CHANGELOG.md`**：版本变更（若用户要求记录变更时可参考）。

---
> Source: [de4321/darkeye](https://github.com/de4321/darkeye) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
