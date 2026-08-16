## deepseek-harness-quant

> > 本文件由 DeepSeek HARNESS 的 agent-instructions 自动加载，所有会话必须遵守。

# DSHQuant 工作区规则（AGENTS.md）

> 本文件由 DeepSeek HARNESS 的 agent-instructions 自动加载，所有会话必须遵守。
> 优先级：本文 > 具体任务指令 > 个人偏好。

## 架构铁律：全部模块动态化

**所有模块必须是动态的：用 API 接入通讯、动态数据驱动渲染，禁止把数据/列表/配置写死在代码里。**

具体约束：

1. **前端页面**：一律通过 `LW.api.get('/api/...')` 拉取数据渲染；禁止在 HTML/JS 里硬编码业务数据（股票列表、策略列表、候选池、评分、状态）。
2. **后端模块**：任何"会变的东西"（策略注册表、ETF 候选池、牛散 persona 元数据、菜单项、阈值、资产清单）必须放在**配置文件**（`config/*.yaml`）或**数据文件/数据库**中，代码只做读取与分发；提供对应 API 供前端消费。
3. **配置模板**：新增配置一律同时提供 `.example` 模板（开源包要求）；缺省值合理兜底，缺失不崩溃。
4. **通讯方式**：页面 ↔ 后端 = HTTP API；后端模块间 = 直接函数调用或服务接口；禁止前端直连数据库。
5. **例外**：性能缓存（如回测价格面板缓存）可用内存缓存；纯展示样式（CSS 颜色、动画）可写死。
6. **新增模块**：落地前先确认数据从哪来（API/配置/文件），先建数据通道再建 UI。

## 数据边界（不可违反）

- 行情数据来自第三方（Tushare 等），**禁止再分发**；只开源代码 + 获取脚本 + 合成演示数据。
- 换手率（turn）2019 年前缺失——**2019 前换手类结论一律作废**。
- 回测口径：T+1 开盘执行、一字板过滤、成本模型、结论分级——见 `.dsh/skills/backtest-acceptance`。

## 工作方式

- 低频主观量化定位；买入指令 = L2 决策卡片聚合（同源，无第二套推荐逻辑）。
- 主仓位 turn_low 分散，pitch 高分仅卫星仓；牛散人格输出不构成投资建议。
- 改动后必须回归验证（页面 200 / API 200 / 语法检查）。

---
> Source: [yuanwang589-dev/deepseek-harness-quant](https://github.com/yuanwang589-dev/deepseek-harness-quant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
