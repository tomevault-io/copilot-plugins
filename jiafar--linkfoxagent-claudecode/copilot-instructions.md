## linkfoxagent-claudecode

> 你可以使用 LinkFoxAgent 进行跨境电商数据分析。它提供 41 个专用工具，覆盖亚马逊/TikTok/eBay/Walmart 产品调研、竞品分析、关键词追踪、评论洞察、专利检测、趋势分析、1688 采购。

# LinkFoxAgent - 跨境电商 AI Agent（41 工具）

你可以使用 LinkFoxAgent 进行跨境电商数据分析。它提供 41 个专用工具，覆盖亚马逊/TikTok/eBay/Walmart 产品调研、竞品分析、关键词追踪、评论洞察、专利检测、趋势分析、1688 采购。

## 环境配置

需要环境变量 `LINKFOXAGENT_API_KEY`。如果未设置，提醒用户：
```bash
export LINKFOXAGENT_API_KEY=你的key
```
获取 Key：https://agent.linkfox.com（右上角 → 设置 → 创建 ApiKey）

## 如何调用

使用项目内的 `linkfox.py` 脚本：

```bash
# 提交任务并等待结果（阻塞模式，任务通常 1-5 分钟）
python3 linkfox.py --wait "你的任务描述"

# 自定义超时（默认 300 秒）
python3 linkfox.py --wait --timeout 600 "你的任务描述"

# 后台模式：提交后立即返回 messageId
python3 linkfox.py "你的任务描述"

# 轮询结果
python3 linkfox.py --poll <messageId>

# JSON 格式输出
python3 linkfox.py --wait --format json "你的任务描述"
```

**重要：** 任务通常需要 1-5 分钟，请使用 `--wait` 模式并设置足够的超时时间。

## 任务 Prompt 写法

### 工具调用语法

用 `@工具中文名` 调用工具，单个任务最多链式调用 10 个工具。

示例：`@卖家精灵-选产品 筛选亚马逊美国站的 "usb charger cable"，返回前40条商品数据`

### 多步骤任务

用编号串联多个步骤，LinkFoxAgent 自动处理数据流转：

```
1、@亚马逊前端搜索模拟 搜索美国站 "computer desk"，返回前2页
2、@对商品标题进行分词 统计上一步商品标题中出现的功能点
3、按功能点统计月销量、月销售额、asin数
```

## 工具选择优先级

**查询亚马逊商品数据**（按优先级）：
1. **Keepa** — 历史/结构化数据，最高效
2. **卖家精灵** — 选品和竞品查找
3. **亚马逊前台** — 需要实时数据或排名顺序时用（较慢）

**数据聚合/统计**：
1. **@智能数据查询** — 动态聚合首选
2. **@Python沙箱** — 需要自定义逻辑时

**@网页解析** — 打开远程浏览器，最慢，仅在其他工具无法获取时使用。

## 全部 41 个工具

