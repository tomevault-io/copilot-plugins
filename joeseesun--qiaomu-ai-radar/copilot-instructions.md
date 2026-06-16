## qiaomu-ai-radar

> **问题**：v2.14 后台 agent 仍用逐条 Edit（50+ 次 API 调用，~5 分钟）

# Daily Topic Selector - 架构文档

> 每日选题助手的系统架构与设计哲学

## 🔖 版本历史

### v2.15 (2026-03-04) - Read→Write 极速过滤翻译 ⚡

**问题**：v2.14 后台 agent 仍用逐条 Edit（50+ 次 API 调用，~5 分钟）

**方案**：Read 一次 → 内存中过滤+翻译 → Write 一次（2 次调用，~10 秒）

**设计哲学**：批量 > 逐条，内存处理最快，消除工具调用开销

### v2.10 (2026-01-10) - AI相关性智能过滤 🧹

**问题**：HN等综合平台全吞模式，铁路/游戏/政治等无关内容进入周刊

**方案**：延续v2.7哲学——脚本采集，Claude过滤

**工作流**：`采集 → 智能过滤（新增）→ 翻译 → 阅读`

**过滤标准**：
- 保留：AI/ML、编程、创业、认知科学、设计、科技动态
- 删除：政治、体育、纯硬件、交通基建、娱乐八卦

**零代码改动**：过滤逻辑在SKILL.md工作流中定义，由Claude执行

### v2.8 (2026-01-10) - 新增 8 个 AI/商业信息源 📰

**新增信息源**（每源限制3篇最新文章）：

**Substack 类**（使用通用 `substack_generic.py` 解析器）：
1. **DC The Median** - 数据科学与AI应用洞察
2. **Mark McNeilly** - AI新闻周报 "The New News in AI"
3. **Business Analytics** - 商业分析与数据驱动决策
4. **AI Leadership Edge** - AI领导力与企业战略
5. **ChinAI Newsletter** - 中国AI发展追踪（Jeff Ding）
6. **Why Try AI** - AI工具测评与实操指南

**Beehiiv 类**（使用通用 `beehiiv_generic.py` 解析器）：
7. **The Rundown AI** - 每日AI新闻速递（百万订阅）
8. **The Neuron Daily** - AI新闻与行业洞察

**架构变更**：
```
新增：scripts/parsers/substack_generic.py (80行)
  - 通用 Substack RSS 解析器
  - 自动将基础URL转换为/feed端点
  - 工厂函数：parse_dcthemedian, parse_markmcneilly 等

新增：scripts/parsers/beehiiv_generic.py (70行)
  - 通用 Beehiiv RSS 解析器
  - 处理 rss.beehiiv.com/feeds/xxx.xml 格式
  - 工厂函数：parse_therundown, parse_theneuron

修改：config/sources.json
  - 新增8个信息源配置
  - 每源 limit: 3（只抓最新3篇）
  - check_interval_hours: 24（每日检查）

修改：scripts/fetch_updates.py
  - 导入新解析器模块
  - PARSERS字典注册8个新解析器
```

**设计哲学**：
- **消除重复**：Substack/Beehiiv 各用一个通用解析器，新源只需配置
- **工厂模式**：`parse_xxx()` 函数只是调用 `parse_substack_rss(source)`
- **单一真相源**：URL和参数在 `sources.json`，代码只负责解析

### v2.7 (2026-01-04) - 周刊工作流 + 零API翻译 📅

**核心变更**：从日志模式切换到周刊模式，翻译由Claude直接完成

**周刊工作流**：
1. **采集模式改变**：
   - 从 `content_log.md`（日志追加）→ `{year}-第{week}周.md`（按周组织）
   - 采集时自动写入本周文件，按ISO周数组织
   - 新文章按日期分组（`## 2026-01-04`）

2. **翻译工作流重构**：
   - **删除**：`llm_utils.py`（114行API调用代码）
   - **原理**：能用LLM直接解决的事，不要写脚本
   - **新流程**：
     1. 采集脚本只记录原始数据（英文标题、中文标题都保留）
     2. Claude读取文件后用Edit工具批量添加翻译
     3. 格式：`- **标题**: English Title / 中文翻译`
   - **收益**：零API调用、零依赖、零token成本

