## pm-prototype

> - Windows + PowerShell（非 Bash）

# 语言与平台
- 中文沟通
- Windows + PowerShell（非 Bash）

# 路径
- 文档/产物中出现的路径一律写相对路径（以项目根目录为基准）

# 终端命令
- 禁止 Bash 语法（`<<`、`&&`、`export` 等）
- 禁止交互（Python REPL、需要人工输入）
- 命令必须可结束；需要常驻时必须声明可停止
- 命令保持短可控；复杂逻辑优先改文件而非塞进一条命令
- 先检索/读文件确认上下文，再运行验证命令

# 脚本（PowerShell + Python）
- 仅允许新增 Python（`.py`）脚本；禁止新增 PowerShell（`.ps1`）脚本
- 正式文本文件（Markdown、JSON、Skill、Agent、PRD、Vue 源码等）的任何修改，必须经 repo 内 Python 工具层或 `.py` 脚本落盘；禁止 shell 直接写入
- `python -c` 仅用于短小、单行、ASCII 为主、无复杂引号或转义的一次性只读命令
- 禁止使用 `@'... '@ | python -` 或任何 PowerShell 管道把中文/多行脚本直接送入 Python stdin，尤其禁止用于正式文本写入
- 禁止使用 `Get-Content`/`Set-Content` 直接改写正式中文文档、Markdown、JSON、Skill、PRD、Vue 源码
- 所有 Python 文本读写必须显式指定 `encoding="utf-8"`，并通过原子写入替换目标文件
- PowerShell 只允许用于只读检查、存在性判断、目录遍历、`npm/git/build` 执行和本地服务启动；不是正式文本生成层
- 复杂写入的唯一允许方式：执行 repo 内 `.py` 文件；必要时可创建临时 `.py` 文件执行，并在完成后删除临时文件

# Pencil 原型文件（.pen）
- **唯一操作方式**：仅允许通过 Pencil MCP 工具操作 `.pen` 文件（读取、修改、截图等）
- **禁止事项**：
  - 禁止使用 Python 脚本、JSON 库、文本编辑等方式直接读写 `.pen` 文件
  - 禁止绕过 Pencil MCP 对 `.pen` 进行任何修改
- **MCP 未连接时**：
  - 禁止对 `.pen` 文件进行任何修改操作
  - 只能通过 Pencil MCP 读取 `.pen` 内容（需在编辑器中打开文件）
  - 必须在交付中标记"风险：MCP 未连接，无法验证原型修改"

# Agent 与 Skill
- Agent（如 `PM_Agent/AGENT.md`）：只写编排协议（触发/输入输出边界/禁止项/顺序/失败停机/门禁存在性）
- Skill（如 `.trae/skills/*/SKILL.md`）：只写执行细则（步骤/增量/校验/缺口输出/允许或禁止推导）
- 同一规则只保留一个“唯一来源”；若必须两处出现：Agent 只写硬边界，细节放 Skill
- 改规则前先检索是否已有；优先修改原处，避免追加新段
- 修改后必须做一致性复核（`TBD` 口径、输入源、写入路径、增量范围、失败策略）

## 全局红线
- 当前环境默认按 Windows 处理；所有新增或改写的文本文件默认使用 UTF-8，不能依赖本地代码页或终端显示效果判断真实内容。

---
> Source: [patsyang/pm-prototype](https://github.com/patsyang/pm-prototype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
