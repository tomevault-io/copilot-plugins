## jev-ui-test

> - **输入是一个自然语言目标。** 不要在策略里加入站点专用的步骤计划或写死的字段值——那是用例（`cases/*.yaml`）该做的事。

# jev-ui-test

## 核心契约

- **输入是一个自然语言目标。** 不要在策略里加入站点专用的步骤计划或写死的字段值——那是用例（`cases/*.yaml`）该做的事。
- **一次请求同时选出操作和该操作对应的目标。** TypeSafe 返回多个目标分支（target head），只消费被选中操作的那一个。
- **目标必须落在已观测元素与受支持操作上。** 绝不让模型产出选择器或可执行代码。
- `TYPE_TEXT` 才调用文本大模型。过期重试时，只有在整个辅助模型输入完全一致的前提下才复用已生成的值。
- **绝不重试浏览器变更操作。** 先记录执行，再去观察结果。
- 截图可选，模型不消费截图。演示素材保持原始速度。
- **凭据保存在服务端**，`.env` 不入库。未经用户允许，测试不得调用付费 API。
- **独立验证最终结果。** 模型选了 `DONE` 不等于成功。
- 示例、README 的说法、原始证据、模型调用次数四者必须保持一致。
- 优先使用简体中文

## 分层与边界

| 层 | 位置 | 职责 |
|---|---|---|
| 库本体 | `jev_ultrafast/agent.py`、`browser.py`、`model.py`、`snapshot.js`、`questions.py` | 观测 → 决策 → 执行的循环，以及全部安全校验 |
| 框架层 | `jev_ultrafast/framework/` | 环境/账号配置、用例加载、断言判定、Allure 报告、E9 登录接入 |
| 用例 | `cases/**/*.yaml` | 用户只写自然语言目标与预期结果 |
| 用例入口 | `tests/test_nl_cases.py` + `tests/conftest.py` | 把 YAML 收集成 pytest 用例 |
| 演示素材 | `examples/` | 演示用例副本、演示报告、演示录屏（**有意入库**，见下方「凭据与仓库边界」的例外） |
| 技能 | `.claude/skills/nl-case-author`、`nl-case-run` | 前者把各种格式的功能用例转成 YAML（`cases/e9/`），后者按用户描述挑用例、执行、出 Allure 报告 |

- **框架是外挂的一层。** 库本体的状态机与各类校验的**语义**不因框架需求而改变；框架只负责用例加载、断言判定与报告。
- 为让内核在真实站点上可用，已对 `snapshot.js` / `browser.py` 做过一批**通用性扩展**（不针对任何特定站点）：元素发现范围、跟随新标签页、执行前滚动、登录态注入、重页面的 CDP 超时、页面稳定等待。**这些扩展只扩大"能看见/能操作什么"，不放松"怎么校验"**——新增动作同样要过新鲜度与遮挡校验。
- 继续扩展时守住这条线：**不要**因为某个站点难搞就放宽安全校验（例如为绕开遮挡检查而跳过 `elementFromPoint`）。那类改法会把"安全的自动化"变成"会误点的自动化"。
- **不要给库本体打 `allure.step` 之类装饰器。** `demo.py`（inspector）也依赖库本体，不应被报告框架污染；而且步骤标题需要动态内容，装饰器做不到。步骤包装放在 `framework/runner.py`。

## 写用例前必须知道的两件事

- **断言只对【终局页面】求值。** 中间步骤做过什么，终局断言看不见——所以要校验的东西
  必须落在最后一站的页面上。典型做法是把"这一步真的发生了"编码进终局 URL 或终局文案。
- **agent 没有"后退"动作。** 路径只能前进，站点内换页要靠页面上的导航，不能指望浏览器历史。

另外：**元素必须落在当前视口内才会成为候选**（`snapshot.js` 的几何过滤）。屏幕外的目标
要么让 goal 明确写"向下滚动，找到 X"，要么把这个动作拆成两步。

## 断言

两级，边界不能混：

