## contractor-go-v2

> 代码变更后第一件事——同步/部署到服务器


# 变更后优先同步服务器

完成与本仓库相关的**任何**功能或修复改动后，**第一件事**是按计划把变更同步到线上（或用户指定的环境），避免本地与生产长期脱节。

## 默认顺序（按实际改动选择执行）

1. **Git**：`git add` → `git commit` → `git push`（推到团队约定分支，触发 CI 若已配置）。
2. **Web（Expo / contractorgo.io）**：在 `mobile` 目录执行 `npm run build:web`，将产物部署到 Vercel（或项目文档中的托管方式）。参考 `mobile/docs/WEB_DEPLOY_CONTRACTORGO_IO.md`。
3. **Supabase Edge Functions**：凡改动 `supabase/functions/**`，对受影响函数执行  
   `supabase functions deploy <函数名>`（可多函数）。参考 `docs/API_ENV_CHECKLIST.md`。
4. **仅改文档/配置**：若用户仍要求「同步」，至少 **push 代码**；Secrets 在 Dashboard 手动核对。

助手在交付代码修改时，应在回复中**明确写出**需要用户或 CI 执行的部署命令；若用户说「帮我部署」且环境允许，可主动执行 `supabase functions deploy` / `npm run build:web` 等（需网络权限）。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/415geek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
