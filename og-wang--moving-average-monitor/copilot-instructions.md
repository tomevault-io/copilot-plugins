## moving-average-monitor

> > 本文档面向 AI 助手，快速了解项目架构、核心逻辑和常见修改点。

# AGENT.md - AI 开发者项目指南

> 本文档面向 AI 助手，快速了解项目架构、核心逻辑和常见修改点。

---

## 📋 项目概述

**均线趋势观测工具** - 每日生成市场趋势观测表，按 20 日均线偏离率排序，观察各标的强弱与趋势状态切换。

- **技术栈**: Python + Flask + SQLite + akshare/yfinance
- **数据源**: 新浪/腾讯/中证官方/东财/同花顺/雅虎财经（多源接力）
- **部署**: 本地运行，Windows 任务计划程序定时更新
- **存储**: 仅存计算结果快照（SQLite），底层行情不落盘

---

## 🏗️ 架构分层

```
用户界面 (Flask Web)
    ↓
编排层 (pipeline.py) ← 配置 (watchlist.json)
    ↓
数据源路由层 (datasource.py) → 9类数据源（新浪/腾讯/中证/东财/同花顺/雅虎等）
    ↓
计算层 (compute.py) → 指标计算 (MA20/偏离率/穿越点/区间涨幅)
    ↓
持久化层 (snapshot.py) → SQLite (data/snapshots.db)
```

---

## 📂 核心文件与职责

### 后端核心

| 文件 | 职责 | 关键函数/类 |
|------|------|------------|
| **`src/pipeline.py`** | 编排层：读配置→取数→计算→排序→存快照 | `run()` - 主流程入口<br>`cutoff_date()` - 计算参考交易日<br>`_reference_date()` - 美股为准的参考日 |
| **`src/datasource.py`** | 数据源路由：9类源接力、重试、代理切换、盘后当日补齐 | `fetch_close_series()` - 统一取数入口<br>`prewarm_ths()` - 预热同花顺token<br>`yf_proxy_env()` - 雅虎代理环境<br>`_fetch_hk_rt_today()` - 港股实时口补当日<br>`_fetch_ths_direct_today()` - 同花顺当日口补当日 |
| **`src/compute.py`** | 指标计算：MA20/偏离率/穿越点/排序 | `compute_metrics()` - 计算单个标的指标<br>`rank_by_deviation()` - 组内排序 |
| **`src/snapshot.py`** | SQLite快照持久化、排序变化、休市标注 | `save_snapshot()` - 保存当日快照<br>`load_snapshot()` - 读取指定日期快照<br>`prev_date()` - 前一交易日 |
| **`src/app.py`** | Flask界面：结果页/配置页/API | `/run` - 触发更新<br>`/api/guess` - 数据源猜测 |
| **`src/resolver.py`** | 新增标的的数据源自动识别 | `guess()` - 根据代码猜测source/fetch_id |

### 前端

| 文件 | 职责 |
|------|------|
| `src/templates/result.html` | 结果页：两张表 + 日期下拉 + 获取数据/智能补缺按钮 |
| `src/templates/config.html` | 配置页：增删监控标的、分组管理 |
| `src/templates/base.html` | 基础模板：样式/导航栏 |

### 配置与文档

| 文件 | 说明 |
|------|------|
| `config/watchlist.json` | 监控清单：分组、标的代码、数据源配置（本地个人配置，不入库；首次运行自动从 `watchlist.example.json` 复制） |
| `config/watchlist.example.json` | 入库的范例配置：同时是能跑通的默认清单和各类数据源的配置范例 |
| `data/snapshots.db` | SQLite数据库：存储每日计算结果快照 |
| `docs/ths_industry_names.md` | 同花顺行业板块标准名清单 |
| `run_daily.py` | 定时任务脚本（挂Windows任务计划程序） |

---

## 🔑 核心逻辑详解

### 1. 日期口径（以美股为准）

**关键概念：参考交易日** = 最近一个**已收盘的美股交易日**

- 美股是全球最晚收盘（美东 16:00），故以此为全表统一基准
- 用**美东时区**判断：`美东时间 < 16:05` → 取昨天，否则取今天
- 所有市场数据都截到 `≤ 参考日` 的最后一根收盘
- 快照以【参考交易日】为主键（同日多次运行互相覆盖）

