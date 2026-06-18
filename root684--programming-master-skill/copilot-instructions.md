## programming-master-skill

> Professional programming mastery for clean code, optimal architecture, and efficient development. Expertise in code comprehension, refactoring, architecture design, multi-language support, problem-solving, and minimal implementation. Use when: designing system architectures, refactoring codebases, optimizing performance, writing clean code, implementing complex features efficiently, solving technical problems across domains, or requiring senior engineer-level insights. Languages: Python, JavaScript/TypeScript, Go, Rust, Java, C#, Swift, Kotlin, SQL, and more. Domains: web, mobile, backend, DevOps, data, AI/ML, systems programming.


# Programming Master - 编程大师

## 核心原则

### 1. 少即是多 (Less is More)
- **用最少的代码实现最多的功能**
- 消除所有不必要的抽象
- 拒绝过度工程化

### 2. 代码质量优先
- 遵循 SOLID 原则
- 保持高内聚低耦合
- 代码即文档

### 3. 深度理解
- 从需求分析到实现的全流程思考
- 理解技术栈的底层原理
- 预见潜在问题

---

## 工作流程

### 步骤 1: 需求深度分析
在写任何代码之前，先：
1. **理解真正的需求** - 区分"想要"和"需要"
2. **识别核心功能** - 找出 MVP 的最小实现
3. **评估技术方案** - 权衡不同实现路径
4. **预估风险** - 识别潜在技术债务

### 步骤 2: 架构设计
**总是从架构开始，而不是从代码开始：**

```
┌─────────────────────────────────────────────────────────┐
│  ARCHITECTURE DECISION TREE                              │
├─────────────────────────────────────────────────────────┤
│  1. Is it a simple script? → Direct implementation     │
│  2. Is it a web app? → Choose framework + architecture   │
│  3. Is it a backend service? → Microservices vs Monolith│
│  4. Is it data-intensive? → Choose database + caching   │
│  5. Is it performance-critical? → Optimize early       │
└─────────────────────────────────────────────────────────┘
```

### 步骤 3: 模块化实现
- **单一职责** - 每个模块只做一件事
- **可测试性** - 代码应该易于测试
- **可扩展性** - 预留扩展点但不过度设计

### 步骤 4: 代码审查与优化
写完代码后：
1. **自我审查** - 假设这是别人的代码
2. **消除冗余** - 删除所有不必要的代码
3. **性能评估** - 识别瓶颈
4. **安全检查** - 确保没有安全漏洞

---

## 编程范式

### 函数式编程原则
- 纯函数优先
- 不可变数据
- 函数组合
- 避免副作用

### 面向对象原则
- SOLID 原则
- 组合优于继承
- 依赖倒置
- 封装变化

### 响应式编程
- 数据流思维
- 事件驱动
- 背压处理
- 错误恢复

---

## 代码重构模式

### 识别代码异味

| 代码异味 | 解决方案 |
|----------|----------|
| 过长函数 | 提取函数、使用策略模式 |
| 过大类 | 拆分类、使用组合 |
| 重复代码 | 提取公共逻辑、使用模板方法 |
| 过长参数列表 | 使用参数对象、构建器模式 |
| 数据泥团 | 引入数据类 |

### 重构技术
1. **提取函数 (Extract Method)**
2. **内联函数 (Inline Method)**
3. **提取变量 (Extract Variable)**
4. **重命名 (Rename)**
5. **移动函数 (Move Method)**
6. **提取类 (Extract Class)**

---

## 架构模式

### Web 应用架构

```
┌─────────────────────────────────────────────────────────┐
│  Modern Web Architecture                                 │
├─────────────────────────────────────────────────────────┤
│  ┌──────────────┐    ┌──────────────┐                 │
│  │   Frontend   │───▶│     API      │                 │
│  │  (React/Vue) │    │  (GraphQL/   │                 │
│  │              │    │   REST)      │                 │
│  └──────────────┘    └──────┬───────┘                 │
│                              │                           │
│                              ▼                           │
│                    ┌─────────────────┐                  │
│                    │   Application   │                  │
│                    │     Layer       │                  │
│                    └────────┬────────┘                  │
│                             │                           │
│                             ▼                           │
│                    ┌─────────────────┐                  │
│                    │   Domain Logic  │                  │
│                    └────────┬────────┘                  │
│                             │                           │
│                             ▼                           │
│                    ┌─────────────────┐                  │
│                    │  Infrastructure │                  │
│                    │     Layer       │                  │
│                    └─────────────────┘                  │
└─────────────────────────────────────────────────────────┘
```

### 微服务架构原则
- 服务边界清晰
- 独立部署
- 数据隔离
- 异步通信

### 事件驱动架构
- 事件溯源
- CQRS
- 最终一致性
- 事件总线

---

## 性能优化

### 前端性能
1. **代码分割** - 按需加载
2. **懒加载** - 延迟非关键资源
3. **缓存策略** - 合理使用缓存
4. **渲染优化** - 避免不必要的重渲染

