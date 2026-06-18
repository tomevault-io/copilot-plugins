## typescript-style

> TypeScript 代码风格与模块解析

# TypeScript 代码规范

- 使用 ES Module 语法（`import`/`export`）。
- 导入路径允许不带扩展名（运行时通过 `--experimental-specifier-resolution=node`，编译期通过 `moduleResolution: bundler`）。
- 保持明确类型：`Express`, `Router` 等关键对象添加显式注解。
- 避免使用 `any`；如需兼容，尽量局部、保守地放宽。
- 避免深层嵌套，优先早返回。
- 与现有格式保持一致（prettier/eslint 若存在则优先遵循）。

## 构建/运行要点
- 源码位于 `src/`，构建产物输出到 `dist/`。
- 开发运行使用：`node --loader ts-node/esm --experimental-specifier-resolution=node`。
- 生产构建使用：`tsc`，生产运行 `node dist/...`。

---
> Source: [iamouyang21/DeepResearch-Langgraph](https://github.com/iamouyang21/DeepResearch-Langgraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
