## sync-docs

> 功能性修改后同步文档


# 文档同步

Agent 结束时 `.cursor/hooks/check-docs-sync.sh` 会自动检查是否遗漏文档更新。

当修改涉及以下内容时，检查 `docs/` 和 `README.md` 是否需要同步更新：

- API 端点或参数变更 → 更新 `README.md`、`docs/development.md`、`docs/agents/web/backend/README.md`
- CLI 参数或默认值变化 → 更新 `README.md`、`docs/development.md`、`docs/agents/apps/README.md`
- 构建 / 部署流程变化 → 更新 `README.md`、`docs/development.md`、`docs/deployment.md`
- 用户可见的功能或行为变更 → 更新 `README.md`、`docs/development.md`、对应前端/后端模块索引

---
> Source: [Neroued/ChromaPrint3D](https://github.com/Neroued/ChromaPrint3D) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
