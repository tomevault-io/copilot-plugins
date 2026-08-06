## codey

> - `README.md` 只写面向使用者的功能描述和必要注意事项，不写技术细节。

# AGENTS.md

## 文档约束

- `README.md` 只写面向使用者的功能描述和必要注意事项，不写技术细节。
- `README.md` 需要覆盖所有主要用户可见能力；不要因为去除技术细节而省略功能面。
- 更新 `README.md` 时，以 Codey 控制台和 Codex 页面内可见增强为核对清单，但不强制按控制台分区拆开书写。
- 不要在 `README.md` 中加入构建命令、发布流程、依赖版本、端口、协议、数据库结构、文件路径、内部模块名、注入/补丁策略或实现原理。
- 内部开发、构建、发布、配置路径、运行机制、性能策略和已知实现限制统一维护在 `INTERNAL_DEVELOPMENT.md`。
- 新增或调整功能时，如果用户能感知到行为变化，可以用非技术语言同步更新 `README.md`；涉及实现、测试、发布或维护细节时，只更新 `INTERNAL_DEVELOPMENT.md`。
- 修改文档前先判断目标读者：普通使用者读 `README.md`，维护者读 `INTERNAL_DEVELOPMENT.md`。

---
> Source: [SuperGness/codey](https://github.com/SuperGness/codey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
