## xhs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 常用命令

```bash
# 交互式菜单（推荐入口）
python start.py

# CLI 直接操作
python pipeline.py --list                           # 列出选题池
python pipeline.py --index N                        # 按编号生成单篇
python pipeline.py --batch --max 3                  # 批量生成
streamlit run app.py                                # 启动审核工作台（6 个 tab，含一键发布助手）

# 发布与数据采集（spider_xhs 集成，首次需小红书 APP 扫码登录）
python scripts/auto_publish.py --dry-run            # 预览 approved/ 待发布笔记（不登录、不实际发布）
python scripts/auto_publish.py --limit 3            # 发布 approved/ 笔记到小红书 -> published/
python scripts/fetch_performance.py                 # 采集已发布笔记互动数据，更新 performance.json
python scripts/competitor_analysis.py --save        # 竞品爆款分析，一键加入选题池
python scripts/xhs_auth_helper.py --creator         # 单独登录/刷新创作者端 Cookie

# 代码质量
pytest                                              # 运行测试
black .                                             # 格式化
ruff check .                                        # Lint
mypy core/                                          # 类型检查
```

## 项目架构

小红书情感账号 AI 内容创作系统。自动生成图文笔记（封面 + 内页 + 审核 + 预设评论），配合 Streamlit 工作台完成审核-发布全流程。

### Multi-Agent 创作流水线

核心创作流程在 `core/agents/orchestrator.py` 中编排，7 个专家 Agent 通过 **MessageBus**（发布/订阅）协作：

```
TopicStrategist → EmotionalWriter → (CoverDesigner ∥ LayoutArtist ∥ ContentEditor)
                                                          ↓
                              ChiefEditor ← 决策 ← 审核报告
                                                          ↓
                                              CommunityManager（预设评论）
```

- **TopicStrategist**：增强选题 brief，补充角度和叙事策略。
- **EmotionalWriter**：LLM 写作，支持 4 种标题公式（问句式/概念解读式/观点冲击式/方法承诺式）和 7 种页面结构（A-G），按公式注入特化指令。
- **CoverDesigner**：提取封面信息（标题/小字/AI 绘画 prompt/视觉风格）。
- **LayoutArtist**：调用 `core/image_generator.py` 生成内页分页图。
- **ContentEditor**：LLM 审核，输出结构化报告（等级 S/A/B/C、问题列表、建议）。
- **ChiefEditor**：主编决策，最多 **5 轮迭代**。决策逻辑：
  - A 级直接通过；B 级 + issues ≤ 2 可通过。
  - 连续 3 轮同级（S/A/B）强制通过，防止审核死循环。
  - 5 轮后仍不达标则 **放弃**（`status: abandoned`）。
- **CommunityManager**：生成 5-8 条拟真预设评论。

每个 Agent 通过 `AgentMemory`（`core/agents/memory.py`）持久化跨会话记忆到 `data/agent_memory/`，记录成功/失败模式。

### MessageBus 通信机制

`core/agents/base.py` 定义了发布/订阅模式的消息总线：
- `MessageType` 枚举定义了 9 种消息类型（BRIEF/DRAFT/DESIGN/REVIEW/REQUEST/RESPONSE/DECISION/COMMENT/NOTIFY）。
- `MessageBus.subscribe()` 注册回调，`publish()` 支持广播（`to_agent=None`）和点对点投递。
- `MessageBus.get_history()` 和 `get_last_message()` 用于查询通信历史。
- `BaseAgent.think()` 调用 LLM 时会自动注入 `AgentMemory` 中的持久化上下文。

### 封面与内页生成

`core/image_generator.py` 负责全部图片渲染：

- **封面生成**（`CoverDesigner` 编排，产物文件名仍为 `cover_ai.png`，历史命名）：
  1. **模板合成**（当前主方案）：纯 Pillow 渲染大字报风封面（粗竖条 + 渐变色块 + 超大标题），7 种配色主题（`warm_grey`/`twilight`/`crimson`/`mist`/`cool`/`warm`/`blank`），动态字号防溢出。
  2. **AI 绘画**（可选，未默认启用）：`generate_cover_ai` 走 Pollinations/SiliconFlow/DALL-E，AI 失败时回退模板。数据证明 AI 封面 CTR 偏低（塑料感 + 文字可读性差），已不作主方案。
  - **配色策略（D-03）**：`CoverDesigner` 优先尊重 `EmotionalWriter` 在笔记里定的视觉风格（情绪基调→配色），弱化冷调强制降权，让 `crimson`（愤怒共鸣）/`mist`（代理正义）等强情绪色真正落地。
- **内页分页**：先由 `_paginate_blocks` 计算排版分页（不渲染），得到准确总页数后多线程（`max_workers=4`）渲染。每页含渐变背景、视觉锚点、金句高亮块、页码。

