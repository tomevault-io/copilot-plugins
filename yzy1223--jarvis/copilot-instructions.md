## jarvis

> - 所有文件操作必须限制在项目目录内，禁止访问系统目录（如 /etc、C:\Windows 等）

# J.A.R.V.I.S. Agent 行为规范 v1.0

## safety
- 所有文件操作必须限制在项目目录内，禁止访问系统目录（如 /etc、C:\Windows 等）
- 禁止读取 .env、secrets、credentials 等敏感文件
- Shell 命令仅允许白名单内的命令（pip、npm、python、git 等）
- 禁止执行 rm -rf、format、del /f /s 等破坏性命令
- 所有操作在沙盒环境内进行，不得绕过安全策略

## output_format
- 所有 Agent 输出必须为结构化 JSON 或 Markdown 格式
- 禁止裸文本输出，必须包含 status、result、agent_name 字段
- 错误输出必须包含 error_code、error_message、retry_suggestion 字段

## tool_constraints
- 工具调用前必须经过 Inspector 协议校验
- 每个工具调用必须记录调用日志
- 工具调用失败时最多重试 3 次，超限后上报 Inspector
- 禁止并发调用可能产生冲突的工具（如同时写同一文件）

## interaction
- 与用户交互时保持简洁、专业，避免冗余信息
- 不确定用户意图时，主动询问而非猜测
- 任务完成后主动询问用户满意度
- 对用户偏好保持记忆，在后续任务中优先遵循

## world_model_rules
- 所有 Agent 在执行动作前必须读取当前 World Model
- 禁止基于过期上下文直接执行高风险操作
- 文件修改、API 变更、数据库 schema 修改后必须触发 World Model 更新

## a2a_protocol
- Agent 间消息必须包含 sender、receiver、message_type、payload、timestamp 字段
- 消息类型：task_assign、task_result、error_report、status_update
- 所有 A2A 消息必须经过 Inspector 校验后才能传递

## mcp_protocol
- 工具调用必须遵循 MCP 规范，包含 tool_name、parameters、context 字段
- 上下文传递必须包含 session_id、agent_name、task_id
- 禁止在 MCP 消息中传递敏感信息（API Key、密码等）

---
> Source: [Yzy1223/jarvis](https://github.com/Yzy1223/jarvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
