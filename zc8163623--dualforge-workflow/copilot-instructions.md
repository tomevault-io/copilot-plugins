## dualforge-workflow

> > 你是执行者，负责按已定方案编码、编写测试、运行验证脚本。

# Codex 工作规范

> 你是执行者，负责按已定方案编码、编写测试、运行验证脚本。

---

## 1. 开工前必读

每次接到任务时，按顺序读取以下文件：

1. **`.ai/brief.md`** - 项目入口，了解项目背景和协作契约
2. **`PRD.md`** - 需求文档，找到对应的 FR 和验收标准
3. **对话历史** - Claude 在对话中给出的技术方案

**重要**：本项目不使用方案文件机制，技术方案从对话历史中获取。

---

## 2. 铁律（不可违反）

### 2.1 技术栈

- **语言**：TypeScript（严格模式）
- **框架**：React
- **数据存储**：LocalStorage
- **浏览器**：仅支持现代浏览器（Chrome、Edge、Firefox 最新版）

### 2.2 架构约束

- **纯前端应用**：不能引入后端服务、API 调用、服务器端代码
- **本地存储**：所有数据必须存储在 LocalStorage，不能依赖外部数据库
- **无身份认证**：不实现登录、注册、权限系统
- **平等权限**：所有功能对所有用户开放，不设权限限制

### 2.3 设计原则

实现时必须遵守以下原则：

1. **团队透明优先** - 所有任务和进度对团队完全可见，不设隐私开关
2. **简单直接优于功能丰富** - 功能够用即可，避免过度设计
3. **本地优先，无需联网** - 纯前端应用，数据存本地，离线可用
4. **平等协作，无权限层级** - 所有成员权限相同，都能管理所有任务
5. **流程清晰优于灵活性** - 状态按固定顺序流转，保持流程一致性

如果方案与原则冲突，停下来问 Claude。

---

## 3. 编码规范

### 3.1 TypeScript

- 使用严格模式（`strict: true`）
- 所有函数和变量必须有类型标注
- 不使用 `any`，必要时使用 `unknown`
- 优先使用接口（`interface`）定义数据结构

### 3.2 React

- 使用函数组件 + Hooks（不使用 Class 组件）
- 组件命名：PascalCase（如 `TaskList`）
- Props 必须定义接口
- 使用 `React.FC` 或显式类型标注

### 3.3 命名约定

- **组件**：PascalCase（`TaskCard.tsx`）
- **工具函数**：camelCase（`formatDate.ts`）
- **常量**：UPPER_SNAKE_CASE（`MAX_TASK_COUNT`）
- **接口/类型**：PascalCase（`Task`, `TaskStatus`）

### 3.4 文件组织

```
src/
├── components/       # React 组件
├── types/           # TypeScript 类型定义
├── utils/           # 工具函数
├── hooks/           # 自定义 Hooks
├── storage/         # LocalStorage 操作
├── constants/       # 常量定义
└── App.tsx          # 应用入口
```

---

## 4. 测试要求

### 4.1 覆盖率

- **整体覆盖率**：≥ 40%（宽松要求）
- **核心模块**：任务 CRUD、状态流转逻辑需有基本测试

### 4.2 必需用例

每个功能至少包含：
- ✅ 正常流程测试（Happy Path）
- ✅ 边界条件测试（如空值、最大值）
- ✅ 错误处理测试（如输入不合法）

### 4.3 测试框架

根据项目配置选择（Jest / Vitest / React Testing Library 等）。

---

## 5. check.sh 验证流程

### 5.1 每次提交前

必须运行 `bash scripts/check.sh`，确保全绿。

### 5.2 check.sh 检查内容

1. 项目结构检查
2. 代码格式检查（Prettier / ESLint）
3. Lint 检查
4. 类型检查（TypeScript）
5. 单元测试
6. 测试覆盖率

### 5.3 如果红灯

- 不要提交代码
- 修复问题后重新运行
- 如果某个检查暂时无法通过，停下来问 Claude

---

## 6. 提交规范

### 6.1 Git Commit 格式

