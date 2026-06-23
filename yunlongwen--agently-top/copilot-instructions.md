## agently-top

> 本文件为本仓库的 AI 协作约定。所有自动化助手在读取、修改或评审本项目时必须遵守。

# AGENTS.md

本文件为本仓库的 AI 协作约定。所有自动化助手在读取、修改或评审本项目时必须遵守。

## 项目概览

这是一个 Python + Vue 全栈 AI 信息源聚合项目。主流程每天抓取 GitHub Trending、Hacker News、少数派、钛媒体、OpenAI、Anthropic、InfoQ AI Development 等信息源，通过 GitHub Models API 生成中文摘要，按来源永久归档到磁盘，并写入 Redis 作为 3 天热数据缓存。

项目同时提供 FastAPI 只读接口和 Vue 3 前端资讯流页面（页面标题"Agently.top"），由 Nginx 静态托管前端、反代 `/api/` 到 FastAPI。`output/latest.json` 作为统一 JSON 兼容旧版接入点继续保留。

## 主要入口与模块

- `main.py`: 主入口，协调所有信息源抓取、AI 摘要、JSON 写出和邮件发送。
- `config.py`: 环境变量配置中心，所有可调参数都应从这里读取。
- `github_trending.py`: GitHub Trending daily / weekly 抓取和项目摘要。
- `hacker_news.py`: Hacker News Top Stories、评论抓取和社区讨论摘要。
- `sspai.py`: 少数派 (sspai.com) RSS 抓取和中文总结。
- `tmtpost.py`: 钛媒体 (tmtpost.com) RSS 抓取和中文总结。
- `official_ai_sources.py`: OpenAI、Anthropic、InfoQ AI Development 信息源抓取。
- `content_items.py`: 统一信息项模型、跨来源 JSON 适配、统一 AI 摘要、JSON 输出。
- `content_store.py`: 按来源归档写磁盘、Redis 最新快照读写、Redis 不可用时降级读磁盘。
- `redis_client.py`: Redis 进程级连接池，连接失败时返回 None 供调用方降级。
- `source_registry.py`: 来源 ID、label、category 注册表，前端/API/Redis key/磁盘路径共用。
- `api.py`: FastAPI 公开只读接口，提供 `/api/sources` 和 `/api/sources/{id}/latest`。
- `access_log.py`: API 访问日志中间件，记录每次请求 IP/路径/耗时/状态码，每小时输出统计汇总。
- `scheduler.py`: FastAPI 进程内采集调度器，按配置时间定时触发 `main.py` 主流程。
- `archive_sync.py`: 采集后把 `output/` 镜像推送到 `archive` 分支（git worktree，主工作区零干扰；默认关闭，由 `ARCHIVE_GIT_ENABLED` 控制）。
- `email_builder.py`: HTML 邮件内容生成。
- `email_sender.py`: SMTP 邮件发送。
- `test_email.py`: SMTP 发送测试脚本。
- `frontend/`: Vue 3 + Vue CLI 前端资讯流，页面标题"Agently.top"，侧边栏来源标签由前端映射覆盖显示。

## 运行方式

安装依赖：

```bash
pip3 install -r requirements.txt
```

本地运行时先确保环境变量已生效。常见方式：

```bash
source ~/.bash_profile
LOG_FILE=/private/tmp/github-trending-spider-run.log python3 main.py
```

服务器定时任务参考：

```bash
source ~/.bash_profile && cd /root/work/workspace/gitee/github-trending-spider && /usr/bin/python3 main.py
```

默认日志路径是 `/root/logs/github-python/trending.log`。本地没有该目录时，应通过 `LOG_FILE` 指向可写路径。

## 环境变量约定

敏感信息只能通过环境变量配置，不要写入代码、README 示例真实值或提交记录。

核心变量：

- `GITHUB_TOKEN`: GitHub Models API token，需要 `models:read` 权限。
- `SMTP_USER`, `SMTP_PASSWORD`, `MAIL_TO`: SMTP 邮件发送配置。
- `LOG_FILE`: 日志路径。
- `OUTPUT_JSON_PATH`: 统一 JSON 输出路径，默认 `output/latest.json`。

数量变量：

- `GITHUB_TRENDING_TOP_COUNT`: GitHub daily / weekly 各获取前 N 个仓库，默认 10。
- `HN_TOP_COUNT`: Hacker News 获取前 N 个帖子，默认 10。
- `HN_COMMENTS_PER_STORY`: 每帖获取前 N 条顶级评论，默认 10。
- `SSPAI_TOP_COUNT`: 少数派 获取前 N 条内容，默认 10。
- `TMTPOST_TOP_COUNT`: 钛媒体 获取前 N 条内容，默认 10。
- `OPENAI_NEWS_COUNT`: OpenAI 获取前 N 条内容，默认 10。
- `ANTHROPIC_NEWS_COUNT`: Anthropic 获取前 N 条内容，默认 10。
- `INFOQ_AI_NEWS_COUNT`: InfoQ 获取前 N 条内容，默认 10。

