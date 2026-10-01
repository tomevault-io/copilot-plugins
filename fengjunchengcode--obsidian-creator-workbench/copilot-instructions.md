## obsidian-creator-workbench

> 本文件面向 Codex CLI 及其他读 `AGENTS.md` 的 Agent。

# Agent 入口

本文件面向 Codex CLI 及其他读 `AGENTS.md` 的 Agent。

**工作流指令、目录规范、评分口径、Obsidian 约定全部写在 [`CLAUDE.md`](CLAUDE.md) 里，请先完整读取那份文件再动手。**
两份文件描述的是同一套系统，`CLAUDE.md` 是唯一真源；不要在本文件里追加规则，否则两端会漂移。

## 最低限度须知（不能替代读 CLAUDE.md）

- 所有新建 `.md` 必须带 YAML front matter（`tags` / `type` / `status` / `created`），否则看板和 Dataview 都识别不到
- 任何选题在评估、推荐、深化、排期前，必须先过 `05-方法论沉淀/选题方法论.md` 第 0 关的方向三问，一票否决
- `predictions/` 里已写完的预测段**不可修改**，有 hook 强制拦截；需要修正就追加新版本段
- 改完 vault 内容后，用 `cd dashboard && python3 build.py` 重建工作台

---
> Source: [fengjunchengCode/obsidian-creator-workbench](https://github.com/fengjunchengCode/obsidian-creator-workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
