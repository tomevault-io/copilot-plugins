## ask-web-ai

> - 开发源码以当前仓库为准；全局安装目录只是通过验证后的同步产物，不直接修改。

# AGENTS.md

## 开始前

- 开发源码以当前仓库为准；全局安装目录只是通过验证后的同步产物，不直接修改。
- 先读 `README.md` 和与改动有关的 `SPEC.md`，再读相邻实现与测试。修改 Provider 或 Runtime 时，同时阅读 `references/` 中对应说明。
- `SPEC.md` 负责运行行为和状态机，本文件负责开发纪律；两者冲突时先澄清并修正文档，不自行选择。

## 职责

- CLI 只处理参数、stdin 和 JSON 输出。
- Coordinator 管理 Job 生命周期、恢复顺序、租约和清理，不解释网页。
- Provider Adapter 管理网站页面语义、定位、观察和浏览器操作。
- Runtime 管理 Tabbit 启动、Task、receipt、resource、checkpoint 和 finish，不包含 Provider 语义。
- Store 各自只管理一种持久化数据；测试通过公开接口验证行为。

## 规则

- Coordinator、状态机和 Provider 语义跨平台共享。只有路径、Git、权限、launcher 和 GUI Runtime 等已证明的平台原语可以隔离，禁止复制整套业务实现。
- 输入暂存与 Send 必须分离。任何可能已经派发的操作只能只读对账，不得盲目重试、换会话或切换浏览器后端。
- 结果必须先持久化并交付，之后才能确认和清理 Task Space。
- 动态生成或发送到另一运行环境的程序必须显式包含依赖，并在隔离环境中实际执行测试；只检查语法不算验证。
- 修复应针对根因和可观察行为，不为通过测试增加平台特判、重复判断或无业务含义的 fallback。
- 允许在大文件中做窄范围修复；新增职责应进入职责明确的模块，不机械拆分，也不继续扩大现有耦合。

## 验证与文档

- 确定性测试证明实现完成；Linux、macOS、Windows CI 全绿才能称为跨平台验证完成。
- 涉及真实 Browser mutation、Runtime 或平台问题时，必须在受影响平台完成对应 live 验证；未执行的验证必须明确标为未验证。
- 行为、状态机、CLI、目录职责或安全约束变化时，同步更新对应的 `README.md`、`SKILL.md`、`SPEC.md`、`docs/`、`references/` 和测试，只更新实际受影响的文件。

---
> Source: [pgw10086/ask-web-ai](https://github.com/pgw10086/ask-web-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