**代码位置：** `pipeline.py` 的 `cutoff_date()` 和 `_reference_date()`

---

### 2. 三种运行模式

#### 模式A：全量抓取 `force_all=True`
- 忽略数据库缓存，强制重新联网抓取所有标的
- 适用场景：盘中运行后盘后更新到收盘价

#### 模式B：智能补缺 `retry_failed_only=True`
- 只联网重抓：**失败的 + 新加入的 + 数据过期的**
- 已成功的标的直接沿用数据库缓存，不再联网
- 适用场景：日常更新、快速修复失败项

#### 模式C：实时模式 `today_mode=True`
- **参考日期**：以【实际抓到的非美股数据】为锚——取非美股标的最新 bar 日期的最大值（封顶为北京时间今天）。A股收盘后运行=今天；凌晨0点后/周末/假日运行会自动落回最近一个真实交易日，不会生成"以无交易日历日为键"的错位快照
- **美股标的**：直接从数据库读昨日数据（**不调用API**）
- **非美股标的**：正常联网抓取当日数据
- **港股特殊处理**：尝试联网，失败时回退到昨日缓存
- **实时模式 + 智能补缺**：非美股标的一律视为"数据过期"，强制重新抓取
- 适用场景：A股收盘后（下午3点后），想立即看当日数据，不等美股收盘

**代码位置：** `pipeline.py` 的 `run()` 函数，约130-305行

---

### 3. 前端按钮功能（已调换）

**UI布局：** `[ 获取数据(蓝) ]  [ 智能补缺(灰) ]  ☑ 不等美股收盘`

| 按钮/开关 | 后端参数 | 功能 |
|----------|---------|------|
| **"获取数据"** | `force_all=True` | 全量抓取（主按钮） |
| **"智能补缺"** | `retry_failed_only=True` | 只补缺失败/新增/过期的 |
| **"不等美股收盘"** | `today_mode=True` | 实时模式：美股用缓存，非美股抓实时 |

**组合效果：**

| 实时模式 | 点击按钮 | 效果 |
|---------|---------|------|
| ❌ 关闭 | 获取数据 | 全市场全量抓取 |
| ❌ 关闭 | 智能补缺 | 全市场智能补缺 |
| ✅ 开启 | 获取数据 | **美股用缓存** + 非美股全量抓取 |
| ✅ 开启 | 智能补缺 | **美股用缓存** + 非美股智能补缺（强制重抓） |

**代码位置：**
- 前端：`src/templates/result.html` 第29-30行（按钮HTML）、132-133行（点击事件）
- 后端：`src/app.py` 第150-164行（`/run` 路由）

---

### 4. 数据过期判断（实时模式关键修改）

**判断逻辑：** `pipeline.py` 第191-197行

```python
outdated = {gc for gc, r in existing_by_code.items()
           if not r.get("error") and r.get("data_date")
           and (dt.date.fromisoformat(r["data_date"]) < ref_date
                or (today_mode and code_to_source.get(gc) != "us"))}
                # ↑ 核心：实时模式下，非美股一律视为过期
```

**为什么需要这个修改？**

- 实时模式下，盘中数据会实时变化（如A股收盘前后价格不同）
- 如果仅用 `data_date < ref_date` 判断，当 `data_date = today` 时，`today < today` 为 False
- 导致智能补缺模式不会重新抓取，卡在盘中价
- **解决方案**：实时模式 + 智能补缺时，强制将所有非美股标的标记为"过期"

---

### 5. 多源接力（避免东财限流）

**问题背景：** 东方财富按IP限频，密集请求会被拉黑

**解决方案：** A股/中证指数采用四级接力

```
新浪 (快、不限流) 
  → 腾讯 
    → 中证官方 (权威、非东财) 
      → 东财 (仅兜底)
```

**特殊处理：**
- 中证红利/新能源等：新浪会返回多年前旧数据，经**新鲜度校验**自动跳到腾讯
- 中证2000/机器人/房地产：新浪腾讯都没有，直接走中证官方

**代码位置：** `datasource.py` 第150-270行（`_fetch_em_index`）

---

### 6. 雅虎源代理处理

**问题：** 雅虎财经在国内被墙，需要代理；但其他源不需要代理

**解决方案：**

