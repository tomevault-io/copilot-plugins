## web-harness

> > 基于 Anthropic & OpenAI 官方工程实践，结合已部署 skills 构建的标准化 web 开发执行框架。

# Web Development Harness

> 基于 Anthropic & OpenAI 官方工程实践，结合已部署 skills 构建的标准化 web 开发执行框架。

## 核心原则

1. **人类定边界，Agent 做执行** - 人类负责架构设计、规则编码、意图对齐与验收标准定义
2. **上下文渐进式披露** - 用结构化仓库文件作为唯一信息源，给 Agent "地图" 而非 "百科全书"
3. **权责分离，闭环反馈** - 生成与评估完全分离，用独立校验机制解决自评估失真
4. **增量执行，状态可追溯** - 每一轮只做一件事，用 git 和结构化文档保留完整状态
5. **机械性约束优先** - 把架构规则、代码规范、质量标准编码为自动化校验

## Harness 架构

```
┌─────────────────────────────────────────────────────────────────┐
│                    Web Development Harness                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │   Planner   │───▶│  Generator  │───▶│  Evaluator  │         │
│  │   (brain-   │    │  (frontend- │    │ (webapp-    │         │
│  │  storming)  │    │  design +   │    │  testing)   │         │
│  │             │    │ fullstack)  │    │             │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│         │                  │                   │                │
│         │                  ▼                   │                │
│         │          ┌─────────────┐             │                │
│         │          │   Review    │◀────────────┘                │
│         │          │  (systematic│                              │
│         │          │  -debugging)│                              │
│         │          └─────────────┘                              │
│         │                  │                                     │
│         ▼                  ▼                                     │
│  ┌─────────────────────────────────────────────────┐           │
│  │              State Management                    │           │
│  │  • feature-list.json (需求清单)                  │           │
│  │  • sprint-contract.md (冲刺合同)                 │           │
│  │  • progress.md (进度追踪)                        │           │
│  │  • evaluation-report.md (评估报告)               │           │
│  └─────────────────────────────────────────────────┘           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 三角色职责

### 1. Planner Agent (需求拆解与架构设计)

**使用技能**: `brainstorming` + `writing-plans`

**核心职责**:
- 将模糊需求转化为完整的 product spec
- 输出结构化 feature list (JSON 格式)
- 定义技术栈与架构边界
- 制定分阶段开发计划

**输入**: 用户 1-4 句核心需求
**输出**: `docs/specs/product-spec.md` + `feature-list.json`

**执行规则**:
- 只定 "做什么" 和 "验收标准"，不定义具体实现细节
- 每个 feature 必须包含端到端测试步骤
- 粒度拆解到用户可感知的最小操作单元

---

### 2. Generator Agent (增量编码实现)

**使用技能**: `frontend-design` + `fullstack-developer` + `test-driven-development`

**核心职责**:
- 基于 sprint contract 增量开发
- 遵循分层架构约束
- 自主迭代决策
- 版本与状态管理

**执行流程**:
```
启动三板斧:
1. pwd 确认工作目录
2. 读取 progress.md + feature-list.json
3. 选择 1 个未完成 feature，提出 Sprint Contract

增量执行:
- 单模块开发，先 API 后 UI
- 自测通过后提交

闭环校验:
- 启动开发服务
- 执行端到端测试
- 更新 feature-list.json (passes: true)
- git commit + 更新 progress.md
```

**Sprint Contract 必备内容**:
- 本轮交付内容
- 验收标准
- 可测试的完成定义
- 预估时间

---

### 3. Evaluator Agent (QA 与质量把关)

**使用技能**: `webapp-testing` + `agent-browser`

**核心职责**:
- 独立、客观、严格地执行验收
- 多维度质量评估
- 输出可落地的修改意见

**评估维度 (Web 开发专属)**:

| 维度 | 权重 | 核心校验标准 |
|------|------|-------------|
| 功能完整性 | 40% | 100% 实现需求，无 stub，API 正常，E2E 全流程可跑通 |
| 设计质量 | 25% | 统一视觉语言，无 AI 模板化，布局合理，交互标准 |
| 代码质量 | 20% | 架构规范，类型安全，无重复代码，lint 通过 |
| 可用性 | 15% | 无引导可操作，交互反馈清晰，响应式正常 |

**硬性规则**:
- 必须使用 `webapp-testing` 进行端到端测试
- 必须使用 `agent-browser` 截图验证
- 任何维度低于 6/10 分直接 FAIL
- 输出具体的修改建议，禁止泛泛评价

---

## 核心工作流

### 入门级流程 (简单需求)

```bash
# 1. 初始化项目
./scripts/init-project.sh <project-name> "<description>"

