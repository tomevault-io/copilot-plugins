## super-agent-memory-manager

> Agent 分层记忆管理技能（v3，适配 OpenClaw 原生记忆结构）。当用户或 Agent 需要执行以下任何操作时必须使用本技能：写入或更新长期记忆（MEMORY.md）、更新用户信息（USER.md）、生成或追加每日日志、执行记忆整合与瘦身、诊断记忆健康度、安装自动化 Hook。v3 在 v2 自进化三件套基础上新增三项关键能力：(1) Hook 自动激活——通过 OpenClaw hooks 机制在每次 Prompt 提交后静默评估、工具出错后立即记录，无需用户手动唤起 evolve；(2) 候选池三次晋升——新观察到的经验先写入 CANDIDATES.md 缓冲区，同一模式被观察满 3 次才提请用户审批晋升到 MEMORY.md，防止单次偶发指令被误固化；(3) 行级来源引用——Agent 依据记忆执行操作时必须标注来源文件及行号（格式：📍 MEMORY.md:L23），实现完整可溯源链路。本技能直接操作 OpenClaw 的原生记忆文件，不创建额外独立目录。触发词包括：初始化记忆、写入记忆、更新 memory、更新 USER.md、记录这件事、写日志、记忆整合、memory 整理、memory 健康检查、自动进化、进化请求、候选池、从这次对话中学到了什么、安装 hook。


# Memory Manager Skill（v3）

**设计理念**：记忆系统是 Agent 的"操作系统内核"——必须始终精简，按需扩展，并对用户完全透明。v3 的核心升级在于：从"被动等待用户要求记录"进化为"Hook 驱动的主动感知 + 候选池防误学 + 操作全程可溯源"。

---

## OpenClaw 记忆结构适配

本技能直接操作 OpenClaw 的原生记忆文件，**不创建独立的 `.agent-memory/` 目录**。

```
~/.workbuddy/                          ← 全局身份层（只读为主）
├── SOUL.md                            ← Agent 灵魂/价值观（memory-manager 不修改）
├── IDENTITY.md                        ← Agent 身份/风格/emoji（memory-manager 不修改）
├── USER.md                            ← 用户信息：偏好、背景 ✏️ memory-manager 可写
└── memory/                            ← 全局记忆目录
    └── （如有全局长期规则可放此处）

{project_workbuddy}/memory/            ← 项目工作层（主要操作目标）
├── MEMORY.md                          ← 长期记忆：规则、错误修正、重要约束 ✏️
├── CANDIDATES.md                      ← 候选池：3次晋升缓冲区（v3 新增）✏️
├── EVOLUTION_LOG.md                   ← 进化记录（v3 继承自 v2）✏️
├── skills/
│   ├── INDEX.md                       ← 技能索引 ✏️
│   └── auto-generated/                ← 自动生成的 Skill 草稿 ✏️
└── 2026-04-XX.md                      ← 每日日志（情节记忆）✏️
```

### 文件职责映射

| v2 概念 | v3 对应的 OpenClaw 文件 | 操作权限 |
|---------|------------------------|---------|
| CORE.md · 用户偏好区块 | `~/.workbuddy/USER.md` | 可写 |
| CORE.md · 重要规则区块 | `{project}/memory/MEMORY.md` | 可写 |
| CORE.md · 错误修正区块 | `{project}/memory/MEMORY.md` | 可写 |
| 情节记忆 episodic/ | `{project}/memory/2026-04-XX.md` | 可写 |
| EVOLUTION_LOG.md | `{project}/memory/EVOLUTION_LOG.md` | 可写 |
| CANDIDATES.md（v3 新增）| `{project}/memory/CANDIDATES.md` | 可写 |
| Skills 索引 | `{project}/memory/skills/INDEX.md` | 可写 |
| SOUL.md / IDENTITY.md | `~/.workbuddy/SOUL.md` / `IDENTITY.md` | **只读** |

**重要**：SOUL.md 和 IDENTITY.md 是 Agent 的身份定义文件，memory-manager **永远不修改**它们。

---

## 四层记忆架构（v3 适配版）

### Layer 0：身份层（只读）
**文件**：`~/.workbuddy/SOUL.md`、`~/.workbuddy/IDENTITY.md`
**定位**：Agent 的价值观和身份定义，由 Agent 配置者设定，memory-manager 只读取，不修改。

---

