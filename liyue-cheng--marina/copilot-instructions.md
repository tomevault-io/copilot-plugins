## marina

> > 这份文件是给为 Marina 贡献代码的 AI agent(Claude Code / Cursor / Codex / 其他)看的。

# AGENTS.md — Marina 项目 AI Agent 工作说明书

> 这份文件是给为 Marina 贡献代码的 AI agent(Claude Code / Cursor / Codex / 其他)看的。
> 你正在 YOLO 模式下工作,大部分时间不需要打扰开发者。但有少数情况你必须立刻停下来。
> 仔细读完整份文件再开始工作。

文档版本:1.3 · 最后更新:2026-05-16

> **v1.3 变更**:与软件定义书 v1.6 / ADR-013 对齐 — 附录 C "关闭弹确认"硬规则增加 Linux 例外脚注:仅当 `lifecycleModel === 'no-persistence'` + 最后一个窗口 + 仍有非 exited session 时,允许弹 `<LastSessionConfirm />`。1.2 节边界 3 例子同步加例外注脚。
> **v1.2 变更**:产品改名 EasyTerm → **Marina**(对齐软件定义书 v1.5,ADR-012)。所有"产品现状"维度的 EasyTerm 字样替换为 Marina;commit message 示例等"历史/惯例"维度的 EasyTerm 保留作为风格参考。`%APPDATA%\EasyTerm\` 全部改为 `%APPDATA%\Marina\`(Electron 由 `productName` 自动派生)。
> **v1.1 变更**:与软件定义书 v1.3 / CP-4 勘误回合对齐 — CP-4 完成标志改 7 套主题;附录 D 新增"勘误回合工作纪律";4.5 章明确"勘误回合修复"也是检查点工作流的一部分。

---

## 0. 你必须先读的文件

按顺序读完以下文件,你才有足够的上下文开始工作:

1. **`docs/软件定义书.md`** — 整个产品的设计共识。读懂第 2 章(设计哲学)和第 8 章(状态机),否则你写的代码会偏离产品方向。
2. **本文件(`AGENTS.md`)** — 你现在在读的。
3. **`docs/ipc-protocol.md`** — IPC 协议规格(若存在)。如果不存在,在你开始实现 IPC 之前需要先和开发者一起定。
4. **`README.md`** — 项目的对外说明。

如果上述任何文件不存在或内容残缺,**立刻停下来通知开发者**,不要自己脑补。

---

## 1. 你的工作模式:YOLO 模式

### 1.1 什么是 YOLO 模式

你被授权在不打扰开发者的前提下,**端到端**完成 Marina 的构建。这意味着:

* **可以自主决策**:文件命名、代码组织、变量名、内部 API 设计、测试用例选择、调试策略
* **可以自主执行**:写代码、跑构建、跑测试、提交 git commit、安装(限定范围内的)依赖
* **不需要每一步问开发者**:写完一个函数不用问、起一个新文件不用问、修复一个 bug 不用问

**你的默认行为是"继续干活"**,不是"等指示"。

### 1.2 YOLO 的三条边界

YOLO 不是"想干啥干啥"。以下三条边界**永远不能越过**,越过即停止:

#### 边界 1:破坏性操作前必须停

包括但不限于:
* 删除文件(除非是你刚才自己创建的临时文件)
* `git push --force` 任何操作
* 修改 `.git/` 目录
* 删除或修改 `~/AppData/Roaming/Marina/` 下的任何文件(即使是测试)
* 卸载 npm 包
* 修改系统注册表(Windows)
* 调用任何"清理"、"重置"、"reset"、"clean"类命令对实际数据生效

遇到这些操作前,**停下来,描述你想做什么,等开发者确认**。

#### 边界 2:超出技术栈的依赖

只允许使用 `软件定义书.md` 第 10.1 节列出的技术栈。如果你觉得需要新加一个 npm 包(例如某个工具库、UI 库、状态管理库等),**停下来问开发者**。

理由:agent 历史经验里经常引入冷门 / 已废弃 / 安全风险的包,这种决策必须人来做。

允许的例外:
* `@types/*` 类型定义包,可自由安装
* 明显的小工具(如 `uuid`、`debounce`),先在对话里告知一声再装
* 测试相关包(jest 等,见第 5 章)

#### 边界 3:产品哲学

`软件定义书.md` 第 2 章的四条哲学原则和第 13.2 节"永远不做"列表是**红线**。如果你发现某个功能实现起来"如果加个 XXX 会简单很多",而那个 XXX 出现在"永远不做"列表里,**不要做,停下来问**。

例如:
* 如果你想加"应用内快捷键来切换 session",**停**(违反"鼠标优先")
* 如果你想加"主窗口"概念来简化 session 持久化,**停**(违反"窗口平等")
* 如果你想给关闭窗口加确认对话框,**停**(违反"窗口零成本开关")
  * **唯一例外(v1.6,软件定义书 ADR-013)**:Linux 上"最后一个窗口关闭 + 仍有非 exited session"时弹同一个 `<LastSessionConfirm />`。非 Linux / 非最后窗口 / 全 exited session 时仍然禁止弹任何确认。详见软件定义书第 2 章 Linux 平台例外脚注与 ADR-013。

### 1.3 YOLO 的成功标准

如果你做到了下面所有事,这次 YOLO 就是成功的:

* [ ] 严格按 `软件定义书.md` 实现所有 V1 必做功能(5.1 节)
* [ ] 没有违反"永远不做"列表(13.2 节)
* [ ] 在每个检查点(见第 4 章)按规格停下来,等开发者测试
* [ ] 后端关键模块有自动化测试覆盖(见第 5 章)
* [ ] 代码注释充分,出问题时开发者能调试(见第 2 章)
* [ ] 所有 commit 历史清晰,git bisect 可用(见第 6 章)
* [ ] 应用能正确打包成 Windows 安装包,在干净的 Windows 11 上能跑

---

## 2. 代码注释要求(关键)

### 2.1 为什么注释要求高

正常项目里,代码 review 是质量保证。**这个项目没人 review 你的代码**。
当你犯错时,开发者要在事后接手调试。如果你的代码他读不懂,他就只能扔掉重写。

所以你的代码必须满足:**任何一个有 TypeScript / Electron 经验的人,在没读过你思考过程的前提下,能在一小时内理解任何一个文件在做什么、为什么这么做、出问题时该看哪里。**

### 2.2 必须写注释的地方

#### 文件头注释(每个 .ts / .tsx 文件必须有)

```typescript
/**
 * @file session-manager.ts
 * @purpose 管理所有 PTY 会话的生命周期(创建、活跃/空闲检测、墓地、销毁)
 *
 * @关键设计:
 * - 每个 Session 在守护进程内是单例,owner_window_id 可为 null
 * - PTY 进程退出后进入"墓地"5 分钟,期间用户可恢复
 * - 字节流通过 IPC 推送给 owner window,无 owner 时仍写 scrollback
 *
 * @对应文档章节:软件定义书.md 第 5.1.2、8.3 节
 *
 * @不要在这里做的事:
 * - 不要解析 OSC 1337(那是 cwd-tracker.ts 的职责)
 * - 不要持久化 session(session 不持久化,设计如此)
 * - 不要管理 path 归属(那是 path-manager.ts 的职责)
 */
