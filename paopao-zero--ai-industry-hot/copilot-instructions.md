## ai-industry-hot

> Fetch and analyze AI industry news from AIHOT and other cited sources, then produce Chinese briefings with AI industry-chain impact mapping, bullish/bearish/neutral/watchlist judgments, push rationale, and risk notes. Use when the user asks about today's AI hotspots, recent AI industry news, OpenAI/Anthropic/Nvidia/Agent/GPU updates, AI产业链利好利空, or AI news suitable for finance/industry research briefings.


# AI Industry HOT

Use this skill to turn AI industry news into a concise Chinese industry-chain briefing. Focus on information screening, factual summarization, impact-path analysis, and risk disclosure. Do not provide investment advice, buy/sell recommendations, target prices, or deterministic price predictions.

## Workflow

1. Determine the user's time window and topic.
   - Default: latest selected AIHOT items from the last 24 hours.
   - If the user asks for "最近几天", set `since` to that window, with a maximum of 7 days for the items endpoint.
   - If the user asks for a company or theme, use `q`, such as `OpenAI`, `Anthropic`, `Nvidia`, `Agent`, `GPU`, `MCP`, or `Claude`.
2. Fetch current AI news before answering. AI news is time-sensitive; do not rely only on memory.
3. Keep only items with clear industry relevance. Prioritize model releases, product launches, infrastructure, compute, cloud, agent applications, financing, M&A, regulation, open source, and technical breakthroughs.
4. Do not invent facts, links, sources, dates, companies, or market reactions. If no relevant result is returned, say so directly and suggest a narrower query.
5. Analyze each selected item through event type, industry-chain mapping, impact direction, push rationale, and risk note.
6. Write in Chinese Markdown, suitable for Feishu/WeChat/classroom presentation.

## AIHOT Data Access

Use AIHOT public REST endpoints as the primary source.

- Selected latest items:
  `GET https://aihot.virxact.com/api/public/items?mode=selected&since=<ISO_TIME>&take=50`
- All latest items, only when the user explicitly asks for 全部/完整/所有/全量:
  `GET https://aihot.virxact.com/api/public/items?mode=all&since=<ISO_TIME>&take=50`
- Keyword search:
  `GET https://aihot.virxact.com/api/public/items?q=<KEYWORD>&take=50`
- Daily briefing:
  `GET https://aihot.virxact.com/api/public/daily`
  or `GET https://aihot.virxact.com/api/public/daily/<YYYY-MM-DD>` when the user asks for a specific daily report.

When calling `/api/public/*`, include a browser-like User-Agent header to avoid access failures:

```bash
curl -H "User-Agent: Mozilla/5.0" "https://aihot.virxact.com/api/public/items?mode=selected&take=50"
```

If AIHOT is unavailable, times out, cannot be accessed, returns too little information, or returns mostly duplicate/low-quality items, do not abandon this skill or downgrade to a free-form summary. Enter fallback mode: use other public sources only when they can be cited with links, such as OpenAI Blog, Anthropic News, Google Blog, NVIDIA Blog, Microsoft Blog, company newsrooms, arXiv, Hugging Face, GitHub, Reuters, CNBC, or official regulator/government pages. Cross-verify candidate items across available sources when possible, then score and select the highest-signal items using these dimensions: source authority (official/regulator/primary sources first), cross-source confirmation, recency relative to the user's requested window, industry-chain relevance, and investment-analysis value (whether the item supports a clear 偏利好/偏利空/中性/待观察 judgment). In fallback mode, keep the same `Output Format` and item template exactly; include source links, source/time, event type, one-sentence summary, impact path, industry-chain impact, push rationale, and risk note for each selected item.

## AIHOT Routing Rules

- Route broad questions like "今天 AI 圈", "过去 24 小时大新闻", and "最近 AI 圈有啥" to the rolling items endpoint with `mode=selected` and a semantic `since` window.
- Route to the daily endpoint only when the user explicitly asks for "日报".
- Use `mode=selected` by default. Use `mode=all` only when the user explicitly asks for all/full/complete items.
- For company or topic queries, use server-side `q=<keyword>` instead of fetching a batch and filtering locally.
- For "最近 N 天 X", always set `since` to N days ago. The items endpoint should not be used beyond the latest 7 days.
- If the user asks for a specific AIHOT category, use the category parameter when helpful:
  - `ai-models`: AI 模型发布/更新
  - `ai-products`: AI 产品发布/更新
  - `industry`: AI 行业动态
  - `paper`: AI 论文研究
  - `tip`: AI 技巧与观点