### 后端性能
1. **数据库优化** - 索引、查询优化
2. **缓存层** - Redis、应用级缓存
3. **异步处理** - 队列、后台任务
4. **连接池** - 复用数据库连接

### 算法优化
- 时间复杂度分析
- 空间优化
- 数据结构选择
- 并行计算

---

## 安全最佳实践

### 认证与授权
- 强密码策略
- 多因素认证
- 最小权限原则
- Token 管理

### 数据安全
- 输入验证
- SQL 注入防护
- XSS 防护
- CSRF 防护
- 数据加密

### 运维安全
- 定期更新依赖
- 安全审计日志
- 漏洞扫描
- 渗透测试

---

## 测试策略

### 测试金字塔
```
        /\
       /E2E\       <-- 少量端到端测试
      /------\
     /Integration\  <-- 集成测试
    /------------\
   /   Unit Tests  \ <-- 大量单元测试
  /----------------\
```

### 测试原则
- **快速** - 单元测试应该快速运行
- **独立** - 测试之间不应该相互依赖
- **可重复** - 每次运行结果应该一致
- **自我验证** - 测试应该能够判断自己是否通过

---

## 多语言最佳实践

### Python
- 使用类型提示
- 遵循 PEP 8
- 使用上下文管理器
- 生成器优于列表

### JavaScript/TypeScript
- 严格模式
- TypeScript 优先
- 异步/等待
- 模块导入导出

### Go
- 简单是美
- 并发原语
- 接口编程
- 错误处理

### Rust
- 所有权系统
- 零成本抽象
- 模式匹配
- 内存安全

---

## 问题解决方法论

### 调试技巧
1. **重现问题** - 稳定的复现步骤
2. **缩小范围** - 二分法定位
3. **验证假设** - 科学方法
4. **文档修复** - 记录根因和解决方案

### 疑难杂症
- **性能问题** - 使用 profiler
- **内存泄漏** - 分析 heap dump
- **死锁** - 检查锁顺序
- **竞态条件** - 使用并发原语

---

## 代码审查清单

### 功能正确性
- [ ] 需求是否完全实现
- [ ] 边界条件是否处理
- [ ] 错误处理是否完善
- [ ] 测试是否覆盖

### 代码质量
- [ ] 命名是否清晰
- [ ] 函数是否简短
- [ ] 注释是否必要
- [ ] 重复代码是否消除

### 架构设计
- [ ] 模块划分是否合理
- [ ] 依赖方向是否正确
- [ ] 扩展性是否考虑
- [ ] 技术选型是否合适

### 性能考虑
- [ ] 有无性能瓶颈
- [ ] 缓存是否合理使用
- [ ] 数据库查询是否优化
- [ ] 资源是否正确释放

### 安全检查
- [ ] 输入是否验证
- [ ] 认证授权是否正确
- [ ] 敏感数据是否保护
- [ ] 漏洞是否修复

---

## 最小实现原则

### YAGNI - You Aren't Gonna Need It
- 只实现当前需要的功能
- 不为未来可能需要的功能预留
- 保持代码简单，需要时再扩展

### KISS - Keep It Simple, Stupid
- 最简单的方案往往是最好的
- 避免过度设计
- 代码应该易于理解

### DRY - Don't Repeat Yourself
- 消除重复代码
- 提取公共逻辑
- 单一事实来源

---

## 持续学习

### 技术栈深度
- 深入理解一门语言
- 掌握核心概念
- 阅读优秀源码
- 参与开源项目

### 技术广度
- 了解不同范式
- 探索新工具
- 关注行业趋势
- 跨领域学习

---

## 参考资源

### 书籍
- 《Clean Code》- Robert C. Martin
- 《Design Patterns》- GoF
- 《Refactoring》- Martin Fowler
- 《Domain-Driven Design》- Eric Evans

### 在线资源
- GitHub 优秀项目
- 技术博客和论文
- 开源社区讨论
- 官方文档

---

## 快速参考

### 常见任务指南

| 任务 | 推荐方法 |
|------|----------|
| 新项目 | 从架构设计开始，用最简单的方案 |
| 代码重构 | 先写测试，再小步重构 |
| 性能问题 | 先测量，再优化 |
| Bug 修复 | 先理解根因，再修复 |
| 技术选型 | 权衡利弊，适合的才是最好的 |

### 设计模式速查

| 场景 | 模式 |
|------|------|
| 创建对象 | 工厂方法、构建器 |
| 结构组织 | 适配器、装饰器、组合 |
| 行为管理 | 策略、观察者、命令 |
| 并发 | 生产者消费者、 futures |

---

## 最后的话

> "任何傻瓜都能写出计算机能理解的代码。优秀的程序员写出人类能理解的代码。"
> — Martin Fowler

> "先让它工作，再让它正确，最后让它快速。"
> — Kent Beck

> "简单是可靠的先决条件。"
> — Edsger W. Dijkstra

这个技能的目标是让你成为一个**编程大师**，用最少的代码实现最强大的功能，同时保持代码的优雅和可维护性。记住：代码是写给人看的，只是顺便让计算机执行。

---
> Source: [root684/programming-master-skill](https://github.com/root684/programming-master-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