```

#### 函数注释(公开函数必须有,私有复杂函数必须有)

```typescript
/**
 * 创建一个新 session 并启动 PTY。
 *
 * @param pathId 该 session 启动时的工作目录所对应的 path id
 * @param templateId 启动模板 id,从 templates.json 读
 * @param ownerWindowId 创建该 session 的窗口 id,设为初始 owner
 * @returns 创建的 SessionInfo
 *
 * @throws 'PathNotFound' 如果 pathId 不存在
 * @throws 'TemplateNotFound' 如果 templateId 不存在
 * @throws 'PtySpawnFailed' 如果 node-pty 启动失败(常见原因:cwd 不存在、shell 不存在)
 *
 * @副作用:
 * - 创建 node-pty 子进程
 * - 把 session 加入 path 的 session 列表
 * - 触发 path 状态机:可能让 path 进入"临时"分类
 * - 广播 pathTreeUpdated 和 sessionStateChanged 事件
 *
 * @常见问题排查:
 * - 如果 PTY 启动后立即退出 → 检查 shell 路径、cwd 权限
 * - 如果 OSC 1337 hook 不生效 → 检查 templateId 对应的 shell hook 文件是否注入
 */
async function createSession(
  pathId: string,
  templateId: string,
  ownerWindowId: string
): Promise<SessionInfo> {
  // ...
}
```

#### 复杂逻辑必须有"为什么这么写"的注释

不是写"这行代码做什么",而是写"为什么这么做,不那么做"。

```typescript
// 我们用 setImmediate 而不是 process.nextTick,因为 nextTick 优先级太高,
// 会饿死后续的 PTY data 事件,导致字节流堆积。这是踩过坑的。
setImmediate(() => {
  this.broadcastPathTreeUpdate();
});

// node-pty 的 onData 回调可能在 PTY 已经标记 destroyed 之后还触发一次
// (经验:Windows ConPTY 的关闭是异步的)。所以这里要 guard。
if (this.destroyed) return;
```

#### 状态机相关代码必须有状态图引用

任何涉及到状态转移的代码,在该函数顶部用注释画出 mini 状态图,或引用文档:

```typescript
/**
 * Session 状态转移逻辑。
 *
 * 状态机参见 软件定义书.md 第 8.3 节(v1.2 ADR-008 后)。
 * 简化:
 *   active <--有/无字节流--> idle
 *   active/idle --PTY 进程退出--> exited (灯显灰,scrollback 保留,无时限)
 *   exited --用户右键关闭 / 应用退出--> destroyed
 *   注:exited 不可回到 active(无重启路径,见 ADR-008)
 */
```

### 2.3 不要写的注释

* `// 设置 x 为 5` ← 重复代码本身,删掉
* `// TODO: fix this later` ← 要么修,要么写明白具体要修什么、为什么不现在修
* 注释掉的代码 ← 直接删,git 里有历史

### 2.4 错误信息要详细

任何你 throw 的 Error 必须包含:
* 出错的操作名
* 出错的关键参数值
* 可能的原因(至少猜两个)
* 建议的下一步

```typescript
// 不好
throw new Error('Failed to create session');

// 好
throw new Error(
  `[SessionManager] Failed to spawn PTY for templateId="${templateId}" cwd="${cwd}". ` +
  `Possible causes: (1) shell binary not found at "${shellPath}", ` +
  `(2) cwd does not exist or no permission, ` +
  `(3) node-pty native module not built for current Node version. ` +
  `Check logs in ~/AppData/Roaming/Marina/logs/main.log for stderr.`
);
```

