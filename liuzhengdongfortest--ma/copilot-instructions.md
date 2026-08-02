## ma

> > 从 [DESIGN.md](DESIGN.md) 延伸：项目规则也应服从“文件系统作为 Source of Truth”。

# AGENTS.md 接入设计

> 从 [DESIGN.md](DESIGN.md) 延伸：项目规则也应服从“文件系统作为 Source of Truth”。

---

## 目标

兼容 `AGENTS.md` 这一类项目级工作规则文件，但不额外创造一套独立的提示词通道。

March 的做法是：

- 如果 **工作目录根下** 存在 `AGENTS.md`
- 则在 **session 初始化** 时自动将其加入 `open_files`
- 并默认标记为 **locked**

这样，AI 看到的是 watcher 维护的真实文件内容，而不是启动时复制出来的一份静态字符串。

---

## 为什么接到 open_files

这里直接复用 `open_files`：

- `AGENTS.md` 是文件，不是隐藏配置对象
- 其内容可能被用户实时修改，最自然的真相源就是 watcher-backed file snapshot
- 现有 `open_files` 已经具备顺序、持久化、watcher 同步、UI 展示、lock 控制这些能力
- 这样接入最小，设计最统一，不需要再维护第二套“规则文件缓存”

换句话说，March 不把 `AGENTS.md` 视为“额外 system prompt 文本”，而是视为“默认打开的一份项目规则文件”。

---

## 加载规则

### Session 初始化

创建新 session 或从持久化状态恢复 session 时：

1. 检查工作目录根下是否存在 `AGENTS.md`
2. 若存在，则确保它出现在 `open_files` 中
3. 并默认将该文件加入 `locked_files`

若用户之前已经手动打开过它，则复用现有条目，不重复插入。

### 恢复已有任务

恢复已有任务时也做同样检查：

- 如果持久化状态里已有 `AGENTS.md`，按已有顺序恢复
- 如果持久化状态里没有，但当前工作目录新出现了 `AGENTS.md`，则自动补入
- 如果文件已不存在，则不保留虚假快照，交给 watcher / 恢复逻辑按真实磁盘状态处理

---

## 与 lock 的关系

`AGENTS.md` 的默认 lock 语义是：

- AI 不能通过 `close_file` 把它从上下文移除
- 用户若明确解锁，才允许后续像普通文件一样被关闭
- lock 只约束“是否保留在 open_files 中”，**不等于禁止编辑**

也就是说，`AGENTS.md` 仍然是普通文件：

- 可以被用户手动修改
- 可以被 watcher 检测变化
- AI 也可以按任务需要提出修改建议，或在用户明确要求时修改它

默认 lock 的目的只是保证“项目规则不要被上下文整理过程顺手丢掉”。

---

## 与自动 open 规则的关系

`AGENTS.md` 是第三种自动进入 `open_files` 的场景：

1. AI 主动 `open_file`
2. AI 写入文件后自动加入
3. 用户 `@path` 引用文件后自动加入
4. session 初始化时自动加入工作目录根下的 `AGENTS.md`

其中第 4 种是系统级规则，不依赖本轮用户消息，也不要求 AI 自己先意识到该文件存在。

---

## 顺序建议

为减少 prompt 抖动，`AGENTS.md` 加入 `open_files` 时应放在稳定靠前的位置：

- 若 session 初始化时自动加入，默认排在 `open_files` 前部
- 其他 AI 主动打开的工作文件继续按打开顺序排在后面

这样符合它“项目级规则文件”的定位，也能减少因普通工作文件频繁增减导致的顺序混乱。

---

## UI 表现

右侧 `open_files` 面板中，`AGENTS.md` 仍按普通 open file 展示，但默认带 locked 状态。

不需要为它单独开新面板，因为：

- 用户本来就应该能直接看到它进入了上下文
- lock 状态已经足够表达“这是系统默认保留的文件”
- 统一放在一处更符合 March 对上下文可见化的设计

如后续需要，可以仅在视觉上给这类系统自动加入的文件加一个轻量标识，但不改变其数据模型。

---
> Source: [liuzhengdongfortest/MA](https://github.com/liuzhengdongfortest/MA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