- **执行期**用 Jev 的 `choice` 概率做连续决策（`model.choose()`）。
- **终局**必须是 pytest 的确定性 `assert`。语义性预期由 Jev 的 `noul` 提供 0–1 的**证据**，**阈值比较与判定留在代码里**——不允许"问模型通过了吗"。
- 断言默认阈值 0.75，收敛在 `framework/config.py: DEFAULT_AI_THRESHOLD`，可按断言/用例覆盖。改阈值前先看 Allure 里的实测概率分布。
- 多断言的合成口径（`and` / `or` / `min_pass`）**逐用例声明**，不设全局默认。
- 断言要写得**原子、正向**：实测否定式表述只能拿到约 0.47，正面陈述同一事实可到 0.9+。
- **元素发现范围**（`snapshot.js`）已扩到：同源 iframe 下钻（含坐标换算与两层命中测试，
  与执行层共用 `window.__jevFast.geometry`）、`tr` 里"行即按钮"的可点整行、
  只有 `title` 的图标按钮、以及"有名字 + 有 onclick/cursor"的自定义按钮。
  这些**只扩大"能看见/能操作什么"**，执行前的遮挡校验一字未动。
  两条踩过的坑：① 判断"主循环是否已收过"只能比**主循环真正用过的那个 selector**，
  拿 `role()` 或"祖上有 role/title"代替都会把「编辑」这类元素两边都漏掉；
  ② 往 selector 里放宽一类元素（如"有 title 又有文字也算按钮"）会把候选表从 72 推到 107，
  原本稳定的用例立刻变得时好时坏——**放宽要按"实测噪声增量"验收，不能只看功能通不通**。
- 语义断言要断言**看得见的具体东西**，不要断言**页面身份**。实测（2026-09-24，E9 流转设置页）
  同一条需求换措辞量出来的分布：
  `「流转设置」是当前选中的那个页签` **0.10** ／ `当前页面是该流程的流转设置界面` **0.24** ／
  `页面显示的是「UI自动化A」这个流程的流转设置界面` **0.40** ／
  而 `页面上有「图形编辑」页签` **0.98** ／ `页面上有「编辑」按钮` **0.97** ／
  `页面上显示着「图形编辑」「节点信息」「出口信息」三个页签和「编辑」按钮` **0.96**。
  原因："是哪个页面"从页面正文里判不出来（`流转设置` 只是四个并列页签之一，选没选中看不见），
  而"页面上有没有这个页签/按钮"是可核的。**先量措辞再决定要不要动阈值**——
  把 0.75 调到 0.7 是把尺子改短，把话说成看得见的事才是把事做对。

## 重跑

- **只允许用例级重跑**（`pytest --reruns N`，或 YAML 里的 `reruns` 字段）。
- **绝不做步骤级重试**：浏览器变更操作重试可能重复提交、产生垃圾数据。
- **决策可以重发，动作不可以。** 这两件事的区别在于"有没有东西被执行过"：
  `model._decision()` 会重发一次**只读**的 TypeSafe 请求（上限 `DECISION_ATTEMPTS`），
  因为它走到重发时校验已经失败，而校验失败发生在任何浏览器动作**之前**——重发不可能
  重复点击。重发次数记在决策结果的 `decision_attempts` 里，报告里看得见，不静默自愈。
- **传输层故障与 429 同等对待。** `model.post_json` 对连接失败/超时也做有界重试
  （不是只对 429/529/503）。判据还是同一条：它的三个调用方（决策、文本生成、语义断言）
  全是只读请求。加这条的直接原因：实测（2026-09）演示用例的首次尝试死在连接失败上，
  靠 pytest 的用例级重跑才过——那次失败本可以在这一层自愈。
- 设计上就该失败的负向对照用例设 `reruns: 0`，避免白跑。
- **决策耗时是服务端的，不是请求体大小决定的。** 实测（2026-09-25）同一个用例、
  同样 99–131 个候选，两次运行决策中位一个是 **6874 ms**、另一个是 **405 ms（快 17 倍）**；
  而全数据集里最小的一次请求（24 KB、71 个候选、goal 192 字）却花了 4491 ms。
  所以看到"决策几秒到几十秒"**别先去砍候选表**，那是拿错了杠杆。