1. **环境隔离**：`yf_proxy_env()` 上下文管理器临时切换代理环境
2. **分阶段抓取**：pipeline 先并行抓直连源，抓完再单独抓 yf 源（串行）
3. **自管会话**：用 yfinance 的 session 管理，正确处理 Yahoo 的 cookie/crumb

**代码位置：**
- `datasource.py` 第345-363行（`yf_proxy_env`）
- `pipeline.py` 第224-244行（分阶段抓取）

---

### 7. 盘后当日补齐机制

**问题背景：** 历史日线接口盘后当天往往尚未追加当日收盘 bar，导致当日数据缺失

**解决方案：** 用实时口补齐当日一根 OHLC

| 市场 | 实时口 | 函数 | 代码位置 |
|------|--------|------|---------|
| 港股指数 | 新浪 `hq.sinajs.cn/list=rt_hk{symbol}` | `_fetch_hk_rt_today()` | `datasource.py` 第225-257行 |
| 同花顺行业/概念 | 同花顺 `today.js` | `_fetch_ths_direct_today()` | `datasource.py` 第419-460行 |
| 同花顺板块直连 | 同花顺 `today.js` | `_fetch_ths_direct_today()` | 同上 |

**核心逻辑：**
- 历史口取到后，比较最新 bar 日期与当日
- 若历史口最新日期 < 当日，用实时口覆盖当日这一根
- 实时口失败不影响历史结果（异常不抛出）

**代码位置：**
- `datasource.py` 第225-257行（港股补齐）
- `datasource.py` 第274-282行（港股补齐集成）
- `datasource.py` 第293-307行（同花顺补齐通用函数）

---

## 🛠️ 常见修改场景

### 场景1：添加新的数据源

**步骤：**

1. **在 `datasource.py` 添加新的 fetch 函数**
   - 函数签名：`def _fetch_xxx(fetch_id: str) -> pd.Series`
   - 返回：日期索引的收盘价序列
   - 处理异常、返回标准化格式

2. **在 `fetch_close_series()` 路由表添加分支**
   ```python
   elif source == "new_source":
       return _fetch_new_source(fetch_id)
   ```

3. **在 `resolver.py` 添加自动识别规则**（可选）
   - 修改 `guess()` 函数，添加代码模式匹配

4. **在配置页下拉菜单添加选项**
   - `src/app.py` 第32-42行（`SOURCES` 列表）

---

### 场景2：修改指标计算逻辑

**位置：** `src/compute.py`

- `compute_metrics()` - 单个标的的指标计算
- `rank_by_deviation()` - 组内排序逻辑

**注意：** 修改后需要重新运行 `force_all=True` 刷新所有快照

---

### 场景3：调整UI样式/按钮

**前端模板：** `src/templates/`

- `result.html` - 结果页（表格、按钮、配色）
- `config.html` - 配置页（表单）
- `base.html` - 全局样式（CSS变量、导航栏）

**配色相关：** `result.html` 第95-107行（Jinja2 过滤器）

---

### 场景4：修改参考日期逻辑

**核心函数：** `pipeline.py`

- `cutoff_date()` - 判断最迟可能交易日
- `_reference_date()` - 计算参考交易日（以美股为准）
- `current_ref_date()` - 轻量确定当前参考日（用于读库判断）

**美股收盘时间：** 第62行 `US_CLOSE = dt.time(16, 5)` （美东16:05）

---

### 场景5：添加新指标列

**步骤：**

1. **`compute.py`** - 在 `compute_metrics()` 添加新指标计算
2. **`pipeline.py`** - 在 `_METRIC_KEYS` 添加新字段名（第29行）
3. **`snapshot.py`** - 在 `_COLUMNS` 添加列定义（第20行），并修改建表SQL（第32行）
4. **`result.html`** - 在表头和表体添加新列显示

---

## 🐛 常见问题排查

### 问题1：某个标的抓取失败

**排查步骤：**

1. 查看错误信息（结果页会显示具体错误）
2. 检查 `datasource.py` 对应源的 fetch 函数
3. 手动测试 akshare API 是否返回数据
4. 检查是否需要切换数据源（如东财→新浪）

---

### 问题2：实时模式下数据没有更新

**可能原因：**

1. **今天是周末/假日** - 非24小时交易的市场没有数据（正常现象）
2. **数据过期判断失效** - 检查 `pipeline.py` 第188-193行的 `outdated` 逻辑
3. **美股缓存没有昨日数据** - 检查 `snapshot.prev_date()` 是否返回有效日期