### 2.5 日志要详细

主进程任何关键路径必须有日志,日志要带模块名 + 关键参数。开发者排查问题时,日志是他的眼睛。

```typescript
log.info(`[SessionManager] createSession: pathId=${pathId} template=${templateId} ownerWindow=${ownerWindowId}`);
log.info(`[SessionManager] PTY spawned: sessionId=${session.id} pid=${pty.pid}`);
log.warn(`[SessionManager] PTY exited unexpectedly: sessionId=${session.id} exitCode=${exitCode}`);
log.error(`[SessionManager] Failed to create session`, error);
```

日志级别:
* `debug` — IPC 消息内容、PTY 字节流(只在 settings.advanced.logLevel = DEBUG 时启用)
* `info` — 重要状态变化(session 创建/销毁、窗口开关、设置变更)
* `warn` — 异常但不致命(PTY 异常退出、单个 IPC 消息失败)
* `error` — 致命错误(无法启动守护进程、数据文件损坏)

---

## 3. 调试中的"10 轮规则"(关键)

### 3.1 什么算"一轮调试"

一轮调试 = 你做了下面任意一件事:
* 修改了代码并重新跑(无论是测试还是手动跑)
* 加了一个 log 重跑
* 改了一个配置重跑
* 试图复现一个错误

### 3.2 触发"10 轮规则"

如果你在调试**同一个问题**(同一个 bug、同一个失败的测试、同一个不工作的功能)超过 10 轮还没解决,**立刻停下来**。

具体触发条件(满足任意一个):
* 同一个测试用例失败了 10 次以上
* 你修改同一段代码超过 10 次,问题仍在
* 你的对话里(或 commit message 里)反复出现"修复 X 问题"超过 10 次
* 你怀疑过 5 个以上不同根因,都不对

### 3.3 触发后该做什么

**不要继续猜**。立刻执行:

1. **停止所有修改**,把当前代码状态保留下来(不要回退)
2. 写一份 `BLOCKED.md` 文件放在仓库根目录,内容包含:
   - 问题简要描述
   - 你尝试过的所有思路(每条一行)
   - 你当前最强的怀疑(以及为什么没修好)
   - 你认为开发者应该看哪几个文件
   - 复现步骤
3. 在终端 / 当前对话里输出明显的求助信号:`🛑 BLOCKED: 调试 10 轮未果,需要开发者介入。详见 BLOCKED.md`
4. **等待**。不要再尝试修。

`BLOCKED.md` 模板:

```markdown
# BLOCKED: [问题一句话描述]

**触发时间**:2026-XX-XX
**调试轮次**:11+
**所在阶段**:Week X, 实现到第 Y 个 SKILL

## 问题表现
[具体的错误信息 / 行为描述]

## 复现步骤
1. ...
2. ...

## 我尝试过的思路
- [ ] 尝试 1:...(结果:...)
- [ ] 尝试 2:...(结果:...)
- [ ] 尝试 3:...(结果:...)
...

## 我目前最强的猜测
[为什么这是猜测?为什么修了还不对?]

## 关键文件
- src/main/xxx.ts:第 Y 行
- ...

## 相关日志
[贴出最具诊断价值的 5-20 行日志,不要全贴]
```

### 3.4 为什么有这条规则

经验告诉我们:agent 在卡死之后,继续往前冲只会让代码越来越糟、越来越难诊断。10 轮是一个保守的上限 — 大多数问题在 3-5 轮就该解决,如果 10 轮还没解决,通常说明你**对问题的理解从一开始就是错的**,这种情况下人介入比 agent 继续猜效率高 100 倍。

不要把这条规则当成"我要努力撑到第 10 轮"。如果你 5 轮就感觉走投无路,提前 BLOCKED 也是合理的。

### 3.5 不属于 10 轮规则的情况

以下不算"卡 10 轮":
* 你在做不同的功能,每个都跑了几次测试 — 这是正常工作,不是卡死
* 你在调试不同的 bug,每个 1-2 轮解决 — 这是正常工作
* 测试基础设施在反复跑(CI 失败重跑)— 不是逻辑卡死

10 轮规则**只针对单一问题反复修而不解决**的情况。

---

## 4. 检查点(关键)

### 4.1 检查点的作用

Marina 的开发被切分成 **N 个检查点**,与 `软件定义书.md` 第 15 章的 Phase 1 路线图对齐。

在每个检查点,**你必须停下来**:
1. 完成一份"自测报告"(你自己先冒烟测试一遍)
2. 完成一份"用户测试指南"(教开发者怎么测)
3. 在终端 / 对话里明显标记:`🛏️ CHECKPOINT N: 等待开发者测试`
4. **不许继续往下做下一个 Phase 的工作**,即使你觉得自己有空

开发者会:
* 按你的指南测试
* 反馈"通过"或"问题清单"
* 通过后,授权你进入下一个检查点

### 4.2 检查点列表

#### 检查点 1:技术骨架可跑(对应 Phase 1 / Week 1-2)

**目标**:有一个能跑的最小 Electron 应用。