- **几十秒的那种要单独认：那是客户端超时 + 一次传输层重试。**
  `CLIENT` 的 timeout 是 25 s，超时后会静默重发一次 → `25 + 0.5 + ≈22 ≈ 48 s`。
  判据看报告里的 **「传输尝试」**（>1 就是重发过）——它与「重发次数」（响应不合法导致
  的决策层重发）是两件事，合起来才解释得清耗时。实测正是靠它把 47.5 s 归因清楚的。

## 凭据与仓库边界（**务必遵守**）

本仓库的 remote 是 **GitHub 公开仓库**（未鉴权即可读取），因此以下文件**只存本机，绝不提交**：

| 文件 | 内容 | 入库的替代物 |
|---|---|---|
| `config.json` | E9 测试环境地址、账号、图谱 MCP 地址 | `config.example.json`（占位模板） |
| `.mcp.json` | 图谱 MCP 的内网地址 | `.mcp.example.json`（脱敏模板）；也可由 `config.json` 的 `mcp` 块重建 |
| `.env` | 各模型 API Key | `.env.example` |

**例外：演示素材是有意入库的。** `examples/allure-report/`（真实执行报告）与
`examples/jev执行真实业务场景的测试报告录屏.mp4`（录屏）**刻意随仓库分发**——
它们已经含内网 E9 的页面截图与地址，这是**已确认的取舍**，不要因为"里面有内网信息"
就去删掉或加 gitignore。但这条取舍**只适用于这两份已确认的素材**：

- **新增或替换演示素材前逐帧 / 逐附件确认**：不带账号密码、不带其它客户或其它系统的信息；
- 素材一入库就进了 git 历史，**删不掉**（要 history rewrite），所以是"先确认再 `git add`"；
- **README / `docs/performance.md` 里的效率数字，原始证据就是报告里每个决策步骤的
  `决策耗时ms`**——改数字必须回报告重新统计，不要沿用旧值（见「核心契约」最后一条）。

提交前自检：

```bash
git check-ignore -v config.json .mcp.json .env   # 三者都必须命中
git status --short | grep -E "config\.json|\.mcp\.json|\.env" && echo "❌ 敏感文件将入库" || echo "✅ 干净"
```

用例里也不要写真实内网地址，用 `{{ base_url }}` 变量。

## 校验

```bash
uv run ruff check .
uv run pytest                                   # 默认离线：143 passed, 6 skipped（自然语言用例全 skip，每个 YAML 一条）
node --check jev_ultrafast/static/app.js
node --check jev_ultrafast/snapshot.js
uv build
```

另外三条按需执行：

```bash
# 浏览器真实控件校验，不调模型
uv run python scripts/check_guards.py           # 期望 PASS: 26 browser guard checks

# 三个 skill 的 evals：真起 inspector、真跑读源脚本、真跑收集与报告链路
uv run python scripts/run_skill_evals.py        # 免费档
uv run python scripts/run_skill_evals.py --paid # 含真跑用例（花钱）

# 自然语言用例（会调用付费 API 并接管一个 Chrome 标签页）
uv run --env-file .env pytest tests/test_nl_cases.py --nl --reruns 1 \
  --alluredir=report/allure-results

# 同上，但顺带生成一份【带时间戳】的 Allure 报告，旧报告不覆盖（推荐）
uv run --env-file .env python scripts/run_nl_report.py --case <用例 id>
```

skill 的 evals 是 skill 的一部分：新增/修改 skill 时**同步改它的 evals**，
并至少跑一遍免费档。断言写在 `.claude/skills/<skill>/evals/run.py` 里，
各目录的 README 记着它验什么、以及它查出过什么。

## 报告层（二期：报告优化）

三条需求（决策传参、执行请求与返回、执行录屏 + 步骤↔视频同步）的落地契约。
这里只记**必须遵守**的部分。

### 三个开关与四级优先级

```text
pytest 参数  >  环境变量 JEV_NL_*  >  config.json  >  framework/config.py 内置默认
```

