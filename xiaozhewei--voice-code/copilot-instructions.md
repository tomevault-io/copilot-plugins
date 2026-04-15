## voice-code

> 本项目默认启动为本机 HTTPS（自签名证书）。


# VoiceCode

## 运行

本项目默认启动为本机 HTTPS（自签名证书）。

```bash
npm start
```

访问：`https://localhost:3000`

## 内网穿透（ngrok）

启动时加 `--tunnel`（或 `--ngrok` / `-t`）会在启动后创建一个公网地址，并在控制台与二维码里优先展示该地址。

前置条件：设置 `NGROK_AUTHTOKEN`（从 ngrok 控制台获取）。

```bash
set NGROK_AUTHTOKEN=YOUR_TOKEN
npm run start:tunnel

# 或
node server.js --tunnel
```

说明：本地服务是自签名 HTTPS，上游 TLS 校验已在 ngrok 转发中关闭（`verify_upstream_tls: false`）。

## 自动启动 AI CLI

可以使用命令行参数在启动终端会话时自动运行特定的 AI 命令行工具。

```bash
# 启动并自动运行 gemini
node server.js --gemini
# 或使用简写
node server.js -g

# 启动并自动运行 claude
node server.js --claude
# 或使用简写
node server.js -c
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xiaozhewei)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/xiaozhewei)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
