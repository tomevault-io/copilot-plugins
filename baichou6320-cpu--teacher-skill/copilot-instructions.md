## teacher-skill

> > **创建日期**：2026-04-23

# CLAUDE.md — Teacher-skill 开发指南

> **版本**：v1
> **创建日期**：2026-04-23
> **作用**：每次开工时 Agent 必须阅读的规则与背景文档

---

## 一、项目背景

**产品**：Teacher-skill — 启发式数字助教
**定位**：解决"理解幻觉"问题，通过「讲解→提问→反馈」闭环引导深度学习
**当前阶段**：Phase 2 开发中（工程基建 + 输入扩展）

**文档地图**：
| 文档 | 内容 | 何时读 |
|------|------|--------|
| [docs/product/PRD.md](docs/product/PRD.md) | 产品需求定义 | 了解"为什么做" |
| [docs/development/status.md](docs/development/status.md) | 当前状态 | 每天开工前 |
| [docs/product/roadmap.md](docs/product/roadmap.md) | 开发路线图 | 决定"今天做什么" |
| [docs/development/changelog.md](docs/development/changelog.md) | 变更日志 | 查历史决策 |
| [docs/product/review-mode.md](docs/product/review-mode.md) | 复习模式扩展 | Phase 2 后期实现 |

---

## 二、核心开发原则

### 2.1 PM 视角：先设计，再编码

**AI 是执行层，不是决策层。开发者必须自己掌控产品定义。**

#### 开工前强制三问

每天开始工作前，先向 Agent 提问：

```
基于当前代码库状态，对照 docs/product/roadmap.md 的里程碑：
1. 当前进度是多少？已完成哪些？
2. 剩余阻塞项是什么？
3. 今天的目标是什么？
```

要求 Agent 输出格式：
```
当前进度: XX% (Milestone N/M)
今日目标: 完成 xxx
阻塞项:   xxx
风险点:   xxx
```

#### 禁止行为
- 不允许 Agent 直接给出下一步指令（除非你明确要求）
- 不允许 Agent 代替你写 PRD/ARCH/TASK 文档
- 如果三份文档写不出来，说明对项目理解还不够，不应该开始编码

---

### 2.2 工程师视角：理解每一行代码

**建立"代码所有权"——你必须理解项目里的每一行代码。**

#### 代码讲解仪式

每完成一个文件，要求 Agent 用以下格式解释：

```
做什么：
不做什么：
职责边界：
  Input:
  Output:

关键设计决策：
Q: 为什么用 XXX 而不是 YYY？
A: ...

我（开发者）的疑问：
```

**规则**：如果 AI 解释完后你仍有模糊点，必须追问到完全理解，否则不继续。

---

### 2.3 独立开发者视角：从"做任务"到"做决策"

#### 20 分钟规则

遇到 AI 建议的方案时：
1. 先花 20 分钟自己想：如果是我，会怎么实现？
2. 再对比 AI 方案：AI 的方案好在哪里？差在哪里？
3. 做决策：采纳、修改、或拒绝

#### 决策日志

在 `docs/product/decisions.md` 中记录重要技术决策：

```markdown
## YYYY-MM-DD: 为什么选择 XXX 而不是 YYY？

**上下文**: ...

**选项 A**:
- 优点：...
- 缺点：...

**选项 B**:
- 优点：...
- 缺点：...

**决策**: 选 B。原因：...

**后续验证**: ...
```

---

## 三、项目架构约束

### 3.1 目录结构

```
Teacher-skill/
├── main.py                 # CLI 唯一入口
├── src/
│   ├── core/              # 核心业务逻辑
│   │   ├── engine.py      # 状态机调度
│   │   ├── memory.py      # 对话记忆
│   │   └── rewards.py     # 连击激励
│   ├── llm/              # LLM 交互层
│   │   ├── client.py      # 客户端（含重试）
│   │   └── translator.py  # 响应解析
│   └── utils/             # 工具模块
│       ├── storage.py     # 文件存储
│       ├── config.py      # 配置管理
│       └── logger.py      # 日志
├── models/                # 数据模型（Pydantic）
├── prompts/               # 当前 split Prompt；legacy/ 为旧版兼容
├── tests/unit/            # 单元测试
├── scripts/               # 手动烟测和开发调试脚本
├── data/                  # 用户数据（JSON）
└── docs/                  # 项目文档
```

### 3.2 模块边界规则

| 模块 | 职责 | 不做什么 |
|------|------|---------|
| `engine.py` | 状态机调度、教学循环控制 | 不直接调用 LLM |
| `client.py` | LLM API 调用、重试、thinking 提取 | 不解析响应 |
| `translator.py` | 解析 LLM JSON 响应 | 不调用 LLM |
| `storage.py` | JSON 文件读写 | 不处理业务逻辑 |

### 3.3 代码风格

- **Python**: Google docstring + type hints 必须
- **错误处理**: 不允许裸 `try-except`，必须指定异常类型
- **Prompt 文件**: 必须用 Markdown，包含 ` ```json ` 包裹的 JSON 输出示例

---

## 四、当前 Phase 2 任务看板

### P0 阻塞任务（不做完 Phase 2 不算完成）

| 任务 | 内容 | 状态 |
|------|------|------|
| A1 | 日志系统（logger.py + 落盘） | ✅ 已完成 |
| A2 | 集中配置（config.yaml + config.py） | ✅ 已完成 |
| A3 | 错误处理（异常细化 + 友好提示） | ⏳ 未开始 |
| B1 | 文件读取（--file 参数 + /load 命令） | ⏳ 未开始 |
| B2 | PDF 读取（pypdf 集成） | ⏳ 未开始 |
| C4 | Prompt 拆分（02_teach + 03_judge） | ⏳ 未开始 |

### P1 重要任务

| 任务 | 内容 | 状态 |
|------|------|------|
| A4 | 单元测试（pytest） | ⏳ 未开始 |
| C1-C3 | 教学风格（Persona） | ⏳ 未开始 |

### 扩展任务（Phase 2 后期）

| 任务 | 内容 | 优先级 |
|------|------|--------|
| D1 | 复习 Prompt（04_review.md） | P1 |
| D2 | 意图识别 + 主题匹配 | P0 |
| D3 | history_topics 数据结构 | P0 |
| D4 | 复习引擎 | P1 |
| D5 | history_topics 写入时机 | P2 |

---

## 五、Git 安全规则

### 5.1 禁止上传的内容

以下文件/目录**永远不要**提交到 git：
- `.env`（包含真实 API Key）
- `__pycache__/`、`.pyc`
- `data/users/`（用户数据）
- `logs/`（日志文件）

### 5.2 开源前必做清单

- [ ] 创建 `.gitignore`（参考 Python 标准模板）
- [ ] 创建 `.env.example`（空值占位符）
- [ ] 从 git 历史中清除 `.env`（使用 BFG Repo-Cleaner）
- [ ] 删除 `吴颉航-简历().pdf`
- [ ] 添加 `LICENSE`（推荐 MIT）

---

## 六、开工检查清单

每次开始新会话时，确保：

```
1. [ ] 已读 docs/development/status.md，确认当前状态
2. [ ] 如果要编码，确认任务属于 Phase 2 任务看板中的某一项
3. [ ] 如果要新增任务，先记录到对应文档，不直接开始写代码
4. [ ] 代码改动后，用"代码讲解仪式"确认理解
5. [ ] 重要决策记录到 docs/product/decisions.md
```

---
> Source: [baichou6320-cpu/Teacher-skill](https://github.com/baichou6320-cpu/Teacher-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