| 开关 | 取值 | 默认 | pytest 参数 | 环境变量 |
|---|---|---|---|---|
| 录屏档位 | `0` / `1` / `-1` | `1` | `--video-record=0\|1\|-1` | `JEV_NL_VIDEO_RECORD` |
| 页面稳定等待 | true / false | **true** | `--wait-stable` / `--no-wait-stable` | `JEV_NL_WAIT_STABLE` |
| 等待成步骤门槛 | 毫秒 | `200` | `--wait-step-ms=200` | `JEV_NL_WAIT_STEP_MS` |

- **档位 `0` 是"不启动录屏"**，不是"录了再删"；**档位 `-1` 是"不编码、不附加"**，
  不是"生成再删"——`allure.attach` 会把字节拷贝进 `allure-results`，附加过就删不掉了。
- **`-1` 档只在"明确通过"时才不留**。终局没过、执行提前停止、过程里出现过页面过期
  或决策重发——一律保留。这些"通过了但不健康"的运行恰恰最值得看。
- 生效值写进「执行摘要」附件，报告里能核对。
- ⚠️ **`--no-wait-stable` 省不了钱，可能更贵**：它省下的只是免费的页面观察，
  却可能让「渲染中途决策 → 页面过期 → 重决策」变多，而**重决策是付费的**。
  定位是调试/诊断，不是回归省钱。想省钱优先调小 `timeout`（25 s → 8 s）。

### 步骤参数与时间：三条"必须是这样的"

1. **参数要在进入 `with` 之前设**（`report_params.step(title, params)`）。
   进入之后再改 `ctx.params` **无效**——params 在 `start_step` 那一刻就被读走了。
2. **补参数必须在 `with` 块【内】**。出了块 `get_item(uuid)` 返回 `None`
   （`stop_step` 会 `_items.pop`），补的东西静默消失。
3. **改步骤时间必须在 `with` 块【外】**（`report_params.step_with_real_times`）。
   `stop_step` 会用 `now()` **覆盖 `stop`**，块内写的会被冲掉；
   而提前抓住的那个对象就是容器里那一份，出块后再写照样落盘。

> 第 2、3 条方向相反，很容易记混。两条都由 `tests/test_report_params.py` 钉住，
> 且各有一条**反向断言**（"这么写会失效"）——它们解释实现为什么长这样。

### 参数值的假值规则（`report_params._text`）

`AllureFileLogger` 落盘时的过滤器是 `asdict(item, filter=lambda _, v: v or v is False)`
（`allure_commons/logger.py:24`）——**假值全部被丢掉**：`""`、`0`、`None` 都不进 JSON，
而报告照常生成、照常打开，只是那个参数不见了。

所以 `_text()` 里那几条转换**不是格式化偏好，是防数据丢失**：
非字符串一律 `str()`（`0` → `"0"`）、`None`/`""` → `"—"`。
**放宽任何一条，对应的参数就会静默消失。**

### 二开插件与版本锁定

`framework/report_plugin/step-video/` 是 Allure 报告的浏览器侧插件（跟框架一起入库）。
它依赖的是**报告运行时的内部结构**，不是稳定协议——升级任一项都可能**静默失效**
（报告照常打开，只是没有时间轴）。所以：

- 版本表在 `report_plugin/step-video/versions.json`，装之前逐项校验，**不匹配就拒绝安装并报错**；
- Backbone / Marionette 在报告页面里、装的时候读不到，由**插件在浏览器里自查**，不一致出横幅；
- **不要自己打包一份 Backbone**：报告页里只有 `window.Backbone`（Backbone 1.3.3 /
  Marionette 3.3.1），没有独立的 `window.Marionette`；自带一份会版本冲突，**可能整个报告白屏**；
- 升级 Allure 后的回归清单见 `report_plugin/step-video/README.md`。

```bash
# 装插件必须在 allure generate 【之后】
allure generate report/allure-results -o report/allure-report --clean
uv run python scripts/install_report_plugin.py report/allure-report
```

### 录屏是"稀疏幻灯片"，不是连续视频

