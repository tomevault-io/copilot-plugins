## news-digest-skill-cn

> AI/科技新闻深度解读工作流。获取 HN + 全网新闻，7 维度分析框架，支持单篇深度洞察。所有依赖已内置，无需额外安装。


# News Digest Skill

AI/科技新闻深度解读工作流，包含三个子命令。所有依赖工具已内置在 `deps/` 目录中。

## 内置工具

以下工具包含在 `{baseDir}/deps/` 中，无需单独安装：

- **hn** — Hacker News CLI。用法：`uv run {baseDir}/deps/hn/scripts/hn.py top -n 100`
- **news-aggregator-skill** — 全网新闻聚合（GitHub Trending、Product Hunt、36Kr、腾讯新闻、华尔街见闻、V2EX、微博）。用法：`python3 {baseDir}/deps/news-aggregator-skill/scripts/fetch_news.py --source all --limit 15 --deep`
- **last30days** — Reddit + X + Web 研究工具。用法：`python3 {baseDir}/deps/last30days/scripts/last30days.py "TOPIC" --emit=compact`

## 子命令

### /myhn
获取 Hacker News 前 100 条热门新闻，筛选 20 条进行 7 维度深度解读。

读取 `{baseDir}/prompts/hn-digest.md` 执行。

### /news
从 HN 以外的全网信息源获取新闻，精选 30 条进行硬核深度拆解。使用内置的 news-aggregator-skill 和 last30days 覆盖 GitHub Trending、Product Hunt、36Kr、腾讯新闻、华尔街见闻、V2EX、微博等。

读取 `{baseDir}/prompts/global-news-digest.md` 执行。

### /洞察 <路径或链接>
对单篇文档或链接进行深度分析，最大化认知增量。包含核心洞察、定位对比、直觉建立、非显而易见的东西等维度。

读取 `{baseDir}/prompts/deep-dive.md` 执行。

## 首次使用

安装后，请编辑 `prompts/` 目录下三个文件中的「我的背景」部分，替换为你自己的信息。AI 会根据这些生成与你个人相关的解读。

同时自定义 `prompts/global-news-digest.md` 中的「对我的具体价值 (Actionable Value)」维度，定义什么样的洞察对你最有价值。

## 深度跟进

看完 digest 后，如果对某条新闻特别感兴趣，可以让 AI 打开原文链接继续深入分析，或使用 `/洞察` 命令。

---
> Source: [RocStone/news-digest-skill-cn](https://github.com/RocStone/news-digest-skill-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
