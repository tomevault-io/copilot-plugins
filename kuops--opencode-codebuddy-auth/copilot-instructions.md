## opencode-codebuddy-auth

> OpenCode 插件，为 CodeBuddy 提供 IOA OAuth 认证和请求拦截。单文件项目，入口 `src/index.ts`。

# AGENTS.md

## 项目概述

OpenCode 插件，为 CodeBuddy 提供 IOA OAuth 认证和请求拦截。单文件项目，入口 `src/index.ts`。

## 构建

```bash
npm install && npm run build   # tsc 编译到 dist/
```

无测试、无 lint、无 CI。只有 `npm run build`。

## 架构要点

- `src/index.ts` 是唯一源文件，导出 `CodeBuddyAuthPlugin`（Plugin 类型）和 default export
- 运行时作为 OpenCode 插件加载，通过自定义 `fetch` 拦截 `/chat/completions` 请求并注入 CodeBuddy 认证 headers
- `@opencode-ai/plugin` 是 peer dependency，仅开发时安装

### 核心 Hooks

1. **config** — 启动时从 `~/.local/share/opencode/auth.json` 读取已保存的 access token，调用 `GET /v3/config` 动态获取 craft agent 可用模型，注入到 `config.provider.codebuddy.models`；未登录或获取失败时 fallback 为 `auto` 默认模型；不覆盖用户手动声明的 models
2. **auth** — IOA OAuth 登录流程（浏览器 → 轮询 token），loader 返回自定义 fetch 拦截请求
3. **chat.params** — 设置 baseURL

### 用户配置

`codebuddy` 不在 models.dev 数据库中，插件通过 `config` hook 自动创建 `provider.codebuddy`（如未声明），并动态注入 models。支持三种配置方式：
1. 只加 `plugin`，不声明 provider（推荐，全自动）
2. 声明 provider 不声明 models（自动发现模型）
3. 手动声明 provider + models（完全手动控制）

## 环境

- 国内版 API：`copilot.tencent.com`，`X-Domain: www.codebuddy.cn`
- 国际版 API：`www.codebuddy.ai`，`X-Domain: www.codebuddy.ai`
- 切换环境需同时改 `CONFIG.serverUrl` 和 `CONFIG.domain`
- 模型列表通过 `GET /v3/config` 获取（需 access token），可能随时变化
- Token 存储路径：`~/.local/share/opencode/auth.json`，config hook 直接读取该文件获取 token

---
> Source: [kuops/opencode-codebuddy-auth](https://github.com/kuops/opencode-codebuddy-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
