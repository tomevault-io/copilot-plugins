## python-code-style

> Darkeye Python/Qt 代码风格（与 docs/development.md 一致）


# 代码规范（写代码时必须遵守）

更完整说明见 `docs/development.md` 中「代码规范」一节。

- **缩进**：4 个空格，禁止 Tab 与空格混用。
- **行长**：文档/注释倾向 ≤79；代码可用 88/100（与 Black 等工具对齐）。
- **命名**：模块/包 `lowercase`；类 `CapWords`；函数/方法/变量 `lower_with_underscores`；常量 `ALL_CAPS`；私有用 `_leading_underscore`。**Qt 信号/槽名用小驼峰**。
- **导入**：标准库 → 第三方 → 本地，组间空一行；避免 `import *`。
- **空格与字符串**：运算符两侧、逗号后留空；括号内不无故加空格；字符串与当前文件风格一致，无特殊理由可统一双引号。
- **文档**：遵守 PEP 257（docstring 格式与写法）。
- **类型**：遵循 PEP 484 及后续（`typing`、`|` 联合类型等）；是否处处标注由项目习惯而定。
- **格式化**：使用 **Black** 整理代码风格。

新代码应优先与**同文件、同目录**已有风格一致；不确定时以本节为准。

---
> Source: [de4321/darkeye](https://github.com/de4321/darkeye) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
