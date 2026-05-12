## burn-in-cceverywhere-codex

> > **English** | [中文](#中文版本)

# Project Guidance for burn-in-cceverywhere-codex

> **English** | [中文](#中文版本)

This document provides project-specific guidance for working with the Codex autonomous agent system.

## Available Skills

This project provides 7 skills installed in `~/.codex/skills/`:

1. **ralph** - PRD to prd.json conversion for autonomous agent loops
2. **coding-standards** - Universal coding standards and best practices
3. **tdd-workflow** - Test-driven development workflow enforcement
4. **backend-patterns** - Backend architecture patterns and API design
5. **frontend-patterns** - React/Next.js patterns and frontend best practices
6. **security-checklist** - Security vulnerability detection and OWASP Top 10
7. **code-review-checklist** - Code review checklist for quality and maintainability

Trigger these skills by asking relevant questions (e.g., "Show me TypeScript coding standards" triggers `coding-standards`).

## Git Workflow

### Commit Message Format

```
<type>: <description>

<optional body>
```

**Types**: feat, fix, refactor, docs, test, chore, perf, ci

### Pull Request Workflow

When creating PRs:
1. Analyze full commit history (not just latest commit)
2. Use `git diff [base-branch]...HEAD` to see all changes
3. Draft comprehensive PR summary
4. Include test plan with TODOs
5. Push with `-u` flag if new branch

### Feature Implementation Workflow

1. **Plan First**
   - Create detailed implementation plan
   - Identify dependencies and risks
   - Break down into phases

2. **TDD Approach**
   - Write tests first (RED)
   - Implement to pass tests (GREEN)
   - Refactor (IMPROVE)
   - Verify 80%+ coverage

3. **Code Review**
   - Use `code-review-checklist` skill
   - Address CRITICAL and HIGH issues
   - Fix MEDIUM issues when possible

4. **Commit & Push**
   - Detailed commit messages
   - Follow conventional commits format

## Common Patterns

### API Response Format

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}
```

### Custom Hooks Pattern

```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}
```

### Repository Pattern

```typescript
interface Repository<T> {
  findAll(filters?: Filters): Promise<T[]>
  findById(id: string): Promise<T | null>
  create(data: CreateDto): Promise<T>
  update(id: string, data: UpdateDto): Promise<T>
  delete(id: string): Promise<void>
}
```

## Performance Guidelines

### Model Selection

**gpt-5.2** (Primary model for all tasks):
- Main development work
- Code generation and pair programming
- Complex coding tasks
- Architectural decisions
- Research and analysis tasks

Note: Codex CLI uses gpt-5.2 as the unified model for all operations.

### Context Window Management

Avoid last 20% of context window for:
- Large-scale refactoring
- Feature implementation spanning multiple files
- Debugging complex interactions

Lower context sensitivity tasks:
- Single-file edits
- Independent utility creation
- Documentation updates
- Simple bug fixes

### Deep Reasoning

For complex tasks:
1. Use detailed planning before implementation
2. Break down complex problems into smaller steps
3. Verify assumptions before proceeding
4. Test thoroughly after implementation

## Build Troubleshooting

If build fails:
1. Analyze error messages carefully
2. Fix incrementally
3. Verify after each fix
4. Run full test suite before marking complete

## Multi-agents (Experimental)

This project supports Codex multi-agents for parallel task execution:

```javascript
// Example: Parallel code review
const security = spawn_agent("Security review of auth.ts", "worker")
const performance = spawn_agent("Performance review of queries.ts", "worker")

wait([security.agent_id, performance.agent_id])