# 2. Planner 拆解需求
# 使用 brainstorming + writing-plans 生成 spec

# 3. Generator 单 Agent 开发
# 循环：读取状态 → 选 feature → 开发 → 自测 → 提交

# 4. Evaluator 最终验收
# 使用 webapp-testing 进行端到端测试
```

### 进阶级流程 (复杂全栈)

```bash
# 1. Planner 生成完整 spec
brainstorming → writing-plans

# 2. Generator 提出 Sprint Contract
# 与 Evaluator 双向评审，达成一致

# 3. Generator 开发
# 单模块增量执行

# 4. Evaluator 评估
# 多维度打分 + 反馈

# 5. 迭代闭环
# 不达标 → Generator 修改 → 重新评估
# 达标 → 下一个 sprint
```

---

## 关键文件结构

```
project/
├── AGENTS.md                 # 本文件 (入口指引)
├── ARCHITECTURE.md          # 架构规范
├── docs/
│   ├── specs/
│   │   └── product-spec.md  # 产品规格
│   ├── plans/
│   │   └── sprint-*.md      # 冲刺计划
│   └── harness/
│       ├── feature-list.json     # 需求清单 (核心!)
│       ├── sprint-contract.md    # 当前冲刺合同
│       ├── progress.md           # 进度追踪
│       └── evaluation-report.md  # 评估报告
├── scripts/
│   ├── init-project.sh      # 项目初始化
│   ├── validate-feature.sh  # Feature 校验
│   └── run-evaluation.sh    # 运行评估
└── .harness/
    ├── templates/           # 模板文件
    └── config.json          # Harness 配置
```

---

## 与现有 Skills 的集成

| Harness 角色 | 使用的 Skills | 用途 |
|-------------|--------------|------|
| Planner | `brainstorming` | 需求澄清与创意探索 |
| Planner | `writing-plans` | 生成结构化计划 |
| Generator | `frontend-design` | 高质量前端实现 |
| Generator | `fullstack-developer` | 全栈开发能力 |
| Generator | `test-driven-development` | TDD 流程 |
| Evaluator | `webapp-testing` | 端到端测试 |
| Evaluator | `agent-browser` | 浏览器自动化验证 |
| Reviewer | `systematic-debugging` | 问题诊断与修复 |

---

## 快速开始

### 新项目初始化

```bash
# 1. 创建项目目录
mkdir my-project && cd my-project

# 2. 初始化 Harness
cp -r /path/to/harness/. .

# 3. 运行初始化脚本
./scripts/init-project.sh my-project "一个待办清单应用"

# 4. 开始开发流程
# Agent 会自动读取 AGENTS.md 并遵循 Harness 规则
```

### 已有项目接入

```bash
# 1. 复制 Harness 文件
cp -r /path/to/harness/. .

# 2. 手动创建 feature-list.json
# 将现有需求转化为结构化格式

# 3. 后续开发遵循 Harness 规则
```

---

## 故障排查

| 问题 | 解决方案 |
|------|---------|
| 上下文溢出、模型提前收尾 | 使用上下文重置，靠仓库文件传递信息 |
| 功能宣称完工但不可用 | 强制 E2E 测试，webapp-testing 验证 |
| 多轮执行后迷路 | 所有状态存入文件，禁止靠对话历史传递 |
| 代码质量平庸 | 启用独立 Evaluator，多维度严格评估 |
| 设计模板化 | frontend-design + 设计原则引导 |

---

## 进阶配置

详见 `docs/harness/advanced.md`:
- 企业级多 Agent 配置
- CI/CD 集成
- 知识库维护
- 技术债务管理

---
> Source: [Drrreistein/web-harness](https://github.com/Drrreistein/web-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
