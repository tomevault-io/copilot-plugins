## one

> 给自动化 agent(Claude Code / Cursor / Copilot 等)的项目说明 —— 帮用户**部署**和**改**这个项目时读它。人类请看 [README.md](README.md) 与 [CONTRIBUTING.md](CONTRIBUTING.md)。

# AGENTS.md

给自动化 agent(Claude Code / Cursor / Copilot 等)的项目说明 —— 帮用户**部署**和**改**这个项目时读它。人类请看 [README.md](README.md) 与 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 这是什么

**one** —— 云端常驻的个人 AI:大脑跑在 Cloudflare(Worker + D1 + Durable Objects),客户端(网页 / 桌面 / 安卓 / 浏览器扩展)是它连上来的「手」。用户对它说一句话,它在用户的设备上把活干完。

## 结构

```
worker/              云核(部署到 Cloudflare)
  server/            后端:AI 内核 + apps(云端数据应用)+ 鉴权 + D1
  ui/                前端(Vue 3),构建产物在 ui/dist
  schema.sql         数据库全量结构 —— 唯一真相,无迁移脚本
  seeds/             出厂小应用源码;seeds/build.mjs 生成 seeds/apps.sql
  wrangler.jsonc     部署配置(已 gitignore;从 wrangler.example.jsonc 复制)
clients/
  tauri/             桌面(Rust + WebView),含电脑控制「手」
  android/           安卓(Kotlin,WebView + 无障碍「手」)
  browser/           Chrome MV3 扩展(CDP「手」)
```

## 帮用户部署(最常见任务)

前提:一个 Cloudflare 账户、Node 18+。全程在 `worker/` 目录下:

```bash
npx wrangler login                                   # 未登录先登录

cd worker && npm install && npm --prefix ui install  # 顺带装前端依赖(部署会构建 ui/)
cp wrangler.example.jsonc wrangler.jsonc             # 待填 account_id / database_id

npx wrangler d1 create one                           # 把输出的 database_id 连同 account_id 填进 wrangler.jsonc
npx wrangler d1 execute one --remote --file=schema.sql
npx wrangler d1 execute one --remote --file=seeds/apps.sql   # 出厂小应用(可选)

npx wrangler secret put AUTH_SECRET                  # 必填:值用 `openssl rand -hex 32` 生成

npm run deploy
```

部署完:打开输出的地址 → 设一个访问密码 → 进「设置」填模型(见下)→ 想让 AI 动手就连一只「手」(`clients/` 下各端,填同一个地址 + 密码即可上线)。

- 不绑自定义域名:删掉 `wrangler.jsonc` 里的 `routes`,直接用 `*.workers.dev` 地址。
- `AUTH_SECRET` 是**必填**的:没配 worker 会拒签/拒验令牌(fail-closed),这是刻意的安全默认,不要为省事去掉。

## 模型配置(部署后在网页「设置」里填)

- **主模型**:API 地址 + API Key + 模型名 + 认证方式。`authMode` = `bearer`(OpenAI 系)或 `x-api-key`(Claude 系)。任何 OpenAI 兼容 / Claude 端点都能接 —— OpenAI、Claude、Gemini、DeepSeek、通义千问、Kimi、智谱 GLM 等,coding plan 端点也行。
- **视觉模型**:可单独配一个(看图、读屏用),也可以勾选让主模型兼任视觉。

## 改代码的硬约定(别踩)

- **机密绝不入库**:`wrangler.jsonc` / `.dev.vars` / `config.json` / 数据库备份 / 任何 key 都已 gitignore,别绕过。`AUTH_SECRET` 只走 `wrangler secret`。
- **数据库无迁移**:`worker/schema.sql` 是唯一真相,改表结构就直接改它,不写迁移脚本。升级前用 `npm run db:backup` 导出备份。
- **种子小应用是生成物**:改 `worker/seeds/<slug>/` 源码后,必须 `npm run seed:build` 重新生成 `seeds/apps.sql`,别手改 apps.sql。
- **小应用四文件规范**:每个应用 = `index.html` + `index.js` + `index.css` + `index.sql`(建表 DDL)。平台打开应用时**先跑 `index.sql`(幂等建表)再加载 `index.js`**;数据表用 `app_` 前缀;不要在 JS 里写建表代码,也不要自己在 html 里引 `index.js`(平台会注入)。
- **路由**:后端能力全在 `/api/*`;`wrangler.jsonc` 的 `run_worker_first` 就是 `["/api/*"]`,其余路径交给前端 SPA。
- **风格随上下文**:注释以简体中文为主,命名/习惯向周围代码看齐;一次改动小而聚焦。

## 客户端构建

- 桌面:`cd clients/tauri && npx tauri build`(需当前平台的构建工具链;`frontendDist` 是本地 `web/`,改完要重新打包才生效)。
- 浏览器扩展:`chrome://extensions` → 开发者模式 → 加载 `clients/browser/`。
- 安卓:见 [clients/android/README.md](clients/android/README.md)。

---
> Source: [yanglongyun/one](https://github.com/yanglongyun/one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