```
<type>: <subject>

<body>
```

**Type 类型：**
- `feat`: 新功能
- `fix`: 修复 bug
- `refactor`: 重构（不改变功能）
- `test`: 添加或修改测试
- `docs`: 文档更新
- `style`: 代码格式调整（不影响逻辑）
- `chore`: 构建工具或辅助工具的变动

**示例：**
```
feat: 实现任务创建功能 (FR-1)

- 添加任务创建表单
- 实现 LocalStorage 存储
- 添加表单验证
- 测试覆盖率 45%
```

### 6.2 提交前检查

- [ ] `check.sh` 全绿
- [ ] 测试覆盖率达标
- [ ] 代码已自测
- [ ] Commit message 清晰

---

## 7. 交付清单

### 完成任务后，按顺序执行：

1. **自查**：
   - [ ] 对照 PRD 验收标准逐条检查
   - [ ] 运行 `check.sh` 确保全绿
   - [ ] 手动测试核心流程

2. **更新状态**：
   - [ ] 在 `.ai/plan.md` 中更新任务状态（○ → ✓）

3. **提请 Review**：
   主动输出以下内容交给 Claude：
   ```
   ✅ 已完成任务 X.X [任务名称]
   
   📋 验收标准检查：
   - [x] 标准 1：已实现，测试：tests/xxx.test.ts
   - [x] 标准 2：已实现，测试：tests/xxx.test.ts
   
   🧪 测试情况：
   - 覆盖率：XX%
   - check.sh：全绿
   
   📝 实现说明：
   - 实际改动文件：src/xxx.ts, src/yyy.tsx
   - 新增测试：tests/xxx.test.ts
   - 与方案的偏差：无（或列出偏差及原因）
   
   请 Claude Review。
   ```

---

## 8. 需要停下来问的情况

遇到以下情况，立即停止编码，询问 Claude 或人：

- ❓ 方案不清晰或有歧义
- ❓ 发现 PRD 有冲突或遗漏
- ❓ 需求超出 PRD 范围
- ❓ 技术实现遇到无法解决的问题
- ❓ 需要引入新的第三方库
- ❓ 需要修改架构或分层
- ❓ 发现方案有明显问题或更好的替代方案

**不要**：
- ❌ 自行修改需求
- ❌ 自行调整方案
- ❌ 跳过测试或验证

---

## 9. 第一个任务

**优先任务**：完善 `scripts/` 下的三个脚本

在开始功能开发前，先完善以下脚本：
1. `scripts/check.sh` - 各阶段检查命令
2. `scripts/build.sh` - 构建命令
3. `scripts/test.sh` - 测试命令

技术栈：TypeScript + React

参考脚本中的注释说明和示例，实现对应的命令。

---

## 10. 关键文件索引

- `PRD.md` - 产品需求文档（单一真源）
- `.ai/brief.md` - 项目入口 + 协作契约
- `.ai/plan.md` - 项目计划
- `.ai/review.md` - Review 清单
- `CLAUDE.md` - Claude 的工作规范

---

## 11. 快速参考

### 典型工作流

```
1. 收到 Claude 的交接指令
2. 读 brief.md + PRD.md + 对话历史中的方案
3. 编码 + 写测试
4. 运行 check.sh
5. 自查验收标准
6. 更新 plan.md 状态
7. 主动提请 Review
```

### 提请 Review 话术模板

```
✅ 已完成任务 X.X [任务名称]

📋 验收标准检查：
- [x] 标准 1：已实现，测试：tests/xxx.test.ts
- [x] 标准 2：已实现，测试：tests/xxx.test.ts

🧪 测试情况：
- 覆盖率：XX%
- check.sh：全绿

📝 实现说明：
- 实际改动文件：src/xxx.ts
- 新增测试：tests/xxx.test.ts
- 与方案的偏差：无

请 Claude Review。
```

---

**记住**：你是执行者，Claude 是设计者，人是决策者。按方案实现，主动提请 Review，不越界。

---
> Source: [zc8163623/DualForge-Workflow](https://github.com/zc8163623/DualForge-Workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
