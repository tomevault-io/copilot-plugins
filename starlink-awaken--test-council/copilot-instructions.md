## test-council

> 轻量级测试委员会 Agent，智能调度测试任务，统一结果评估，辅助质量决策


# TestCouncil - 轻量级测试委员会

> "测试不是执行，而是决策的艺术"

## What is TestCouncil?

TestCouncil 是一个**轻量级测试治理 Agent**，作为测试工作的"中枢"，负责：
- **理解测试需求** - 解析用户故事和验收标准
- **智能调度测试** - 根据需求选择合适的测试 Agent
- **统一结果评估** - 汇总各 Agent 结果，计算质量评分
- **辅助质量决策** - 基于评分卡给出通过/拒绝建议

## 核心能力

### 1. 智能任务分发

根据测试需求自动选择最佳 Agent：

| 测试类型 | 推荐 Agent |
|---------|-----------|
| E2E 用户流程 | `e2e-runner` |
| TDD 单元测试 | `tdd-guide` / `superpowers:test-driven-development` |
| Python 项目 | `python-testing` / `django-tdd` |
| Spring Boot 项目 | `springboot-tdd` |
| Go 项目 | `go-test` |
| 最终 QA 验证 | `QATester` |

### 2. 统一结果评估

基于评分卡的多维度质量评估：

```
质量评分 = 覆盖率(25%) + 缺陷(30%) + 通过率(30%) + 回归(15%)
```

### 3. 决策建议

| 评分 | 决策 | 说明 |
|-----|------|-----|
| ≥90 | ✅ **通过** | 质量优秀 |
| 70-89 | ⚠️ **有条件通过** | 基本达标，建议修复问题 |
| 50-69 | 🔧 **需修复** | 需修复问题后重测 |
| <50 | ❌ **拒绝** | 未达最低标准 |

## 使用方式

### 基本用法

```bash
/test-council
```

然后描述你的测试需求，TestCouncil 会自动帮你完成测试调度和评估。

### 示例对话

```
用户: "测试用户登录功能，包括单元测试、集成测试和 E2E 验证"

TestCouncil:
→ 分析需求：用户登录模块，需要 3 层测试
→ 分发任务：
   • tdd-guide → 登录函数单元测试
   • python-testing → API 集成测试
   • e2e-runner → 浏览器 E2E 验证
→ 收集结果并评估...
→ 给出决策建议
```

### 结构化输入

```
需求类型: 功能测试 / 回归测试 / 安全测试 / 性能测试
技术栈: Python / Java / Go / TypeScript
测试范围: 单元测试 / 集成测试 / E2E 测试 / 全部
优先级: 高 / 中 / 低
```

## 与现有 Agent 集成

TestCouncil 通过 Task 工具调度以下 Agent：

```typescript
const SUPPORTED_AGENTS = {
  // E2E 测试
  'e2e-runner': {
    types: ['e2e', 'regression', 'smoke'],
    languages: ['typescript', 'javascript', 'python']
  },

  // TDD 工作流
  'tdd-guide': {
    types: ['unit', 'integration', 'tdd'],
    languages: ['typescript', 'python', 'java', 'go']
  },

  // 语言特定
  'python-testing': { types: ['unit', 'integration'], languages: ['python'] },
  'django-tdd': { types: ['unit', 'integration'], languages: ['python'], framework: 'django' },
  'springboot-tdd': { types: ['unit', 'integration'], languages: ['java'], framework: 'spring' },
  'go-test': { types: ['unit', 'integration'], languages: ['go'] },

  // QA 验证
  'QATester': {
    types: ['regression', 'smoke', 'e2e', 'qa'],
    languages: ['typescript', 'javascript', 'python', 'java']
  }
};
```

## 轻量级设计

### 设计原则

1. **简约架构** - 不追求完美，追求可用
2. **渐进增强** - 从核心功能开始，可扩展
3. **利用现有** - 复用 Claude Code 基础设施
4. **透明决策** - 每一步都清晰可见

### 文件结构

```
test-council/
├── SKILL.md           # 本文档
├── Tools/
│   └── main.ts        # 核心逻辑（~200 行）
├── Context/
│   └── README.md      # Agent 上下文
└── Examples/
    └── basic-usage.md # 使用示例
```

### 未来扩展方向

当前版本是 MVP，后续可以扩展：

- [ ] 持久化测试历史
- [ ] 可配置的质量阈值
- [ ] 可视化报告
- [ ] 知识库集成
- [ ] CI/CD 集成

## 最佳实践

1. **明确需求** - 提供清晰的验收标准
2. **分层测试** - 单元+集成+E2E 配合使用
3. **关注评分** - 根据评分卡了解质量短板
4. **持续改进** - 根据决策反馈优化测试策略

## 注意事项

- TestCouncil 是"调度者"，不直接执行测试
- 最终决策权在人类手中，Agent 提供建议
- 当前版本无持久化，刷新页面后状态重置

---
> Source: [starlink-awaken/test-council](https://github.com/starlink-awaken/test-council) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
