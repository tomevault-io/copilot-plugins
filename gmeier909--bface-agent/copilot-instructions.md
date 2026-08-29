## bface-agent

> > 红队作战 MCP（24 工具）。本文件同时作为 CLAUDE.md / AGENT.md，供 Claude Code 与 omp 读取。

# bface-agent — 操作规则与 MCP 用法

> 红队作战 MCP（24 工具）。本文件同时作为 CLAUDE.md / AGENT.md，供 Claude Code 与 omp 读取。
> MCP 接入与连接方式见 README_MCP.md；工具参数以运行时 tool schema 为准。

## 工程纪律

### 1. 先读再写

- 动任何文件之前，**先读那个文件**，搞清楚已有的模式、import、依赖
- 照搬项目里现成的写法，别凭空发明
- 项目用 `fetch` 就别上 `axios`，用 `crypto.randomUUID()` 就别引 `uuid` 包
- 加依赖前先 grep 项目是否已有替代方案

### 2. 先想再敲

- 动手前先说清楚要做什么，列出计划
- 模糊需求要拆解："加认证" = 登录 + 注册 + token管理 + 路由守卫 + 错误处理
- 不懂就停下来问——别用看起来像回事实则一跑就崩的代码糊弄

### 3. 极简主义

- 写解决**眼前问题**的最少代码，不是解决所有未来版本的
- 测试标准：某样东西被抽象的唯一理由是"以防万一" → 过度工程了
- 别写现在用不上的接口、配置项、扩展点

### 4. 精准手术

- diff 和任务一样小，没让碰的别碰
- 不要顺手格式化——会把真正重要的三行改动埋在三百行无关变更里
- 判断标准：每一行改动都能直接关联到用户需求吗？不能，撤回

### 5. 验证

- "觉得能跑"和"真的能跑"之间隔着测试
- 修 bug：先写测试复现 → 改代码 → 测试通过才算完
- 测那些真会在用户面前炸掉的场景，别只测鸡毛蒜皮
- 某样东西怎么都测不了 → 代码设计有问题，不是测试的问题

### 6. 目标驱动执行

- 动手前把"做完了"定义清楚，且可验证
- "加个验证"太模糊 → "邮箱未填或格式错，弹出明确报错，两种情况都测过"
- 多步骤：先列计划，别闷头干一小时方向就是错的

### 7. 调试

- 东西坏了：去查，别猜
- 读完整报错和堆栈，先复现再改，一次只改一个地方

### 8. 依赖管理

- 每一个依赖都是你无法控制的永久代码
- 加之前问：标准库能不能搞定？
- 加了说明为什么——让选择可见

### 9. 沟通

- 说你做了什么、为什么，不只是丢代码
- "我不确定这个库是否支持流式传输" ← 好沟通
- "我觉得应该能行" ← 不叫沟通

### 10. 常见翻车模式

| 模式 | 症状 | 对策 |
|------|------|------|
| 厨房水槽 | 修水龙头拆了整个厨房 | 只改和任务直接相关的文件 |
| 错误抽象 | 复制粘贴好几遍没合并 | 发现3次重复立刻抽取 |
| 盲目乐观 | 只考虑顺利情况 | 问自己：输入错了？网络断了？服务挂了？ |
| 失控连锁 | 改一个文件倒了十几个 | 每次 commit 前看 diff 范围，越界就撤回 |

## 反幻觉纪律（证据纪律）

大模型的通病：把"推测"写得像"事实"。任何结论落进档案/报告/回复前，先过这道闸：

- **每个结论必须有来源**：来源 = 工具原始输出、命令回显、档案原文、测试结果。没有来源的断言 → 幻觉，删除。
- **工具输出 ≠ 结论**：工具的"疑似 / VULNERABLE / 可能"只是线索，必须手动复核拿到确凿证据（请求+响应、身份回显、数据原文）才算数。
- **状态三态**：`假设`（未复现）→ `已验证`（复现成功）→ `已利用`（实际利用）。严禁把 `假设` 当 `已验证` 写入档案。
- **"没找到"是合法结果**：如实写"无发现"，禁止用"可能有 XX"填结果。空结果也是结果，硬凑 = 编造。
- **引用原文，不凭记忆**：写进档案/报告的每句话都来自读到的文件或跑出的输出。记忆会补全、美化、编造细节。
- **禁止脑补**：没读过的文件、没跑过的命令、没拿到的回显，一律不得写进结论或报告。

## MCP（bface-agent）能力与同步约定

### 工具速查

| 类 | 工具 | 作用 |
|---|---|---|
| C2 | c2_listener / c2_session / c2_task / c2_task_manage / c2_payload / c2_event / c2_profile / c2_file | 监听器·会话·任务下发·任务管理·payload·事件·profile·文件 |
| WebShell | manage_webshell_{add,list,update,delete,test} + webshell_{exec,file_list,file_read,file_write} | 连接 CRUD + 命令执行 + 文件读写 |
| 信息收集 | gather_assets | FOFA/Hunter/Quake 资产测绘（按 ip:port 去重） |
| 项目黑板 | manage_project / manage_project_fact | 项目 + 事实 DAG（edges/from_edges 拉攻击图） |
| 漏洞账本 | manage_vulnerability | 漏洞 CRUD + 三态流转（假设→已验证→已利用） |
| 知识库 | list_knowledge_risk_types / search_knowledge_base | 风险类型清单 + 语义检索 |
| 视觉 | analyze_image | 截图/证据图 → 模型文字研判 |

### 同步约定（硬性）

1. **信息收集 → 同步项目黑板**：`gather_assets` 结果自动 upsert（fact_key=asset.<ip>:<port>）；侦察/手工发现的其它事实用 `manage_project_fact action=upsert` 写入，可带 edges/from_edges 拉攻击图边。
2. **发现漏洞 → 同步漏洞账本**：用 `manage_vulnerability action=create` 登记，必填 severity + status + 可复现 proof；只有可复现才升 status=confirmed，报告只收 confirmed。

---
> Source: [gmeier909/bface-agent](https://github.com/gmeier909/bface-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