归档推送变量：

- `ARCHIVE_GIT_ENABLED`: 是否在采集后把 `output/` 镜像推送到 `archive` 分支，默认 false。
- `ARCHIVE_GIT_BRANCH`: 归档分支名，默认 archive。
- `ARCHIVE_GIT_WORKTREE`: worktree 检出目录，默认 .archive-worktree。
- `ARCHIVE_GIT_DIR`: worktree 内归档子目录，默认 archive。
- `ARCHIVE_GIT_REMOTE`: 推送 remote，默认 origin。

数量配置遵循“最多取 N 条”：如果配置为 100，但源头实际只解析到 14 条，则只展示 14 条。

## 输出与忽略文件

- `output/latest.json`: 运行生成的统一信息项 JSON，供后续后端读取。
- `output/email_preview.html`: 可用于本地预览邮件效果。
- `output/` 已在 `.gitignore` 中忽略，运行产物不要提交。
- `.env` 已忽略，真实密钥不要提交。
- `.task/` 已忽略，用于存放 AI 任务计划和任务记录。

## 任务文件规则

以后所有新任务计划、任务拆解、执行清单、阶段性记录，都必须写入 `.task/` 目录，不要再新增或扩写根目录任务文件。

生成新任务文件之前，必须先参考已有任务记录，按以下顺序读取：

1. 根目录 `tasks.md`: 当前 AI 后端专项信息源 v1 的完整实施历史。
2. `.task/tasks.md`: 更早的 HN 与 TLDR AI 接入历史。
3. `.task/` 下其他同类任务文件: 如果存在，优先参考同主题最近任务。

参考时重点看：任务标题粒度、涉及文件、勾选项写法、验证记录、是否有历史遗留问题。不要重复规划已经完成的内容。

命名格式：

```text
.task/YYYY-MM-DD_N-short-slug.md
```

示例：

```text
.task/2026-05-29_1-add-source-count-config.md
.task/2026-05-29_2-fix-infoq-feed.md
```

要求：

- 每个独立需求使用单独任务文件。
- 同一天多个任务用 `_1`, `_2`, `_3` 区分。
- 文件名 slug 使用英文小写和连字符，表达任务主题。
- 任务文件内至少记录：目标、涉及文件、执行步骤、验证结果、遗留问题。
- 根目录 `tasks.md` 只视为历史记录；新任务不要继续写入该文件。

## 当前任务历史摘要

当前仓库已有两批主要任务记录，可作为后续任务拆解模板。

### `.task/tasks.md` 历史任务

- 新增 Hacker News Top 10 + 评论总结功能：
  - 新增 `hacker_news.py`，通过 HN Firebase API 获取 Top Stories 和评论。
  - 抽出 `email_sender.py`、`email_builder.py`，让邮件发送和 HTML 生成从 GitHub 爬虫中解耦。
  - 新建 `main.py` 作为统一入口，协调 GitHub + HN + AI + 邮件流程。
  - 引入独立容错：GitHub 和 HN 任一成功即可发送邮件。
- 新增 TLDR AI 中文内容接入：
  - 新增 `tldr_ai.py`，从 TLDR AI 官方归档页解析最新 issue。
  - 接入 TLDR AI 阶段，支持未配置 `GITHUB_TOKEN` 时降级展示英文摘要。
  - 更新邮件模板和 README，形成 GitHub + HN + TLDR AI 三源日报。

### 根目录 `tasks.md` 当前任务

- AI 后端专项信息源 v1：
  - 新增 `content_items.py`，统一字段为 `source/category/title/url/published_at/original_summary/chinese_summary/backend_focus`。
  - 新增 `official_ai_sources.py`，接入 OpenAI、Anthropic、InfoQ AI Development。
  - `main.py` 调整为 6 个源独立抓取、独立容错，同时生成邮件和 `output/latest.json`。
  - `email_builder.py` 保留 GitHub/HN/TLDR AI 旧展示，并新增 OpenAI、Anthropic、InfoQ 分板块展示。
  - InfoQ 单个 `ai-development/news` feed 当前条目少，已改为聚合 AI Development、Artificial Intelligence、Generative AI 多个官方 RSS。
  - 数量配置环境变量化：`GITHUB_TRENDING_TOP_COUNT`、`HN_TOP_COUNT`、`TLDR_AI_TOP_COUNT`、`OPENAI_NEWS_COUNT`、`ANTHROPIC_NEWS_COUNT`、`INFOQ_AI_NEWS_COUNT` 等。