| 分类 | 工具名 | 用途 |
|------|--------|------|
| **Keepa** | @Keepa-亚马逊-商品搜索 | 按关键词/BSR/价格/销量筛选商品 |
| **Keepa** | @Keepa-亚马逊-商品详情 | 批量 ASIN 详情查询 |
| **Keepa** | @Keepa-亚马逊价格历史 | ASIN 价格历史和趋势 |
| **亚马逊前台** | @亚马逊前端搜索模拟 | 搜索模拟（可设配送地址） |
| **亚马逊前台** | @亚马逊前端-商品详情 | 商品详情/五点描述/A+ |
| **亚马逊前台** | @亚马逊-商品评论 | 按星级获取评论 |
| **亚马逊前台** | @亚马逊前端-以图搜图 | 以图搜图 |
| **亚马逊前台** | @ABA-数据挖掘 | Amazon Brand Analytics |
| **Sif** | @SIF-ASIN的关键词 | ASIN 反查关键词 |
| **Sif** | @SIF-关键词流量来源 | 关键词流量来源分析 |
| **Sif** | @SIF-ASIN流量来源 | ASIN 流量结构 |
| **Sif** | @SIF-关键词竞品数量 | 关键词竞争密度 |
| **卖家精灵** | @卖家精灵-选产品 | 按类目筛选选品 |
| **卖家精灵** | @卖家精灵-查竞品 | 按关键词查竞品 |
| **极目** | @极目-亚马逊-细分市场评论 | 细分市场评论挖掘 |
| **极目** | @极目-亚马逊-细分市场信息 | 细分市场概览 |
| **极目** | @极目-亚马逊-产品挖掘 | 精细筛选产品 |
| **谷歌趋势** | @谷歌趋势-时下流行 | 实时热门话题 |
| **谷歌趋势** | @谷歌趋势-关键词趋势信息 | 关键词趋势 |
| **1688** | @店雷达-1688商品榜单 | 1688 商品排行 |
| **1688** | @店雷达-1688选品库 | 1688 选品采购 |
| **检索** | @网页检索 | 实时网页搜索 |
| **检索** | @网页解析 | URL 内容提取 |
| **TikTok** | @EchoTik-TikTok新品榜 | TikTok 新品排行 |
| **TikTok** | @EchoTik-TikTok商品搜索 | TikTok 商品搜索 |
| **Walmart** | @walmart前端-商品列表 | Walmart 商品搜索 |
| **eBay** | @ebay前端-商品列表 | eBay 商品搜索 |
| **专利** | @智慧芽-专利图像检索 | 外观专利图片搜索 |
| **专利** | @睿观-外观专利检测 | 外观专利侵权检测 |
| **专利** | @睿观-版权检测 | 版权检测 |
| **专利** | @睿观-图形商标检测 | 图形商标检测 |
| **专利** | @睿观-文本商标检测 | 文本商标检测 |
| **专利** | @睿观-发明专利检测 | 发明专利检测 |
| **专利** | @睿观-政策合规检测（纯图检测） | 政策合规检查 |
| **AI** | @按商品主图相似度分组 | 主图相似度分组 |
| **AI** | @分析商品主图 | 主图分析 |
| **AI** | @对商品标题进行分词 | 标题分词 |
| **沙箱** | @智能数据查询 | 动态数据聚合 |
| **沙箱** | @excel内容提取并分析 | Excel 分析 |
| **沙箱** | @Python沙箱 | Python 代码执行 |
| **沙箱** | @智能Excel处理 | 智能 Excel 处理 |

## 参考文档

每个工具分类的详细参数说明在 `references/` 目录下：
- `references/keepa.md` — Keepa 3 个工具
- `references/amazon-frontend.md` — 亚马逊前台 5 个工具
- `references/sif.md` — Sif 4 个工具
- `references/seller-sprite.md` — 卖家精灵 2 个工具
- `references/jimu.md` — 极目 3 个工具
- `references/google-trends.md` — 谷歌趋势 2 个工具
- `references/tiktok.md` — TikTok 2 个工具
- `references/walmart.md` — Walmart 1 个工具
- `references/ebay.md` — eBay 1 个工具
- `references/1688.md` — 1688 2 个工具
- `references/patent.md` — 专利检索 7 个工具
- `references/ai-tools.md` — AI 工具 3 个
- `references/sandbox.md` — 沙箱 4 个工具
- `references/web-search.md` — 网页检索 2 个工具

当你需要某个工具的具体参数格式时，读取对应的 reference 文件。

## 常用任务模板

### 市场分析
```
1、@卖家精灵-选产品 筛选亚马逊美国站的 "usb charger cable"，返回40条商品数据
2、@智能数据查询 按品牌、评分、价格段统计月销量、月销售额、占比
3、生成市场分析报告
```

### 评论挖掘
```
@亚马逊-商品评论 @亚马逊前端-商品详情 亚马逊美国站，asin为B00163U4LK的详情及每个星级各100条评论
总结：人群特征、使用场景、未被满足的需求、好评、差评、购买动机，给出改良建议
```

### 竞品 Listing 优化
```
1）查询竞品 ASIN 的商品详情
2）查询每个 ASIN 的关键词
3）构建关键词价值打分表
4）生成符合 FABE 法则和 Amazon COSMO 算法的五点描述
```

### 视觉市场分析
```
1、@亚马逊前台模拟搜索工具 搜索美国站 necklaces for women，第一页
2、统计不同挂件形状的销售额占比
3、用精美 HTML 网页展示数据
```

## 错误处理

如果工具调用失败，返回中会有错误详情。常见问题：
- 参数超出范围（检查 min/max 约束）
- 格式不对（检查 regex 模式）
- 单次任务工具数超过 10 个

---
> Source: [jiafar/linkfoxagent-claudecode](https://github.com/jiafar/linkfoxagent-claudecode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
