## canvas-editor

> - **任务前置检查**：在开始任何开发任务之前，必须首先检查项目根目录下的 `tasks/` 目录。

# Project Rules for AI Assistants

## 任务管理与进度追踪 (Task Management)

- **任务前置检查**：在开始任何开发任务之前，必须首先检查项目根目录下的 `tasks/` 目录。
  - 阅读 `tasks/overview.md` 以获取项目当前阶段的宏观进展。
  - 阅读具体的任务文件（如 `tasks/data-model.md`）以确认当前正在进行的细项任务，严禁进行重复工作。
- **任务后置更新**：任务执行完成后（包括编写代码、测试通过、打包验证等步骤），必须立即在对应的 `.md` 文件中更新进度。
  - 使用 `[x]` 勾选已完成的任务项。
  - 如果任务产生了新的子任务，应及时补充到清单中。
- **任务更新原则**：每实现一个功能，都必须在 `tasks/` 目录下找到对应的任务清单。如果之前不存在该任务，应先补齐；功能实现并验证后，必须立即勾选为已完成。
- **一致性**：确保 `tasks/` 目录下的进度描述与代码库的实际状态始终保持同步。

## 架构原则 (Architectural Principles)

- **数据模型 (Data Model)**：核心数据模型必须严格遵循 **OpenXML (OOXML)** 风格。
  - 命名空间、标签名（如 `w:p`, `w:r`, `w:t`）以及属性结构（如 `w:pPr`, `w:rPr`）应与标准一致。
  - 内部类继承架构（`DocumentElement` -> `ParagraphElement`, `RunElement`）必须保持清晰的职责分离。
- **Monorepo 管理**：使用 `pnpm workspace` 进行包管理。
  - 跨包依赖使用 `workspace:*`。
  - 库子包使用 `tsdown` 打包，应用子包使用 `Vite`。
- **高清渲染**：Canvas 渲染器必须考虑设备的 `devicePixelRatio` 和应用的 `zoom` 缩放，确保在所有屏幕下保持锐利。

## 开发规范 (Development Standards)

- **类型安全**：所有公共接口必须定义在 `@eiinu/editor-protocol` 中。
- **构建与测试**：每次提交代码前，必须确保 `pnpm build` 和 `pnpm test` 在全量环境下通过。
- **文档同步**：代码中的核心逻辑变更应同步更新对应的技术文档（如 `docs/data-model-spec.md`）。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eiinu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