3. **"值得写"索引**：
   - 新增 `generate_worth_writing.py`
   - 从 `state.json` 提取标记为 `📒 to_write` 的文章
   - 生成 `值得写.md` 供Claude分析和推荐
   - 自动在采集流程后更新

**文件变化**：
```
新增：scripts/generate_worth_writing.py (86行) - 生成"值得写"索引
新增：scripts/update_indexes.py (67行) - 一键更新所有索引
修改：scripts/fetch_updates.py
  - append_to_log() → append_to_weekly_log()
  - 简化翻译逻辑（114行API代码→0行）
  - 移除自动索引生成（职责分离）
删除：scripts/shared/llm_utils.py (不再需要)
```

**设计哲学的胜利**：
- **简单消灭复杂**：114行API代码 → 0行，翻译质量更好
- **单一职责**：脚本负责数据，Claude负责智能
- **消除边界情况**：
  - 无需处理API超时、重试逻辑
  - 无需处理中英文混合的edge case
  - 无需维护翻译prompt模板
- **好品味的代码**：让人说"操，原来可以这么简单"

**工作流示例**：
```bash
# === 自动部分（采集） ===
python3 fetch_updates.py
# 1. 同步上次的emoji标记 → state.json
# 2. 采集新文章 → 2026-第1周.md
# 3. 提示下一步操作

# === 手动部分（处理） ===
# 1. Claude批量翻译周刊标题
#    Read → Edit → "English Title / 中文翻译"

# 2. 用户阅读周刊，打emoji标记
#    ⭐️ 收藏  📒 值得写  ✅ 已读  ❌ 跳过

# 3. 一键更新所有索引
python3 update_indexes.py
# → 同步标记 → 更新收藏索引 → 更新值得写索引
```

**文件职责分离**：
- `fetch_updates.py` - **只采集**，不生成索引
- `update_indexes.py` - **只更新索引**（打标后运行）
- 收藏索引/值得写索引 - **跨周累积**，不随周刊清空

**关键理解**：
- 周刊（`2026-第1周.md`）= **Inbox**（本周待处理，下周清空）
- 收藏索引（`⭐️收藏精选.md`）= **Archive**（永久保存，跨周累积）
- 值得写（`值得写.md`）= **创作清单**（待创作的选题）

### v2.6 (2026-01-04) - 新增 4 个顶级播客源 🎙️

**新增播客源**：
1. **Lex Fridman Podcast** (lexfridman.com)
   - AI、科学、哲学深度访谈
   - 嘉宾包括顶级研究者、创业者、思想家
   - 长篇对话（2-4小时），深度探讨

2. **Cognitive Revolution** (cognitiverevolution.ai)
   - AI 前沿技术与应用深度解析
   - AI 创业案例与产业洞察
   - 技术趋势前瞻

3. **80,000 Hours Podcast** (80000hours.org)
   - 有效利他主义（Effective Altruism）
   - 职业选择与社会影响力
   - AI 安全、全球优先事项

4. **Latent Space** (latent.space)
   - AI 工程实践与产品开发
   - AI 创业公司深度访谈
   - 技术栈选择与架构设计

**技术实现**：
- 创建 4 个 podcast parser：`lexfridman.py`, `cognitiverevolution.py`, `hours80k.py`, `latentspace.py`
- 全部使用 RSS feed（零依赖，简单可靠）
- Latent Space 初始使用 Flightcast URL 超时，改用 Substack `/feed` endpoint 成功
- 测试通过，成功采集 40 篇节目（10+10+10+10）

**内容矩阵再扩展**：
| 维度 | v2.5 覆盖 | v2.6 新增播客 |
|------|----------|-------------|
| AI 技术深度 | 袁超发、HF、Import AI | **Lex Fridman**（顶级对话）<br>**Cognitive Revolution**（前沿技术） |
| AI 工程实践 | - | **Latent Space**（工程与创业） |
| AI 安全/伦理 | - | **80,000 Hours**（有效利他） |
| 跨学科视野 | Wait But Why | **Lex Fridman**（科学/哲学） |

