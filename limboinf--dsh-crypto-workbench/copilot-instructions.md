## dsh-crypto-workbench

> 合约实验室：主流币 USDT 永续的 AI 辅助交易工作台（受控执行，2026-09-11 用户拍板开放下单），以插件形式挂载到 DeepSeek Harness（dsh），

# dsh-crypto-workbench —— Agent 工作指引

合约实验室：主流币 USDT 永续的 AI 辅助交易工作台（受控执行，2026-09-11 用户拍板开放下单），以插件形式挂载到 DeepSeek Harness（dsh），
**不改动 dsh 核心代码**。技术方案：`docs/合约实验室技术实现方案-20260911.md`（实现即按它落地）；
产品边界：`docs/加密货币工作台可行性与MVP方案-20260911.md` §8.1–8.3。

- dsh 源码仓库（本机 clone，路径自行替换；需先 `pnpm install` + `pnpm run build:lib`）
- 插件开发官方文档：https://deepseek-harness.github.io/deepseek-harness/develop/basic
- 仓库根 README.md 有完整的接入/验证命令，改动部署相关内容时先同步读它

## 常用命令

```bash
npm run typecheck   # tsc --noEmit
npm test            # node --import tsx --test test/*.test.ts（node:test，无 vitest/jest）
npm run live-check  # 联网走 OKX 公共 REST：BTC/ETH 行情 + USDT-CNY 汇率（账户/消息需在 dsh 里验）
npm run build       # tsdown → lib/index.js（host）+ lib/client.js（client）
```

## 目录结构

```
src/index.ts             插件入口（export name/inject/apply）：装配传输（dsh MCP 主 + REST 兜底）、
                         OkxReadonlyClient、NewsAdapter、SnapshotBuilder、CryptoStore，再 registerCryptoTools
src/crypto-dto.ts        面板数据契约（host ⇄ client 唯一载荷）：纯类型 + 纯函数、零 import。
                         EXPERIMENT_PROFILE 固定实验参数；<!--crypto:kind …--> 标签编解码
src/risk.ts              风险引擎纯函数：指标公式、闸门阈值（GATE_THRESHOLDS）、合约规格
                         （CONTRACT_SPECS）、仓位计算 buildTradePlan（executable=风控是否通过）、planToInput 执行前重算
src/crypto-normalize.ts  OKX/消息原始 JSON → DTO：信封解析、账户/仓位/行情规范化、消息分类/等级/ID
src/okx-adapter.ts       适配层：ToolTransport（dsh ctx.tools.execute / 公共 REST / 兜底组合）、
                         Result<T,SourceError>、READONLY_TOOLS + OkxReadonlyClient、WRITE_TOOLS + OkxTradeClient
src/execute.ts           执行链路：三道门复核 → 设杠杆 → 限价单附止损/止盈 → 回执 → 成交 import → 执行日志；平仓
src/news-adapter.ts      消息源（OKX news / OKX 日历 / CRYPTO_NEWS_TOOL 外部源）+ 去重 + 时间线
src/opportunity.ts       机会雷达（只读扫描面）：24h/3d/7d 三窗全部同向才给方向，四因子评分
                         （趋势强度 35/动量质量 20/资金费健康度 25/区间位置 20）≥70 才显示；
                         扫描面宽（对话可配：crypto_radar_config → radar.json，允许全集
                         RADAR_ALLOWED_COINS，观察币种上限 8）执行面=同一份白名单（2026-09-12 拍板放开，BTC/ETH 固定在列）；
                         RADAR_REFRESH_MS=60s 独立计时，不随 5s 快照轮询重取（2026-09-12 重做）
src/jev-service.ts       Jev 决策建议层（2026-09-18 接入）：OpenRouter ~typesafe/jev-latest 的 choice
                         客户端（fetch+超时+fail-open）+ 5s 行情环形缓冲（内存不落盘）+ 语义桶
                         （趋势按 2σ 噪声带宽判档——数字只算一次，不喂裸价格序列）+ watch 状态机
                         （tick fire-and-forget、陈旧响应丢弃、conf<0.5 一律观望）；决策动作转变时
                         写 decision-logs（kind=note，tags 带 jev）；纯建议层，不进执行链
src/jin10-mcp.ts         金十标准 MCP HTTP 客户端：Bearer、握手/发现、session、SSE/JSON、structuredContent
src/jin10-service.ts     quote://codes 与报价/快讯/资讯/日历的聚合和字段规范化
src/jin10-dto.ts         金十 tab 独立载荷契约（零 import）与 <!--jin10:payload …--> 编解码
src/jin10-tools.ts       5 个 crypto_jin10_* 工具 + /jin10 影子命令
src/snapshot.ts          SnapshotBuilder：并行取数 + 分段 TTL 缓存 + 失败保留上一份 + 闸门叠加（快照只在内存流转，不落盘）
src/store.ts             本地仓储：experiment.json / daily / plans / fills / decision-logs，
                         原子写 + 记录校验 + JSONL fail-closed
src/crypto-render.ts     payload → markdown（模型/人读）；数字只在这里格式化一次
src/crypto-tools.ts      13 个工具（OKX/交易 + crypto_radar_config 雷达配置 + crypto_jev_watch Jev
                         监控开关，含 crypto_execute_plan / crypto_cancel_plan_order /
                         crypto_close_position） + /crypto（含 jev / jevchart 子命令）、
                         /crypto-activity 命令（裸 JSON-Schema）
src/env.ts               数据目录与 env 文件解析（含金十 URL / Bearer Token / 默认报价品种 / OpenRouter Key）
src/client/index.ts      client 入口：shell.overlay 面板、tool.call.toolview 工具行、input.left 快捷按钮
src/client/channel.ts    影子会话取数（executeCommandText）+ 对话联动（linkSend）+ ctx 最小类型
src/client/side-panel.ts 右侧停靠面板壳（拖拽调宽、开合记忆、对话列让位 CSS）
src/client/crypto-panel.ts 面板体七区（总览/仓位/委托/行情/闸门/消息/计划） + 消息线 OKX/金十双 tab
                         + 币种详情视图（点行情卡片进入：SVG 走势+决策点、语义桶、Jev 决策卡、
                         决策流水；Esc/返回退出；走势种子走 /crypto jevchart、监控开关走 /crypto jev）
src/client/jin10-panel.ts 金十 tab：切入时按需调用 /jin10，不参与 5s 轮询
skills/crypto-briefing/  简报规范（事实/推断/行动分段，数字只抄工具）
skills/crypto-risk/      风险审查规范（第二道闸：可否决，不可推翻引擎）
test/                    node:test；fixtures.ts 是 OKX 1.4.6 信封样本，helpers.ts 是可编程假传输
scripts/live-check.ts    REST 兜底链路联网自检
cordis.dev.patch.yml     开发模式：OKX MCP（market,account,news,swap）+ 源码直载（command/profile 是占位符；
                         本机实际值放 cordis.dev.local.patch.yml，已 gitignore，启动用它）
cordis.patch.yml         安装模式：只插本插件，MCP 由用户 profile patch 另挂
```