close_agent(security.agent_id)
close_agent(performance.agent_id)
```

Enable in `~/.codex/config.toml`:
```toml
[features]
collab = true
```

## Testing Requirements

- Minimum 80% test coverage
- Unit tests for all functions
- Integration tests for APIs
- E2E tests for critical flows

Use `tdd-workflow` skill for TDD guidance.

## Security

Use `security-checklist` skill before commits:
- No hardcoded secrets
- Input validation on all endpoints
- SQL queries parameterized
- Output escaped (XSS prevention)
- `npm audit` clean

## Code Quality

Use `code-review-checklist` skill after writing code:
- Functions < 50 lines
- Files < 800 lines
- No deep nesting (< 4 levels)
- Immutability patterns
- Proper error handling

## Resources

- Ralph Documentation: `~/.codex/ralph/AGENTS.md`
- Skills: `~/.codex/skills/*/SKILL.md`
- Examples: `docs/examples/`

---

# 中文版本

> [English](#project-guidance-for-burn-in-cceverywhere-codex) | **中文**

本文档为 Codex 自主代理系统提供项目特定的指导。

## 可用技能

本项目在 `~/.codex/skills/` 中提供了7个技能：

1. **ralph** - PRD到prd.json的转换，用于自主代理循环
2. **coding-standards** - 通用编码标准和最佳实践
3. **tdd-workflow** - 测试驱动开发工作流程
4. **backend-patterns** - 后端架构模式和API设计
5. **frontend-patterns** - React/Next.js模式和前端最佳实践
6. **security-checklist** - 安全漏洞检测和OWASP Top 10
7. **code-review-checklist** - 代码审查清单，确保质量和可维护性

通过提问相关问题来触发这些技能（例如，"告诉我TypeScript编码标准"会触发 `coding-standards`）。

## Git工作流

### 提交信息格式

```
<类型>: <描述>

<可选的正文>
```

**类型**：feat, fix, refactor, docs, test, chore, perf, ci

### Pull Request工作流

创建PR时：
1. 分析完整的提交历史（不仅仅是最新提交）
2. 使用 `git diff [base-branch]...HEAD` 查看所有更改
3. 起草全面的PR摘要
4. 包含测试计划和待办事项
5. 如果是新分支，使用 `-u` 标志推送

### 功能实现工作流

1. **先规划**
   - 创建详细的实现计划
   - 识别依赖关系和风险
   - 分解为多个阶段

2. **TDD方法**
   - 先写测试（RED）
   - 实现以通过测试（GREEN）
   - 重构（IMPROVE）
   - 验证80%+覆盖率

3. **代码审查**
   - 使用 `code-review-checklist` 技能
   - 解决CRITICAL和HIGH级别的问题
   - 尽可能修复MEDIUM级别的问题

4. **提交和推送**
   - 详细的提交信息
   - 遵循conventional commits格式

## 常见模式

### API响应格式

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}
```

### 自定义Hooks模式

```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}
```

### Repository模式

```typescript
interface Repository<T> {
  findAll(filters?: Filters): Promise<T[]>
  findById(id: string): Promise<T | null>
  create(data: CreateDto): Promise<T>
  update(id: string, data: UpdateDto): Promise<T>
  delete(id: string): Promise<void>
}
```

## 性能指南

### 模型选择

**gpt-5.2**（所有任务的主要模型）：
- 主要开发工作
- 代码生成和结对编程
- 复杂编码任务
- 架构决策
- 研究和分析任务

注意：Codex CLI使用gpt-5.2作为所有操作的统一模型。

### 上下文窗口管理

避免在最后20%的上下文窗口中进行：
- 大规模重构
- 跨多个文件的功能实现
- 调试复杂交互

较低上下文敏感性任务：
- 单文件编辑
- 独立实用程序创建
- 文档更新
- 简单错误修复

### 深度推理

对于复杂任务：
1. 实施前使用详细规划
2. 将复杂问题分解为更小的步骤
3. 在继续之前验证假设
4. 实施后彻底测试

## 构建故障排除

如果构建失败：
1. 仔细分析错误消息
2. 逐步修复
3. 每次修复后验证
4. 在标记完成前运行完整测试套件

## Multi-agents（实验性）

本项目支持Codex multi-agents进行并行任务执行：

```javascript
// 示例：并行代码审查
const security = spawn_agent("Security review of auth.ts", "worker")
const performance = spawn_agent("Performance review of queries.ts", "worker")

wait([security.agent_id, performance.agent_id])

close_agent(security.agent_id)
close_agent(performance.agent_id)
```

在 `~/.codex/config.toml` 中启用：
```toml
[features]
collab = true
```

## 测试要求

- 最低80%测试覆盖率
- 所有函数的单元测试
- API的集成测试
- 关键流程的E2E测试

使用 `tdd-workflow` 技能获取TDD指导。

## 安全性

提交前使用 `security-checklist` 技能：
- 无硬编码秘密
- 所有端点上的输入验证
- SQL查询参数化
- 输出转义（XSS预防）
- `npm audit` 干净

## 代码质量

编写代码后使用 `code-review-checklist` 技能：
- 函数 < 50行
- 文件 < 800行
- 无深度嵌套（< 4层）
- 不可变性模式
- 适当的错误处理

## 资源

- Ralph文档：`~/.codex/ralph/AGENTS.md`
- 技能：`~/.codex/skills/*/SKILL.md`
- 示例：`docs/examples/`

---
> Source: [hellangleZ/burn-in-cceverywhere-codex](https://github.com/hellangleZ/burn-in-cceverywhere-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