**采集内容示例**：
- Latent Space: "The Agent Labs Thesis", "Agentic Commerce Protocol"
- Lex Fridman: 对话顶级 AI 研究者与创业者
- Cognitive Revolution: AI 应用案例深度分析
- 80,000 Hours: AI 安全与职业影响力

**设计哲学**：
- **播客优先 RSS**：所有 podcast 都有标准 RSS，稳定可靠
- **URL 调试**：Latent Space 从 Flightcast → Substack，找到最稳定endpoint
- **零额外依赖**：继续复用 `web_utils.py` 的 `fetch_html()`

### v2.5 (2026-01-04) - 新增 4 个高质量信息源 🌐

**新增信息源**：
1. **袁超发技术博客** (yuanchaofa.com)
   - LLM/AI 技术深度文章
   - 手写大模型组件系列（Transformer、LoRA、GQA等）
   - 中文技术内容，工程实践导向

2. **Farnam Street Brain Food** (fs.blog)
   - 认知科学、心理学、决策理论
   - 225+ 期周更 newsletter
   - 深度思考与智慧洞察

3. **Austin Kleon** (austinkleon.substack.com)
   - 创意写作与艺术实践
   - "Steal Like an Artist" 作者
   - 创作过程与生活洞察

4. **Paul Graham Essays** (paulgraham.com)
   - 创业哲学与黑客文化
   - Y Combinator 创始人
   - 经典长文（How to Do Great Work 等）

**技术实现**：
- 创建 4 个新 parser：`yuanchaofa.py`, `brainfood.py`, `austinkleon.py`, `paulgraham.py`
- 3 个使用 RSS feed（简单可靠），1 个 HTML 解析（Paul Graham 无 RSS）
- 更新 `sources.json` 和 `fetch_updates.py`
- 测试通过，成功采集 45 篇文章（10+10+10+15）

**内容矩阵扩展**：
| 维度 | 原有源 | 新增源 |
|------|-------|--------|
| AI技术 | HN, HF, Import AI | **袁超发**（深度实现） |
| 创业/产品 | PH, HN | **Paul Graham**（哲学） |
| 认知/思维 | James Clear, Wait But Why | **Brain Food**（决策） |
| 创意/艺术 | - | **Austin Kleon**（创作） |

**设计哲学**：
- **优先 RSS**：3/4 使用 RSS，稳定可靠
- **HTML 兜底**：Paul Graham 无 RSS，简单 HTML 解析
- **零额外依赖**：复用 `web_utils.py` 的 `fetch_html()`

### v2.4 (2026-01-04) - Ben's Bites 解析器重构 ⚡

**问题**：
- Ben's Bites 采集失败，Playwright 访问主页超时（90秒）
- 过度工程化：用浏览器自动化对抗 Cloudflare，但 RSS feed 本身可直接访问

**解决方案**：
- 重写 `parsers/bensbites.py`，移除 Playwright 依赖
- 使用 `web_utils.py` 的 `fetch_html()` 直接访问 RSS feed
- 从 114 行减少到 107 行，零额外依赖

**性能对比**：
| 维度 | 修复前（Playwright） | 修复后（curl） |
|------|---------------------|---------------|
| 速度 | 90秒超时，失败 | 2-3秒完成 |
| 依赖 | Playwright + Chromium | 零依赖 |
| 可靠性 | 容易因网站变化失败 | RSS 格式稳定 |

**设计哲学**：
- **简单优于复杂**: Simple is better than complex
- **消除假想敌**: RSS feed 是公开的，不需要对抗 Cloudflare
- **能消失的复杂性永远比能写对的复杂性更优雅**

### v2.3 (2026-01-04) - LLM 增强推荐 🤖

**功能增强**：
- 推荐引擎集成 LLM 能力，自动翻译标题并打分
- 三维评分体系：兴趣匹配度 + 深度潜力 + 写作可行性
- 生成乔木风格写作大纲（开头/核心观点/类比/结尾）
- 双文件输出：`recommendations.md`（英文）+ `recommendations_enhanced.md`（中文增强版）

**设计哲学**：
- 能用 LLM 直接解决的事，不要写脚本
- 简单、高效、零依赖

### v2.2 (2026-01-03) - 路径配置化重构 🏗️

