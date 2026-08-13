## downcity

> 1. 所有架构设计、代码重构、公开 API 与跨平台实现必须遵循 [`docs/engineering-design-standard.md`](docs/engineering-design-standard.md)。

# 工程设计规范

1. 所有架构设计、代码重构、公开 API 与跨平台实现必须遵循 [`docs/engineering-design-standard.md`](docs/engineering-design-standard.md)。
2. 设计决策依次从产品意图、领域职责、所有权与生命周期、依赖方向、最小公开 API 推导，不能从现有目录或最少改动倒推架构。

# Package

1. 如果 package 更新新特性和用户使用相关， 则需要 更新 homepage 当中的 docs。
2. 模块最多800-1000行，超过之后拆分成多个模块。
3. 尽量不要使用动态导入
4. 每个模块都应该写详细的文档注释。
5. 关键节点所有的注释都使用中文
6. 类型统一放到 types/ 下面
7. package 对外能力、SDK API、用户可见行为发生变化并准备提交时，必须使用 patch 脚本完成版本号自增与构建：
   - 只改 `@downcity/agent`：`pnpm agent:patch:build`
   - 只改 `@downcity/city`：`pnpm city:patch:build`
   - 只改 `downcity`：`pnpm cli:patch:build`
   - 涉及多个 package：根据实际影响范围显式指定，例如 `pnpm patch:build -- --city --services --database-d1`
   - 只允许 bump 对外能力实际发生变化的 package；依赖 package 可以参与构建验证，但不能因此自动 bump
   - 仅验证不需要 bump 时，显式指定验证范围并使用 `pnpm patch:build -- --no-bump --city --services`

# Patch + Commit

1. commit 前先确认改动范围，只 stage 本次任务相关文件，不要混入用户未要求的改动。
2. 如果属于 package 用户可见更新，先运行对应 patch 脚本，再补跑受影响区域的 typecheck / lint。
3. patch 脚本通过后再 `git add`、`git commit`，commit message 使用明确作用域，例如 `feat(agent): add session system prompts api`。
4. commit 完成后用 `git status --short` 确认没有遗漏本次任务应提交的文件。


# Homepage

1. 仅面向用户写文档，不需要写开发文档，开发文档在 docs 中


# 其他开发

1. 永远不要考虑向后兼容，直接迭代。保证功能一致，采用做合适的实践。
2. 所有的模块必须添加文件注释/模块注释
3. 你在创建任何 type 类型的时候，都需要每个字段都给到详细的注释说明。
4. 变量命名和function命名使用snaker，例如 current_time, 不要使用驼峰命名。
5. 不要补丁式开发，而应该使用最佳实践。遇到bug先考虑最佳实践方案，如果改动较大，需要让用户确认。

---
> Source: [genesiscosmos/downcity](https://github.com/genesiscosmos/downcity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
