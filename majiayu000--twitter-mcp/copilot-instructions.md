## twitter-mcp

> TypeScript + Playwright 浏览器自动化，通过 MCP 协议操作 Twitter/X。

# Twitter MCP 服务

TypeScript + Playwright 浏览器自动化，通过 MCP 协议操作 Twitter/X。

## 运行

```bash
npm install && npx playwright install chromium
npm run mcp
```

## 端口

**18071**（不可更改，已在全局端口分配表中注册）

## MCP 注册

```bash
claude mcp add --transport http twitter-mcp http://localhost:18071/mcp
```

## 环境变量

复制 `.env.example` 到 `.env` 并填写 `TWITTER_USERNAME` 和 `TWITTER_PASSWORD`。

## 传输模式

- `MCP_TRANSPORT=http`（默认）— StreamableHTTP，端点 `/mcp`
- `MCP_TRANSPORT=sse` — SSE 模式
- `MCP_TRANSPORT=stdio` — 标准输入输出

## VibeGuard 豁免

U-16 exempt: `src/mcp.ts` → 1500 — MCP server 主文件聚合 21 个 tool 定义与 handler，正在分步拆分中

---
> Source: [majiayu000/twitter-mcp](https://github.com/majiayu000/twitter-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
