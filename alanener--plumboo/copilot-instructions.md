## plumboo

> 授权目标 only。中文。有可验证方向就继续；方向穷尽则记录并切换。

# Plumboo — 黑盒渗透认知 Runtime

授权目标 only。中文。有可验证方向就继续；方向穷尽则记录并切换。
仅 scope 争议、缺外部凭据时问人。不问是否继续。每个接口独立验证。

> **Runtime 不替你决定下一步、不判洞、不跑 SOP、不按关键词加载知识。**
> 它只做：**发请求并留证、保存当前世界、把世界投影给你、记下你的结论。**
> 怎么打，是你的事。它要是抢了，这套东西就是扫描器。

## 开局

读 `cognition/`（固定顺序，别挑）：

```
cognition/object_boundary.md           什么是对象，什么不是
cognition/observation_vs_fact.md       观察 ≠ 事实
cognition/hypothesis_to_conclusion.md  假设 → 实验 → 结论
cognition/surface_vs_reality.md        表层 ≠ 真实
```

这四份是**推理透镜**，不是操作手册。按目标换着读就输了。

## Runtime

产物在 `results/<target>/`：`responses/`、`experiments/`、`report.md`。
当前世界在 `state/memory.md`（唯一 active）。

```bash
python runtime/doctor.py                                           # 环境自检（开跑前跑一次）
python runtime/fetch_tools.py [--download] [--extract]             # 取外部工具（默认只报告）
python runtime/http_request.py <url> [-X POST] [-d ..] [-H ..]    # 禁止裸 curl
python runtime/http_request.py <url> [--max-body N] [--raw]       # 大文件/二进制
python runtime/execute.py --host-dir results/<t> -- <cmd...>       # 非 HTTP
python runtime/portscan.py <host> [--ports ..] [--host-dir ..]
python runtime/sourcemap.py <url> [--extract] [--calls]
python runtime/subdomain.py run --target <domain> [--source auto|oneforall|passive]
python runtime/subdomain.py list --target <domain> [--alive-only] [--hosts-only]
python runtime/subdomain.py show --target <domain> --host <sub>
python runtime/listen.py --host-dir results/<t> --port 8080 --bg  # 带外监听（SSRF/JNDI/XXE）
python runtime/listen.py --host-dir results/<t> --show            # 读回回调
python runtime/listen.py --host-dir results/<t> --stop
python runtime/graph.py show                                       # 当前世界投影
python runtime/graph.py switch --target <host> --scope "..."       # 开/换世界
```

展示截断 ≠ 全文，判断以磁盘为准。四个 IO 工具**共用一套 `req_NNN` 编号**，
全部落 `results/<target>/responses/`，账本 FK 只认这个目录 ——
端口扫描写 `req_NNN.json`，HTTP 写 `req_NNN.txt`，编号自动接续不撞车。
带外回调是**第三类证据** `cb_NNN`，落 `results/<target>/callbacks/`，FK 同样认。

读证据时先看头部这几行，它们区分"读全了"和"读残了"：

```
body-bytes=1183053              # 服务端实际给了多少
body-view=complete              # complete / binary-omitted / [TRUNCATED] …
raw=responses/req_004.body      # 文本视图不完整时，完整字节一定在这里
network=直连（已忽略环境代理 …） # 这次出网经过了谁 —— 没这行=不可信
```

**出网路径必须自证**：`http_request.py` 默认**直连**（显式压掉 `http_proxy`/
`https_proxy` 环境变量，否则流量会被中间人接走、目标会"返回" 502 而证据里
一字不提）。要真走代理必须 `--proxy` 明说，或 `PLUMBOO_USE_ENV_PROXY=1` 显式尊重环境。
`subdomain.py` / `sourcemap.py` 共用同一套逻辑（`runtime/netenv.py`）。

`--calls` 抽出的 `_api_calls.txt` 是**接口清单**，`_path_keys.txt` 是 `path:` 键
（vue-router 路由居多，别当接口打）。`_api_calls.json` 里 `base_urls` 是前缀不是接口。

**三态硬约束：有 / 没有 / 没测到。** 「我没拿到」不等于「它没有」。
只有两态的工具会把"没测到"算进"没有"，而它总是选"没有" —— 你会把一块攻击面
从地图上永久划掉。（这次读数是真是假拿不准时，翻 `knowledge/instrument_failures.md`。）

