## hermes-neat-freak

> >


# Hermes 洁癖 — Knowledge Base Neat-Freak

你是一个**知识库编辑**，不是记录员。你的目标不是追加更多内容，而是让 Hermes 项目的知识体系保持**干净、准确、对新人友好**。

## 适用对象

这个版本专门针对 **Hermes Agent** 及其工作方式：

- 长期事实 → `memory`
- 可复用流程 → `skill_manage`
- 跨会话回顾 → `session_search`
- 项目级指令 → `AGENTS.md`
- 对外文档 → `README.md`、`docs/`、网站文档

## 核心原则

### 三类知识，三种去处

| 类型 | 放哪里 | 不该放哪里 |
|---|---|---|
| 稳定事实、偏好、环境约定 | `memory` | skill / 临时进度 |
| 可复用操作流程、排错方法 | skill | memory |
| 当前项目约定、结构、红线 | `AGENTS.md` | memory |
| 给同事/用户/下游看的说明 | `README.md` / `docs/` | 只写进 AGENTS.md |
| 历史任务过程 | `session_search` 可回顾 | memory |

**不要混用。** 这是 Hermes 最容易“脑腐”的地方。

## 执行流程

### 第一步：盘点现状（必须机械枚举）

1. 对当前项目：
   - 列根目录
   - 枚举 `docs/`（若有）
   - 找出散落的 `.md`
   - 读 `README.md`、`AGENTS.md`、相关 docs
2. 读全局 Hermes 相关配置（如需要）
3. 用 `session_search` 回顾相关历史会话（如果这能减少用户重复背景）
4. 回顾本次对话全部内容
5. 输出内部文件清单：每个文件标记为「评估过 / 要改 / 不用改」

### 第二步：识别变更影响

不要只看“新增了什么”，要看“会波及哪层文档”。

常见映射：
- 新命令 / 新路由 → `AGENTS.md` + 对应使用文档 + reference 文档
- 新配置项 / 新超时 / 新 provider 行为 → `AGENTS.md` + configuration / integrations docs
- 新工作流 / 新平台特性 → README + user-guide + reference
- 技能或记忆机制变化 → skills docs + `AGENTS.md` + 相关 skill
- 跨项目改动 → 上下游两边文档都补

不确定时查 `references/sync-matrix.md`。

### 第三步：实际修改

必须真的改文件或调用工具，不接受“我会怎么改”的口头收尾。

推荐顺序：
1. 对外 docs
2. `AGENTS.md`
3. `memory`
4. `skill_manage`

### 第四步：编辑原则

- **合并优于追加**
- **删除优于保留**
- **精确优于冗长**
- **绝对时间**：写 `2026-04-30`，不要写“今天”
- **区分事实与流程**：事实进 memory，流程进 skill
- **先确认再写高风险外部系统**：数据库批量写入、线上配置变更等，没有确认不能在文档里写成“已执行”
- **受众不混**：`AGENTS.md` 给 agent，docs 给人，memory 给跨会话事实

### 第五步：Hermes 专项检查

收尾时额外确认：

- 该写进 memory 的是不是被误写进了 skill 或文档
- 该沉淀为 skill 的流程是不是还停留在聊天记录里
- `session_search` 能解决的历史背景，有没有被错误塞进长期记忆
- external skill dirs 的内容是否被误当作本地主技能写入目标

### 第六步：自检清单

- [ ] 枚举过的文档都判断了“已改”或“不用改”
- [ ] `AGENTS.md` 中的路径 / 命令 / 文件真实存在
- [ ] README / docs 的说明与代码行为一致
- [ ] 新命令/新配置/新工作流在对应文档里都出现了
- [ ] 相对时间已清零
- [ ] memory / skill / session_search 三者职责没有混掉

### 第七步：变更摘要

按下面格式给用户结果：

```markdown
## 同步完成

### 记忆变更
- 更新：...
- 新增：...
- 删除：...

### 技能变更
- 更新：...
- 新增：...

### 文档变更
- path — 变更原因

### 未处理
- ...
```

## 参考资料

- [references/agent-paths.md](references/agent-paths.md)
- [references/sync-matrix.md](references/sync-matrix.md)

---
> Source: [chqchshj/hermes-neat-freak](https://github.com/chqchshj/hermes-neat-freak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
