## xueqiu-skills

> **最新提交**: 756179c Initial commit

# xueqiu-skills 项目知识库

**生成时间**: 2026-03-06  
**最新提交**: 756179c Initial commit  
**分支**: main

## 项目概览

轻量级 AI 技能仓库，专为 AI Agent 协作设计。核心功能：爬取雪球投资时间线，AI 分析观点并生成 PDF 报告。

**技术栈**: Python 3.10+ (uv) + Playwright (agent-browser) + Markdown/PDF 转换

## 仓库结构

```
xueqiu-skills/
├── AGENTS.md              # 本文件，AI 编程工具指引
├── README.md              # 人类用户文档
├── LICENSE
└── skills/
    └── crawl-xueqiu-my-timeline/
        ├── SKILL.md              # 技能详细文档（AI 主要参考）
        ├── assets/
        │   └── github-markdown.css
        ├── evals/
        │   └── evals.json
        └── scripts/
            ├── check-cdp.sh
            ├── check-agent-browser.sh
            └── crawl_xueqiu_home_timeline_api.py
```

## 模块说明

| 模块 | 职责 |
|------|------|
| `skills/crawl-xueqiu-my-timeline/` | 雪球时间线爬取技能（唯一技能） |
| `scripts/` | 可执行脚本（环境检查 + 爬取主逻辑） |
| `assets/` | PDF 样式文件 |
| `evals/` | 技能评估测试用例 |

## 代码地图

### 入口点

| 文件 | 作用 |
|------|------|
| `scripts/crawl_xueqiu_home_timeline_api.py` | 主入口，`main()` 函数协调爬取、解析、输出 |

### 关键函数

| 函数 | 位置 | 作用 |
|------|------|------|
| `main()` | `crawl_xueqiu_home_timeline_api.py:273` | 解析 CLI 参数，调度爬取流程 |
| `fetch_timeline_in_range()` | `crawl_xueqiu_home_timeline_api.py:167` | 分页爬取时间线，返回原始帖子列表 |
| `parse_status()` | `crawl_xueqiu_home_timeline_api.py:139` | 解析单条帖子数据（内容、时间、引用） |
| `group_by_author()` | `crawl_xueqiu_home_timeline_api.py:218` | 按发言人分组，生成 Markdown |
| `get_home_timeline()` | `crawl_xueqiu_home_timeline_api.py:49` | 浏览器 API 调用 fetch |
| `open_xueqiu()` | `crawl_xueqiu_home_timeline_api.py:32` | 打开雪球首页并登录 |

## 命令速查

```bash
# 环境检查
sh skills/crawl-xueqiu-my-timeline/scripts/check-cdp.sh
sh skills/crawl-xueqiu-my-timeline/scripts/check-agent-browser.sh

# 爬取最近 24 小时
uv run skills/crawl-xueqiu-my-timeline/scripts/crawl_xueqiu_home_timeline_api.py

# 爬取最近 N 小时
uv run skills/crawl-xueqiu-my-timeline/scripts/crawl_xueqiu_home_timeline_api.py --hours N

# 爬取最近 N 天
uv run skills/crawl-xueqiu-my-timeline/scripts/crawl_xueqiu_home_timeline_api.py --days N

# 指定日期范围
uv run skills/crawl-xueqiu-my-timeline/scripts/crawl_xueqiu_home_timeline_api.py \
  --start-date 2026-03-01 --end-date 2026-03-06
```

## 技能触发条件

当用户提及以下关键词时，**立即触发** `crawl-xueqiu-my-timeline` 技能：

✅ **触发关键词**：
- 雪球时间线、雪球关注、雪球大 V
- 投资要闻、市场热点、投资热点
- 投资分析报告、生成 PDF 报告
- 爬取雪球、整理雪球观点

❌ **不触发**：
- 打开雪球网站（简单浏览器操作）
- 查看某只股票行情（单一数据查询）
- 登录雪球（简单网页操作）

## 执行流程

### 步骤 1：环境检查

```bash
sh skills/crawl-xueqiu-my-timeline/scripts/check-cdp.sh
sh skills/crawl-xueqiu-my-timeline/scripts/check-agent-browser.sh
```

如未安装，脚本会自动安装。脚本生成：`home_timeline_YYYYMMDD_YYYYMMDD.md`

### 步骤 2：执行爬取

```bash
uv run skills/crawl-xueqiu-my-timeline/scripts/crawl_xueqiu_home_timeline_api.py
```