- `portscan.py` 输出 `开放 / 明确关闭 / 被过滤 / 未能判定` 四数。
  **只有 RST 拒绝才算关闭**；`被过滤`（超时）和 `未能判定`（本地错误）都**不能**当端口不存在。
  记账要照实写这三态，别把四数压成"开了 N 个"。
- `sourcemap.py` **退出码 0 = 干净结论（含真阴性），3 = 侦查不完整，结论不可用**。
  看到 `⚠ 结果不完整` 就不许写"没有 sourcemap"。
- `http_request.py` 证据头部有 `redirects=` / `final-url=`；`--no-redirect` 是真不跟随。
- `graph.py` 未设 target 时会 WARN"证据引用**未经校验**" ——
  "没警告"从来不代表"已校验"。
- `listen.py` 收不到回调有**四种解释**（不可达 / 到不了我们 / payload 形态不对 /
  监听没起来），`--show` 会原样列出，**别当阴性结论用**。
  `--show` 还会**实测回环**判断监听是否还活着（不信 `listener.json`）。
  反过来：**空连接（连上没发数据）≠ payload 被投递**，会被标 `(空)` 单列。

**子域枚举是 IO，不是决策。** `subdomain.py` 只负责跑工具、归一化落盘、原样投影，
**不排序、不打分、不挑"脆弱的"**。清单是客观的，挑谁是你的判断。
产物在 `results/<target>/recon/`（含 OneForAll 全部原始字段，别只看预览）。
OneForAll 默认只探 80/443，`alive=0` 不等于不存在 —— `resolve=1` 但 `alive=0` 的要看。

**外部工具是可选的，`runtime/` 不依赖它们。** `runtime/` 是纯标准库；只有 `subdomain.py`
会去找 OneForAll，找不到就退回被动源（crt.sh / hackertarget）。**"工具没装"会让功能降级，
但它必须先被说出来** —— `doctor.py` 就是把这件事提前说出来的地方（`0` 可开跑 / `3` 别硬跑）。
缺了要取：`fetch_tools.py`（从官方 release 取到 `_tools/`，**不内置、不执行**）。
它报 `被隔离` 时别当成"没装"——那是"装过但被杀软删了"，两者该问的问题不同。
工具不在 PATH 里也能用：`PLUMBOO_ONEFORALL`（含 `oneforall.py` 的目录）、
`PLUMBOO_ONEFORALL_PY`（解释器）、`PLUMBOO_SEARCH_ROOTS`（浅扫根目录）。
哪些工具擅长什么、去哪拿，见 `knowledge/tool_index.md`（**是索引不是路由** ——
它不回答"该用哪个"，那不归它管）。

**`runtime/` 覆盖不到的操作，走 `execute.py`。** 它跑任意命令、把产物落成 `exp-NNN`
（`cmd` / `stdout` / `stderr` / `rc`），stdout 直接吐 `exp-NNN` 供账本引用 ——
外部工具、复杂管道、**你现写的脚本**都从这一个口子走，不因此往 `runtime/` 加脚本。
一次性脚本放 `_scratch/`（已 gitignore，不进仓库）；想留下来复用的先问一句
**"它能不能写成一条通用命令？"** —— 能，就说明它该走 `execute.py`。
只有某操作**产生新的一类证据形态**（像 `listen.py` 引入 `cb_NNN`）才值得进 `runtime/`；
常驻监听这种 `execute.py` 干不了的（它同步阻塞），只有 `listen.py`。
落不了盘的操作用法上不可复核 —— 照实说"这段没证据"，别编引用。

## 记账（每轮末尾提交一次 DELTA）

```bash
python runtime/graph.py merge --file delta.txt
```

```
DELTA:
surface: /api/login 存在，POST 返回 200
hypothesis [observed] h-001: /api/login 的身份判定可能只看 cookie 是否存在
counter: 去掉 cookie 返回 401 (req_007)
```

`hypothesis` 可带稳定编号 `h-NNN`：**同编号 = 同一条**（Runtime 按编号覆盖状态，
不再靠文本比对）。改措辞但仍是同一条假设 → **沿用同一 `h-NNN`**，
否则会被当成两条，旧的那条永远悬着。没有编号时退回按文本精确比（旧行为）。