后续任务若涉及新增源、调整邮件、调整 JSON 结构或改运行配置，必须先对照以上历史，复用已有模块边界，不要重新设计主流程。

### `.task/2026-05-29_1-frontend-redesign.md` 前端重设计

- 全量重写 `frontend/src/App.vue` 视觉层，风格定位「科技资讯媒体 · Editorial」。
- 引入 Google Fonts：DM Sans + Noto Sans SC，替换系统默认字体。
- 品牌区：渐变图标（蓝→紫）+ 标题"Agently.top" + 副标题 + "⏱ 每 8 小时更新" chip。
- 删除搜索框及相关 `keyword` / `filteredItems` / `servedFromText` 逻辑。
- 删除 feed toolbar 中的更新时间、数据来源、条数计数展示。
- 内容卡片精简为：标题 + 中文摘要 + "阅读原文 →"，去掉 `backend_focus`、meta tags、`published_at`。
- 新增 `SOURCE_DISPLAY_MAP` 前端覆盖映射（不改后端 `source_registry.py`）：
  - github-daily → 今日开源热榜 / GitHub · 日榜
  - github-weekly → 本周开源精选 / GitHub · 周榜
  - hacker-news → 硅谷社区热议 / Hacker News
  - tldr-ai → AI 速报精选 / TLDR AI
  - openai → OpenAI 最新动态 / 官方更新
  - anthropic → Anthropic 最新动态 / 官方更新
  - infoq → AI 工程实践 / InfoQ AI
- 2026-06-17：因 V2EX API 在当前网络环境不可达、TLDR AI 被 Cloudflare 拦截，
  把这两个源替换为 少数派 (sspai.com/feed) + 钛媒体 (tmtpost.com/rss)，
  新增 `sspai.py` / `tmtpost.py` 两个 RSS 模块，沿用 `official_ai_sources.py` 的
  RSS 抓取 + AI 总结模式；`source_registry`、`main.py` 流程、`App.vue` 侧边栏
  映射、邮件板块同步更新；新增 `SSPAI_FEED_URL / SSPAI_TOP_COUNT / SSPAI_MAX_RETRIES`
  与 `TMTPOST_FEED_URL / TMTPOST_TOP_COUNT / TMTPOST_MAX_RETRIES` 六个环境变量。
- loading 状态改为 3 个 shimmer 骨架卡片动画，替代纯文字"正在加载数据"。
- 侧边栏 active 状态增加左竖线 indicator；卡片 hover 增加左竖线 + 背景过渡。
- `public/index.html` title 同步改为"Agently.top"。

## 开发约定

- 项目已引入 FastAPI + Vue 前端作为标准 Web 服务层，不再需要从零引入；新功能直接在现有 `api.py` / `frontend/` 上扩展。
- 新增信息源时，应优先选择官方 RSS/API；HTML 页面解析只能作为兜底。
- 每个信息源必须独立容错，不能因单个源失败导致其他源无法输出。
- 新增来源必须适配到 `content_items.py` 的统一字段：
  - `source`
  - `category`
  - `title`
  - `url`
  - `published_at`
  - `original_summary`
  - `chinese_summary`
  - `backend_focus`
- AI 摘要失败或缺少 `GITHUB_TOKEN` 时，必须保留原始标题、链接和原文摘要，并给出明确降级文案。
- 不要把运行生成的 `output/`、日志、`.env`、缓存文件加入版本控制。

## 验证建议

基础检查：

```bash
python3 -m py_compile main.py config.py github_trending.py hacker_news.py sspai.py tmtpost.py official_ai_sources.py content_items.py content_store.py redis_client.py scheduler.py source_registry.py api.py access_log.py email_builder.py email_sender.py
```

本地完整运行：

```bash
source ~/.bash_profile && LOG_FILE=/private/tmp/github-trending-spider-run.log python3 main.py
```

如果只想验证邮件模板或 JSON 写出，优先使用小样本构造测试，避免频繁发送真实邮件。

日志排查命令（部署后可用）：

```bash
grep "\[访问\]" /root/logs/github-python/trending.log   # 每次请求记录
grep "\[数据\]" /root/logs/github-python/trending.log   # 数据来源追踪（Redis/磁盘）
grep "\[统计\]" /root/logs/github-python/trending.log   # 每小时访问汇总
```

## 沟通与安全

- 回答用户时默认使用中文。
- 对不确定点先询问，不要猜测。
- 修改前先查代码事实，避免凭 README 或记忆判断。
- 任何真实 token、邮箱授权码、密码只允许报告“已设置/未设置/长度”，不要明文输出。

---
> Source: [yunlongwen/agently.top](https://github.com/yunlongwen/agently.top) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
