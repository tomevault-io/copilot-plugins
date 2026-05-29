## wechat-mp-harvest

> - 在这个仓库处理非琐碎任务前, 先读 `./.codex/AGENTS.md`.

# Vault 根级 Agent 说明

## 入口协议

- 在这个仓库处理非琐碎任务前, 先读 `./.codex/AGENTS.md`.
- 新线程首次接手仓库时, 先执行 `codex-memo quick "<task summary>"`, 再读 `merged_must_read` 与 route hits.
- 这个仓库的项目记忆统一放在 `./.codex/memory/`.
- 仓库级多步任务默认使用 `project-memory-loop`, 具体治理规则以该 skill 为准.

## 职责分层

- `./.codex/memory/` 只存项目记忆.
- `./.codex/tasks/` 只在长任务 workflow 启用时承担规划与执行状态.
- `docs/` 默认仍是给人看的正式文档, 除非用户明确要求别的用途.

<!-- CODEX-PROJECT-LAYER:START -->
## Codex Project Layer

- 处理非琐碎任务前, 先读 `./.codex/AGENTS.md`.
- 新线程首次接手仓库或处理非琐碎任务时, 优先执行 `codex-memo quick "<task summary>"`, 读取返回的 `merged_must_read`, `execution_gate` 与高置信 route hits.
- 仅当任务摘要未知时, 先执行 `codex-memo ov`, 再补 `codex-memo quick "<task summary>"` 或 `codex-memo r --task "<summary>"`.
- 若需要本地 capability 索引, 执行 `codex-memo a`, 先查看 `.codex/cache/asset-index.json` 的 `summary`, 再按需打开完整索引.
<!-- CODEX-PROJECT-LAYER:END -->

---
> Source: [Sophomoresty/wechat-mp-harvest](https://github.com/Sophomoresty/wechat-mp-harvest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