| kind | 记什么 |
|---|---|
| `target` | 当前目标（出现在 DELTA 里即触发换世界，旧的自动归档） |
| `tech` `port` `artifact` | 目标档案 |
| `surface` | 对象存在性 / 技术事实 |
| `hypothesis` | **解释不了的关系**，可证伪。不是待办任务 |
| `counter` | 反证：推翻某个假设，或标记边界 |
| `dead` | 路径不可达（做过差分才能写） |
| `cred` | 拿到的凭据 |
| `vuln` | 可复现且有影响 |
| `future` | 战略传承，不参与本轮推理 |

`hypothesis` 状态来自验证结果，不是猜测：
`[observed]` 未验证 · `[confirmed]` 实验支持 · `[contradicted]` 有反证 · `[blocked]` 缺条件 · `[unknown]`

**结论 vs 事实**：越权/绕过/泄露/否定等**探索结论**必须用 `hypothesis [状态]` / `vuln` / `counter` 表达并带证据编号。
`surface` 只记事实。

**冲突即记账**：观察跟当前模型打架或说不清 → 立刻留可证伪 `hypothesis:`。
可换主线，不可因新故事遗忘未闭合 hypothesis。

## 打点与取得入口

- **探索 ≠ 破门。** 探索扩大世界模型；破门进入已有状态或验证已有假设。
- 大量未解释关系时探索优先；阻断机制仅当有 hypothesis 指向它时才投入。
- 不规定先扫哪类面。判据：**这一跳能否扩大世界模型，或证伪某个冲突。**

## 从观察到假设

- **单次观察不是事实。** `server: nginx/1.18` 是观察，不等于存在 nginx 漏洞。
- `hypothesis` 描述**关系**不描述任务。错："测试 SQL 注入"；对："search 参数是否影响 SQL 拼接"。
- **路径 / act / 函数名默认不是同一对象。** 合并要关系证据，不能靠长得像。
- 只记解释不了的关系，禁止 interesting。未做差分禁止 `dead:`。

## 差分与证据

- 孤立的单次响应弱于受控差分。Baseline 与只改一个条件的 mutation 之间有稳定差异，才能缩小替代解释。
- **无差异只约束本次上下文，不证明所有路径等价。**
- 403 可能来自 WAF / 网关 / 应用鉴权 / 路由隐藏——这些是替代解释，不是裁决。
- 负结果只关闭**被实际区分**的那段关系。没区分的保留 unknown。

## 什么算进展、何时收口

- **进展不只是"又多了一个洞"。** 把一个未知缩小、把一个假设验掉或排除掉，同样是往前走了。
- **探索完成是条件性的**：新证据可能推翻旧假设、旧优先级、旧收口判断。
  访问条件/权限边界/可达范围变化时，重新审视此前被降权的邻接未知。
- 对象可暂停，不应因写报告而视为闭合。
- 超时 / 纯文案 / 无差分 → 禁止 `vuln:`。

## 报告

写 `results/<target>/report.md` 时逐洞过滤，不得凭类型名称自判（过滤口径见 `knowledge/report_criteria.md`）。

- 一洞一节
- 每节必填：标题、域名、类型、等级、简述、**完整绝对 URL**、分步复现（每步附可粘进 Burp 回放的原始请求包）、证据编号、修复
- 禁止无证据的"可能"；禁止只有结论没有包

**World 完整 > Report 精简。** 低价值发现照样记进 world，只在写报告时过滤。

## 知识库

`knowledge/` **不自动加载**。卡住了才自己翻，它是过滤器不是答案。

**准入判据：这条内容能不能写成 nuclei 模板？** 能 → 它不该在这个目录里，
也不该占你的轮次。指纹 / CVE 清单一律不收（已经清掉过一次，别再加回来）。
只收：对象清单、认知纪律、平台收录规则。

## 子代理（可选，用于隔离上下文）

`.claude/agents/`：`plumboo-recon`（子域枚举 + 挑候选）、`plumboo-survey`（铺面）、
`plumboo-explore`（单假设验证）、`plumboo-report`（出报告）。

派给它的必须是**有界的问题**，它返回 DELTA 文本，**你审完自己 merge**。
它不替你决定打哪 —— recon 交回来的是「清单 + 候选与理由」，选谁是你看完之后定的。

---
> Source: [Alanener/Plumboo](https://github.com/Alanener/Plumboo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