### 关键兼容层

`pipeline.py` 保留了大量向后兼容的辅助函数（`_extract_cover_info`、`_extract_visual_style`、`_clean_md`等）。**`app.py` 依赖这些函数的正则逻辑来解析 `note.md`**。修改这些正则或 `note.md` 输出格式时，必须同步检查 `app.py` 中的解析逻辑。

### 数据持久化

`core/config.py` 提供原子写入 JSON（`tempfile.mkstemp` + `os.replace`）和跨平台文件锁（`fcntl`），防止并发损坏 `data/topics.json` 和 `data/performance.json`。

### LLM 调用层

`core/writer.py` 提供底层 LLM 调用：
- `_call_api()`：OpenAI 兼容 API，内置 3 次指数退避重试（处理 429/5xx）。
- `_extract_content()`：从 OpenAI 兼容响应中提取文本。

高层写作/审核/评论逻辑由 Agent 层处理（`EmotionalWriter.write()`、`ContentEditor.review()`、`CommunityManager.generate_comments()`）。

### 输出目录结构

每篇笔记生成到 `docs_agent/pending/{topic_name}_{hash}/` 下，随生命周期推进自动迁移：

```
docs_agent/
  pending/                          # 新生成未审核（pipeline 默认输出目录）
    {topic}_{hash}/
      note.md                       # 主文件（正文 + 审核 + 封面/内页路径 + 预设评论）
      cover_ai.png                  # AI 绘画封面
      inner_page_1.png              # 内页第 1 页
      inner_page_2.png              # 内页第 2 页
      ...
  approved/                         # 已审核通过，等待在小红书发布（app.py 点击"通过"后迁移）
    {topic}_{hash}/...
  published/                        # 已在小红书发布，等待录入数据（用户确认"我发布了XX"后迁移）
    {topic}_{hash}/...
  archived/                         # 已发布且已录入数据（录入互动数据后自动迁移）
    {topic}_{hash}/...
```

**生命周期迁移规则**：
- `pipeline.py` 生成笔记 → `pending/`
- `app.py` 点击「✅ 通过」→ 整目录 `shutil.move` 到 `approved/`
- 用户确认「我发布了 XX」→ 整目录 `shutil.move` 到 `published/`
- `app.py` 录入互动数据 → 整目录 `shutil.move` 到 `archived/`

⚠️ **重要**：审核通过不等于已发布。只有经 `auto_publish.py` / 🚀「一键发布助手」tab 实际发布，或用户明确说"我发布了 XX 笔记"，才迁到 `published/`。

一次性迁移脚本：`python scripts/reorganize_notes.py`（首次把历史混放的笔记按状态分到 4 个子目录）。

### 自动发布与数据采集（spider_xhs 集成）

`spider_xhs/` 是 git 子模块（`cv-cat/Spider_XHS`），提供小红书 PC 端采集 + 创作者端发布能力。`spider_xhs/__init__.py` 二次封装并导出 4 个类：`XHS_Apis`（PC 采集/搜索）、`XHS_Creator_Apis`（创作者发布）、`XHSPcAuth` / `XHSCreatorAuth`（两套登录认证）。运行时依赖 Node.js（用 `node_modules/crypto-js` 生成请求签名）。

**认证层**（`scripts/xhs_auth_helper.py`）：Cookie 持久化到 `.cache/xhs_{pc,creator}_cookie.json`，首次扫码登录、后续复用、过期自动重扫。`get_pc_auth()` 供采集脚本用，`get_creator_auth()` 供发布脚本用。

**三个自动化脚本**（均经 spider_xhs）：

| 脚本 | 作用 | 目录迁移 |
|---|---|---|
| `scripts/auto_publish.py` | 登录创作者端，发布 `approved/` 笔记（`--dry-run` 预览 / `--limit N`），URL 写回 `note.md` | `approved/ -> published/` |
| `scripts/fetch_performance.py` | 登录 PC 端采集 `published/`+`archived/` 互动数据，算 S/A/B/C 等级，更新 `performance.json`，触发熔断 | 仅更新数据 |
| `scripts/competitor_analysis.py` | 登录 PC 端搜情感赛道爆款（>1000 赞），分析标题公式/标签，结合历史表现优化搜索词，可 `--save` 加入选题池 | 反哺 `topics.json` |

**Streamlit 工作台**（`app.py`）现含 6 个 tab：📝待审核 / 📚选题池 / 📖创作标准 / 📈数据复盘 / 📅内容日历 / 🚀一键发布助手。其中：
- `tabs/publish.py`（🚀 一键发布助手）：直接调 `XHS_Creator_Apis + get_creator_auth`，UI 勾选 approved 笔记即可自动发布，免去命令行。
- `tabs/analytics.py`（📈 数据复盘）：录入互动数据、熔断预警、公式/支柱分析。

