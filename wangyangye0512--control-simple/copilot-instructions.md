## control-simple

> 项目架构


# Tiny Orchestrator — Architecture (Telegram 按钮增强版)

## 目标

- 在 Telegram 指定群组的**某个 Topic**内，用**中文按钮与卡片**操作两台 Freqtrade 实例（long、short）完成：
  - 查看/设置“篮子”
  - 一键开多（确认按钮）
  - 一键反向开空（先平多→确认→再开空）
  - 一键全平（确认按钮）
  - 查看状态与最近一次结果
- **不引入** 数据库/复杂状态机/外部 Web 服务；只在现有 Tiny-MVP 基础上增加 **InlineKeyboard** 与 **CallbackQuery** 回调。

---

## 目录结构

orchestrator/
├─ bot.py                 # 主程序（Telegram命令 + 中文按钮UI + 回调 + Freqtrade客户端）
├─ config.yml             # 基础配置（Telegram、两实例、默认stake/延时、权限）
├─ watchlist.yml          # 篮子文件（对称标的列表）
└─ runtime/
└─ audit.log           # 轻量审计（不存在时自动创建）

---

## 配置样例（config.yml）

```yaml
telegram:
  token: "xxxxx:yyyyy"
  chat_id: -1001234567890        # 你的群ID
  topic_id: 4567                 # 目标Topic的 message_thread_id
  admins: [12345678, 23456789]   # 允许交易的用户ID
  require_arm: true              # 是否需要“武装”才允许执行
  arm_ttl_minutes: 15            # 武装有效期（分钟）

freqtrade:
  long:
    base_url: "http://10.0.0.11:8080/api/v1"
    user: "admin"
    pass: "secret"
  short:
    base_url: "http://10.0.0.12:8080/api/v1"
    user: "admin"
    pass: "secret"

defaults:
  stake: 200
  delay_ms: 300
  poll_timeout_sec: 60           # 反向时等待清仓的超时
  poll_interval_sec: 2
````

### 篮子（watchlist.yml）

```yaml
basket: ["BTC/USDT", "ETH/USDT", "SOL/USDT"]
```

---

## 核心组件

### 1) Telegram 命令入口（中文）

- 只在 `chat_id + topic_id` 生效；交易类命令仅 `admins` 可用。
- 命令清单：

  - `/basket`：显示当前篮子与参数（中文面板）
  - `/basket_set BTC/USDT ...`：覆盖设置篮子（校验：大写、去重、`BASE/QUOTE`）
  - `/stake 200`：设置每笔名义（仅内存；可选落回 `config.yml`）
  - `/status`：显示探活（long/short）、最近一次摘要
  - `/go_long`：不立刻下单 → 发送“确认卡片（中文按钮）”
  - `/go_short`：不立刻下单 → 发送“反向执行确认卡片（中文按钮）”
  - `/flat`：不立刻平仓 → 发送“确认卡片（中文按钮）”
  - `/arm <口令>`：武装一段时间（如启用 require\_arm）

### 2) 中文按钮 & 面板（InlineKeyboard）

- **确认卡片**（开多 / 反向开空 / 全平）

  - 标题：`🧺 当前篮子：{N} 个 | 每笔：{stake} USDT | 延时：{delay_ms}ms`
  - 二级提示（反向开空）：`⚠️ 将先平多端仓位（至 0 或超时），再在空端批量开空。`
  - 按钮（示例）：

    - 行1：`✅ 立即开多` / `❌ 取消`
    - 行2（可选快速参数）：`每笔 100`、`每笔 200`、`每笔 300`
    - 行3（可选快速参数）：`延时 200ms`、`延时 300ms`、`延时 500ms`
- **篮子面板（可选）**

  - 预设按钮：`🚀 主动强势 8`、`🛡️ 稳健大盘 8`、`🧪 实验 8`、`🧹 清空篮子`
  - 逐项移除：`BTC/USDT 【移除】`（回调后更新并回显）

### 3) CallbackQuery 回调

- 回调统一走 `on_callback_query()`，解析 `callback_data`：

  - 结构建议：`<OP>|<ACTION>|<OP_ID>|<KV...>`

    - 例：`GL|CONFIRM|1724130000|stake=200;delay=300`（Go Long）
    - 例：`GS|SET_STAKE|1724130000|stake=300`
    - 例：`GS|CONFIRM|1724130000|stake=300;delay=200`
    - 例：`FLAT|CONFIRM|1724130001|`
- 回调校验：

  - `user_id ∈ admins` 且 消息 `message_thread_id == topic_id`
  - 若 `require_arm=true`：检查武装未过期
  - 幂等：同一 `OP_ID` 的 `CONFIRM` 只执行一次

### 4) Freqtrade 迷你客户端（HTTP）

- 方法：

  - `list_positions()`（404→`[]`）
  - `cancel_open_orders()`
  - `forcebuy(pair, stake_amount)`
  - `forceshort(pair, stake_amount)`
  - `forcesell(pair)`（或基于 trade\_id）
  - `forcecover(pair)`（或基于 position\_id）
- 端点命名兼容：对多个常见路径轮询尝试，先成功者为准。
- 请求间隔：相邻操作 `sleep(delay_ms/1000)`。

### 5) 轻量审计（NDJSON）

- `runtime/audit.log` 行级记录：

  - 命令开始/结束（`cmd, user, summary`）
  - 每笔动作（`cmd, pair, side, action, result, message`）
- 错误不阻塞整体；汇总时输出失败摘要（前 1–2 条）。

---

## 交互流程（按钮增强）

### A. `/go_long` → 确认卡片 → 执行

1. 用户发送 `/go_long`（仅管理员 & 指定 Topic）。
2. 机器人发送确认卡片（按钮：`✅ 立即开多`、`❌ 取消`、快速参数组）。
3. 用户点击：

   - 若点**参数按钮**：仅改变本次 `op` 的临时参数，不执行；卡片标题实时更新（编辑消息）。
   - 若点\*\*✅ 立即开多\*\*：生成 `op_id` → 逐个 `forcebuy(pair, stake)`（串行 + delay）。
4. 执行中：在**同一主题**线程化输出进度：

   - `[1/8] BTC/USDT → ✅`
   - `[2/8] ETH/USDT → ❌（最小名义不足）`
5. 结束：汇总 `✅ 成功 {ok} / ❌ 失败 {err} | ⏱️ 用时 {sec}s`；写审计。

### B. `/go_short`（反向）

1. 发送反向**确认卡片**（含警告）：`⚠️ 将先平多端仓位…`
2. 用户点\*\*✅ 立即反向开空\*\* → 生成 `op_id`：

   - `long` 实例：`cancel_open_orders()` → `forcesell` 全部 → 轮询 `list_positions()` 直至 0 或超时（`poll_timeout_sec`，每 `poll_interval_sec` 一次）
   - `short` 实例：逐个 `forceshort(pair, stake)`（串行 + delay）
3. 输出进度与汇总；写审计。

### C. `/flat`（全平）

1. 确认卡片：`🧯 一键平仓（两侧）`
2. 点击\*\*✅ 确认\*\* → 生成 `op_id`：

   - `short` 实例：`cancel_open_orders()` → `forcecover` 全部
   - `long` 实例：`cancel_open_orders()` → `forcesell` 全部
3. 输出进度与汇总；写审计。

---

## 按钮/回调数据设计（示例）

- 开多（Go Long）

  - `GL|CONFIRM|<op_id>|stake=200;delay=300`
  - `GL|SET_STAKE|<op_id>|stake=100`
  - `GL|SET_DELAY|<op_id>|delay=500`
  - `GL|CANCEL|<op_id>|`
- 反向开空（Go Short）

  - `GS|CONFIRM|<op_id>|stake=200;delay=300`
  - `GS|SET_STAKE|<op_id>|stake=300`
  - `GS|SET_DELAY|<op_id>|delay=200`
  - `GS|CANCEL|<op_id>|`
- 全平（Flat）

  - `FLAT|CONFIRM|<op_id>|`
  - `FLAT|CANCEL|<op_id>|`

> 统一在 `on_callback_query()` 解析三段 `<OP>|<ACTION>|<OP_ID>|<KV…>`，KV 用 `;` 连接，`=` 赋值。

---

## 中文文案（可直接使用）

- **标题**：`🧺 当前篮子：{N} 个 | 每笔：{stake} USDT | 延时：{delay_ms}ms`
- **反向提示**：`⚠️ 将先平多端仓位（至 0 或超时），再在空端批量开空。`
- **按钮**：

  - `✅ 立即开多` / `✅ 立即反向开空` / `🧯 立即全平`
  - `❌ 取消`
  - `每笔 100`、`每笔 200`、`每笔 300`
  - `延时 200ms`、`延时 300ms`、`延时 500ms`
  - `📦 选择预设篮子`、`🧹 清空篮子`
- **进度**：

  - `开始：{cmd} | 篮子 {N} | 每笔 {stake} | 延时 {delay_ms}ms`
  - `[{i}/{N}] {pair} → ✅ 已完成` / `❌ 失败（{reason}）`
- **汇总**：

  - `✅ 成功 {ok} / ❌ 失败 {err} | ⏱️ 用时 {sec}s`
  - `失败摘要：\n- {pair}: {reason}\n- {pair}: {reason}`

---

## 安全与幂等

- 范围限制：仅 `chat_id + topic_id`；按钮回调也需二次校验。
- 权限：仅 `admins` 能触发交易类执行；`/basket`、`/status` 可放宽（可选）。
- 武装机制：`require_arm=true` 时，未武装或过期不显示“✅ 确认”按钮，或点击后提示 `⛔ 未武装，请发送 /arm <口令>`。
- 幂等：每次确认执行前生成 `op_id`；同一 `op_id` 的 `CONFIRM` 回调只处理一次（内存记忆或临时集合去重）。

---

## 错误处理与可观测

- 单笔失败不阻塞整体；在汇总中展示失败条目（最多2条摘要）。
- 审计 `audit.log`：

  - 命令开始/结束：`{"ts","cmd","user","summary"}`
  - 动作级：`{"ts","cmd","pair","side","action","result","message"}`
- `/status` 返回最近一次结果摘要（保存在内存：`last_summary`）。

---

## 依赖与运行

- 依赖（最小）：`python-telegram-bot`、`httpx` 或 `requests`、`pyyaml`
- 运行：

  1. 填写 `config.yml` 与 `watchlist.yml`
  2. 确保两台 Freqtrade 的 `api_server` 启用，且 long 支持 `forcebuy/forcesell`，short 支持 `forceshort/forcecover`
  3. `python bot.py`
  4. 在 Telegram 指定 Topic：

     - `/basket` 查看 → `/go_long` 发卡片 → 点 **✅ 立即开多**
     - `/go_short` → 点 **✅ 立即反向开空**
     - `/flat` → 点 **✅ 立即全平**

---

## 验收清单（按钮版）

- [ ] `/basket` 正确显示篮子；`/basket_set` 能更新并回显
- [ ] `/stake` 修改后在确认卡片与 `/status` 中生效
- [ ] `/go_long` 仅发卡片；点 **✅** 才执行；串行下单且有进度与汇总
- [ ] `/go_short` 发卡片；**先平多**轮询至 0 或超时，再开空；有进度与汇总
- [ ] `/flat` 发卡片；点 **✅** 后两侧取消挂单并清仓
- [ ] 非管理员或错误 Topic 的命令/按钮回调被拒绝（或无响应）
- [ ] `require_arm=true` 时未武装不允许执行；`/arm` 后在有效期内允许
- [ ] `runtime/audit.log` 有开始/动作/结束三类记录；失败项被记录

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WangyangYe0512)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/WangyangYe0512)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
