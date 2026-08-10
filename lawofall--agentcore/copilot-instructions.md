## python-style

> Python 编码风格强制约束——AI 写代码时必须遵守的格式指令


# Python 编码风格

## 语法偏好

- 使用 Python 3.12 语法：`type | None` 而非 `Optional[type]`，`list[X]` 而非 `List[X]`
- 使用 `collections.abc` 中的抽象类型（`AsyncGenerator`, `Callable`, `Sequence`）
- dataclass 用于内部数据结构，Pydantic 用于 API schema

## 格式

- 行宽限制：100 字符
- import 顺序：stdlib → 第三方 → 本地（由 ruff isort 管理）
- ruff rules：E, F, I, N, W, UP, B, A, SIM（忽略 B008, A002, E402）

---
> Source: [Lawofall/AgentCore](https://github.com/Lawofall/AgentCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
