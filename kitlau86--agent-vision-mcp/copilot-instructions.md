## agent-vision-mcp

> 为非视觉模型提供图片分析能力的 MCP 服务器。单一工具 `analyze_image`，调用用户配置的 OpenAI 兼容视觉 API。

# agent-vision-mcp
为非视觉模型提供图片分析能力的 MCP 服务器。单一工具 `analyze_image`，调用用户配置的 OpenAI 兼容视觉 API。

**技术栈：** TypeScript + tsx + tsc + npm | Node >= 18
**核心依赖：** `@modelcontextprotocol/sdk` v1.x + `openai` v4.x + `zod` v3.x

## 命令
- `npm run dev` — tsx 运行源码
- `npm run build` — tsc 编译到 dist/
- `npm start` — node dist/index.js

## 架构
`index.ts` → McpServer + StdioServerTransport 注册 `analyze_image` 工具 → handler 串联 `image-processor`（图片来源检测/base64）→ `cache`（SHA256 文件缓存，默认关闭）→ `vision-client`（openai SDK 调用 chat.completions.create）

## 约定
- 中文 `README-CN.md` 与英文 `README.md` 必须同步修改：改动其一时，务必同步更新另一个，保持内容一致
- 日志只打 `console.error`（stderr），stdout 是 MCP 通道
- 注释/文档用中文，标识符用英文，关键字英文
- Zod `.strict()`，TS 严格模式
- 工具层捕获所有异常返回 `{ isError: true }`，不向上抛
- 详见 `docs/`

## 参考文档（操作前先读对应文档）
- 修改数据流/模块职责/工具注册 → `docs/architecture.md`
- 改动环境变量/MCP 客户端配置/各供应商示例 → `docs/configuration.md`
- 构建/编译/依赖/开发命令相关 → `docs/development.md`
- 本地联调、发布前全链路测试、stdio 接入验证 → `docs/local-testing.md`

---
> Source: [kitlau86/agent-vision-mcp](https://github.com/kitlau86/agent-vision-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
