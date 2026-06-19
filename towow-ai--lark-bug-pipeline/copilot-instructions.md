## lark-bug-pipeline

> 飞书 Bug 反馈 → 自动修复 → PR 的端到端流水线。用户在飞书群里 @bot 扔一句话 bug，15 分钟后 GitHub 上自动开好 PR。依赖 Claude Code harness（headless `claude -p`）做 triage 和 guardian-fixer 的 8 Gate 闭环。


# 飞书 Bug → PR 全自动流水线

## 一句话

**用户在飞书群 @bot 发一条 bug，Mac 本地两个 launchd 常驻进程捡起来，调 `claude -p` 跑 triage + guardian-fixer 8 Gate 修复流程，最后自动开 PR 到 GitHub。**

你（维护者）只需要：
1. 飞书建个自建应用 + 把 bot 拉进目标群（不需要多维表格，默认 IM-only）
2. 填 `~/.towow/.env.lark`（4 个必填变量）
3. `bash .claude/skills/lark-bug-pipeline/install.sh`
4. 关电脑不管它

下次开机登录桌面后，两个 LaunchAgent 自动拉起，pipeline 继续待命。

---

## 这个东西解决的问题

早期产品有一堆 bug，但开发者看不到 / 用户懒得写 / 开发者懒得修的三重循环。传统方案是 Sentry + Linear + Jira 三件套，需要用户愿意填 form、开发者愿意分诊、团队愿意挤进 sprint。三个链条任何一环断了，bug 就烂在后台。

这个 skill 把这条链路全部压缩到 15 分钟：

| 环节 | 传统做法 | 本 skill |
|---|---|---|
| 用户反馈 | 提 GitHub Issue / 写邮件 / 填表 | 飞书群 @bot 一句话 |
| 分诊 | 产品经理人工判断优先级 | `lark-triage` skill 自动判断 auto / needs_user_clarification / out_of_scope / needs_nature |
| 修复 | 开发者进入 sprint 排期 | `guardian-fixer` skill 8 Gate 走完（PLAN → REVIEW → TASK → REVIEW → 实现 → TEST → FINAL-REVIEW → CLOSURE） |
| PR | 人工写描述 + 补测试 + 找 reviewer | `gh pr create` 自动带双语标题 + reporter attribution + 8 Gate artifact |
| 成本 | $$$ 人天 | **0 API 成本**（走 Claude Code 订阅而非 API billing） |

