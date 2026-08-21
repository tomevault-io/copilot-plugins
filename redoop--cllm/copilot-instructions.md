## cllm

> - 当用户要求"请继续"或"请测试"时，自动执行相应操作

# cLLM 持续执行工作流程规则

## 核心原则
- 当用户要求"请继续"或"请测试"时，自动执行相应操作
- 遇到错误时，自动分析并修复，然后继续执行
- 完成一个任务后，检查是否需要后续步骤

## 持续执行场景

### 1. 修复-测试-验证循环
```
用户: "修复某个bug"
AI: 修复代码
用户: "请测试"
AI: 自动执行:
  - 编译项目
  - 运行测试
  - 如果失败，分析错误并修复
  - 重复直到成功
```

### 2. 多步骤任务自动完成
```
用户: "实现功能X"
AI: 
  - 创建TODO列表
  - 逐步实现每个步骤
  - 每完成一步，自动验证
  - 继续下一步直到完成
```

### 3. 错误自动处理
```
如果遇到编译错误:
  - 分析错误原因
  - 修复问题
  - 重新编译
  - 重复直到成功

如果遇到运行时错误:
  - 查看日志
  - 定位问题
  - 修复并测试
```

## 工作流程规则

1. **编译任务**: 自动使用 `make -j4` 或检测 CPU 核心数
2. **测试任务**: 自动运行相关测试并报告结果
3. **错误处理**: 遇到错误时自动分析、修复、重试
4. **进度报告**: 完成每个步骤后简要报告进度

## 常用命令模式

- "请继续" -> 继续执行当前任务的下一步
- "请测试" -> 编译、测试、验证当前修改
- "如果失败，请修复" -> 遇到错误时自动修复
- "完成后继续优化" -> 完成当前任务后自动进入优化阶段

## 代码编写规范
- 请遵守.codebuddy/rules/always/00_core_constraints.md中的约束
- 请遵守.codebuddy/rules/always/01_architecture_rules.md中的架构规则
- 请遵守.codebuddy/rules/always/02_workflow_standards.md中的工作流程标准
- 请遵守.codebuddy/rules/manual/performance_optimization.md中的性能优化规则
- 请遵守.codebuddy/rules/manual/refactoring_guide.md中的代码重构规则
- 请遵守.codebuddy/rules/requested/tokenizer_integration.md中的tokenizer集成规则

---
> Source: [redoop/cLLM](https://github.com/redoop/cLLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