---

### 问题3：被东财限流/IP拉黑

**解决方案：**

1. **检查是否用了东财源** - 理论上A股已切到新浪/腾讯，不应该频繁敲东财
2. **等待一段时间** - IP限流通常几分钟后自动解除
3. **使用 `force_all=False`** - 用智能补缺模式减少请求

---

### 问题4：雅虎源（境外指数）抓取失败

**排查步骤：**

1. **检查本地代理是否启动** - 默认 `http://127.0.0.1:7890`（Clash Verge）
2. **测试代理连通性** - 浏览器设置代理后访问 Google
3. **检查环境变量** - 可用 `MA_YF_PROXY` 自定义代理地址
4. **查看 yfinance 版本** - 可能需要更新：`pip install -U yfinance`

---

## 📊 数据流示意图

```
用户点击"获取数据"
    ↓
app.py /run 路由 (force_all=True)
    ↓
pipeline.run() 主流程
    ↓
├─ 判断运行模式 (force_all / retry / today_mode)
├─ 读取配置 (watchlist.json)
├─ 实时模式：加载昨日快照（美股用）
├─ 智能补缺：识别需要抓取的标的
    ↓
├─ 并行取数阶段1：直连源（新浪/腾讯/东财/同花顺等）
├─ 并行取数阶段2：雅虎源（单独、走代理）
    ↓
├─ 确定参考交易日 (以美股为准)
├─ 计算指标 (compute.compute_metrics)
│   ├─ MA20
│   ├─ 偏离率
│   ├─ 穿越点
│   └─ 区间涨幅
    ↓
├─ 实时模式：填充美股昨日数据
├─ 组内排序 (compute.rank_by_deviation)
├─ 保存快照 (snapshot.save_snapshot)
    ↓
返回结果 → 前端刷新页面
```

---

## 🔐 安全与限流注意事项

1. **不要频繁请求东财API** - 会被拉黑IP（几分钟到几小时）
2. **雅虎源已用 yfinance session** - 避免 429 限流
3. **并行线程数已限制** - `MAX_WORKERS = 8`（`pipeline.py` 第27行）
4. **重试机制** - 各数据源已内置指数退避重试

---

## 📝 测试建议

### 单元测试（手动）

```python
# 测试单个数据源
from src import datasource
df = datasource.fetch_close_series("em_index", "399006")
print(df.tail())

# 测试指标计算
from src import compute
metrics = compute.compute_metrics(df)
print(metrics)

# 测试智能补缺
from src import pipeline
result = pipeline.run(retry_failed_only=True)
print(result)
```

### 数据可用性测试

运行 `test_data_availability.py` 检查各数据源能否获取到最新数据：

```bash
python test_data_availability.py
```

---

## 🎯 开发优先级建议

**高优先级（影响核心功能）：**
1. 数据源失效修复（如API变更、接口下线）
2. 日期逻辑bug（影响所有数据对齐）
3. 实时模式/智能补缺的缓存逻辑

**中优先级（提升体验）：**
1. 新增数据源或标的
2. UI/样式优化
3. 新增指标列

**低优先级（锦上添花）：**
1. 性能优化（当前10-20秒已可接受）
2. 日志系统
3. 单元测试覆盖

---

## 📚 相关文档

- **用户文档**: `README.md` - 用户快速开始指南
- **同花顺板块名**: `docs/ths_industry_names.md` - 新增行业板块时参照
- **依赖清单**: `requirements.txt` - Python包依赖

---

## 🤝 AI 助手协作建议

1. **修改前先阅读本文档** - 了解核心逻辑和常见坑
2. **测试数据源前先运行** `test_data_availability.py` - 确认API可用性
3. **涉及日期逻辑修改** - 务必测试周末/假日/跨时区场景
4. **修改指标计算** - 提醒用户用 `force_all=True` 刷新历史快照
5. **添加新数据源** - 注意东财限流问题，优先用新浪/腾讯
6. **修改前端按钮** - 前后端参数对应关系见"前端按钮功能"章节

---

**最后更新**: 2026-06-18
**版本**: v1.0
**维护者**: AI Agent 协作文档

---
> Source: [OG-Wang/moving-average-monitor](https://github.com/OG-Wang/moving-average-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
