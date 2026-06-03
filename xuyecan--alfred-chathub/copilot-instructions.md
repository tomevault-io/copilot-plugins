## alfred-chathub

> - `src/`: Python sources for providers and runtime (e.g., `openai.py`, `anthropic.py`, `gemini.py`, `qwen.py`, `ollama.py`, orchestration in `chat.py`, helpers in `helper.py`, streaming core in `llm_service.py`).

# Repository Guidelines

## Project Structure & Module Organization
- `src/`: Python sources for providers and runtime (e.g., `openai.py`, `anthropic.py`, `gemini.py`, `qwen.py`, `ollama.py`, orchestration in `chat.py`, helpers in `helper.py`, streaming core in `llm_service.py`).
- `info.plist`: Alfred workflow definition (triggers, variables, connections).
- `assets/`: Screenshots and icons used by the README and workflow.
- `icon.png`: Workflow icon.  `README.md`: Usage and screenshots.

## Build, Test, and Development Commands
- Python version: use system `python3` and ensure `#!/usr/bin/env python3` resolves correctly.
- Syntax check: `python3 -m py_compile src/*.py` (quick static validation).
- Package for sharing (UI): Alfred Preferences → Workflows → right‑click workflow → Export.
- Package via CLI (example): `zip -r Alfred-Chathub.alfredworkflow info.plist src icon.png assets` (ensure no secrets are included).

## Coding Style & Naming Conventions
- Python: PEP 8, 4‑space indents, explicit/clear identifiers. Remove unused imports.
- Comments: only for复杂逻辑且必须使用中文注释；避免赘述性的显而易见注释。
- Trailing whitespace: none. After `git add`, run `/Users/xuyecan/.macconfig/script/strip-trailing-ws-in-diff --staged`.
- Logs/output: technical output不得包含表情符号。
- Curly‑brace languages (if introduced): opening brace on same line; always use braces for single‑line bodies.
- Author header (when a source file requires it): `@author Created by Claude Code on 2025-10-10`.

## Testing Guidelines
- No formal test suite yet. Validate by: launching Alfred, triggering the workflow, verifying request/response, and checking `alfred_workflow_data` artifacts.
- Quick checks: compile step above; avoid printing secrets; use environment variables configured in the workflow for API keys and proxies.
- If adding tests, place them under `tests/` and prefer `pytest` with simple I/O unit tests for provider adapters.

## Commit & Pull Request Guidelines
- Commits: 使用中文消息与约定式前缀（`feat:`, `fix:`, `docs:`, `refactor:`, `chore:`, `config:`）。
- 模板：
  - 单行摘要
  - 空一行
  - 以短横线开头的要点清单（各行之间不留空行）
- 示例：
  - `feat: 新增 Deepseek 解析优化`
  - `fix: 代理参数在 socks5 模式下未生效`
- PR 需包含：变更说明、动机、涉及的文件（尤其是 `info.plist`）、验证步骤与必要截图；关联相关 issue。

## Security & Configuration Tips
- 配置 API Key、模型与代理于 Alfred 的环境变量；切勿将密钥写入代码、日志或导出的包。
- 避免在 `assets/` 与版本控制中包含敏感信息；提交前请自检。

---
> Source: [xuyecan/alfred-chathub](https://github.com/xuyecan/alfred-chathub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