### Layer 1：用户信息层
**文件**：`~/.workbuddy/USER.md`
**定位**：存储用户的偏好、背景信息、沟通风格等。在 OpenClaw 的 workspace 注入机制下，此文件每次 session 自动加载。

**写入内容**：
- 用户语言偏好
- 用户沟通风格和习惯
- 用户明确要求记住的个人背景信息
- 工作领域和专业背景

**写入门槛**：用户明确表达，或候选池观察满 3 次的偏好类信息。

**v3 行号管理**：USER.md 头部维护行号速查注释，每次写入后更新。

---

### Layer 2：长期记忆层
**文件**：`{project_workbuddy}/memory/MEMORY.md`
**定位**：项目级长期规则、错误修正、重要约束。每次 session 由 OpenClaw 注入。

**写入内容**：
- Agent 犯过的错误 + 修正后的规则（不删除，标注"已修正"保留）
- 用户确认的重要规则和行为约束
- 技能索引引用（`→ skills/INDEX.md`）
- 月度日志归档引用（仅文件名）

**写入门槛**：高。判断标准：**"如果忘了这条，下次 Agent 一定会出错或违背用户意图"**，才写入。

**大小限制**：不超过 200 行，超过时触发 `consolidate`。

**v3 行号速查表**：文件头部维护行号索引，每次写入后更新：
```markdown
<!-- 行号速查（每次写入后更新）
L6-L20   重要规则
L22-L40  错误修正记录
L42-L48  技能索引引用
L50-L55  日志归档引用
L57-L62  进化历史摘要
-->
```

---

### Layer 1.5：候选池（v3 新增）
**文件**：`{project_workbuddy}/memory/CANDIDATES.md`
**定位**：介于"对话临时上下文"和"已确认记忆"之间的缓冲层。所有尚未验证足够重要的经验，先在此处积累观察次数。

**晋升规则**：
- 同一模式被观察到 **≥ 3 次** → 触发进化请求卡片，用户审批后正式写入 MEMORY.md 或 USER.md
- 用户**明确说出**"记住"/"以后都这样"/"永远不要" → 跳过候选池，直接生成进化卡片（1 次即可晋升）
- 候选项超过 **30 天未再次观察** → 自动丢弃（记录到 EVOLUTION_LOG.md）

**直接丢弃（不进候选池）**：
- 一次性指令（"现在帮我做 X"）
- 特定文件/上下文的临时指令（"在这个文件里用 XX 格式"）
- 假设性表达（"如果...应该怎么..."）

---

### Layer 3：情节记忆层
**文件**：`{project_workbuddy}/memory/2026-04-XX.md`（每日日志）
**定位**：记录"今天发生了什么"，对应 OpenClaw 的每日日志文件。

**写入时机**：
- 完成一次重要任务后
- 解决了一个复杂问题后
- session 结束时的简要总结

**衰减规则**：
- 超过 30 天的日志 → 执行 `consolidate` 时压缩为月度汇总
- 超过 90 天的月度汇总 → 可归档或删除（需用户确认）

---

## v3 三项新机制

---

### 机制一：Hook 自动激活

**解决的问题**：v2 依赖用户或 Agent 手动执行 `evolve`，容易遗漏。v3 通过 OpenClaw hooks 实现无感后台触发。

#### 两个 Hook

**① `activator.sh`（UserPromptSubmit）**
- 触发时机：每次用户提交 Prompt
- 作用：在 Agent 上下文末尾注入一段轻量提醒，让 Agent 在回复时顺带扫描是否出现学习信号
- Token 开销：约 40–60 tokens
- 关键原则：仅注入提示，不自动写入任何文件——写入必须经用户审批

**② `error-detector.sh`（PostToolUse）**
- 触发时机：每次工具调用（Bash、文件操作等）完成后
- 作用：检测退出码，出错（exit code ≠ 0）时立即将此次错误写入 CANDIDATES.md 作为候选经验
- 意义：工具出错是最有学习价值的时刻，不应等到 session 结束才记录
- 安全注意：此 hook 会读取 `CLAUDE_TOOL_OUTPUT`，**敏感环境中不要启用**

#### 安装命令

执行 `memory-manager install-hooks` 或手动：

