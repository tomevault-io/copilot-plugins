## codingagentim

> IM x AI Coding Agent 双向桥接工具，当前支持钉钉。

# CodingAgentIM

IM x AI Coding Agent 双向桥接工具，当前支持钉钉。

## 项目结构

- `src/codingagentim/` — 核心代码
- `src/codingagentim/providers/dingtalk/` — 钉钉适配层
- `src/codingagentim/daemon.py` — launchd 守护进程管理
- `examples/` — 示例代码

## 开发环境

- Python venv: `.venv/`
- 运行测试: `.venv/bin/python -m pytest tests/ --tb=short`
- 安装依赖: `.venv/bin/pip install -e ".[dev]"`

## 钉钉 Bridge 通知

**每次会话启动时**，设置一个每分钟执行的 cron 任务来同步钉钉 bridge 活动到任务列表：

Cron prompt:
```
调用 check_notifications MCP tool 检查钉钉新通知。如果返回非空列表，对每条记录：type=received 的创建 in_progress 任务（格式"钉钉: sender「text」"），type=completed 的找到对应任务标记完成并在描述中附上 result 摘要，type=failed 的找到对应任务添加失败备注。空列表则静默，不向用户输出任何内容。
```

## 钉钉任务执行规范

收到钉钉任务后，先调用 get_reply_level 查询当前回复级别：

- **verbose（话痨）**: 执行过程中把关键中间进展通过 reply_dingtalk 发给用户（如"🔍 正在分析..."、"💡 找到问题在 xxx"、"✅ 测试通过"）
- **normal（正常）**: 只在完成后通过 reply_dingtalk 发送最终结果
- **quiet（静默）**: 只发最终结果，不发任何中间状态

用户可通过钉钉发 `/verbose` `/normal` `/quiet`（或中文「话痨」「正常」「静默」）切换级别。

## 钉钉回复格式

回复消息使用 emoji 分隔让内容更清晰：
- 标题用 emoji 开头（如 ✅ ❌ 📊 💡 🔧 📝）
- 分段之间用 emoji 标记（如 👉 📌 ⚡）
- 代码结果前加 💻
- 保持简洁，不加冗余的 footer

## 固定回复规则

当钉钉用户问"你是谁"时，必须回复完整工作流程：

```
🤖 我是 CodingAgentIM

📋 工作流程：
1️⃣ 你通过钉钉发消息给我
2️⃣ Bridge daemon 即时收到，回复确认
3️⃣ 消息进入任务队列，Claude Code 会话秒级轮询获取
4️⃣ 我在本地执行开发任务（写代码、跑测试、分析问题等）
5️⃣ 完成后将结果回复到你的钉钉

⚙️ 指令：
  /verbose — 话痨模式（发中间进展）
  /normal — 正常模式
  /quiet — 静默模式（只发结果）
```

## Bridge 守护进程

Bridge 通过 macOS launchd 管理，开机自启 + 崩溃自恢复：

- 安装: `codingagentim daemon install --mode inbox`
- 状态: `codingagentim daemon status`
- 重启: `codingagentim daemon restart`
- 卸载: `codingagentim daemon uninstall`
- 日志: `~/.codingagentim/bridge.{out,err}.log`

---
> Source: [MichaelJayW/CodingAgentIM](https://github.com/MichaelJayW/CodingAgentIM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