**架构变更**：
- 新增 `config/paths.json` - 统一输出路径配置
- 新增 `scripts/shared/path_utils.py` - 公共路径工具模块
- 重构所有脚本，移除硬编码路径，实现配置化
- **输出目录迁移**: 从技能内部 `data/` 移至用户知识库 `~/乔木新知识库/01.inbox/素材索引/`

**修改的文件**：
- `fetch_updates.py` - 使用 `get_content_log_path()`
- `recommend_topics.py` - 使用 `get_recommendations_path()`
- `generate_weekly_index.py` - 使用 `get_weekly_index_path(year, week)`
- `generate_starred_index.py` - 使用 `get_starred_index_path()`

**设计哲学**：
- **配置优于硬编码**: 路径可配置，适应不同用户的知识库组织
- **单一真相源**: 所有路径配置集中在 `paths.json`
- **向后兼容**: 若 `paths.json` 不存在，回退到默认路径
- **关注点分离**: 内部状态 (`state.json`) 保留在技能目录，用户可见输出保存到知识库

**文件组织哲学**：
```
内部数据(state.json) → 技能目录      # 隐藏实现细节
用户输出(索引文件)   → 知识库inbox  # 用户可见可管理
```

### v2.0 (2025-12-27) - 智能推荐升级

**架构变更**：
- 新增 `config/user_preferences.json` - 用户偏好配置
- 重写 `recommend_topics.py` - 智能过滤+主题聚类+深度评分
- 新增 6大主题分类：AI模型/应用/政策/哲学/工程/创业

**核心算法**：
1. 兴趣过滤（减少66%噪音：112篇→38篇）
2. 深度评分（来源基础分+关键词加分）
3. 主题聚类（按关键词分组）
4. 乔木风格推荐模板（开头/核心观点/类比/结尾）

**设计哲学**：
- 从"通用推荐"到"个性化推荐"
- 信息采集≠知识筛选
- 数量不等于质量，好的推荐=过滤噪音+提供角度

### v1.0 (2025-12-24) - 初始版本

基础功能：内容抓取、URL去重、简单推荐

---

## 📐 系统架构

```
qiaomu-ai-radar/
├── config/
│   ├── sources.json          # 信息源配置（URL、parser、抓取间隔）
│   ├── user_preferences.json # 用户偏好配置（v2.0）
│   └── paths.json            # 输出路径配置（v2.2）✨
├── scripts/
│   ├── fetch_updates.py      # 核心抓取引擎（遍历 sources，调用 parsers）
│   ├── recommend_topics.py   # 选题推荐引擎（分析素材，生成推荐）
│   ├── generate_weekly_index.py   # 周刊索引生成器
│   ├── generate_starred_index.py  # 收藏索引生成器
│   ├── parsers/              # 解析器库（每个信息源一个独立 parser）
│   │   ├── hackernews.py     # [聚合器] HN 热门话题
│   │   ├── importai.py       # [RSS] Import AI newsletter（AI 政策、研究）
│   │   ├── huggingface.py    # [HTML] Hugging Face 社区热门论文
│   │   ├── jamesclear.py     # [归档] James Clear 3-2-1 newsletter
│   │   ├── waitbutwhy.py     # [博客] Wait But Why 深度长文
│   │   ├── producthunt.py    # [聚合器] Product Hunt 新产品
│   │   ├── bensbites.py      # [RSS] Ben's Bites（v2.4 修复，curl直接访问）
│   │   ├── tldrai.py         # [RSS] TLDR AI（AI 行业快讯）
│   │   ├── yuanchaofa.py     # [RSS] 袁超发技术博客（v2.5 新增）
│   │   ├── brainfood.py      # [RSS] Farnam Street Brain Food（v2.5 新增）
│   │   ├── austinkleon.py    # [RSS] Austin Kleon Substack（v2.5 新增）
│   │   ├── paulgraham.py     # [HTML] Paul Graham Essays（v2.5 新增）
│   │   ├── lexfridman.py     # [Podcast] Lex Fridman（v2.6 新增）🎙️
│   │   ├── cognitiverevolution.py  # [Podcast] Cognitive Revolution（v2.6 新增）🎙️
│   │   ├── hours80k.py       # [Podcast] 80,000 Hours（v2.6 新增）🎙️
│   │   └── latentspace.py    # [Podcast] Latent Space（v2.6 新增）🎙️
│   └── shared/
│       ├── web_utils.py      # 共享工具（curl wrapper，HTML 提取）
│       └── path_utils.py     # 路径配置加载器（v2.2）✨
└── data/
    ├── state.json            # URL 去重状态（MD5 hash，保留在技能目录）
    ├── auto_run.log          # 自动运行日志
    ├── launchd.log           # launchd 标准输出
    └── launchd_error.log     # launchd 错误输出

输出文件（v2.2 迁移到用户知识库）:
~/乔木新知识库/01.inbox/素材索引/
├── content_log.md            # 历史素材库（按日期组织）
├── recommendations.md        # 最新推荐（英文版，recommend_topics.py 生成）
├── recommendations_enhanced.md  # 智能推荐（中文翻译+打分+写作大纲，v2.3）
├── 2025-第1周.md            # 周刊索引（按周组织）
├── 2025-第2周.md
└── ⭐️收藏精选.md           # 收藏索引（跨周累积）
```