```bash
# 复制 hook 脚本
HOOKS_SRC="{skill_path}/hooks"
HOOKS_DST="$HOME/.openclaw/hooks/memory-manager"
mkdir -p "$HOOKS_DST"
cp "$HOOKS_SRC/activator.sh" "$HOOKS_DST/"
cp "$HOOKS_SRC/error-detector.sh" "$HOOKS_DST/"
chmod +x "$HOOKS_DST/"*.sh

# 启用
openclaw hooks enable memory-manager

# 验证
openclaw hooks list | grep memory-manager
```

#### 精细控制

```bash
openclaw hooks enable memory-manager/activator      # 只启用 Prompt 提醒
openclaw hooks enable memory-manager/error-detector # 只启用错误检测
openclaw hooks disable memory-manager               # 临时全部禁用
openclaw hooks uninstall memory-manager             # 彻底卸载
```

---

### 机制二：候选池三次晋升（防误学）

**解决的问题**：v2 中只要 Agent 发现值得持久化的经验就直接呈现进化卡片，存在将偶发指令误固化为永久规则的风险。

#### CANDIDATES.md 文件格式

```markdown
# 候选记忆池（v3）
> 由 memory-manager v3 自动维护 | 最后更新：YYYY-MM-DD
> 晋升规则：同一模式观察 ≥3 次，或用户明确指示

---

## 🌱 观察中

| ID | 类型 | 内容摘要 | 首次观察 | 最近观察 | 次数 | 目标 |
|----|------|---------|---------|---------|------|------|
| C001 | 规则 | PowerShell 删除文件用 Remove-Item 而非 rm | 2026-04-15 | 2026-04-17 | 2/3 | 3次 |
| C002 | 偏好 | 代码注释优先用中文 | 2026-04-18 | 2026-04-18 | 1/3 | 3次 |

---

## ✅ 已晋升

| ID | 内容摘要 | 晋升日期 | 写入位置 |
|----|---------|---------|---------|
| C000 | 回复语言：中文 | 2026-04-10 | USER.md:L8 |

---

## 🗑️ 已丢弃

| ID | 内容摘要 | 原因 | 日期 |
|----|---------|------|------|
| C-x | 临时调试参数 | 30天无再次观察 | 2026-04-01 |
```

#### 候选项生命周期决策树

```
发现学习信号
      │
      ▼
用户说过"记住"/"以后都这样"/"永远不要"等明确指示词？
      │ YES → 跳过候选池，直接生成进化卡片（1次晋升）
      │ NO
      ▼
CANDIDATES.md 中是否已有同类候选项？
      │ YES → 次数 +1 | 更新"最近观察"日期
      │         └── 次数 ≥ 3？YES → 下次 evolve 触发进化卡片
      │ NO  → 新建候选项（次数 1/3），静默记录，不打扰用户
      │
      ▼
距首次观察 > 30 天且次数 < 3？
      → 标记丢弃，记录 EVOLUTION_LOG.md
```

#### 1次晋升的快捷触发词

用户说出以下词语时，无需等待 3 次，立即生成进化卡片：
- 中文：记住、以后都、永远不要、必须、这是规定、下次也
- 英文：Remember, Always, Never, Make sure you always

---

### 机制三：行级来源引用

**解决的问题**：v2 中用户不知道 Agent 为什么这样决策，记忆系统对用户是黑盒。v3 要求 Agent 在依据记忆执行操作时主动标注来源。

#### 引用格式

```
依据 MEMORY.md 中的规则：
  操作描述（📍 MEMORY.md:L23）

依据 USER.md 中的偏好：
  操作描述（📍 USER.md:L8）

依据今日日志（情节记忆）：
  操作描述（📍 2026-04-17.md）

依据候选池中尚未晋升的观察：
  操作描述（📍 CANDIDATES.md:C001，已观察 2/3 次）

找不到精确行号时降级引用：
  操作描述（📍 MEMORY.md · 重要规则区块）
```

#### 何时必须引用，何时可省略

**必须引用**：
- 根据记忆中的规则拒绝或修改了用户请求
- 自动调整了输出语言/格式/风格
- 选择了某个特定工具/方案而非其他备选（有规则依据时）

**可省略**：
- 普通问答，没有依赖任何记忆规则
- 技术性事实（文档知识、编程语言特性）
- 用户当场在本 session 内明确给出的新指令

#### 行号速查表维护

每次执行 `write-memory` 或 `write-user` 后：

