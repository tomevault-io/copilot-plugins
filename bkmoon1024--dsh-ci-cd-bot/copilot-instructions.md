## dsh-ci-cd-bot

> 减少常见 LLM 编码错误的行为指南。根据需要与项目特定说明合并使用。

# CLAUDE.md

减少常见 LLM 编码错误的行为指南。根据需要与项目特定说明合并使用。

**权衡：** 这些指南偏向谨慎而非速度。对于简单任务，自行判断。

## 1. 先思考再编码

**不要假设。不要隐藏困惑。暴露权衡。**

在实现之前：
- 明确陈述你的假设。如果不确定，询问。
- 如果存在多种理解，呈现它们——不要默默选择。
- 如果存在更简单的方法，说出来。在必要时提出反对意见。
- 如果有什么不清楚的地方，停下来。指出困惑之处。询问。

## 2. 简单优先

**用最少的代码解决问题。不要臆测。**

- 不添加未要求的功能。
- 不为一次性代码创建抽象。
- 不添加未被要求的"灵活性"或"可配置性"。
- 不为不可能发生的场景添加错误处理。
- 如果你写了 200 行但 50 行就够了，重写它。

问自己："资深工程师会说这过于复杂吗？"如果是，简化。

## 3. 精准修改

**只触碰你必须改的。只清理你自己造成的混乱。**

在编辑现有代码时：
- 不要"改进"相邻的代码、注释或格式。
- 不要重构没有问题的东西。
- 匹配现有风格，即使你会采用不同做法。
- 如果你注意到无关的废弃代码，提及它——不要删除。

当你的修改产生了孤儿代码：
- 移除因你的修改而变得无用的导入/变量/函数。
- 除非要求，不要移除预先存在的废弃代码。

检验标准：每一行被修改的代码都应直接追溯到用户的需求。

## 4. 目标驱动的执行

**定义成功标准。循环执行直到验证通过。**

将任务转化为可验证的目标：
- "添加验证" → "为无效输入编写测试，然后让它们通过"
- "修复 bug" → "编写一个能复现问题的测试，然后让测试通过"
- "重构 X" → "确保测试在前后都能通过"

对于多步骤任务，陈述简要计划：
```
1. [步骤] → 验证：[检查项]
2. [步骤] → 验证：[检查项]
3. [步骤] → 验证：[检查项]
```

强大的成功标准让你能独立循环。弱标准（"让它工作"）则需要不断澄清。

---

**这些指南生效的标志是：** diff 中不必要的更改更少、因过度复杂化而重写的次数更少、澄清性问题在实现之前而非犯错之后提出。

## 5. 项目特定规则

- **全程使用中文进行对话**
- **禁止主动操作 Git**：不允许主动执行 `git commit`、`git push`、`git push --force`、`git cherry-pick`、`git rebase`、`git reset --hard` 等不可逆操作。只有当用户明确要求时才可以执行
- **Windows 环境下执行命令默认使用 Git Bash**，而非 PowerShell，除非Git Bash不可用

## 6. 前端界面与文案规则

**用分区与视觉引导产品，不用描述性文字填充界面。**

- **界面文字以中文为主**：产品标题、功能标题、导航、按钮一律使用中文，不写中英对照（如 "AGENT CONSOLE"、"ROLE PROFILES" 这类英文副标题一律删除）。英文品牌名保留原样（如 MARKETPULSE）；技术专有名词保留英文原词（API Key、Base URL、Temperature、tokens 等）。
- **不在功能区写常驻的状态/说明文字**：如 "API 已连接"、"5 AGENTS" 一律不写。状态用视觉元素表达——状态点、颜色、图标；数量用徽标数字。需要补充说明时用 `title`/tooltip 悬浮提示，不占常驻位置。
- **功能描述不进界面**：像 "每次保存都会归档上一版配置" 这类解释，靠分区、导航、层级、空状态本身传达。确需说明的（危险操作确认、安全提示）才写，且要短。
- **提交前端代码前自查**：有没有中英对照标题？有没有常驻 "X 已连接 / N 个 Y" 式文案？功能区里有没有可删的功能描述？

## 7. dsh-ci-cd-bot 仓库规则

- **本仓库是 DSH Web GUI 插件**（shipped 形态，npm 包 `dsh-ci-cd-bot`）：宿主半区跑在普通 Node 进程，可自由用 node 内置 / fetch / dsh SDK 包；浏览器半区必须产出 `window.__ModuleLoader__.load` 协议包（由 `build.mjs` 包装）。
- **构建/检查命令**：`npm run build`（`build.mjs` 直调 esbuild 平台二进制，勿改回 JS API——沙箱禁管道捕获子进程输出）；`npm run typecheck`；`npm test`（Node ≥ 23.6，原生 TS 导入，11 个无头行为场景）。
- **源码兼容 Node strip-only 类型剥离**：禁止 TS 参数属性（`constructor(readonly x)`）、`enum`、`namespace`。
- **安装依赖必须带 `npm install --ignore-scripts`**（esbuild 二进制经 optionalDependencies 提供，无需 postinstall）。
- **`lib/` 构建产物随源码提交**（DSH 插件家族惯例，可直接 link 安装）。
- **测试不碰真实 GitHub / agent**：GitHub API 用 `globalThis.fetch` 打桩，agent 走 Runner 的 `runAgent` 注入 seam，git 流用临时裸仓库。
- **Token 安全**：宿主配置 `~/.dsh/dsh-ci-cd-bot.json`（0600）；Token 永不下发浏览器（GET /config 只回 `hasToken`）。
- **CLAUDE.md 与 AGENTS.md 必须逐字同源**：本仓库不使用软链，修改任何一份后必须立即同步另一份，防止两份规则分叉。
- **宿主侧程序化运行 agent 的唯一方式**：`ctx.agents.create({ meta: { cwd } })` → `followup` → `whenIdle` → 读 `session.events` 的 `turn/end.reason.kind` 判成败（subagent seam 无法指定任意 cwd），详见 `docs/design.md`。

---
> Source: [bkMoon1024/dsh-ci-cd-bot](https://github.com/bkMoon1024/dsh-ci-cd-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