**完成标志**:
- [ ] `npm install && npm run dev` 能在 Windows 上启动应用
- [ ] 能看到一个窗口,内含一个 xterm.js 实例
- [ ] xterm 里能正确显示 PowerShell 提示符,能输入命令并看到输出
- [ ] 关闭窗口 → 应用进入纯托盘模式(托盘图标还在,任务管理器里 Marina.exe 还在)
- [ ] 单击托盘图标 → 重新打开一个窗口(窗口编号变了)
- [ ] 右键托盘 → 看到菜单,菜单里"完全退出"能真正退出应用
- [ ] 启动第二次 Marina.exe → 在已运行实例上新开一个窗口(单实例锁工作)
- [ ] 至少 3 个 main 进程模块的单元测试存在并通过

**用户测试指南必须包含**:
1. 如何运行(precise commands)
2. 上述每一条 checkbox 对应的具体测试步骤(点哪里、按什么键)
3. 预期结果 vs 失败现象
4. 失败时去哪个日志文件看(精确路径)

#### 检查点 2:核心数据模型 + 多窗口(对应 Phase 1 / Week 3-4 前半)

**目标**:三栏侧栏可见,Path 状态机工作,多窗口共享数据。

**完成标志**:
- [ ] 侧栏显示"收藏 / 临时 / 最近"三栏(均为空时也显示)
- [ ] 能通过 "+" 按钮选文件夹加入收藏,关闭再开应用,收藏还在
- [ ] 能通过 Explorer 拖文件夹到侧栏加入收藏
- [ ] 在某收藏路径双击 / 单击新建终端按钮 → 该路径下出现一个 session
- [ ] 关闭那个 session → 路径仍在收藏里(因为是收藏)
- [ ] 在某非收藏路径新建终端 → 路径自动出现在"临时"分类
- [ ] 关闭该路径所有终端 → 该路径自动从"临时"移到"最近"
- [ ] 开第二个窗口(从托盘菜单)→ 第二个窗口看到相同的侧栏数据
- [ ] 在窗口 A 改设置(主题)→ 窗口 B 立即同步
- [ ] 关闭窗口 A 时持有的所有 session → 在窗口 B 里那些 session 变成"无 owner",可以接管
- [ ] 后端核心模块测试覆盖率 > 70%

#### 检查点 3:Session 完整 + cwd 跟踪(对应 Phase 1 / Week 3-4 后半)

**目标**:Session 状态机完整,cwd 跟踪工作,启动模板可用。

> v1.2 起本检查点的部分要求已修订,详见软件定义书 ADR-008:path 与 cwd 解耦、砍墓地。

**完成标志**:
- [ ] Session 有"活跃 / 空闲 / 已退出"三种状态显示,自动切换
- [ ] 在 session 里 `cd` 到另一个路径 → 该 session 在所属路径下不动,但其标签出现 ⚠️ 提示真实 cwd
- [ ] Session 进程退出后,标签灯显灰色 ⚫,scrollback 完整保留;**无时限自动消失**(用户右键"关闭"才销毁)
- [ ] 启动模板有 4 个内置:Shell / Claude Code / Codex / OpenCode(命令可执行,即使 claude 实际未安装也要能尝试启动并报错)
- [ ] 在收藏路径设置默认模板,双击该路径直接启动该模板
- [ ] OSC 1337 hook 注入对 PowerShell 工作(更新 session.currentCwd,不动 path 树)
- [ ] OSC 1337 hook 注入对 cmd.exe 工作
- [ ] OSC 1337 兜底:启动后 5 秒内若未收到任何 OSC,启动 NtQueryInformationProcess 轮询;收到首条后关闭轮询
- [ ] scrollback 2MB 环形缓冲(尾部裁切),owner 切换/接管时一次性推给 renderer
- [ ] 状态机相关模块测试覆盖率 > 80%

#### 检查点 4:UI 完整 + 主题 + 设置(对应 Phase 1 / Week 5-6)

**目标**:产品对外可用版本。