---

## 🧠 设计哲学

### 1. 模块化解析器设计

**问题**: 不同网站结构千差万别，HTML/RSS/API 格式各异
**方案**: 每个信息源一个独立 parser，统一返回格式

```python
# 所有 parser 返回统一结构
[
    {
        "url": "文章 URL",
        "title": "标题",
        "summary": "摘要（可选）"
    }
]
```

**好处**:
- 单一职责：每个 parser 只负责一个网站
- 易于扩展：添加新源 = 写新 parser + 注册
- 故障隔离：一个 parser 失败不影响其他

### 2. URL 去重的哲学

**为什么用 MD5 hash 而非直接比对 URL?**

```python
# ❌ 坏方案：直接存储完整 URL
seen_urls = ["https://...", "https://..."]  # 占用大量内存

# ✅ 好方案：存储 12 位 hash
seen_hashes = {"a8bbd636a456": "...", ...}  # 紧凑高效
```

**权衡**:
- Hash 冲突概率: MD5(12位) ≈ 1/16^12 ≈ 可忽略
- 内存节省: ~90%（完整 URL 通常 50+ 字符 → 12 字符）
- 查找速度: O(1) dict lookup

### 3. 为什么不用数据库？

**设计决策**: 文件系统 > SQLite > PostgreSQL

**理由**:
- **简单性**: Markdown 可直接阅读，无需 SQL 查询
- **便携性**: 整个 skill 是一个文件夹，可直接复制/同步
- **零依赖**: 不需要安装数据库
- **够用就好**: 即使每天抓取 100 篇文章 × 365 天 = 3.6 万条记录，Markdown 文件也就几 MB

**何时需要数据库**:
- 需要复杂查询（多维度筛选、全文搜索）
- 数据量 > 10 万条
- 需要并发写入

当前场景：**过度工程** 🚫

### 4. 选题推荐的启发式算法

**策略**: 基于规则的推荐 > 机器学习

```python
# 推荐优先级
if source == "Import AI":
    priority = "high"  # 深度分析，适合写洞察
elif source == "Hugging Face Papers":
    priority = "medium"  # 论文解读
elif "replaced" in title or "simple" in title:
    priority = "high"  # 工程哲学类
```

**为什么不用 ML?**
- 数据太少（每天 < 100 篇）
- 目标明确（找适合乔木风格的话题）
- 规则可解释、可调整

**Linus 的品味**: 简单规则 > 黑盒算法

---

## 🔧 关键设计决策

### 抓取频率

| 信息源 | 频率 | 理由 |
|--------|------|------|
| Hacker News | 24h | 每天有新热点 |
| Import AI | 168h (周) | 每周发布 |
| Hugging Face | 24h | 论文更新频繁 |
| James Clear | 168h | 每周三发布 |

**原则**: 匹配内容更新频率，避免重复抓取

### 错误处理

**哲学**: Fail gracefully，永远交付结果

```python
# ✅ 好的错误处理
for source in sources:
    try:
        articles = parser(source)
    except Exception as e:
        print(f"⚠️  {source} 失败: {e}")
        continue  # 跳过这个源，继续其他

# 即使所有 parser 都失败，也返回空列表而非崩溃
```