- Fetch serially and avoid high-frequency polling. Treat cursor-like pagination tokens as opaque if the response includes them.
- Convert `publishedAt` from UTC ISO time into Beijing time and a human-readable form such as `今天上午 09:48`, `2 小时前`, `昨天`, or `5/7 00:43`. Do not show raw ISO timestamps to the user.
- Prefer `title` for Chinese output. Use `title_en` only when the user asks for English or `title` is empty.

## Event Types

Classify each news item into one primary event type:

- 模型发布: foundation model, reasoning model, multimodal model, embedding/reranker, voice/video model.
- 产品发布: AI assistant, coding tool, search, office tool, enterprise SaaS feature, consumer AI app.
- 算力硬件: GPU, ASIC, HBM, server, data center, networking, optical module, power and cooling.
- 云服务: model API, cloud AI platform, inference service, model marketplace, enterprise deployment.
- Agent应用: coding agent, browser agent, workflow agent, enterprise automation, MCP/tool-use ecosystem.
- 投融资并购: funding, acquisition, IPO, strategic investment, valuation, partnership with commercial implications.
- 政策监管: AI safety rule, copyright, export control, data policy, antitrust, industry standards.
- 开源生态: open model, developer framework, benchmark, dataset, infrastructure library.
- 论文技术突破: research with likely product or infrastructure implications.

## Industry-Chain Mapping

Map the event to affected links in the AI industry chain.

- 上游: GPU, AI chips, HBM, servers, data centers, power, cooling, optical modules, networking.
- 中游: foundation models, cloud vendors, API platforms, MLOps, data services, developer tools, model deployment platforms.
- 下游: AI applications in office, education, finance, healthcare, content generation, enterprise services, coding, robotics, smart devices.

Use impact labels conservatively:

- 偏利好: credible demand, adoption, pricing power, ecosystem, or cost-efficiency benefit.
- 偏利空: credible pressure from regulation, competition, substitution, cost, supply constraint, or monetization uncertainty.
- 中性: mainly factual update with unclear business impact.
- 待观察: important event but insufficient evidence on adoption, revenue, cost, or regulation.

## Output Format

Start with a concise disclaimer:

> 以下为产业研究视角的信息整理，不构成投资建议。

Then produce two layers:

1. `## 今日最值得关注` or `## 本期最值得关注`
   - Select 3-5 highest-signal items.
2. Categorized sections when enough items exist:
   - `## 模型与基础设施`
   - `## 产品与应用`
   - `## 资本与公司`
   - `## 政策与风险`

Use this item template:

```markdown
### 1. <新闻标题>
- 来源与时间：<来源>，<时间>
- 原文链接：<URL>
- 事件类型：<模型发布/产品发布/算力硬件/...>
- 一句话摘要：<一句话事实摘要>
- 影响链条：<事件 -> 需求/成本/生态/监管变化 -> 产业链影响>
- 产业链影响：
  - 上游：<受影响环节>，<偏利好/偏利空/中性/待观察>，<简短原因>
  - 中游：<受影响环节>，<偏利好/偏利空/中性/待观察>，<简短原因>
  - 下游：<受影响环节>，<偏利好/偏利空/中性/待观察>，<简短原因>
- 推送理由：<为什么这条值得用户看>
- 风险提示：<不能直接外推的限制、不确定性或后续观察点>
```

Keep each item compact. Prefer 4-8 bullets per item. If an upstream/midstream/downstream link is not materially affected, write `暂无明确影响` instead of forcing a conclusion.

## Quality Rules

- Always include source links for cited news.
- Separate fact from inference. Use phrases like `可能影响`, `偏利好`, `待观察`, and `从产业链视角看`.
- Explain the impact path rather than only naming companies.
- Avoid broad market claims unless the source supports them.
- If the user asks for a specific company, analyze the company as part of the industry chain but do not recommend trading action.
- If the API returns duplicate or low-value items, merge or omit them.
- If the user requests a push-ready format, make the output shorter and keep the top 3 items only.
- Do not expose raw endpoint paths, query parameters, rate limits, cache headers, cursors, HTTP status codes, or backend mechanics in the user-facing briefing. Write human-readable metadata such as `最近 3 天命中 OpenAI 关键词的精选条目`, `按发布时间倒序`, or `共 12 条`.
- Do not treat AIHOT `summary` as a direct quotation. For direct claims or quotes, verify against the source URL.
- If the daily report is unavailable because it has not been generated yet, explain that the latest daily report may not be ready and use the latest rolling items or the previous daily report.
- If the API returns no relevant items, say no relevant result was found in the selected time window; do not fill the gap from memory.

---
> Source: [paopao-zero/ai-industry-hot](https://github.com/paopao-zero/ai-industry-hot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