## 设计纪律（改代码前必读，违反即破坏本项目核心原则）

1. **零运行时依赖**：不得 import 任何 `@deepseek-ai/*` 运行时符号。工具用裸 JSON-Schema 注册，
   ctx 类型在 `src/index.ts` / `src/okx-adapter.ts` / `src/client/channel.ts` 里自声明（刻意宽松）。
2. **执行三道门**：写操作只走 `OkxTradeClient`（WRITE_TOOLS：永续设杠杆/下单/撤单/平仓/查单/查成交，无现货/期权/划转）；
   `crypto_execute_plan` 必须同时满足：用当前快照重算的 blockers 为空、计划 status=confirmed、`confirm=true`，
   且最新价偏离入场参考价 ≤1%、计划未过期。用户只说「确认」不等于「执行」，模型不得替用户说这两句。
   `test/execute.test.ts` 锁死了这个顺序，别绕。模型直连 `mcp__okx__*`：写动词（isOkxWriteTool）全局 guard 拒绝、
   读工具放行（查单/成交/行情核验回执用）；未知形态 fail-closed。想退回只读观察：patch 加 `--read-only`，执行工具自动报通道未启用。
3. **数字只算一次**：所有金额/杠杆/回撤/强平距离/张数在 host 确定性层算完进 DTO；client 只格式化，
   模型只解读。闸门由 `risk.ts` 算死，模型与前端不能覆盖。
4. **不猜数**：除数 ≤ 0 返回 null；数据过期可以展示但 `blocked`；账户 `partialFailure` 直接不出净值；
   「无消息」不是信号；成交只能人工录入或从 OKX 成交流水 import（按 tradeId 幂等）。
5. **DTO 稳定**：`crypto-dto.ts` 零 import，client 直接 import；改字段先想清楚 client 与 Skill 两边。
   换传输通道（Typert 直连）时 DTO 一行不改。

## 代码约定

- TypeScript strict + ESM；`verbatimModuleSyntax`，类型导入必须 `import type`；相对导入带 `.ts`。
- 注释与用户可见文案均为中文；注释风格是「设计纪律」式的（解释为什么这么写）。
- 金额：`Money { usdt, cny }`，USDT 4 位、CNY 2 位（`money()` 是唯一入口）；汇率 `FxRate` 带来源与时间。
- 标的 = 白名单全集 8 个 USDT 永续：BTC/ETH/SOL/BNB/SUI/DOGE/ZEC/HYPE（`RADAR_ALLOWED_COINS`/`isCryptoSymbol`，2026-09-12 拍板定稿），
  白名单外一律丢弃并写 `DataIssue`；`CONTRACT_SPECS` 必须覆盖全集（test/risk.test.ts 锁一致性）。
- 扫描面与执行面共用白名单：想交易某个币必须先加进雷达扫描（快照行情只覆盖核心+雷达标的，
  `buildTradePlan` 对行情缺失/过期按标的 fail-closed；全局 stale 闸门只看核心 BTC/ETH）。