**案例**: Ben's Bites 被 Cloudflare 拦截
- ❌ 糟糕方案：整个抓取失败
- ✅ 优雅方案：禁用该源，其他照常工作

---

## 📊 数据流

```
[定时任务 9:00 AM]
       ↓
fetch_updates.py
       ↓
遍历 sources.json
       ↓
调用各个 parser ──→ [网络请求] ──→ HTML/RSS/JSON
       ↓
提取文章 {url, title, summary}
       ↓
URL 去重 (hash 比对 state.json)
       ↓
追加到 content_log.md
       ↓
更新 state.json
       ↓
[用户调用 skill]
       ↓
recommend_topics.py
       ↓
读取 content_log.md（最近3天）
       ↓
分析主题、来源、关键词
       ↓
生成推荐 → recommendations.md
       ↓
返回给用户
```

---

## 🚀 扩展指南

### 添加新信息源（3 步）

#### Step 1: 写 parser

```python
# parsers/newsource.py
def parse_newsource(source):
    html = fetch_html(source["url"])
    # 提取文章...
    return [{"url": ..., "title": ..., "summary": ...}]
```

#### Step 2: 注册 parser

```python
# fetch_updates.py
from newsource import parse_newsource

PARSERS = {
    # ...
    "newsource": parse_newsource,
}
```

#### Step 3: 添加配置

```json
// config/sources.json
{
  "id": "newsource",
  "name": "New Source",
  "url": "https://...",
  "parser": "newsource",
  "enabled": true,
  "check_interval_hours": 24
}
```

**就是这么简单** 💪

---

## 📈 性能特征

- **抓取速度**: ~5 秒/源（网络延迟为主）
- **内存占用**: < 50 MB（纯 Python，无数据库）
- **存储增长**: ~10 KB/天（假设 50 篇文章 × 200 字符）
- **可扩展性**: 单机可支持 100+ 信息源

**瓶颈**: 网络 I/O，非计算或存储

---

## ⚠️ 已知限制

### 1. Cloudflare 防护
- **影响**: Ben's Bites、部分 Substack 站点
- **方案**: 改用 RSS feed 或 Playwright

### 2. HTML 结构变化
- **风险**: 网站改版导致 parser 失效
- **缓解**: 定期检查 cron.error.log

### 3. 推荐算法简陋
- **现状**: 基于规则的启发式
- **改进**: 可引入 TF-IDF、主题建模

### 4. 无全文搜索
- **现状**: 只能按日期浏览
- **改进**: 可添加 SQLite FTS5

---

## 🎯 设计权衡总结

| 决策 | 选择 | 权衡 |
|------|------|------|
| 存储 | Markdown 文件 | 简单 vs 查询能力 |
| 去重 | MD5 hash | 内存效率 vs 极小冲突风险 |
| 推荐 | 基于规则 | 可解释 vs 智能程度 |
| 错误处理 | Fail gracefully | 可靠性 > 完美性 |
| Parser 架构 | 独立模块 | 可维护 > 代码复用 |

**核心哲学**: **实用主义 > 完美主义**（Linus Good Taste ✅）

---

## 📝 维护清单

### 每周检查
- [ ] `cron.error.log` 有无新错误
- [ ] `content_log.md` 文件大小（建议 < 10 MB）

### 每月检查
- [ ] 各 parser 是否仍工作（网站结构是否变化）
- [ ] `state.json` 大小（建议 < 5 MB，可清理 90 天前记录）

### 按需优化
- [ ] 添加新信息源
- [ ] 优化推荐算法
- [ ] 清理历史数据（保留最近 1 年）

---

**哲学总结**:
这个 skill 的设计遵循 Linus 的 Good Taste 原则——**消除特殊情况，让边界自然融入常规**。每个 parser 独立，每个错误被隔离，整个系统即使部分失效也能继续工作。简单、可靠、易扩展。

*Good code doesn't need exceptions. Good architecture doesn't need rescue.*

---
> Source: [joeseesun/qiaomu-ai-radar](https://github.com/joeseesun/qiaomu-ai-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
