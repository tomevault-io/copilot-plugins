## 008-loudon-command-comments

> 禁止修改含 #command by loudon 的注释行与注释块（产品指令，软约束）


# Loudon Command Comments（软约束）

## 识别标记

源码中出现以下标记（大小写不敏感）时，表示 **loudon 的产品指令注释**：

- `#command by loudon`
- `command by loudon`（同一注释内等价）

常见写法：`//`、`/* */`、JSX `{/* */}`、`#`（Python/Shell）等。

## 禁止（Agent 须遵守）

1. **禁止修改**带上述标记的**整行注释**（含标记行本身）。
2. **禁止修改**以该标记为起点的**整块注释**（从含标记的注释起始到该注释闭合符为止，例如 `*/`、`*/}`）。
3. **禁止**在重构、格式化、删 dead code、启用被注释代码、lint 自动修复时触碰上述区域。
4. **禁止**将注释块解开为可执行代码，或删除/缩短/改写注释内的说明文字，除非用户 **loudon** 明确要求并指明文件与块。

## 允许

- 在**注释块之外**正常改业务代码。
- 用户或 loudon 明确授权并点名文件与注释块时，方可改动该块。

## 改前自检

若计划中的 diff 会触及含 `#command by loudon` 的行或其注释块：

- **停止**，不要提交该 diff。
- 在回复中说明被锁定的文件与行号，请用户确认是否由 loudon 授权变更。

本项目**无** CI / lock / 脚本自动校验；仅依赖本规则与 `AGENTS.md` 的说明约束 Agent 行为。

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
