## ai-investment-advisor

> 基于 Claude Code 的私人投资分析系统，支持多模型协同决策。

# 私人投资分析系统 v2.1

## 项目概述

基于 Claude Code 的私人投资分析系统，支持多模型协同决策。

## 回复语言要求

- **必须使用中文回复用户**

**核心功能**：
- 每日投资简报（分析性，非数据罗列）
- 市场扫描与机会发现
- 个股深度分析
- 交易记录与验证
- 周期性复盘
- **多模型投资委员会**（Claude + Codex + Gemini）

---

## 文件结构

```
个人系统/
├── AGENTS.md                          # 本文件，系统说明
├── 使用手册.md                        # 用户操作手册
├── .claude/
│   └── skills/
│       ├── brief/SKILL.md             # /brief 每日简报
│       ├── scan/SKILL.md              # /scan 市场扫描
│       ├── analyze/SKILL.md           # /analyze 个股分析
│       ├── trade/SKILL.md             # /trade 交易记录
│       ├── review/SKILL.md            # /review 周期复盘
│       └── committee/SKILL.md         # /committee 多模型委员会
│
└── 股市信息/
    ├── Config/
    │   ├── Holdings.md                # 持仓明细（唯一真相源）
    │   ├── Watchlist.md               # 关注池（行业、标的、逻辑）
    │   ├── Profile.md                 # 投资者画像（用户自述）
    │   ├── Principles.md              # 投资原则
    │   ├── Context.md                 # 上下文摘要（自动生成）
    │   └── Insight.md                 # 用户洞察（动态观察，Claude维护）
    │
    ├── Daily/
    │   └── YYYY-MM-DD-Brief.md        # 每日简报
    │
    ├── Analysis/
    │   └── [代码]-[名称].md           # 个股分析报告
    │
    ├── Records/
    │   ├── trades.md                  # 交易记录
    │   └── reviews/                   # 复盘报告
    │
    ├── Committee/                     # 多模型决策
    │   ├── Input/                     # 统一输入数据
    │   ├── Opinions/                  # 各模型观点
    │   ├── Sessions/                  # 会议记录
    │   └── Templates/                 # 模板文件
    │
    └── scripts/
        └── fetch_market_data.py       # 数据获取脚本 v2.0
```

---

## 数据源架构（核心）

### 数据来源优先级

```
第一优先：fetch_market_data.py 脚本输出
├── indices      - 主要指数（上证、沪深300、科创50等）
├── holdings     - 持仓行情（价格、涨跌、盈亏）
├── watchlist    - 关注池行情
├── macro        - 宏观数据（PMI、CPI、M2）
├── north_flow   - 北向资金
└── news         - 财联社快讯

第二优先：配置文件
├── Holdings.md   - 持仓（成本、数量、买入日期）
├── Watchlist.md  - 关注方向
├── Profile.md    - 投资风格（用户自述）
├── Context.md    - 上下文摘要
└── Insight.md    - 用户洞察（Claude维护）

第三优先：WebSearch（谨慎使用）
├── 仅用于：政策原文、公司公告、研报观点
├── 必须标注"来源：网络搜索"
└── 价格数据以脚本为准
```

### 数据真实性原则

1. **价格必须来自脚本**：禁止估算、假设、使用记忆数据
2. **脚本失败时明确告知**：禁止使用默认值或编造
3. **来源标注**：所有数据标注来源和获取时间
4. **时效说明**：提醒用户数据可能有15-30分钟延迟

---

## 六大 Skill

## Skills 同步规则

- Claude 技能目录：`.claude/skills/`
- Codex 技能目录：`~/.codex/skills/`（用户目录下）
- **任何技能新增/修改/删除都必须同步更新两边，保持内容一致**
- 同步时不影响 Claude 现有技能，仅做镜像或复制到 Codex

### /brief - 每日简报

**触发词**："简报"、"今日市场"、"持仓分析"

**执行流程**：
1. 运行 `fetch_market_data.py` 获取数据
2. 读取 Holdings.md、Watchlist.md、Profile.md
3. 从脚本 `news` 字段提取相关快讯（替代 WebSearch）
4. 综合分析生成简报（必须覆盖全部持仓）
5. 保存到 `Daily/YYYY-MM-DD-Brief.md`

### /scan - 市场扫描

**触发词**："有什么机会"、"推荐"、"扫描市场"

**执行流程**：
1. 读取用户偏好（Profile、Watchlist、Holdings）
2. 运行 `fetch_market_data.py` 获取数据
3. 分析指数风格、北向资金、快讯热点
4. 筛选关注池机会
5. 生成扫描报告（最多推荐3个标的）

### /analyze - 个股分析

**触发词**："分析XX"、"看看XX怎么样"、"XX值得买吗"

**执行流程**：
1. 识别标的代码
2. 运行 `fetch_market_data.py` 获取价格
3. 读取持仓配置（判断是否已持有）
4. WebSearch 补充公司基本面信息（谨慎使用）
5. 生成分析报告并自动保存

### /trade - 交易记录

**触发词**："买了"、"卖了"、"加仓"、"减仓"

**执行流程**：
1. 确认交易信息
2. 运行脚本验证价格（与当前价偏差不超过5%）
3. 更新 `trades.md`
4. 更新 `Holdings.md`
5. 更新 `Context.md`

### /review - 周期复盘

**触发词**："复盘"、"回顾"、"这周怎么样"

**执行流程**：
1. 读取交易记录和简报
2. 运行脚本获取最新价格
3. 验证历史建议准确性（30天/90天验证）
4. 生成复盘报告
5. 沉淀经验到 Principles.md

### /committee - 投资委员会

