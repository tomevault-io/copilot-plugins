## agent-hub

> 用户的私人 Agent 仓库（GitHub: Dy1ngn/agent-hub），存放各个 Agent 的**实现代码**。

# Agent Hub — 项目记忆

## 项目定位
用户的私人 Agent 仓库（GitHub: Dy1ngn/agent-hub），存放各个 Agent 的**实现代码**。
每个 Agent 是顶层 `agents/` 目录下的一个独立文件夹，内含代码和一份 `README.md` 介绍文件。

## 目录结构
- `agents/` — 所有 Agent，一个文件夹 = 一个 Agent
  - `agents/<agent 名>/README.md` — 该 Agent 的介绍文件（汇总到顶层 README 的数据来源）
  - `agents/<agent 名>/...` — 该 Agent 的实现代码
- `scripts/sync_agents.py` — 顶层 README Agent 列表自动同步脚本
- `.claude/settings.json` — hook 配置（PostToolUse + SessionStart 触发同步）
- `README.md` — 项目介绍，包含自动维护的 Agent 列表区块

## 核心约定（必须遵守）
1. **顶层 README.md 中的 Agent 列表必须始终与 `agents/` 目录的实际内容保持一致** —— 这是用户明确要求长期自动维护的事项。
2. 正常情况下 hook 会自动同步；若发现列表与实际不符，立即手动运行：`python3 scripts/sync_agents.py`
3. 不要手动编辑顶层 README.md 中 `AGENTS:START` / `AGENTS:END` 标记之间的内容（会被自动覆盖）。
4. 新增 Agent 时：在 `agents/` 下建一个文件夹，写好代码，并创建 `README.md` 介绍文件（见 README「如何添加新 Agent」）。

## 介绍信息提取规则（sync_agents.py 的逻辑）
- 名称优先级：frontmatter `name` > 正文第一个 `#` 大标题 > 文件夹名
- 描述优先级：frontmatter `description` > 正文第一段 > 「（暂无描述）」
- 缺少 README.md 的 Agent 会在列表中标注「（缺少 README.md 介绍文件）」

## 运行环境（本机）
- Python 3.9.6（系统自带），anthropic SDK 已装（0.120.2+）
- 本机 Claude Code 通过**阿里云 Anthropic 兼容网关**连接（`ANTHROPIC_BASE_URL` 指向 token-plan.cn-beijing.maas.aliyuncs.com），模型名 `qwen3.8-max`；`ANTHROPIC_AUTH_TOKEN` 只在 Claude Code 进程环境里有，用户终端需要自行 export
- 官方 Anthropic API key（ANTHROPIC_API_KEY）未在本机配置；agent 代码默认用 `claude-opus-5`（官方 API），走网关时用 `--model qwen3.8-max`
- agents 的模型调用代码要做**参数降级兼容**（第三方网关可能不支持 thinking/fallbacks 等新参数），参考 `agents/file-butler/agent.py` 的 `call_model()`

## 现有 Agent
- `agents/file-butler` — 本地文件管家（function-call 文件操作，沙盒安全，已通过端到端测试）

## 常见问题
- 删除 Agent 文件夹后列表未即时更新 → SessionStart hook 会在下次会话开始时同步，或手动运行同步脚本。
- hook 未生效（如刚克隆仓库、或配置是本会话中途创建的）→ 手动运行同步脚本，或重启 Claude Code / 打开一次 /hooks。

---
> Source: [Dy1ngn/agent-hub](https://github.com/Dy1ngn/agent-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
