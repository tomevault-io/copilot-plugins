## codex-project-governor-skill

> - 默认使用简体中文回复；代码、命令、路径和 Git 标识保持原文。

# AGENTS.md - codex-project-governor-skill

- 默认使用简体中文回复；代码、命令、路径和 Git 标识保持原文。
- 修改公开说明时同步维护 `README.md` 与 `README_EN.md`。
- Skill 主体位于 `skills/codex-project-governor/`；可执行工具放在 `scripts/`，详细规则放在 `references/`。
- 保持 `AGENTS.md` 与 `codex/` 的职责边界：前者保存稳定规则，后者保存动态状态与断点。
- 修改 Skill 后运行 Python 编译、`unittest`、`git diff --check` 和 `skill-creator` 的 `quick_validate.py`。
- 未经明确要求不创建远端仓库、不添加远端、不推送或发布。
- `codex/` 保存本地执行状态并保持忽略。

---
> Source: [MDX-Tom/codex-project-governor-skill](https://github.com/MDX-Tom/codex-project-governor-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