CDP screencast 是**重绘驱动**的：页面不动就没有合成提交，也就没有帧
（实测一条 26 秒的用例只抓到个位数帧；连续滚动才会到 20 fps）。

这不是缺陷：时间轴用每帧的真实时长，**seek 到某一刻看到的就是那一刻页面的样子**。
但要注意：
- 「录屏说明」附件里有**帧数**，是判断录屏是否正常的第一个线索；
- 视频最后一帧保持到录制结束，所以视频时长 == 真实录制跨度；
- **断言与终局判定的步骤在录屏范围之外**（录屏在用例收尾前就停，那时浏览器已关），
  插件对这类步骤给说明而**不静默跳到末尾**。

### 插件里的两条已定行为（别改回去）

1. **点一步 = 跳过去并停住；再点同一步 = 继续播。** 不要改回"跳过去就自动播"
   ——绝大多数步骤只有 1 秒上下，自动播会直接跑过关键的那一瞬。
   判据是**状态**（`paused` 且位置没被拖走过），不是简单 toggle。
2. **光标是合成的，不是录下来的。** `Page.startScreencast` 不含系统光标，
   只能按执行步骤参数里的「操作坐标」画上去。坐标由 `browser.py` 的 act 返回
   （`execute_result["point"]`），三种 `via` 语义不同，**别混为一谈**：
   `mouse` 真发了鼠标事件 / `wheel` 是滚动的固定坐标 / `js` 是 select 直接赋值。

两条都靠"不能静默失效"这一条线：

- 换算的**分母是报告的 `videoViewport` 标签**，不写死 1120、也不从 `video.videoWidth`
  推断——等比缩小时推断恰好对，不成比例就**静默偏**；比例不符时插件出红横幅；
- `object-fit: contain` 在插件 CSS 里**显式钉住**，不依赖浏览器默认值；
- 没有「操作坐标」（旧报告）时不画光标，但**说明原因**，不静默什么都不显示；
- **"哪些是执行步骤"按「有 `操作` 参数」认，不要按标题里有没有"执行"认。**
  标题判据会把「终局：**执行**结果」算进去，而它在录屏结束之后——自检拿它比视频时长
  会弹**假的**"录屏没覆盖到"横幅（实测只差 0.57 s，容差 1.0 s，**越慢的机器越会误报**）。
  这个 bug **在界面上看不出来**：计数对、横幅也没弹，只能回到数据源头数。

> 验证这两条要用**真鼠标事件**（`Input.dispatchMouseEvent`）：合成的 `li.click()`
> 不算用户激活，`play()` 会被自动播放策略拒绝，测出来是**假失败**。
> 工具在本机 `artifacts/plugin-cursor/`（gitignore，不入库）。

### ⚠️ Windows：`subprocess` 不认没有扩展名的 `.bat`

```text
shutil.which("allure")                  → D:\...\allure.BAT
subprocess.run(["allure", "--version"]) → FileNotFoundError (WinError 2)
```

症状极具误导性：看着像"allure 没装"，其实装了但叫不出来。
调用外部工具一律先过 `resolve_tool()`（`report_plugin/install.py` 与 `framework/encode.py` 各一份）。

## 其它规则

1. 大模型的思考和回复优先使用简体中文，新增的文档与代码备注也优先使用简体中文。
2. 未经用户明确要求，不要 commit 或 push。
3. **测试/调试的临时产物一律写到 `artifacts/`**（已 gitignore），不要落在仓库根目录或其它受版本管理的目录。
   包括：探针脚本、截图、录屏、生成的报告、中间数据。`report/` 也是 gitignore 的，Allure 报告照旧写那里。
   注意：工具常以「当前工作目录」为基准解析相对路径，所以 **Playwright 截图、写探针文件、导出报告时都要显式给
   `artifacts/` 下的绝对路径**——用相对路径会落到根目录，一不留神就被 `git add -A` 收进仓库。
   提交前的自检里顺手看一眼：`git status --porcelain -uall | grep -v '^?? artifacts/'`。

---
> Source: [buer2233/jev-ui-test](https://github.com/buer2233/jev-ui-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
