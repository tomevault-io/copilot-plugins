## cli-agent-collaboration

> CLI Agent 协作 — 修改文件前同步磁盘状态


# CLI Agent 协作

本项目会同时使用 **CLI Agent** 与编辑器内助手协作，同一仓库可能被多方改动。

- **在修改任何已有文件之前**：必须先 **重新读取磁盘上的当前内容**（例如用 `read_file`/等价方式），确认与自身 assumptions 一致后再打补丁。
- **禁止**仅凭会话记忆或旧片段直接覆盖，避免覆盖他方已提交的变更。

---
> Source: [liangchusan/Flownana](https://github.com/liangchusan/Flownana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
