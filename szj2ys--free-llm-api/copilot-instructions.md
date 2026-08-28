## free-llm-api

> > 多平台网页版 LLM 统一代理（Kimi / 千问 / 智谱 GLM / DeepSeek）。OpenAI 兼容入口 `/v1/chat/completions`，按 `model` 名前缀路由，客户端只配一个供应商。

# AGENTS.md — free-llm-api

> 多平台网页版 LLM 统一代理（Kimi / 千问 / 智谱 GLM / DeepSeek）。OpenAI 兼容入口 `/v1/chat/completions`，按 `model` 名前缀路由，客户端只配一个供应商。

## 怎么跑
- 本机：`npm install && npm run build && node dist/index.js`（端口读 `configs/dev/service.yml`，当前 8787，host 0.0.0.0）
- Docker：`docker compose up -d --build`，验证 `curl http://127.0.0.1:8787/ping` → `pong`
- 开发热重载：`npm run dev`（tsup watch + 自动重启子进程）

## 技术栈
- TypeScript + Koa2 + tsup（打包 cjs/esm + dts），无数据库
- 凭据在 `configs/dev/credentials.yml` 服务端托管；多账号用列表 → round-robin 轮换；可选 `master_key` 统一鉴权；优先级：请求头 Bearer > 环境变量 > credentials.yml
- 路由 `src/api/routes/*`，适配器 `src/api/controllers/*`（每平台一个）
- 入口是 `src/index.ts`（由 tsup 打包为 `dist/index.js`），无独立守护进程脚本

## 目录与约定
- 新增平台：写 `controllers/<platform>.ts` → 在 `routes/chat.ts` 加模型前缀分支 → `routes/models.ts` 加静态条目
- 模型列表三层合并：静态基线(18) + 智谱动态拉取(`paas/v4/models`，缓存 10min) + 实测成功自动注册(内存态，重启清空)
- 别名 `@/*` → `src/*`（仅 tsup 友好，运行时勿用）；DeepSeek PoW 依赖根目录 `sha3_wasm_bg.*.wasm`（勿删）
- 凭据字段名 = 浏览器里实际 token/Cookie 名（F12 叫什么这里就叫什么）

## 当前状态与下一步
- 已实测：Kimi / 千问 / DeepSeek 全链路；GLM 卡在账号试用额度（429 / code 1113，需先领试用）
- 遗留待修：① gemini/claude 适配器沿用原项目实现、未针对各平台单独验证（已知限制，README 已注明）
- 红线：`configs/dev/credentials.yml` 含真实平台 token，**切勿提交到 git 或打进 Docker 镜像**；改凭据优先用环境变量覆盖（compose 已支持）

---
> Source: [szj2ys/free-llm-api](https://github.com/szj2ys/free-llm-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