如需指定时间范围，添加参数（见 README.md）。

### 步骤 3：创建 TODO 列表

读取生成的 Markdown 文件，统计每位发言人的发言数量，按**单个用户的发言条数**分配 TODO：

| 用户发言数量 | TODO 分配方式 | 示例 |
|-------------|-------------|------|
| **>10 条** | 独占一个 TODO | @A(15 条) → TODO: 分析@A |
| **5-10 条** | 3-5 个用户合并一个 TODO | @B(7 条)+@C(6 条)+@D(5 条) → TODO: 分析@B/@C/@D |
| **<5 条** | 所有用户合并一个 TODO | @E(3 条)+@F(2 条)+@G(1 条) → TODO: 分析@E/@F/@G |

使用 `todowrite` 工具创建 TODO 列表。

### 步骤 4：启动 Subagent 分析

**并发限制**：同时运行的 subagent 不超过 3 个

每个 subagent 的任务：
1. 读取时间线文件，定位指定用户的发言
2. 只总结本人观点（排除 `//` 或 `>` 开头的内容）
3. 提取：核心观点、涉及标的（`$xxx$` 格式）、情绪倾向
4. 返回分析结果（不写中间文件）

**重试机制**：失败时自动重试，最多 2 次

### 步骤 5：生成分析报告

整合所有 subagent 的结果，写入 Markdown 格式报告（保存为 `雪球时间线_YYYYMMDD_YYYYMMDD.md`）：

```markdown
# 雪球关注时间线分析报告

## 时间范围
2026-03-05 至 2026-03-06

## 发言人统计
- @产业驱动博弈：19 条
- @管我财：6 条

## 发言观点总结

### @发言人 A
- **发言数量**: X 条
- **主要观点**:
  - 观点 1
  - 观点 2
- **涉及标的**: $XXX$, $YYY$
- **情绪倾向**: 乐观/中性/悲观

## 市场热点 TOP3
1. 热点主题 1 - 讨论人数 X
2. 热点主题 2 - 讨论人数 X
3. 热点主题 3 - 讨论人数 X

## 投资建议/风险提示
...
```

### 步骤 6：转换为 PDF

```bash
bunx mdpdf 雪球时间线_YYYYMMDD_YYYYMMDD.md \
  --style=skills/crawl-xueqiu-my-timeline/assets/github-markdown.css
```

### 步骤 7：清理临时文件

```bash
rm 雪球时间线_YYYYMMDD_YYYYMMDD.md
```

## 重要规则

### 观点归属判断

只有**不以** `//` 或 `>` 开头的内容才是发言人自己的观点：

| 标记 | 含义 | 是否总结 |
|------|------|---------|
| 无标记行 | 发言人本人的发言 | ✅ 需总结 |
| `//` 开头 | 被评论的其他用户发言 | ❌ 非本人观点 |
| `>` 开头 | 被引用的转发内容 | ❌ 非本人观点 |

**注意**：`回复@XXX：` 前缀是雪球自动生成的，整行都是发言人自己的观点，应总结。

### 报告核心原则

- ✅ 必须包含所有发言人，即使只有 1 条发言
- ✅ 按发言数量降序排列
- ✅ 只总结本人观点
- ✅ 分析失败的发言人在报告中标注"⚠️ 分析失败"

## 错误处理

| 错误 | 处理方式 |
|------|---------|
| Chrome Debug 模式未启动 | 运行 check-cdp.sh 自动启动 |
| agent-browser 未安装 | 运行 check-agent-browser.sh 自动安装 |
| 雪球验证页面 | 提示用户手动完成验证后重试 |
| Subagent 分析失败 | 自动重试 2 次，仍失败则标注"⚠️ 分析失败" |
| mdpdf 转换失败 | 检查样式文件路径，保留 Markdown 文件供用户查看 |

## 环境适配

根据当前环境选择工具：

- **OpenCode**: 使用 `task` 启动 subagent，`todowrite` 管理 TODO
- **Claude Code**: 使用适合的方式启动 subagent
- 其他环境：遵循相同逻辑，适配对应工具

## 完整技能文档

详细技能说明：[skills/crawl-xueqiu-my-timeline/SKILL.md](skills/crawl-xueqiu-my-timeline/SKILL.md)

---
> Source: [CNife/xueqiu-skills](https://github.com/CNife/xueqiu-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