- 记录 ID `<kind>-<本地日期>-<时间戳>`（`store.newId`）；计划按创建日分目录，`findPlan` 只扫 ID 里那天。
- 新鲜度阈值 `FRESHNESS_MS`（行情 10s / 账户 30s / 消息 5min）；刷新 TTL `REFRESH_MS`（行情 5s / 账户 15s /
  消息 60s）。两组常量各管一件事，别混用。快照历史不落盘（2026-09-11 用户拍板删除），复盘靠 plans/fills/decisions。
- 数据目录 `CRYPTO_DATA_DIR`，默认 `~/.dsh/crypto-workbench/`。

## OKX / dsh 接入的坑

- OKX MCP 1.4.6 信封：`{tool, ok, data:{data:[...]}, capabilities:{readOnly,demo}}`；字段全是字符串数字，
  空串=缺失；`account_get_balance_all` 的 data 是 `{trading,funding,valuation,meta}`。样本在 `test/fixtures.ts`。
- USDT→CNY 用 `market_get_index_ticker instId=USDT-CNY`（2026-09-11 实测可用）。
- host 调 MCP 走 `ctx.tools.execute({callId,name:'mcp__okx__<tool>',arguments,signal})`（无 agent 作用域）。
  若 dsh 侧对 MCP 工具要求审批（ask），无 agent 会被拒——表现为 issue `unavailable/…requires approval`，
  届时要么调 dsh 审批策略，要么切 CLI adapter（方案 §8「工具 schema 成本」有备选）。**首次在 dsh 里跑必须验证这一点。**
- Node 的 fetch 不认 `HTTP_PROXY`：REST 兜底在代理环境要 `NODE_USE_ENV_PROXY=1`（live-check 已带）；
  MCP 子进程由 OKX Kit 自己处理代理。dsh MCP client 会清洗 `KEY|PASSWORD|SECRET|TOKEN` 环境变量，
  OKX 凭证走 `~/.okx/config.toml` 即可。
- 账户是 hedge（long_short_mode）：设杠杆/下单/平仓都要带 posSide；execute.ts 通过 account_get_config 自动判定。
- 事件黑窗只认 impact=critical（CORE_MACRO_RE：CPI/非农/FOMC/GDP/PCE）；provider 的 importance=3 还包括密歇根信心等，只标 high。
- 开始实验时日初净值重置为当前净值（入金前定格的 0 会把入金算成当日盈利）。
- 金十直接走 `https://mcp.jin10.com/mcp` 标准 Streamable HTTP，不经 dsh MCP 注册表。Token 只放
  `~/.dsh/crypto-workbench/env` 的 `CRYPTO_JIN10_BEARER_TOKEN`；协议固定 `2025-11-25`，机器读取优先
  `structuredContent`。金十是独立只读 tab，不进入 OKX 快照、风险闸门或执行链。
- Jev 决策走 OpenRouter `POST /api/alpha/decisions`（alpha 端点，形态可能漂移——jev-service 解析处
  fail-open，坏响应只翻状态位）。Key 优先 `CRYPTO_OPENROUTER_API_KEY`（放数据目录 env 文件最稳），
  回退进程环境 `OPENROUTER_API_KEY`；dsh 从 GUI 启动时不继承 shell env，别依赖 ~/.zshrc。
- 开发模式改了 `src/*.ts` 必须**重启 dsh**（host 常驻）；client 改完 `npm run build:client` **也要重启 dsh**——client 包在启动时合并打包并按 rev 缓存，只刷浏览器拿不到新代码（2026-09-11 实测）。
- 面板轮询在页面 `visibilityState=hidden` 时跳过；工具行里的旧载荷只在比当前更新时才喂面板。
- 面板取数走影子会话（`session-crypto-panel-*`），否则 5s 轮询会把命令记录写进用户真实对话。
- `--patch` 是根启动器参数，必须写在 `web` 之前。
- **dsh 0.1.5-rc.2 的命令入参是 `invocation.rawInput`（带前导空格），不是 input/args/text**
  （2026-09-18 实测：crypto-tools 的 commandArg 只认旧形态时，「/crypto radar」「/crypto jev」
  的参数被静默吞掉、一律当空处理——radar 按钮因此悄悄废了很久，被 60s 自动重扫掩盖）。
  commandArg 现按 rawInput → input → args → text 逐个认，test/plugin.test.ts 有回归锁。

## 敏感区前置阅读

- 动闸门阈值 / 仓位公式之前：读方案 §7、§8.3，改 `risk.ts` 必须同步 `test/risk.test.ts` 与 README 闸门表。
- 动简报 / 风险审查输出格式之前：读 `skills/crypto-briefing/SKILL.md`、`skills/crypto-risk/SKILL.md`。
- 动 DTO 之前：确认 client 七区与两个 Skill 的引用字段。

---
> Source: [limboinf/dsh-crypto-workbench](https://github.com/limboinf/dsh-crypto-workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