```bash
# 快速定位各区块起始行号
grep -n "^##" ~/.workbuddy/USER.md
grep -n "^##" {project}/memory/MEMORY.md
# 按输出更新文件头部的 <!-- 行号速查 --> 注释
```

---

## 子命令列表（v3 完整版）

| 命令 | 触发场景 | v3 变化说明 |
|------|---------|-----------|
| `init` | 首次部署 | 新增初始化 CANDIDATES.md；询问是否安装 hooks |
| `install-hooks` | 启用自动化 | **v3 新增**：一键安装并启用 hook 脚本 |
| `write-memory` | 写入长期规则/错误修正 | 原 write-core；写入后更新行号速查表 |
| `write-user` | 写入用户偏好/背景 | 原 write-core 偏好区块；操作 USER.md |
| `write-log` | 记录今日事件 | 原 write-event；操作每日日志文件 |
| `candidates` | 查看/管理候选池 | **v3 新增**：list / promote / drop 子操作 |
| `consolidate` | 月度整合/记忆瘦身 | 新增候选池清理（30天丢弃过期项） |
| `add-skill` | 新增/更新技能索引 | 无变化 |
| `health-check` | 定期维护诊断 | 新增候选池健康检查 + hooks 状态检查 |
| `handoff` | 交接给新 Agent | 新增导出候选池状态 |
| `evolve` | session 结束时 | 与候选池联动；区分首次观察（静默）和满3次（卡片）|
| `auto-skill` | 多步骤工作流完成后 | 无变化 |

详细执行流程见：`commands.md`

---

## 写入决策树（v3 完整版）

```
收到需要记忆的信息
        │
        ▼
这是当前 session 的临时工作上下文？
        │ YES → 不持久化，仅在 context window 保留
        │ NO
        ▼
这是可复用的操作流程（≥3步）？
        │ YES → auto-skill：生成草稿到 skills/auto-generated/，提请审批
        │ NO
        ▼
这是一次性事件（今天发生了什么）？
        │ YES → write-log：写入今日日志（2026-04-XX.md）
        │ NO
        ▼
用户说过明确晋升触发词（记住/以后都/永远）？
        │ YES → 直接生成进化卡片（跳过候选池）
        │ NO
        ▼
这是用户偏好/背景类信息？
        │ YES → 候选池判断 → 满3次 → 进化卡片 → 写入 USER.md
        │ NO
        ▼
这是规则/错误修正/行为约束？
        │ YES → 候选池判断 → 满3次 → 进化卡片 → 写入 MEMORY.md
        │ NO
        ▼
        丢弃（不是所有信息都值得记忆）
```

---

## 进化请求卡片（v3 格式）

```
╔════════════════════════════════════════════════╗
║  💡 进化请求 · 发现 N 条可持久化的经验          ║
╠════════════════════════════════════════════════╣
║                                                ║
║  [1] 📌 规则 → 写入 MEMORY.md                 ║
║      来源：候选池晋升（C001，观察 3 次）        ║
║      首次观察：2026-04-15                      ║
║      内容：[2026-04] PowerShell 删除文件须用   ║
║             Remove-Item，不可用 rm             ║
║      操作：ADD（无冲突）→ 预计写入 L24         ║
║                                                ║
║  [2] 👤 偏好 → 写入 USER.md                   ║
║      来源：用户明确指示（本次直接晋升）         ║
║      内容：[2026-04] 代码注释优先使用中文      ║
║      操作：ADD → 预计写入 USER.md:L12         ║
║                                                ║
║  [3] ⚠️ 错误修正 → 写入 MEMORY.md             ║
║      来源：error-detector Hook 触发            ║
║      内容：npm install 在此项目失败 →          ║
║             改用 pnpm install                  ║
║      操作：ADD → 预计写入 L31                 ║
║                                                ║
║  回复 "确认" / "全部" / "1,3" / "跳过"        ║
╚════════════════════════════════════════════════╝
```

**v3 新增字段**：
- `来源` 区分四种来源类型：候选池晋升 / 用户明确指示 / Hook 触发 / Agent 自省
- `首次观察` 提供完整溯源链（来自候选池时显示）
- `预计写入 Lxx` 显示写入后的预期行号，便于后续引用

---

## v3 自进化完整时序图