**完成标志**:
- [ ] **7 套**主题都可切换,即时生效,xterm 颜色与 UI 同步(v1.3 起:Rose Pine / Rose Pine Dawn / Rose Pine Moon / Cutie / Business / Ubuntu / Windows Terminal)
- [ ] 设置页面 7 个分类都可访问,所有 V1 设置项都工作
- [ ] 设置即改即生效,无保存按钮
- [ ] 跨窗口设置同步
- [ ] 终端右键菜单(复制 / 粘贴 / 清屏 / 搜索)工作
- [ ] 终端搜索(Ctrl+F)工作 — 搜索栏显示命中数 `current/matches`(SearchAddon `onDidChangeResults`)
- [ ] Ctrl+F / Esc 通过 `term.attachCustomKeyEventHandler` 拦截,**不**透传成 ^F / 0x1B 给 PTY
- [ ] 多行粘贴前弹原生 `confirm`(行数 + 200 字预览),用户确认后再写 PTY
- [ ] 字体下拉枚举 `window.queryLocalFonts()`,推荐组置顶 + 系统已装组(main 端 `setPermissionRequestHandler` 自动放行 `local-fonts`)
- [ ] UI 系统图标走 lucide-react(不再 emoji);用户数据(Template.icon)保持 emoji
- [ ] 选中即复制 + 右键弹菜单的两种行为都工作
- [ ] 完全退出前的二次确认弹窗工作
- [ ] 关闭单个窗口绝不弹任何对话框(已验证)
- [ ] 启动模板编辑子页面工作(增删改自定义模板)
- [ ] 数据导出 / 导入工作 — **导入走 in-memory replace**(Manager.replaceAll + emit),不调 `app.relaunch`,运行中 PTY 不被关(ADR-009)
- [ ] CSP 通过 main 进程 `webRequest.onHeadersReceived` 注入(dev 含 unsafe-eval 给 React Refresh,prod 严格);移除 `index.html` 的 meta CSP
- [ ] 终端状态机有兜底:`createSession` 末尾立即 `scheduleIdleCheck()`,首波 PTY 数据是纯 OSC 时不会卡 active(CP-4 勘误 #5)
- [ ] 应用打包(`npm run build`)产生 Windows 安装包(.exe / .msi)
- [ ] 在干净的 Windows 11 虚拟机或机器上,安装该包,能正常启动并运行
- [ ] 后端整体测试覆盖率 > 75%

#### 检查点 5:开源准备(对应 Phase 2)

**目标**:可以公开发布的状态。

**完成标志**:
- [ ] README.md 中英双语完整(含截图)
- [ ] CONTRIBUTING.md 完整
- [ ] CHANGELOG.md 有 v1.0.0 条目
- [ ] LICENSE 文件存在(MIT)
- [ ] 所有代码注释中无脏话、无内部缩写、无敏感信息
- [ ] `.gitignore` 完整(node_modules / dist / 用户数据等)
- [ ] GitHub Actions CI 配置就位:lint + test + build for Windows
- [ ] 第一个 GitHub Release 草稿就位

### 4.3 检查点之间的工作纪律

* **不要在检查点 N 之前做检查点 N+1 的工作**。即使代码上看起来很容易顺手做完。
* **每个检查点的代码,在通过之前不能往 main 分支 merge**。每个检查点用一个 feature branch:`checkpoint-1`、`checkpoint-2`...
* **检查点失败时,你修复后必须重跑全部完成标志**,不能只修被指出来的那一项。
* **检查点的"自测报告"是必须做的**。你不能让开发者帮你找编译错误。

### 4.4 自测报告 vs 用户测试指南

每个检查点你提交两个东西:

#### 自测报告(`docs/checkpoints/CP-N-self-test.md`)

你自己跑过哪些测试,结果如何。这是给开发者看的"我做完了什么"。

```markdown
# CP-1 自测报告

## 跑过的测试
- [x] `npm test` 通过(35/35)
- [x] `npm run dev` 启动后,xterm 显示 PowerShell 提示符,输入 `dir` 有输出
- [x] 关闭窗口后,任务管理器查看 Marina.exe 仍在
- ...

## 已知不工作的事(需要开发者关注)
- 第二次启动 Marina.exe 时,新窗口的位置和第一个重叠了 → 这个属于 CP2 的窗口位置记忆功能,目前先不修
- ...

## 我没测的东西(需要开发者帮忙)
- 没有干净 Windows 11 虚拟机环境,无法验证安装包在新机器上工作
- ...
```

#### 用户测试指南(`docs/checkpoints/CP-N-user-test-guide.md`)

教开发者一步一步怎么测。**写得像给一个不熟悉这个项目的人看**。

```markdown
# CP-1 用户测试指南

## 准备
1. 确保 Node.js 20+ 已安装(`node --version`)
2. 在项目根目录跑 `npm install`(首次约 2-3 分钟)

## 测试 1:基础启动(预计 1 分钟)
1. 跑 `npm run dev`
2. 等待 ~10 秒,应该看到一个 Marina 窗口出现
3. 窗口右半部分应有一个黑色终端区域,显示 PowerShell 提示符,如 `PS C:\Users\xxx>`
4. **预期**:你能在终端里输入 `dir` 并看到当前目录列表
5. **失败现象**:窗口空白 / 终端显示乱码 / 终端输入无反应
6. **失败时**:打开 `~/AppData/Roaming/Marina/logs/main.log`,把最后 50 行贴给 agent

## 测试 2:窗口与应用解耦(预计 30 秒)
1. 关闭那个窗口(点右上角 ×)
2. 看 Windows 系统托盘(右下角),应该有一个 Marina 图标
3. 打开任务管理器,搜索 "Marina",应该看到 Marina.exe 进程仍在
4. **预期**:窗口关了,但应用还在
5. 单击系统托盘的 Marina 图标
6. **预期**:重新出现一个窗口,标题栏写 "Window 2"(不是 Window 1)

## 测试 3 ~ 测试 N:...

## 全部通过后
回复 agent:"CP-1 通过,可以开始 CP-2"
```

### 4.5 检查点的等待行为

当你标记 `🛏️ CHECKPOINT N: 等待开发者测试` 后:
* **不要往下做事**
* **不要"乘等待之机"重构旧代码**
* **可以**做的事:整理文档、更新 CHANGELOG、给现有代码补注释、修明显的 lint 警告
* 收到开发者"通过 CP-N"后,才能开始 CP-(N+1)
* 收到开发者"CP-N 失败,xxx 不工作"后,先修复,再次自测,再次提交检查点

### 4.6 勘误回合(checkpoint errata)

每个检查点开发者测试后,如果发现一批问题,会创建 `docs/cp{N}勘误.md` 把问题整理成自由文本。**勘误回合是检查点工作流的一部分,不是另一种特殊状态**。

工作纪律:
1. 通读勘误每一条(顺序无关),把每一条转成 task(`TaskCreate`),按用户原文写描述。
2. 若涉及包/技术栈选择(图标库、新依赖等),用 `AskUserQuestion` 在动手前对齐方向 — 这是边界 2 的延伸。
3. 实现完每条后 `TaskUpdate completed`;不能"批量做完一起更新",会丢追溯。
4. 至少跑过一次 `npm run typecheck` + `npm test` + `npm run lint`,通过再回报。
5. 把这次回合的总结沉淀到:
   - **软件定义书**:对应章节用"v{X.Y} 起 / CP-{N} 勘误"标注的方式更新,新增 ADR(若涉及决策变更)。
   - **AGENTS.md**:CP-{N} 完成标志列表把变更条目补上。
   - **CP-{N}-self-test.md**:在文末加"勘误回合修复"章节(参考 CP-3 自测报告的写法)。
6. 已通过的检查点代码"封箱"规则(第 7 章)在勘误回合内**临时解冻** — 你被授权改它们,因为正是它们出了问题;改完仍在同一 feature branch,不要新开。

历史回合参考:
* `docs/ch2勘误.md` — CP-2(banner / 主题切换 / 标签顺序)
* `docs/cp3勘误.md` — CP-3(banner 完全砍 / 状态点输出驱动 / tab 闪绿 / 右键无反应 / 标签顺序)
* `docs/cp4勘误.md` — CP-4(滚动条主题 / 移除跟随系统主题 / 字体真枚举 / 多行粘贴 / 图标库 / in-memory 导入 / CSP)

---

## 5. 自动化测试要求

### 5.1 后端必须有测试,前端不需要

**必须有测试**:
* `src/main/` 下所有非平凡逻辑模块
* `src/shared/` 下所有共享类型 / 协议 / 工具

**不需要写测试**(也不要写):
* `src/renderer/` 下任何代码(UI 由人工测)
* `src/preload/` 下的代码(简单转发)

理由:
* 主进程是产品的"大脑",数据状态机和 PTY 管理出错代价大,自动测试性价比高
* UI 测试在 Electron 里成本高(需要 spectron/playwright + Electron 的兼容性),收益低于人工测,资源不应花在这里
* 人工测试反而对 UI 更好,因为人能感知"丑"和"反人类",自动测试感知不到

### 5.2 测试栈

* **测试框架**:Vitest(若 Jest 配置更顺手则用 Jest,你决定)
* **mock 库**:Vitest 内置或 sinon
* **PTY 测试**:用 mock,不要 spawn 真的 PowerShell(慢且不稳)
* **文件系统测试**:用 `memfs` 或临时目录,不要写真实数据目录

### 5.3 必须测试的场景(后端)

#### 状态机类(必测)
* Path 状态机的所有转移(收藏 ↔ 临时 ↔ 最近)
* Session 状态机的所有转移(active ↔ idle → exited → destroyed,v1.2 ADR-008 后)
* 应用生命周期状态机(启动 → 有窗口 ↔ 纯托盘 → 退出)

#### 核心管理器(必测)
* SessionManager:创建 / 销毁 / 状态查询 / owner 切换
* PathManager:增删改 / 自动分类 / 容量限制(最近最多 30 个)
* SettingsManager:读 / 写 / 验证 / 默认值合并 / 版本迁移
* WindowManager:窗口编号分配 / owner 关系维护

#### 协议类(必测)
* IPC 消息的序列化 / 反序列化
* 消息 schema 验证

#### 持久化类(必测)
* 原子写(写临时文件 → rename)
* 损坏恢复(JSON 损坏时回退到 .bak / 默认值)
* 版本迁移

#### 解析类(必测)
* OSC 1337 序列解析
* PTY 字节流的状态识别(active / idle 阈值)

### 5.4 不需要测试的(后端)

* 第三方库的 wrapper(如 node-pty 的简单封装)
* 简单的 getter / setter
* IPC handler 仅做转发的部分(转发逻辑测,handler 本身不测)
* `console.log` 等纯日志代码

### 5.5 测试覆盖率目标

* CP-2 时:核心数据模块 > 70%
* CP-3 时:状态机模块 > 80%
* CP-4 时:整个 `src/main/` > 75%

不追求 100%,追求**关键路径有保护**。

### 5.6 测试要"会出错"

测试不仅测 happy path,还要测:
* 错误输入(null、undefined、错误类型、超长字符串)
* 边界条件(0 个 / 1 个 / N 个 session;路径数量到 30 上限)
* 并发(同一个 session 被两个窗口同时 claim)
* 异常(PTY 启动失败、JSON 损坏、磁盘写失败)

### 5.7 跑测试

* `npm test` 跑所有测试
* `npm run test:watch` 开发时用
* CI 必须跑测试,失败必须阻止 merge

---

## 6. Git 提交纪律

### 6.1 commit 颗粒度

**每个独立的功能点 / bug 修复 / 重构,一个 commit**。

不要 "一天一个 commit 包含所有改动"。理由:出问题后 `git bisect` 是开发者唯一的救命稻草。

例子:
* 好:`feat(session-manager): add session creation` + `feat(session-manager): add tombstone logic` + `fix(session-manager): handle PTY exit before owner assigned`
* 坏:`progress on session manager`

### 6.2 commit message 格式

用 Conventional Commits:

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

**type**:`feat` / `fix` / `refactor` / `test` / `docs` / `chore` / `perf`
**scope**:`main` / `renderer` / `session` / `path` / `tray` / `settings` / `ipc` / `theme` 等
**subject**:50 字以内,陈述句,小写开头

例子:
```
feat(session): add tombstone state with 5-minute retention

Session 进程退出后不立即销毁,保留 5 分钟以便用户从
墓地恢复。计时器在 SessionManager 内集中管理,避免每
个 session 各自起 setTimeout。

对应 软件定义书.md 8.3 节。
```

### 6.3 分支策略

* `main`:只接受通过检查点的代码
* `checkpoint-N`:每个检查点一个分支
* `fix/xxx`:checkpoint 失败后的修复分支,基于 `checkpoint-N`

不要直接在 `main` 上写代码。

### 6.4 不要 force push

`git push --force` 任何分支都需要先停下来问开发者。

### 6.5 commit 前必须

* 跑过 `npm run lint`,无错误(警告可接受)
* 跑过 `npm test`,通过
* 代码格式化(prettier)

---

## 7. 不许重构已通过的代码

### 7.1 规则

一旦一个检查点通过,该检查点对应的代码就是"已封箱"的。在后续检查点,你**不许重构这些代码**,除非:

* 后续功能强制要求改它(比如 CP-2 加多窗口必须改 CP-1 的窗口管理代码)
* 它是个 bug 阻碍当前工作(那就修 bug,不是重构)
* 开发者明确指示"重构 X"

不许的"重构"包括:
* "我觉得这里命名不够好,改一下"
* "把 callback 改成 async/await"
* "把这段抽出一个工具函数"
* "顺便加个抽象层方便以后"

### 7.2 为什么

* agent 重构经常引入 regression
* 没人 review,引入了你也不知道
* 检查点已经测过了,改了等于让开发者重测一遍

### 7.3 例外:注释和格式

可以做的:
* 给已有代码补注释(满足第 2 章要求)
* 跑 prettier 格式化
* 修明显的 typo

这些不算重构。

---

## 8. 平台抽象与跨平台

### 8.1 V1 只测 Windows,但代码必须 platform-aware

`软件定义书.md` 第 12 章定义了 PlatformAdapter 抽象。

**你必须**:
* 所有平台特定 API 通过 `src/main/platform/` 调用
* `windows.ts` 完整实现
* `macos.ts` 和 `linux.ts` 留占位 throw `Not implemented`
* 不要往 `windows.ts` 之外的地方写 `process.platform === 'win32'` 之类的判断
* 测试时,如果某测试需要 platform adapter,用 mock,不要直接调 windows.ts

### 8.2 不要"顺手"实现 macOS 或 Linux

即使你觉得"这个 API node 标准库就有,顺便做了 Linux 也行",**也不要做**。

理由:
* 你不会在 macOS / Linux 上测,实现了等于发布未经测试的代码
* 这违反"V1 只 Windows"的承诺
* 留给社区贡献者去做,有意义的开源协作

如果你忍不住,先在 PR / commit 中提议,等开发者明确同意。

---

## 9. 数据安全

### 9.1 不许碰用户的真实数据

测试 / 调试 / 开发过程中,**永远不要**:
* 删除 `~/AppData/Roaming/Marina/` 下任何文件
* 修改 `~/AppData/Roaming/Marina/` 下任何文件(除非是应用本身的正常写入)
* 用真实的 Marina 数据目录跑测试

测试用临时目录或内存文件系统:
```typescript
// 测试里这么用
const tempDir = await fs.mkdtemp(path.join(os.tmpdir(), 'marina-test-'));
// 测试结束清理
await fs.rm(tempDir, { recursive: true, force: true });
```

### 9.2 不许提交敏感数据

* `.env`、`.env.local` 加入 `.gitignore`
* 不要在代码 / 注释 / commit message 里写真实路径(用 `~/projects/example` 这种占位)
* 不要写 API key / token / 密码,即使是注释里的"示例"
* 不要把测试用的真实输出贴进 commit message

### 9.3 测试不许跨进程影响

如果你的测试需要 spawn 真实进程,只用 mock 或在隔离环境(如 Docker)。绝不能让测试 spawn 真的浏览器、shell、应用。

---

## 10. 性能要求(底线)

V1 只是底线,不追求极致:

* 启动到第一个窗口可见 < 3 秒(冷启动)
* 创建一个 session 到 PTY ready < 1 秒
* PTY 字节流到 xterm 显示的延迟 < 50ms(目测无延迟)
* 设置变更到所有窗口同步 < 200ms
* 内存:稳态(10 个 session,2 个窗口)< 500MB

如果你发现某个操作明显慢(超出上述底线 2 倍以上),停下来评估是否设计有问题。不要"先实现着,以后再优化"。

不要在没问题的地方做性能优化(过度抽象、缓存、异步队列等)。**保持简单是最好的性能策略**。

---

## 11. 工作流总结

正常工作流(不出问题情况):

```
1. 读 docs/软件定义书.md + AGENTS.md(本文件)+ ipc-protocol.md
2. 切到 checkpoint-1 分支,开始干活
3. 写代码 + 写测试 + commit(细颗粒)
4. 完成 CP-1 所有完成标志
5. 自测(写 docs/checkpoints/CP-1-self-test.md)
6. 写用户测试指南(docs/checkpoints/CP-1-user-test-guide.md)
7. push checkpoint-1 分支
8. 输出 🛏️ CHECKPOINT 1: 等待开发者测试
9. ★ 等待 ★(可以做小的注释 / 文档整理)
10. 收到 "CP-1 通过" → 切到 checkpoint-2,继续
11. 收到 "CP-1 xxx 不工作" 或 "看 docs/cp1勘误.md" → 进入勘误回合(见 4.6),
    通读勘误 → 转 task → 修复 → typecheck/test/lint → 文档同步 → 再次回报
12. 重复直到 CP-5 完成
13. 输出 🎉 ALL CHECKPOINTS COMPLETE: Marina V1 构建完成
```

异常工作流(出问题):

```
A. 调试某 bug 到第 8 轮,感到走投无路
   → 主动 BLOCKED,写 BLOCKED.md,等开发者(不必撑到 10 轮)

B. 想加一个超出技术栈的依赖
   → 在对话里说"想装 X 包,理由 Y",等开发者回复

C. 发现某 V1 必做功能在文档里描述模糊
   → 在对话里说"软件定义书 X 节对 Y 描述不清,我的猜测是 Z,继续吗",等开发者回复

D. 在执行过程中发现产品哲学冲突
   → 立刻停下来描述冲突,等开发者裁决,不要自己绕过去

E. 检查点之间想"顺手"重构 / 加新功能
   → 不要做,记到 BACKLOG.md 里待开发者评估
```

---

## 12. 关键提醒(给你的最后嘱托)

如果你在工作中陷入困惑,回到这几条:

1. **你的工作不是"写出最优雅的代码"**,而是"实现一个可用的 V1 Marina,让开发者接手后能维护"。
2. **写注释不是浪费时间**。开发者读不懂你的代码 = 你白干了。
3. **YOLO 不是"不顾后果"**,而是"在边界内自主"。边界(技术栈 / 哲学 / 破坏性操作)永远不能越。
4. **检查点是给你机会校准方向**,不是麻烦事。一次跑偏到 CP-4 才发现,损失大于 4 次检查点的开销之和。
5. **10 轮规则保护你和开发者**。不是失败,是诚实。
6. **测试是写给未来的你**,不是写给开发者看的。当你在 CP-3 改了 SessionManager 时,CP-2 写过的测试会告诉你有没有破坏旧逻辑。
7. **当你不确定时,问开发者比猜测好**。问的代价 = 一条对话消息;猜错的代价 = 几小时返工 + 信任流失。

---

## 附录 A:文件 / 目录权限速查

| 路径 | 你能读吗 | 你能写吗 | 你能删吗 |
|------|---------|---------|---------|
| 项目根目录及子目录 | ✅ | ✅ | ⚠️ 仅限你自己创建的临时文件 |
| `node_modules/` | ✅ | ⚠️ 通过 npm | ⚠️ 通过 npm |
| `.git/` | ✅ | ❌ | ❌ |
| `~/AppData/Roaming/Marina/` | ⚠️ 仅 Marina 应用代码 | ❌(测试不许碰) | ❌(永远不许) |
| 系统 / 注册表 | ⚠️ 通过 PlatformAdapter | ❌(只在 V1.2 启用 Explorer 集成时可,现在不许) | ❌ |
| 任意 `~/projects/*` 等用户文件 | ❌ | ❌ | ❌ |

## 附录 B:你可能想问但没必要问的事

* **"我能用 X 这个工具吗"** → 看第 1.2 节边界 2,只要不超出技术栈、不安装新包,就能用
* **"代码风格用什么"** → ESLint + Prettier,跟仓库 `.eslintrc` 和 `.prettierrc` 走
* **"要不要加国际化"** → V1 中英双语,UI 文字写中文,英文版稍后(看具体 SKILL 决定)
* **"npm install 失败怎么办"** → 检查 Node 版本(20+),清 `node_modules` 重装,还不行就 BLOCKED
* **"我能跑 npm run build 吗"** → 能,但产物只放在 `dist/`,不许往项目根目录拷
* **"提交时可以用 emoji 吗"** → commit subject 不要,body 偶尔可以(但别滥用)

## 附录 C:你绝对不能问的事(因为答案永远是不)

* "能不能加一个 vim 模式快捷键"(违反哲学,永远不)
* "能不能加 project / workspace 概念让 session 持久化"(违反哲学,永远不)
* "能不能让 main window 在关闭时弹确认"(违反哲学,永远不,而且根本没有 main window)
  * **唯一例外(v1.6,软件定义书 ADR-013)**:Linux 上当**最后一个窗口**关闭且**仍有非 exited session**时,弹同一个 `<LastSessionConfirm />` modal。Windows 关窗 / macOS 关窗 / 任何非最后窗口的关闭 / 全 exited session 的最后窗口关闭 —— **仍然永远不弹**。
* "我能不能 force push"(永远不,除非开发者明确叫你做)
* "我能不能为了过测试 mock 掉这个核心模块"(永远不,这是自欺欺人)

---

**说明书结束**

> 这份说明书会随 Marina 项目演化而修订。如果你看的版本是 1.0,而 git 里有更新版本,以最新版本为准。
>
> 当你完成 V1 构建,你的最后一个 commit 应该是更新本文件的 "Last Updated" 字段并加一行:
> "构建完成于 2026-XX-XX,by [agent identifier]"。

---
> Source: [Liyue-Cheng/marina](https://github.com/Liyue-Cheng/marina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