**触发词**："开会"、"投资委员会"、"多模型分析"

**执行流程**：
1. 运行脚本获取市场数据
2. 读取 Context.md 获取持仓上下文
3. 确定决策问题，生成统一输入文件
4. Claude 先输出分析意见
5. 引导用户获取 Codex、Gemini 意见
6. 用户返回后汇总三方共识
7. 保存会议记录到 `Committee/Sessions/`

---

## 上下文同步机制

### Context.md 作用

- 自动生成的持仓摘要
- 供各 Skill 和多模型读取
- 确保上下文一致性

### 更新时机

- `/trade` 执行后自动更新
- 手动触发时更新

### 内容包含

- 持仓统计和列表
- 最近操作记录
- 待验证交易
- 关注方向摘要
- 风险提示

---

## 用户洞察机制

### Insight.md 作用

与 Profile.md（用户自述）互补，记录 Claude 通过交互观察到的用户特征：
- **行为模式**：决策特征、买卖习惯
- **心理画像**：情绪触发点、认知偏差
- **成长轨迹**：进步记录、待改进事项
- **决策采纳**：对各类建议的采纳率和结果

### 更新时机

| 触发场景 | 更新内容 |
|----------|----------|
| `/trade` 交易记录 | 分析决策心理，记录是否遵循规则，追踪建议采纳情况 |
| `/review` 复盘 | 总结周期行为模式，更新进步/退步记录 |
| `/brief` 简报 | 读取 Insight 以个性化提醒，观察用户对建议的反应 |
| `/committee` 投委会 | 记录用户对多模型共识的采纳情况 |
| 日常对话 | 捕捉情绪、观点变化、关键洞察 |

### 核心追踪维度

1. **采纳率追踪**：记录各类建议（简报/投委会/分析）的采纳情况和结果
2. **行为验证**：对比用户自述弱点与实际行为（如"不会止盈"是否改善）
3. **模式识别**：发现用户更容易接受什么类型的建议

### 使用原则

- 读取：每次 skill 执行前读取 Insight.md，提供个性化服务
- 更新：skill 执行后根据用户行为更新观察记录
- 诚实：记录真实观察，不美化也不苛责

---

## 多模型协同（已实现）

### 目标

让 Claude、Codex、Gemini 共同读取相同数据，独立分析，提取共识。

### 架构

```
统一输入层
├── Context.md                    - 持仓上下文
├── Committee/Input/YYYY-MM-DD-Input.md - 当日输入数据
└── 决策问题                       - 用户指定的讨论问题

独立观点层
├── Committee/Opinions/Claude.md  - Claude 分析意见
├── Committee/Opinions/Codex.md   - Codex 分析意见
└── Committee/Opinions/Gemini.md  - Gemini 分析意见

共识提取层
├── 强共识 (3/3一致) → 可考虑直接执行
├── 弱共识 (2/3一致) → 参考多数意见
└── 分歧区 (各不相同) → 需用户决策
```

### 操作流程

1. 用户说"开会"或 `/committee`
2. Claude 生成统一输入数据，输出自己的分析
3. 用户复制输入数据到 Cursor/Copilot（获取 Codex 意见）
4. 用户复制输入数据到 Gemini（获取 Gemini 意见）
5. 用户返回告诉 Claude "三个都弄好了"
6. Claude 读取三方意见，提取共识，生成决策汇总

### 模板文件

- `Committee/Templates/prompt_template.md` - 给其他模型的输入提示词
- `Committee/Templates/opinion_template.md` - 标准观点输出格式
- `Committee/Templates/consensus_template.md` - 共识汇总格式

---

## 分析原则

### 1. 不要数据罗列
```
❌ 消费ETF -33.29%
✅ 消费ETF深套977天，以旧换新资金主要利好家电汽车对白酒提振有限
```

### 2. 结合用户持仓
```
❌ 铜价创新高，有色板块强势
✅ 你11月买入时有色已涨很多，铜价创新高后今日回调，建议止盈一半
```

### 3. 给可操作建议
```
❌ 建议关注
✅ 如果回调10%可以考虑加仓，分3次买入
```

### 4. 诚实评估风险
```
❌ 科技方向好，可以买入
✅ 科技方向长期正确，但当前估值高位，现在追高风险较大
```

---

## 使用方式

| 需求 | 命令 |
|------|------|
| 生成今日简报 | `/brief` 或 "简报" |
| 扫描市场机会 | `/scan` 或 "有什么机会" |
| 分析某个标的 | `/analyze 腾讯` 或 "分析下比亚迪" |
| 记录交易 | `/trade` 或 "今天买了红利ETF" |
| 周期复盘 | `/review` 或 "复盘" |
| 多模型讨论 | `/committee` 或 "开会" |

---

## 数据获取脚本 v2.0

### 输出结构

```json
{
  "metadata": { "date": "2026-01-12", "time": "10:41", "version": "2.0" },
  "indices": { "上证指数": { "price": 4135, "change": 0.36 } },
  "holdings": [
    { "code": "00700", "name": "腾讯", "price": 620, "cost": 344, "pnl": 80.1 }
  ],
  "watchlist": [...],
  "macro": { "pmi": {...}, "cpi": {...} },
  "north_flow": { "today": { "net_flow": 50.3 } },
  "news": [{ "time": "10:30", "title": "..." }]
}
```

### 模块开关

在脚本中可配置开启/关闭各模块：
- indices, holdings, watchlist (默认开启)
- macro, north_flow, news (默认开启)
- notices (默认关闭，较慢)

---

*最后更新：2026-01-14*

---
> Source: [AllenAI2014/ai-investment-advisor](https://github.com/AllenAI2014/ai-investment-advisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
