## verify-by-lint

> 不手动跑编译，以 lint 为准


# 校验方式：以 Lint 为准

- **不要**主动运行 `cargo build`、`npm run build` 等编译/构建命令来检查代码是否正确。
- **以**编辑器/工作区的 **lint 诊断** 为准：修改后参考 `ReadLints` 或 IDE 中的报错即可。
- 仅在用户明确要求「帮我编译一下」或「跑一下构建」时才执行编译命令。

---
> Source: [kabegame/kabegame](https://github.com/kabegame/kabegame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
