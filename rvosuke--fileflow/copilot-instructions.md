## fileflow

> 2. 修改共享核心文件前，先在 `INBOX.md` 发锁消息

# Gemini <-> Team

## 协作入口

- 实时短消息：`INBOX.md`
- 长文档总结：本文件

## 当前协议

1. 开始一轮工作前先读：
   - `INBOX.md`
   - `GIT_WORKFLOW.md`
   - 相关协作文档
2. 修改共享核心文件前，先在 `INBOX.md` 发锁消息
3. `[NEED-REPLY]` 消息需要优先回执
4. Git 分支前缀使用 `gemini/<scope>`

## 当前项目状态

- `main` 已包含：
  - feedback learning / preview / config edit
  - manual rules CRUD
  - content preview enhancement (.json, .tar.*)
- Codex 当前在推进：
  - FastAPI API skeleton (`fileflow/api/app.py`, `fileflow/cli.py` serve)
- Claude 当前在推进：
  - `tests/test_watcher.py`
  - `tests/test_rule_cache.py`
  - `tests/test_executor.py` / `tests/test_dedup.py` coverage

## Gemini 的工作清单

1. **[Web] 前端骨架搭建 (Vue 3 + Vite)** [DONE]
   - 目标：在 `frontend/` 初始化项目并完成基础路由与 API 对接。
   - 状态：已完成 Dashboard, Rules, History, Corrections 视图的编写。
2. **[Core] 文件内容预览增强** [DONE]
   - 目标：在 `fileflow/analyzer/content.py` 中实现摘要/预览提取。
   - 状态：已支持 JSON 格式化预览和多格式压缩包列表。
3. **[Docs/Chore] 完善项目 README.md 与打包调研** [IN PROGRESS]
   - 目标：更新文档以适应新 API 结构，调研 PyInstaller 打包方案。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rvosuke)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Rvosuke)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
