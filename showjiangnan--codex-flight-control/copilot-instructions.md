## codex-flight-control

> - `code/` 只存放本插件的源码工程、构建配置、测试和插件包资源。

# Repository Engineering Rules

## 目录边界

- `code/` 只存放本插件的源码工程、构建配置、测试和插件包资源。
- `docs/` 集中存放产品、架构、决策、质量、运维、计划和参考文档。
- 仓库根目录仅保留必要的入口与工程控制文件，例如 `README.md`、`AGENTS.md` 和 `.gitignore`。
- 不要读取、修改或提交 `.tmp_*` 等与本项目无关的临时资产。

## 文档先行

- 开始实现前先阅读 `docs/README.md`、相关架构文档、需求追踪表和已接受 ADR。
- 任何改变产品边界、状态机、数据契约、安全不变量、持久化格式或宿主集成假设的改动，必须先更新文档或新增 ADR。
- 代码、测试、数据迁移和文档必须在同一个交付中保持一致。
- 文档维护规则以 `docs/development/documentation-maintenance.md` 为准。

## 产品硬约束

- 本项目不实现 Computer Use；相关能力应由未来独立插件承担。
- 所有子代理保持为 Lead 的一层子节点，不建立失控的嵌套代理树。
- 大目标按 Wave 顺序推进；一个 Run 同一时刻最多有一个活动 Wave。
- 子代理遇到困难时通过本地 MCP 战术指导信箱求助，不得猜测关键事实。
- 私有 Skill 只能由用户显式选择创建命令后熔炼，不得自动创建。
- GitHub 来源必须先克隆到独占临时目录，再从本地安全快照熔炼。
- 自动 Git 操作只允许本地提交，禁止 push、PR、发布或修改远端状态。

## 工程完成标准

- 修改必须通过与风险相称的格式、类型、单元、集成、恢复和安全测试。
- 不得用“模型认为成功”替代确定性验证证据。
- 不得静默忽略无效配置、失败迁移、过期回信或权限拒绝。
- 不得为了让测试通过而放宽安全不变量。
- 每个手写、可维护的源码、测试、脚本和代码化配置文件不得超过 1200 个物理行；接近上限时必须按职责拆分，并由结构门禁持续检查。可重建的 bundle 和 source map 生成物不作为可维护源码计数，禁止手工编辑。
- 未选择实现语言和工具链前，不要擅自生成大规模源码脚手架。

## Git

- 默认分支为 `main`。
- 保留用户已有改动，禁止破坏性重置。
- 不执行 `git push`、创建 PR、发布包或其他远端写操作。
- 每个本地提交应是可评审、可验证、自洽的交付单元。

---
> Source: [showjiangnan/codex-flight-control](https://github.com/showjiangnan/codex-flight-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
