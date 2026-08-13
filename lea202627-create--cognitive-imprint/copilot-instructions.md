## cognitive-imprint

> Cognitive Imprint — 抓取作者公开文章（RSS/blog，未来含 X 帖子），用 LLM 识别其认知结构与认知判断，并支持纵向跟踪其思考演变。**这是认知分析工具，不是内容摘要工具。**

# CLAUDE.md

Cognitive Imprint — 抓取作者公开文章（RSS/blog，未来含 X 帖子），用 LLM 识别其认知结构与认知判断，并支持纵向跟踪其思考演变。**这是认知分析工具，不是内容摘要工具。**

## 治理文件（改代码前先读）

- [docs/CONSTITUTION.md](docs/CONSTITUTION.md) — 项目宪法：设计理念、分析伦理红线、工程原则、演进路线。与其冲突的改动一律不做。
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — 当前架构、两段式分析管道、数据模型。
- [docs/ENGINEERING.md](docs/ENGINEERING.md) — 分支/提交规范、prompt 变更规范、合入前验证清单。

## 常用命令

```bash
npm run dev        # 开发服务器 (localhost:3000)
npm run build      # 类型检查 + 构建（合入前必须通过）
npm run db:push    # schema 变更推送到 Supabase
npm run db:studio  # Drizzle Studio (localhost:4983)
```

环境变量：`.env.local` 需要 `DATABASE_URL`（Supabase **transaction pooler** URI，端口 6543）和 `OPENROUTER_API_KEY`。可选：`OPENROUTER_MODEL`、`OPENROUTER_SITE_URL`、`CRON_SECRET`。

## 硬性规则（速查）

1. `lib/analyzer.ts` prompt 中的伦理红线（不奉承作者、不推断私人心理、结论必须带原文证据、语料不足必须降置信度）**只能加强，不能删除**。
2. `types/index.ts` 与 analyzer prompt 的 JSON schema 是同一契约——改一侧必须同步另一侧，并同步报告页与 `lib/export.ts`。
3. `articles.raw_text` 与 `extracted_features` 只增不删；`imprint_reports` 追加版本，不覆盖历史。
4. 业务逻辑在 `lib/`，API route 只做校验/转发；页面不直连数据库。
5. 两段式管道（逐篇提取 → 语料聚合）不合并：逐篇特征是可重跑聚合的稳定中间表示。
6. LLM 走 OpenRouter（`lib/analyzer.ts` 中的 `callModel`，纯 fetch，无 SDK 依赖）；模型 ID 只写在该文件一处，可用 `OPENROUTER_MODEL` 覆盖。
7. 数据库是 Supabase Postgres + postgres.js；serverless 下必须用 transaction pooler 且 `prepare: false`（已在 `lib/db.ts` 设置，勿改）。
8. 数据源只限作者公开发布的内容；不做风格模仿、不做私人心理画像（宪法明确排除项）。
9. **本项目是 Next.js 14**：动态路由的 `params` 是普通对象，直接 `params.id` 取值。不要用 Next 15 的 `params: Promise<...>` + `use(params)` 写法——那会在运行时抛 "An unsupported type was passed to use()"，且 `npm run build` 检查不出来（只有真正访问页面才暴露）。

## 访问控制

`middleware.ts` 对全站做 HTTP Basic Auth。`BASIC_AUTH_PASSWORD` 未设置时不启用（本地开发无感），生产环境**必须设置**——所有 API 都能花钱（ingest/analyze 调 LLM）或毁数据（`DELETE /api/authors`）。`GET /api/track` 带正确 `Bearer $CRON_SECRET` 时绕过 basic auth（Vercel Cron 发不了 basic 凭据），且该绕过只对这一个路径生效。新增可被外部调用的路由时不要绕开 middleware。

## Feed 解析

添加作者时只需填**网站地址**，`resolveFeedUrl` 会自动发现 feed：先试输入本身是不是 feed，再读页面 `<link rel="alternate">`，最后试 `/feed`、`/rss`、`/atom.xml` 等常见路径；全部失败会报出试过哪些地址。

`lib/fetcher.ts` 的 `parseFeed` 支持 RSS 2.0、RSS 1.0/RDF（`<item>` 带属性、`dc:date`）和 Atom（`<entry>`、href 属性取链接、优先 `rel="alternate"`、`published` 缺失时回落 `updated`）。喂进非 feed 的 URL 会**显式抛错**，不静默返回"没有新文章"。

## 分析输出语言

分析文字默认输出**简体中文**（`ANALYSIS_LANGUAGE` 可改，见 `lib/analyzer.ts` 的 `LANGUAGE_RULES`）。**原文证据摘录永远保持原语言不翻译**——翻译过的引文不再是证据，这条与伦理红线同级，不得放宽。`confidenceNote` 在 TS 代码里生成，改语言逻辑时要同步 `zh` 分支。

## 已知限制

- 文章发布时间依赖 feed 里的日期字段；全部缺失时报告的 date range 显示 unknown。
- 逐篇分析只取正文前 6000 字符。

---
> Source: [lea202627-create/Cognitive-imprint](https://github.com/lea202627-create/Cognitive-imprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
