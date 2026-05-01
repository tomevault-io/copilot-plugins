## telegram-relay-for-agent

> 本项目的 Bot 会在 Cloud Agent 启动时自动运行，接收 Telegram 上传的文件并保存到 `./uploads/`。与 Agent 对话时可直接引用该目录下的文件。

# Telegram Relay for Cursor Cloud Agent

本项目的 Bot 会在 Cloud Agent 启动时自动运行，接收 Telegram 上传的文件并保存到 `./uploads/`。与 Agent 对话时可直接引用该目录下的文件。

## 配置步骤

### 1. 创建 Telegram Bot

1. 在 Telegram 中搜索 `@BotFather`
2. 发送 `/newbot`，按提示创建 Bot，获得 **Bot Token**（形如 `123456:ABC-DEF...`）
3. （可选）私聊你的 Bot 发一条消息，用 `getUpdates` 获取你的 **Chat ID**：
   ```bash
   curl "https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates"
   ```
   返回的 `message.chat.id` 即为 Chat ID

### 2. 配置 Cursor Cloud Agent Secrets

1. 打开 [cursor.com/dashboard?tab=cloud-agents](https://cursor.com/dashboard?tab=cloud-agents)
2. 在 **Secrets** 中添加：
   - `TELEGRAM_BOT_TOKEN`：Bot Token
   - `TELEGRAM_CHAT_ID`：你的 Chat ID（可选，不填则接受所有聊天；建议填写以限制来源）

### 3. 将本项目加入你的 Agent 环境

在你的项目根目录创建或编辑 `.cursor/environment.json`，在 `terminals` 中加入 Telegram Relay：

```json
{
  "install": "npm install",
  "terminals": [
    {
      "name": "Telegram Relay Bot",
      "command": "cd /path/to/telegram-relay-for-agent && node index.mjs"
    }
  ]
}
```

若本项目与你的代码在同一仓库的子目录，可写为：

```json
{
  "terminals": [
    {
      "name": "Telegram Relay Bot",
      "command": "cd telegram-relay-for-agent && npm install && node index.mjs"
    }
  ]
}
```

### 4. 使用流程

1. 启动 Cursor Cloud Agent
2. 在 Telegram 中向你的 Bot 发送文件/视频/图片
3. 文件会保存到 `./uploads/`（或项目内的 `telegram-relay-for-agent/uploads/`）
4. 在 Cursor 中对 Agent 说：「处理一下 uploads 里刚上传的 xxx 文件」

## 文件保存位置

- 默认：`./uploads/`（相对于 Bot 进程工作目录）
- 文件名格式：`原名_时间戳.扩展名`

## 注意事项

- Cloud Agent 每次任务会启动新 VM，任务结束后 VM 销毁，`uploads/` 中的文件也会消失
- 建议配置 `TELEGRAM_CHAT_ID`，避免他人滥用你的 Bot
- Telegram Bot API 单文件最大约 20MB

---
> Source: [spudhsu-lemon/telegram-relay-for-agent](https://github.com/spudhsu-lemon/telegram-relay-for-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