**熔断机制**：最近 3 篇全 C 级 -> 触发熔断，侧边栏红字警告暂停发布、复盘选题（`app.py:_check_circuit_breaker` 与 `fetch_performance.py:_check_circuit_breaker`）。

**话题标签真链接解析**（`core/publish_helpers.py: resolve_topics`）：发布前逐个调 `get_topic` 把标签解析为真话题链接（进话题流量池、可点击），解析失败的降级为纯文本 `#tag`（仍可见但不进流量池），避免冷门标签解析 `raise RuntimeError` 中断整篇发布。dry-run 无 api，全部纯文本展示。

**完整闭环**（原"手动迁移"已部分自动化）：

```
pipeline 生成 -> pending/ -> app.py「通过」-> approved/
   -> auto_publish.py / 🚀tab（创作者端 API 自动发布）-> published/（URL 写回 note.md）
   -> fetch_performance.py（PC 端 API 采集互动）-> 更新 performance.json + 等级 + 熔断
   -> competitor_analysis.py（历史表现反哺选题）-> 选题池
```

### 废笔记健康检查（防 LLM 思考链泄漏）

glm-5.2 等模型可能把英文推理过程（`Let me analyze...`）混进 `content` 返回，`core/writer.py:_extract_content` 不剥离，导致整篇笔记变成思考链（正文英文占比异常、无实质内容）。审核 LLM 同样泄漏且可能给通过，必须代码兜底。三道防线：

- **`core/utils.py: check_note_health(content)`**：检测思考链特征词（≥2 处，如 `Let me`/`I need to`/`Hmm`）+ 英文占比异常（英文词 >20 且 > 中文字 ×0.5），返回问题列表。标签段缺失**不**在此检查（由 `check_tags_compliance` 负责，避免误伤漏标签的正常笔记）。
- **生成时拦截**：`ChiefEditor._make_decision` 加 health 硬门禁（与标签门禁并列），废笔记 -> revise，feedback 指出思考链泄漏。
- **发布前置校验**：`auto_publish.py:parse_note` 与 `tabs/publish.py` 发布前校验，废笔记跳过发布；🚀tab UI 标红 + 禁用勾选 + `auto_publish_notes` 双保险拦截。

实测：思考链泄漏废笔记（命中 49 处特征词）被精准拦截；正常中文笔记（含 `#不懂就问有问必答` 等标签）无误判。

## 平台调性约束

- **陪伴感优先于分析感**：情感内容必须像闺蜜夜聊，禁止理性分析和下定义式表达。
- **封面生成策略**：以**模板大字封面**为主（文件名 `cover_ai.png` 为历史命名）。AI 绘画封面因 CTR 偏低已不作主方案。配色由写手定的视觉风格驱动（见 D-03）。

## 涨粉优化约定（2026-06 改造，详见 `docs/涨粉优化执行计划.md`）

- **话题标签（D-01/D-02）**：3-5 个精准标签，禁用泛词（#情感 #恋爱 等）。`core/utils.py: sanitize_tags` 在写手产出后确定性兜底，`ChiefEditor` 硬门禁拦截不合规标签。**凡 prompt 写了但 LLM 不稳定遵守的规则，一律用代码兜底**，不依赖 LLM 自觉。发布时 `resolve_topics` 把标签解析为真话题链接进流量池（见「自动发布与数据采集」）。
- **内页气泡（D-05）**：`_parse_to_blocks` 识别"角色：内容"对话行，渲染成左右聊天气泡（主角方右暖色、对方左灰），不再拍扁成纯文字。block 结构为 4 元组 `(text, is_bold, is_sep, side)`，向后兼容 3 元组调用与索引访问。
- **金句海报（D-06）**：任何页结尾短句居中放大成海报页（`last_block_font_boost=18`），便于截图传播。
- **选题（D-07）**：`generate_topics.py` 强制情绪光谱均衡（≥4 种情绪）+ 搜索型问句配额（≥2 个），避免单一"隐性伤害"赛道锁死。
- **人设与系列（D-08）**：固定人设 = 31 岁互联网打工人（见 `prompts/agent_writer.md`）。每个选题带 `series` 字段（5 个固定系列），写手结尾输出系列引导建立追更感。

## 环境依赖

- Python 3.13+，复制 `.env.example` 为 `.env` 并填写 `LLM_API_KEY`。
- 必须提供思源黑体字体：`assets/fonts/NotoSansSC-Regular.ttf` 和 `NotoSansSC-Bold.ttf`。缺失时按平台 fallback 系统黑体，但排版效果会下降。
- `data/topics.json` 为必选文件，缺失时程序直接报错。

---
> Source: [orangeforu/xhs](https://github.com/orangeforu/xhs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