**真实运行证据**：PR [NatureBlueee/Towow#90](https://github.com/NatureBlueee/Towow/pull/90) 是这条流水线处理的第一条真用户反馈，从"飞书 @bot 发消息"到"GitHub PR OPEN"总耗时约 15 分钟，triage 2 分钟 + fixer 6.7 分钟 + bookkeeping。

---

## 架构一眼图

```
┌─────────────────┐
│ 飞书群 @bot 发 bug │
└────────┬────────┘
         │ WebSocket long-poll
         ▼
┌──────────────────────┐      ┌─────────────────────────┐
│ bug_daemon.py        │      │  ~/.towow/queue/        │
│ (LaunchAgent #1)     │─────▶│  <ts>-lark-im-*.json    │
│ lark-oapi 长连接      │      │  (JSONL 队列)            │
└──────────────────────┘      └───────────┬─────────────┘
                                          │
                                          │ 30s poll
                                          ▼
                              ┌─────────────────────────┐
                              │ bug_worker.py           │
                              │ (LaunchAgent #2)        │
                              │ batch + bundle + route  │
                              └───────────┬─────────────┘
                                          │
                    ┌─────────────────────┼─────────────────────┐
                    │                     │                     │
                    ▼                     ▼                     ▼
         ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
         │ claude -p        │  │ git worktree add │  │ claude -p        │
         │ --skill          │  │ + stage issues   │  │ --skill          │
         │   lark-triage    │  │                  │  │   guardian-fixer │
         │ (opus-4-6)       │  │                  │  │ (opus-4-6)       │
         └────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘
                  │ state file          │                     │ PR url
                  ▼                     ▼                     ▼
           docs/issues/*.md       isolated branch         gh pr create
```

三个关键隔离：
1. **daemon 和 worker 进程隔离** — daemon 挂了不影响已排队的 bug 继续修
2. **worker spawn 子进程跑 `claude -p` 而不是 inline 调 API** — 用 Claude Code 订阅，不烧 API tokens
3. **每个 bundle 一个 git worktree** — 多个 bug 可以并行修互不踩，失败也不污染主工作区

---

## 和 harness 的关系

这个 skill **不是**一个独立可跑的东西。它假设你背后已经有一套 Claude Code harness：

| 依赖 | 是什么 | 哪里拿 |
|---|---|---|
| `claude` CLI | Claude Code 命令行，headless 模式 `claude -p ... --permission-mode bypassPermissions` | <https://claude.com/claude-code> |
| Claude 订阅 | 别用 API key —— 两条修复走 API 要 $$$，走订阅是 flat rate | Claude Max 订阅 |
| `lark-triage` skill | 把用户一句话翻译成结构化 issue 草稿 + escalation 判定 | 本 skill 的 `runtime/lark-triage.md`，install.sh 会拷到 `.claude/skills/lark-triage/` |
| `guardian-fixer` skill | 8 Gate 修复流程（规划/审查/实现/测试/closure） | **你自己要有**。本 skill 只依赖它的 slash-command 接口；参考 Towow 主仓的 `.claude/skills/guardian-fixer/SKILL.md` |

`guardian-fixer` 是 harness 里最重的一块，本 skill 没把它包进来的原因：它是一套跟整个仓库工程规范深度耦合的东西（测试基线、review 规范、ADR-030 不可降级要求、commit 双语规则等等），强行打包没意义。你自己的仓库有自己的 guardian-fixer 实现，或者参考 Towow 的版本改一个。

---

## 安装

### 0. 前置条件

- macOS 14+
- Python 3.9+，`pip install lark-oapi`
- `claude` CLI 已登录，`claude -p "hello"` 能跑通
- 你的仓库里已经有 `.claude/skills/guardian-fixer/SKILL.md`

### 1. 飞书侧（详见 `docs/feishu-setup.md`）

1. 在 <https://open.feishu.cn/app> 创建「自建应用」
2. 申请 scope：`im:message`, `im:message.group_at_msg`, `bitable:app`
3. 开启「事件订阅」→ 消息与群组 → `im.message.receive_v1`
4. 新建多维表格「Bug 反馈」，字段见 `docs/feishu-setup.md`
5. 把应用加到你想收 bug 的群，给机器人一个名字

### 2. 环境变量

```bash
mkdir -p ~/.towow
cp .claude/skills/lark-bug-pipeline/templates/env.lark.example ~/.towow/.env.lark
# 按注释填实际值（App ID, Secret, Table Token, Bot Open ID 等）
```

### 3. 一键装

```bash
cd <你的项目根目录>
bash .claude/skills/lark-bug-pipeline/install.sh
```

install.sh 会：
- 检查 macOS / python3 / claude CLI / lark-oapi
- 拷贝 `bug_daemon.py` 和 `bug_worker.py` 到你仓库的 `scripts/lark/`
- 拷贝 `lark-triage` 子 skill 到 `.claude/skills/lark-triage/`
- 渲染两个 LaunchAgent plist 到 `~/Library/LaunchAgents/`（替换 `{{PYTHON}}` / `{{REPO_ROOT}}` / `{{HOME}}` / `{{PATH_PREFIX}}`）
- `launchctl bootstrap` 启动两个服务
- 验证 state=running，打印运维命令

### 4. 验证

```bash
# 在飞书群里 @bot 发一条 bug
# 然后看日志
tail -f ~/.towow/logs/lark-worker.log
```

看到 `Triage start` → `Fixer start` → `PR url: https://github.com/...` 就说明这条链路活了。

---

## 使用条件

| 前提 | 状态要求 |
|---|---|
| Mac 开机 | ✅ 必须 |
| 用户登录到桌面 | ✅ 必须（LaunchAgent 是登录触发，不是开机触发） |
| 网络联通飞书 + GitHub | ✅ 必须 |
| 任何 terminal / Claude Code session | ❌ 不需要 |
| 你人在电脑前 | ❌ 不需要 |

换句话说：**你把 Mac 放着，出去喝咖啡回来就能看到新 PR**。

---

## 运维

```bash
# 看日志
tail -f ~/.towow/logs/lark-worker.log ~/.towow/logs/lark-daemon.err

# 重启某一个（改完 .env.lark 后 daemon 需要重启才能生效）
launchctl kickstart -k gui/$(id -u)/net.towow.lark-daemon
launchctl kickstart -k gui/$(id -u)/net.towow.lark-worker

# 查看运行状态
launchctl print gui/$(id -u)/net.towow.lark-daemon | grep -E "state|pid"

# 彻底卸载
launchctl bootout gui/$(id -u)/net.towow.lark-daemon
launchctl bootout gui/$(id -u)/net.towow.lark-worker
rm ~/Library/LaunchAgents/net.towow.lark-{daemon,worker}.plist
```

---

## 故障排查

| 症状 | 一般原因 | 怎么治 |
|---|---|---|
| 飞书 @bot 没反应 | `LARK_BOT_OPEN_ID` 没填 | 第一次 @bot 后看 daemon 日志，抄 `open_id` 填进 `.env.lark`，重启 daemon |
| Triage exit 1 + no state file | Budget 不够 / 网络挂 / skill 写顺序错了 | 看 `~/.towow/logs/lark-worker.err`，调大 `TRIAGE_BUDGET_USD`（bug_worker.py 顶部常量） |
| Fixer blocked `issue file not found in worktree` | 主仓 issue draft 没进 worktree | 这是已知坑。worker.py `run_fixer_for_bundle()` 已经加了 `shutil.copy2` stage 逻辑，如果还报说明你拿的是老版本 |
| PR 没开但 fixer exit 0 | `gh` CLI 没登录 / 没 push 权限 | `gh auth status` 检查 |
| LaunchAgent 每 30 秒重启 | 脚本崩了触发 `KeepAlive` + `ThrottleInterval` | 看 `~/.towow/logs/*.err` 找真崩因 |

---

## 设计决策速查

（详见 `docs/architecture.md` 和 Towow 主仓的 `docs/decisions/ADR-040-lark-bug-pipeline.md`）

1. **为什么 LaunchAgent 不是 cron**：cron 在用户 session 外跑，没法访问 Keychain（`claude` CLI 依赖）。LaunchAgent 登录触发就自然进 session。
2. **为什么 `claude -p` 不是直接调 API**：Claude Max 订阅覆盖实际成本，`--max-budget-usd` 只是 API 价格估算的 circuit breaker。一次 triage + 一次 fixer 估算 $3+ API 成本实际 $0。
3. **为什么每个 bundle 一个 git worktree**：修复时需要 dirty working dir（写文件、跑测试、commit），不能污染主工作区；而且未来可以多 bundle 并行。
4. **为什么 triage 和 fixer 分两个子 skill 而不是一个大 prompt**：triage 快（2 min, 判断 + 写 issue 草稿）、fixer 慢（6-10 min, 真改代码 + 跑测试 + PR）。把慢动作隔离出去，worker 可以给 triage 卡更严的 budget、更短的 timeout。
5. **为什么 state file 是权威不是 exit code**：`claude -p` headless 模式的 exit code 不稳定（budget 边界会吐 exit 1 但状态是对的）。state file 是结构化产物，内容对就认。
6. **为什么 state file 写在 issue draft 之前**：crash-only software 排序原则。几十字的契约产物必须在几 KB 的叙述产物之前落盘，这样进程半路死掉 pipeline 也不回退到 needs_nature。

---

## 宣传文案（社媒 / 博客 / 发布）

> **你的飞书群现在可以自己修 bug 了。**
>
> 用户 @bot 扔一句话，15 分钟后 GitHub 上就有一个 PR。不是 GPT 写两行伪代码然后让你 review 半小时——是 Claude Code 跑完完整 8 Gate 修复流程（规划 / 独立审查 / 实现 / 测试 / final review / closure），带双语 commit，带 reporter attribution，带诚实的测试状态标记。
>
> 用的是你电脑上那个 `claude` CLI 订阅。零 API billing。
>
> 架构简单到你不想相信：
> - `bug_daemon.py` 挂着飞书 WebSocket，事件落 JSONL 队列
> - `bug_worker.py` 每 30 秒 poll 队列，spawn `claude -p` 子进程跑 triage，再 spawn 一个跑 fixer，最后 `gh pr create`
> - 两个进程都是 macOS LaunchAgent，开机登录自动拉起，崩了 30 秒自动重启
>
> 关 terminal、关编辑器、关 Claude Code 客户端都不影响它跑。你人不在电脑前它也在修。
>
> 这不是 AI demo，这是一套真在生产用的 skill。第一个真实 bug 走完闭环耗时 15 分钟，PR #90，已合。
>
> 安装一条命令：
> ```
> bash .claude/skills/lark-bug-pipeline/install.sh
> ```

---

## AI 指导模式

> 这一节是写给 AI 读的，不是写给用户读的。当有人对 Claude Code（或任何等价 harness）说 "我想装这个飞书 bug pipeline"，你——那个 AI——读这一节，按它走。

### 谁是你，谁是用户

这套东西不是一键安装包。一半动作你可以代劳——写文件、跑 `sed`、`launchctl bootstrap`、tail 日志；另一半必须由用户亲自完成，因为涉及账号凭证、开发者后台权限、真人身份。你替用户去飞书控制台点「创建应用」是越界，用户替你写 plist 是低效。这一节的目的是帮你分清这两类动作，并以合适的节奏把用户领过去。

想象你是 **上周刚帮另一个人装过同样东西的工程师朋友**，现在坐在这个用户旁边。你对每一个坑都熟悉到可以闭眼绕过，但你没有忘记对方是第一次；你不按小时收费，所以不会为了显示专业而多说废话。

- 你不是远程客服：不让用户「填工单等回复」
- 你不是教学视频：不把 WebSocket 或 launchd 的原理讲一遍
- 你不是代工外包：不替用户去飞书后台点按钮
- 你是 **让事情真正跑起来的人**。教学是副产品，不是目标

### 两种动作的边界

每一步开始前，先判断它属于哪一类，然后明确告诉用户。

**「你来做」——用户必须亲自完成**
- 登录 <https://open.feishu.cn/app> 创建自建应用
- 开通 scope 并发布版本
- 把应用加到目标群 / 给机器人起名字 / 第一次 @bot 触发事件
- 复制 `App Secret`（凭证只在用户屏幕上闪一次）
- `claude login`（如果 `claude` CLI 没登录过）

这些动作你说得再详细也没用——你的手伸不进他的屏幕。你能做的是：告诉他去哪点、点什么、点完之后你需要看到什么信号才能继续。

**「我来做」——你直接代劳**
- 写 `~/.towow/.env.lark`（等用户把凭证贴给你）
- 跑 `bash .claude/skills/lark-bug-pipeline/install.sh`
- `tail` / `grep` 日志做诊断
- 从 daemon 日志里抠 `bot_open_id` 回填 env 文件
- 任何文件编辑、任何 shell 命令

这些动作用户也能做，但你更快更准，而且不容易漏步骤。**不要出于「让用户学习」的理由把它们推回给用户** —— 用户的学习是副产品，不是这次会话的目的。

### 前置检查（所有后续步骤之前）

在让用户动任何东西之前，先自己跑这些验活命令：

```bash
uname                             # 必须是 Darwin
python3 --version                 # >= 3.9
python3 -c "import lark_oapi"     # 失败就 pip install --user lark-oapi
which claude                      # 空的话让用户去 claude.com/claude-code 装
claude -p "say ok"                # 非零退出说明没登录
gh auth status                    # fixer 开 PR 要用
git status                        # 必须在 git 仓库里，推荐 main 分支
```

七条里任何一条红了，先修再继续。有一个反模式叫 **「前置检查懒癌」**：先跑 install.sh，失败之后再回头补检查——比一开始就检查多花 3 倍时间。不要犯。

### 默认路径：群聊 IM-only

默认装最小路径——只装飞书 IM 群聊模式。多维表格、状态写回、看板视图都是可选升级，用户明确要求之前不要主动启用。

这意味着：
- `LARK_BUG_TABLE_TOKEN` / `LARK_BUG_TABLE_ID` 在 env 文件里 **留空**，不是让用户填占位符
- 飞书权限配置 **不需要** `bitable:*` scope、不需要创建多维表格
- 用户只需要：建自建应用 → 开 `im:*` scope → 开事件订阅长连接 → 把 bot 加群 → 在群里 @bot 发消息

这是三到五步而不是十步。不要主动说「顺便把多维表格也建了吧」，用户想要时会问。

### 节奏启发式

**默认节奏：每次只给一步。** 把下一步说清楚，等用户做完反馈，再给下一步。不要一次甩 10 步清单——用户会跳过其中 3 步然后卡在第 7 步，而你没有信号知道哪里断了。

例外：当用户明显熟练时（主动跳过你的说明、用你没教过的命令、问细节而不是问步骤），可以一次给更密的信息。

**元信号对照**：

| 用户问 | 他在做 | 你的反应 |
|---|---|---|
| "为什么要这样" | 在学习 | 可以稍微展开原因 |
| "我做完了，下一步" | 在执行 | 直接给下一步，不展开 |
| "这是什么意思" | 被术语吓到 | 换非术语的说法，**不要解释术语原理** |
| 沉默很久 | 卡住了 | 主动问「现在你看到的是什么屏幕」 |

### 水平感知（不要公开宣布）

不要在第一句话里问「你对 launchd 熟不熟」——这是个既可能暴露对方不懂、又可能让懂的人觉得被小看的问题。改为观察：

- 他粘过来的第一条命令 / 错误消息的形态
- 他用不用技术词（进程、环境变量、OAuth）
- 他的打字节奏（一堆截图 vs 一行字）

三条信号足以判断水平。判断之后 **不要公开宣布**（「我看你是个老手」是典型居高临下）—— 只是调整信息密度。

### 成功信号

**不要声称「装好了」**，直到你和用户一起看到以下两条信号：

1. **两个 LaunchAgent 都在跑**
   ```bash
   launchctl print gui/$(id -u)/net.towow.lark-daemon | grep state
   launchctl print gui/$(id -u)/net.towow.lark-worker | grep state
   # 两条都必须是 "state = running"
   ```

2. **端到端穿透**——让用户去群里 @bot 发一条假 bug（例：`@Bot 测试：登录按钮点了没反应`），然后一起 `tail -f ~/.towow/logs/lark-worker.log`。必须在 2 分钟内看到 `Triage start`，15 分钟内看到 `PR url: https://github.com/...`。

**看到 PR URL 之前不要说「搞定」**。两个服务 running 不等于链路通，中间任何一步断了都会让信号在某处停下。

### 失败诊断顺序（按故障频率降序）

端到端穿透失败时，按这个顺序查，**不要乱查**：

1. `~/.towow/logs/lark-daemon.err` 有新行吗？没有 → 飞书事件没到 → scope 或事件订阅没开对
2. `~/.towow/logs/lark-worker.err` 有 `Triage exit` 行？有 `no state file` → budget 不够或 claude CLI 没登录
3. fixer 跑完但没 PR → `gh auth status`，八成没登录 gh
4. 两个日志都静悄悄 → `launchctl print` 看 state
5. state 不是 running → `.err` 最后 30 行找崩因

**不要一上来让用户「把所有日志贴给我」** —— 这是 **「日志倾倒」反模式**。先问具体信号，按顺序缩小范围。

### 双向校准：你在两面墙之间走

下面两份清单是两面墙，帮你识别自己是不是在某一侧过头了。

**过度放手的信号**（在这样做 → 停）
- 一次甩 10 步清单让用户自己对着做
- 假设用户知道 `launchctl bootstrap` 是什么
- 用户报错时回答「你 google 一下」
- 没跑前置检查就让用户配 env
- 给链接但不说用户应该在那个页面做什么
- 飞书后台报错就让用户「去找飞书客服」
- 声称「装好了」但没看到 PR URL

**过度保姆的信号**（在这样做 → 停）
- 每一步前都问「准备好了吗」「可以继续吗」
- 解释 WebSocket / launchd / OAuth 的原理
- 用户已经贴过 App ID 了还在重复念 checklist
- 反复强调「这一步很重要哦」
- 用户跳步骤时强行拉回「我们还是按顺序来吧」
- 在用户没卡时主动展开背景知识
- 用户贴了一条报错，你先回「别担心我们一起解决」而不是先诊断
- 每次回复末尾追加「总结我刚才做了什么」

两份清单 **不对等** —— 过度保姆那一侧列得多一点。这是因为 RLHF 训练出的 AI 天然偏向过度解释、过度安抚、过度复盘。你更可能在这一侧出问题。

### 一句话总括

你的工作是 **让用户在一小时内看到第一个 PR**，不是让用户理解整套系统。理解是副产品。把用户必须自己点的按钮清楚地交给用户，把你能代劳的命令直接跑掉，中间不要拖泥带水。

---

## 文件结构

```
.claude/skills/lark-bug-pipeline/
├── SKILL.md                              # 本文件
├── install.sh                            # 一键安装
├── runtime/
│   ├── bug_daemon.py                     # 飞书 WebSocket 长连接 daemon
│   ├── bug_worker.py                     # 队列 poller + triage/fixer 编排
│   └── lark-triage.md                    # triage 子 skill 副本
├── templates/
│   ├── env.lark.example                  # 环境变量模板
│   ├── net.towow.lark-daemon.plist.tmpl  # LaunchAgent 模板
│   └── net.towow.lark-worker.plist.tmpl
└── docs/
    ├── feishu-setup.md                   # 飞书自建应用 + 多维表格建表步骤
    └── architecture.md                   # 架构细节 / 队列格式 / 扩展指引
```

---
> Source: [Towow-ai/lark-bug-pipeline](https://github.com/Towow-ai/lark-bug-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
