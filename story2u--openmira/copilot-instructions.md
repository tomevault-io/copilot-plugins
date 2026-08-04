## openmira

> 本文件是 Codex、Claude Code 及其他编码代理进入仓库时的短入口，不是项目百科。

# AI 开发入口

本文件是 Codex、Claude Code 及其他编码代理进入仓库时的短入口，不是项目百科。
`docs/` 是项目知识库；代码、迁移、测试和可执行配置是行为的最终证据。

## 开始任务前

1. 先读 [文档导航](docs/README.md)，再按任务类型加载必要文档，避免一次读完所有资料。
2. 修改跨模块行为前读 [架构总览](docs/architecture/overview.md)；修改产品行为前读
   [功能地图](docs/product/feature-map.md)。
3. 读目标代码及其相邻测试，验证文档描述；发现漂移时在同一变更中修正文档。
4. 复杂、跨层、涉及迁移或需求不确定的工作，先按 [执行计划规范](docs/plans/README.md)
   在 `docs/plans/active/` 建计划；小改动可在会话内维护轻量计划。

## 全局约束

- 保持改动小而完整，不顺手重构无关代码，不覆盖用户未提交的修改。
- 后端遵循现有分层边界；`domain` 不得依赖框架、数据库、API 或 worker。
- API 契约变化必须同步 Pydantic DTO、前端类型/调用方、测试与功能地图。
- 数据库结构变化必须新增 Alembic 迁移；不得只改 SQLModel，也不得重写已发布迁移。
- 生产路径不得静默回退到 mock。演示逻辑必须明确标注且不得被描述成已交付能力。
- 认证、密钥、webhook、IM 发送和 AI 自动回复按高风险变更处理；遵守
  [安全基线](docs/quality/security.md)，不得提交秘密或真实用户数据。
- 优先复用现有模式和依赖。新增依赖、跨层抽象或不可逆决策时记录 ADR。

## 工作闭环

按“理解 → 计划 → 实现 → 验证 → 更新知识库”推进：

- 先定义验收标准和影响面，再编辑代码。
- 用最小相关测试获得快速反馈，完成后运行与风险匹配的完整检查。
- 修复根因；禁止通过删除测试、放宽断言、吞异常或关闭检查制造通过。
- 未实际运行的检查必须明确写成“未运行”，不得声称通过。
- 变更架构、功能状态、命令、配置、运维方式或已知限制时，同步更新对应文档。

## 验证入口

- harness 与架构边界：`make harness-check`
- 后端快速验证：`make backend-check`
- 前端静态验证：`make frontend-check`
- 本地完整验证：`make check`

命令细节、依赖前提和按改动类型选择检查的方法见
[开发命令](docs/development/commands.md)与[测试策略](docs/quality/testing.md)。

## 按任务加载文档

- 系统边界、依赖方向、数据流：[架构总览](docs/architecture/overview.md)
- 页面、API、能力成熟度、已知断点：[功能地图](docs/product/feature-map.md)
- Python、React、API、迁移和日志规范：[开发规范](docs/development/standards.md)
- 功能开发、缺陷修复和文档维护流程：[开发工作流](docs/development/workflow.md)
- 环境变量、容器、队列和部署：[运行与运维](docs/operations/runtime.md)
- 方案决策记录：[ADR 索引](docs/decisions/README.md)
- 长任务状态、决策日志和技术债：[执行计划](docs/plans/README.md)
- harness 原则及维护方式：[Harness 原则](docs/harness/principles.md)

## 完成定义

实现满足验收标准；相关检查通过；安全与失败路径已考虑；没有把真实能力退化为 mock；
文档与代码一致；交付说明列出改动、验证结果及仍存在的风险。

---
> Source: [story2u/OpenMIRA](https://github.com/story2u/OpenMIRA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
