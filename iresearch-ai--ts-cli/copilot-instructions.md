## ts-cli

> **Collab Spec 版本：** 2.3.0

# AGENTS.md

**Collab Spec 版本：** 2.3.0

**项目：** `ts-cli` —— 用 Go 编写为单二进制命令行工具，提供 AI 编码工具 token 用量解析、增量同步、Device Flow 初始化、远程 Summary 查询、跨平台 daemon、Skill 安装/卸载、配置管理、状态查看与重置。

**命名约束：**
- 二进制名称：`ts-cli`
- Go 模块路径：`github.com/iResearch-AI/ts-cli`
- 用户命令入口：`ts-cli init`、`ts-cli sync`、`ts-cli summary`、`ts-cli daemon`、`ts-cli skill`、`ts-cli config`、`ts-cli status`、`ts-cli reset`

**启用 AI 工具/接入方式：** Codex（原生读取 AGENTS.md + 项目级 `.codex/skills/`，如存在）、Kilo（专用路径 `.kilo/instructions.md`，如存在）
**受管指令文件：** `.kilo/instructions.md`（如存在；不含 AGENTS.md 自身）

## 权威来源优先级

| 优先级 | 来源 | 裁决范围 |
|--------|------|---------|
| **P0** | `AGENTS.md` | 工程结构、命名、不可协商原则与 Agent 驾驶规则 |
| **P1** | `README.md` 与 `README_zh_cn.md` | 公开产品范围、用户可见行为与项目入口 |
| **P2** | `docs/install.md` 与 `docs/release.md` | 安装、卸载、打包、发布与迁移细则 |
| **P3** | `agent/git.md` 与未来新增的 `agent/*.md` | Git 提交、分支、Tag/发布、提交前检查与其他工程细则 |

**冲突裁决：** 低优先级与高优先级冲突时，以高优先级为准；实现与公开文档冲突时，通过 Issue/PR 确认预期行为，并在同一变更中同步代码、测试和文档。

## 宪法

1. **声明目录结构。** 当前 Go CLI 实现在 `src/` 下。默认使用 `src/cmd/ts-cli/`（入口）、`src/internal/cmd/`（Cobra 子命令）、`src/internal/parser/`（解析器接口与 23 个实现）、`src/internal/api/`（HTTP 与 Device Flow 客户端）、`src/internal/state/`（增量同步状态）、`src/internal/config/`（配置读写）、`src/internal/i18n/`（国际化翻译）、`src/internal/daemon/`（daemon 与平台服务管理）、`src/internal/skill/`（Skill 安装/卸载）、`src/internal/output/`（终端输出）、`src/internal/sqlite/`（SQLite 只读封装）、`src/internal/tools/`（工具数据目录检测）、`src/embed/`（嵌入内容）、`src/rec/`（图标等二进制资源嵌入）、`src/scripts/`（在线/离线安装脚本）、`docs/`（公开安装与发布文档）、`agent/`（工程规范）、`outputs/`（生成产物，若需要）。不新增未声明的顶层目录；如需新增，先更新本文件。
2. **公开文档与用户文案支持中英文。** `README.md` 使用英文，`README_zh_cn.md` 提供中文版；CLI 面向用户输出通过 i18n 支持中英文，工程文档默认使用中文；代码标识符、协议字段、命令名保持英文。
3. **公开文档与行为同步。** README 描述产品范围，`docs/` 描述安装和发布细节；新增或修改用户可见行为时，在同一变更中更新实现、测试与相关文档。
4. **骨架先行，按模块迭代。** 项目级先保证 Cobra 根命令、模块边界、配置/状态/输出基础设施，再按 parser、api、sync、init、summary、daemon、installer 等模块推进。每次迭代留记录。
5. **单文件不超过 1000 行。** Go 源码、测试、文档均遵守；接近上限时按模块拆分，并在文档间显式链接。
6. **功能范围保持与 README 一致。** Go 版当前包含 23 个解析器、Device Flow + 手动 key、content-hash 增量同步、远程 Summary、daemon、Skill、Config、Status、Reset、Windows 系统托盘、安装脚本和并行解析器执行。
7. **服务端地址完全可配置。** 不硬编码任何业务服务端域名；`apiUrl` 来源优先级为 `--api-url` > `TS_CLI_API_URL` > 已保存的 `config.apiUrl`。`init` 时必须能写入后续复用的配置。服务端开源项目为 [`iResearch-AI/TokenScope`](https://github.com/iResearch-AI/TokenScope)；源码仓库地址不得作为默认业务服务端地址。
8. **解析器失败不阻断整体同步。** 23 个解析器通过 goroutine 并行执行；单个解析器错误应记录并跳过，不让整个同步失败。
9. **增量同步按批提交状态。** bucket/session 使用关键字段 JSON 序列化后的 SHA-256 content-hash；仅上传变化条目；每批上传成功后才提交 state；需要清理已消失的 key。
10. **SQLite 只读访问。** 使用 `modernc.org/sqlite`；默认 `mode=ro`；遇到锁库时复制 db、`-wal`、`-shm` 到临时目录后读取副本。
11. **跨平台 daemon 隔离实现。** macOS 使用 launchd，Linux 使用 systemd user service，Windows 使用注册表启动托盘模式；Windows GUI 依赖必须用 build tag 隔离，非 Windows 平台不得编译托盘依赖。
12. **本地构建以单二进制为核心，GitHub Release 仅发布源码。** Makefile 保留跨平台单二进制构建、测试与打包能力；官方 GitHub Release 不上传编译产物，仅使用 GitHub 为 annotated Tag 自动生成的源码归档。依赖二进制 Assets 的安装与升级能力在提供独立兼容发布源前不得声明为官方可用。
13. **完成前必须验证。** Go 代码变更至少运行相关包测试；涉及公共行为时运行 `cd src && go test ./...`。文档变更至少检查路径、引用和术语一致性。
14. **本文件仅维护工程结构声明与不可协商原则。** 仓库结构、权威文档、命令命名或核心范围变化时同步更新。

## Agent 驾驶规则

1. 先读权威来源，再改代码或文档；涉及产品范围和核心算法时，先核对 AGENTS.md、README 和相关测试。
2. 保持变更小而聚焦，不顺手重构无关模块。
3. 不覆盖用户未提交改动；遇到相关脏改动时先理解再协作。
4. 代码优先沿用现有 Go 包分层、Cobra 命令风格和测试结构。
5. 新增或修改用户可见行为时，同步测试和必要文档。
6. 涉及 Git 提交、分支、Tag/发布、提交前检查或不应提交内容时，遵循 `agent/git.md`。
7. 完成后说明变更点、验证结果和未验证风险。

## 项目指令文件同步

如启用 `.kilo/instructions.md`、项目级 `.codex/skills/` 或未来新增 `agent/*.md`，应与本文件保持结构、命名、目录与不可协商原则一致；同步规则不得反向覆盖本文件的权威声明。

---
> Source: [iResearch-AI/ts-cli](https://github.com/iResearch-AI/ts-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