```
每次 UserPromptSubmit
        │
        ▼
[activator.sh 触发] ← Hook 注入
        │ 在 Agent 上下文末尾添加轻量评估提醒
        │
        ▼
Agent 处理对话 / 执行工具调用
        │
        │ 工具出错？
        │ YES ──→ [error-detector.sh 触发]
        │             └─→ 写入 CANDIDATES.md（新建或次数+1）
        │ NO
        │
        ▼
Agent 回复中发现学习信号？
        │ YES ──→ 候选池判断（见决策树）
        │ NO  ──→ 正常继续
        │
        ▼
Session 结束 / 用户手动触发 evolve
        │
        ▼
加载 CANDIDATES.md
        │
        ├── 候选项满 3 次 → 加入进化卡片晋升列表
        ├── 明确指示项 → 加入进化卡片晋升列表
        ├── 首次观察项 → 写入候选池（静默，不打扰用户）
        └── 工作流 ≥3 步 → auto-skill 草稿候选
        │
        ▼ （有晋升候选时）
呈现进化请求卡片（同一 session 最多 2 次）
        │
        ▼
用户审批
        ├── 确认 → write-memory / write-user（含行号速查表更新）
        │         → 更新 EVOLUTION_LOG.md
        │         → 更新 CANDIDATES.md（标记已晋升，填写写入位置）
        └── 跳过 → 本次不写入，候选池状态保留
```

---

## MEMORY.md 文件模板（v3 适配 OpenClaw）

```markdown
# 长期记忆（MEMORY.md）
> 最后更新：YYYY-MM-DD | 由 memory-manager skill v3 维护
> 行数：N 行

<!-- 行号速查（每次 write-memory 后更新）
L6-L20   重要规则
L22-L38  错误修正记录
L40-L45  技能索引引用
L47-L52  日志归档引用
L54-L58  进化历史摘要
-->

---

## 📌 重要规则
<!-- 格式：[YYYY-MM] 规则描述 | 来源：{用户指定/候选池C00x/Hook触发} -->

## ⚠️ 错误修正记录
<!-- 格式：[YYYY-MM] 曾犯的错误 → 修正后的做法 | 来源：{Hook/用户纠正/自发现} -->
<!-- 错误记录不删除，标注"已修正"后保留，供后续 session 学习 -->

## 🔗 技能索引
→ 详见 skills/INDEX.md
→ 草稿待审批：skills/auto-generated/（如有）

## 📅 日志归档引用
<!-- 只写文件名，不展开内容 -->

## 🔄 进化历史摘要
<!-- 最近 3 次：日期 | 来源 | 写入内容摘要 | 详见 EVOLUTION_LOG.md -->
```

---

## USER.md 文件模板（v3 适配 OpenClaw）

```markdown
# 用户信息（USER.md）
> 最后更新：YYYY-MM-DD | 由 memory-manager skill v3 维护
> 行数：N 行

<!-- 行号速查（每次 write-user 后更新）
L6-L12   基本偏好
L14-L22  工作背景
L24-L30  沟通风格
-->

---

## 👤 基本偏好
<!-- 格式：[YYYY-MM] 偏好描述 | 来源：{用户指定/候选池C00x} -->

## 🏢 工作背景
<!-- 职业、领域、技术栈等 -->

## 💬 沟通风格
<!-- 语言偏好、格式偏好、回复长度偏好等 -->
```

---

## 注意事项（v3）

- **SOUL.md 和 IDENTITY.md 只读**：这两个文件是 Agent 身份的根基，memory-manager 永远不修改它们
- **候选池是防误学护城河**：非明确指示的经验，一律先进候选池，不允许单次信号直接写入 MEMORY.md 或 USER.md
- **Hook 可选**：不安装 hooks 时，本技能完全可用，只是需要手动调用 evolve；敏感环境中可只启用 activator 不启用 error-detector
- **行号速查表必须维护**：每次写入后必须更新行号速查表，否则来源引用不准确
- **引用自然嵌入**：行内 📍 引用应嵌入句末，不要单独罗列，不要每句都加
- **错误记录不删除**：标注"已修正"后保留，供后续 session 学习
- **进化频控**：同一 session 内进化请求卡片最多出现 2 次，避免频繁打扰
- **子 Agent 隔离**：子 Agent 不能直接写 MEMORY.md，需提交候选建议到自己的日志文件，由主 Agent 在 evolve 时审核

---
> Source: [2017java/super-agent-memory-manager](https://github.com/2017java/super-agent-memory-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
