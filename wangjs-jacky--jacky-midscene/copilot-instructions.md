## learning-plan

> Midscene 学习计划 - 从使用到实现的完整学习路径


# Midscene 学习计划

## 📚 学习目标

1. **学会使用 Midscene 工具库**：能够使用 Midscene 完成各种自动化任务
2. **理解 Midscene 实现原理**：从零开始理解并能够实现类似的功能

---

## 📋 学习管理规则

### 作业任务管理

**作业文件位置**：`docs/plan/`

在学习过程中，所有具体的作业任务和练习案例都应该单独记录在 `docs/plan/` 目录下，格式为 Markdown 文档。

**文件命名规范**：
- 按阶段和章节命名，例如：`stage1-1.1-环境搭建.md`、`stage1-1.2-playwright基础.md`
- 或者按作业类型命名，例如：`作业-环境配置.md`、`作业-商品搜索提取.md`

**文件内容结构**：
```markdown
# 作业标题

## 作业描述
[作业的详细描述]

## 完成步骤
1. [步骤1]
2. [步骤2]
...

## 完成状态
- [ ] 未开始
- [ ] 进行中
- [x] 已完成

## 完成时间
[完成日期]

## 遇到的问题
[记录遇到的问题]

## 解决方案
[问题的解决方案]

## 学习总结
[完成作业后的总结和收获]
```

### 常见问题记录（FAQs）

**FAQ 文件位置**：`docs/FAQs/`

在学习过程中遇到的所有问题和解答都应该记录在 `docs/FAQs/` 目录下，格式为 Markdown 文档。

**文件命名规范**：
- 按主题分类，例如：`环境配置问题.md`、`API使用问题.md`、`桥接模式问题.md`
- 或者按时间顺序，例如：`FAQ-2024-01.md`

**文件内容结构**：
```markdown
# 常见问题 - [主题]

## Q1: [问题标题]
**问题描述**：[详细描述问题]

**答案**：[问题的解答]

**相关资源**：
- [相关文档链接]
- [相关代码位置]

**标签**：#环境配置 #API #错误排查

---

## Q2: [问题标题]
...
```

**FAQ 记录原则**：
1. 遇到问题时，先记录问题描述
2. 解决问题后，及时补充答案和解决方案
3. 添加相关资源链接和代码位置，方便后续查阅
4. 使用标签分类，方便检索
5. 定期整理和更新 FAQ 文档

### AI 助手协作规则

当用户在学习过程中：
1. **提问时**：AI 助手应该：
   - 回答用户的问题
   - 提醒用户将问题和答案记录到 `docs/FAQs/` 目录
   - 帮助用户创建或更新 FAQ 文档

2. **开始作业时**：AI 助手应该：
   - 帮助用户在 `docs/plan/` 目录创建作业文档
   - 将具体的作业任务详细记录在作业文档中
   - 跟踪作业完成进度

3. **完成作业时**：AI 助手应该：
   - 帮助用户更新作业文档的完成状态
   - 记录遇到的问题和解决方案
   - 总结学习收获

---

## 🎯 第一阶段：基础使用（1-2周）

### 目标
掌握 Midscene 的基本使用方法，能够完成简单的自动化任务。

### 学习内容

#### 1.1 环境搭建与配置
- 安装 Midscene 依赖包
- 配置 LLM API（环境变量设置）
- 运行连接测试（`connectivity-test/`）
- 理解 Midscene 支持的模型类型

**📝 作业任务**：
1. **基础作业**：
   - 在 `midscene-example/connectivity-test/` 目录下创建 `.env` 文件
   - 配置你的 LLM API Key（OpenAI 或其他支持的模型）
   - 运行 `npm install` 安装依赖
   - 运行 `npm run test` 执行连接测试
   - 确保测试通过，能够成功调用 LLM API

2. **验证作业**：
   - 尝试使用不同的模型（如 OpenAI GPT-4o、Qwen 等）
   - 记录不同模型的响应时间和成本差异
   - 创建一个简单的测试脚本，验证 API 配置是否正确

**🤔 思考问题**：
1. 为什么需要配置环境变量而不是直接在代码中写 API Key？
2. Midscene 支持哪些 LLM 模型？它们之间有什么区别？
3. 如果连接测试失败，可能的原因有哪些？如何排查？
4. `.env` 文件的作用是什么？为什么不应该提交到 Git？

**🚀 扩展问题（附加题）**：
1. 研究 Midscene 的模型配置文档，尝试配置一个本地模型（如 Ollama）
2. 编写一个脚本，自动检测和验证所有配置的环境变量
3. 创建一个配置管理工具，支持在不同环境（开发/生产）之间切换配置

#### 1.2 Playwright 自动化基础
- 学习 Playwright 集成（`playwright-demo/demo.ts`）
- 理解 Agent 的基本概念
- 掌握核心 API：
  - `aiAction()` - 执行操作
  - `aiQuery()` - 查询数据
  - `aiWaitFor()` - 等待条件
  - `aiAssert()` - 断言
  - `aiTap()` - 点击
  - `aiBoolean()` / `aiNumber()` / `aiString()` - 数据提取
  - `aiLocate()` - 定位元素

**📝 作业任务**：
1. **基础作业**：
   - 运行 `playwright-demo/demo.ts` 示例，观察执行过程
   - 理解每一行代码的作用，在代码中添加注释说明
   - 修改示例，将 `headless: true` 改为 `headless: false`，观察浏览器操作过程

2. **实战作业 1 - 商品搜索与提取**：
   - 在 eBay 上搜索"laptop"（或任意商品）
   - 提取前 5 个商品的信息，包括：商品名称、价格、评分、链接
   - 将提取的数据保存为 JSON 文件
   - 使用 `aiAssert()` 验证至少找到了 5 个商品

3. **实战作业 2 - 登录流程自动化**：
   - 选择一个有登录功能的网站（如 GitHub、Reddit 等）
   - 使用 `aiAction()` 完成登录流程
   - 使用 `aiWaitFor()` 等待登录成功
   - 使用 `aiAssert()` 验证登录状态
   - 注意：不要硬编码密码，使用环境变量

4. **实战作业 3 - 数据提取**：
   - 访问一个新闻网站（如 BBC、CNN）
   - 提取首页所有新闻标题和链接
   - 使用 `aiQuery()` 提取结构化数据
   - 将数据保存为 CSV 文件

**🤔 思考问题**：
1. `aiAction()` 和 `aiTap()` 有什么区别？什么时候用哪个？
2. `aiQuery()` 返回的数据结构是如何确定的？如何确保提取的数据格式正确？
3. `aiWaitFor()` 和普通的 `sleep()` 有什么区别？为什么推荐使用 `aiWaitFor()`？
4. 如果 `aiAction()` 执行失败，可能的原因有哪些？如何调试？
5. `aiBoolean()`、`aiNumber()`、`aiString()` 这三个方法的使用场景分别是什么？
6. Agent 是如何理解自然语言指令的？它能看到页面的哪些信息？

**🚀 扩展问题（附加题）**：
1. **挑战任务 1**：创建一个自动化脚本，在电商网站上完成完整的购物流程：
   - 搜索商品 → 查看详情 → 添加到购物车 → 结算（不实际支付）
   - 使用 `aiAssert()` 在每个步骤验证操作是否成功

2. **挑战任务 2**：实现一个数据监控脚本：
   - 定期访问某个网站（如股票价格、天气信息）
   - 提取关键数据
   - 如果数据超过阈值，发送通知（可以打印到控制台）

3. **挑战任务 3**：创建一个表单自动填写工具：
   - 读取 JSON 配置文件中的表单数据
   - 自动填写表单并提交
   - 验证提交是否成功

4. **研究任务**：对比 Midscene 和传统 Playwright 脚本：
   - 用传统 Playwright 实现同样的任务
   - 对比代码量、可读性、维护成本
   - 分析 Midscene 的优势和劣势

#### 1.3 桥接模式（Bridge Mode）深入学习
- 理解桥接模式的概念和优势
  - 复用桌面 Chrome 的 cookies、插件和页面状态
  - "man-in-the-loop" 自动化模式
  - 与自动化脚本协作完成任务
- 安装和配置 Chrome 扩展
  - 从 Chrome 应用商店安装 Midscene 扩展
  - 理解扩展的 Bridge Mode 功能
- 学习桥接模式的基本使用
  - 连接新标签页（`connectNewTabWithUrl()`）
  - 连接当前标签页（`connectCurrentTab()`）
  - 理解 WebSocket 通信机制
- 学习桥接模式的配置选项
  - 远程访问配置（`allowRemoteAccess`）
  - 主机和端口配置
  - 安全注意事项

**📝 作业任务**：
1. **基础作业**：
   - 从 Chrome 应用商店安装 Midscene 扩展
   - 运行 `bridge-mode-demo/demo-new-tab.ts` 示例
   - 运行 `bridge-mode-demo/demo-current-tab.ts` 示例
   - 理解两种连接方式的区别和使用场景
   - 观察浏览器控制台和 Node.js 控制台的输出

2. **实战作业 1 - 已登录网站操作**：
   - 在 Chrome 中手动登录一个网站（如 GitHub、Gmail）
   - 使用 `connectCurrentTab()` 连接当前标签页
   - 编写脚本执行自动化操作（如读取邮件、查看通知等）
   - 验证操作是否成功

3. **实战作业 2 - 利用浏览器插件**：
   - 安装一个浏览器插件（如广告拦截器、翻译插件）
   - 使用桥接模式访问一个网站
   - 观察插件是否正常工作
   - 编写脚本验证插件效果（如验证广告是否被拦截）

4. **实战作业 3 - YAML 脚本集成**：
   - 查看 `bridge-mode-demo/yaml-new-tab.yml` 示例
   - 创建一个 YAML 脚本，使用桥接模式
   - 运行 YAML 脚本，验证功能
   - 对比 YAML 脚本和 TypeScript 脚本的优缺点

**🤔 思考问题**：
1. 桥接模式和 Playwright 模式的主要区别是什么？各自的适用场景是什么？
2. `connectNewTabWithUrl()` 和 `connectCurrentTab()` 的区别是什么？什么时候用哪个？
3. 为什么桥接模式可以复用浏览器的 cookies 和插件？这是如何实现的？
4. WebSocket 通信在桥接模式中的作用是什么？数据是如何在 Node.js 和浏览器之间传输的？
5. "man-in-the-loop" 自动化模式是什么意思？它解决了什么问题？
6. 桥接模式的安全注意事项有哪些？为什么需要配置 `allowRemoteAccess`？
7. 如果连接失败，可能的原因有哪些？如何排查连接问题？

**🚀 扩展问题（附加题）**：
1. **挑战任务 1**：实现一个混合自动化脚本：
   - 使用桥接模式打开一个已登录的网站
   - 执行一些需要登录状态的操作
   - 切换到 Playwright 模式执行其他操作
   - 对比两种模式在执行相同任务时的差异

2. **挑战任务 2**：创建一个远程控制工具：
   - 配置 `allowRemoteAccess: true`
   - 从另一台机器连接并控制浏览器
   - 实现基本的远程操作功能
   - 注意安全配置，不要在生产环境使用

3. **挑战任务 3**：实现一个协作自动化工具：
   - 使用桥接模式，允许用户手动操作浏览器
   - 脚本检测用户操作，自动执行后续步骤
   - 实现"人机协作"的自动化流程

4. **研究任务**：深入理解桥接模式的实现：
   - 查看 Chrome 扩展的源代码（如果可访问）
   - 理解 WebSocket 通信协议
   - 尝试修改连接配置，观察行为变化
   - 记录通信过程中的数据流

#### 1.4 YAML 脚本使用
- 学习 YAML 脚本格式（`yaml-scripts-demo/`）
- 理解 YAML 配置结构：
  - `web` 配置（URL、viewport、输出等）
  - `tasks` 任务列表
  - `flow` 步骤定义
- 运行 YAML 脚本示例

**📝 作业任务**：
1. **基础作业**：
   - 查看 `yaml-scripts-demo/midscene-scripts/` 中的所有示例脚本
   - 理解每个示例的配置结构和执行流程
   - 运行至少 3 个不同的示例脚本
   - 观察生成的报告和输出文件

2. **实战作业 1 - 完整用户流程**：
   - 编写一个 YAML 脚本，自动化电商网站的购物流程：
     - 访问网站 → 搜索商品 → 查看商品详情 → 添加到购物车
   - 使用 `flow` 定义每个步骤
   - 使用 `assert` 验证每个步骤是否成功
   - 生成执行报告

3. **实战作业 2 - 数据提取任务**：
   - 编写一个 YAML 脚本，从新闻网站提取数据：
     - 访问新闻网站首页
     - 提取所有新闻标题、链接、发布时间
     - 将数据保存为 JSON 文件
   - 使用 `output` 配置输出路径
   - 验证提取的数据格式是否正确

4. **实战作业 3 - 多任务脚本**：
   - 创建一个包含多个 `tasks` 的 YAML 脚本
   - 每个任务执行不同的操作
   - 配置任务之间的依赖关系（如果有）
   - 运行脚本，观察任务执行顺序

**🤔 思考问题**：
1. YAML 脚本和 TypeScript 脚本各自的优缺点是什么？什么时候应该用哪个？
2. YAML 脚本中的 `web` 配置项有哪些？它们的作用分别是什么？
3. `flow` 中的步骤是如何执行的？是顺序执行还是可以并行？
4. 如何在 YAML 脚本中处理错误？如何配置重试机制？
5. `output` 配置的作用是什么？可以输出哪些类型的数据？
6. YAML 脚本如何与桥接模式结合使用？
7. 如何调试 YAML 脚本？如果执行失败，如何定位问题？

**🚀 扩展问题（附加题）**：
1. **挑战任务 1**：创建一个复杂的 YAML 脚本：
   - 包含至少 5 个不同的任务
   - 每个任务有多个步骤
   - 使用条件判断（如果支持）
   - 实现错误处理和重试机制

2. **挑战任务 2**：实现一个 YAML 脚本模板系统：
   - 创建可复用的 YAML 模板
   - 支持参数化配置
   - 能够根据不同的输入生成不同的脚本

3. **挑战任务 3**：对比 YAML 和 TypeScript：
   - 用两种方式实现相同的自动化任务
   - 对比代码量、可读性、灵活性
   - 分析各自的适用场景

4. **研究任务**：深入理解 YAML 脚本的执行机制：
   - 查看 Midscene 如何解析和执行 YAML 脚本
   - 理解 YAML 到代码的转换过程
   - 尝试扩展 YAML 脚本的功能（如果可能）

#### 1.5 Playwright 测试框架集成
- Playwright 测试集成（`playwright-testing-demo/`）
- 理解如何在测试中使用 Midscene
- 学习测试报告和调试

**📝 作业任务**：
1. **基础作业**：
   - 查看 `playwright-testing-demo/` 中的示例
   - 理解如何在 Playwright 测试中使用 Midscene
   - 运行示例测试，观察测试报告
   - 理解测试的结构和断言方式

2. **实战作业 1 - 编写 E2E 测试**：
   - 选择一个网站（如电商网站、新闻网站）
   - 使用 Midscene 编写一个完整的 E2E 测试：
     - 测试关键用户流程
     - 使用 `aiAssert()` 进行断言
     - 验证页面元素和数据
   - 运行测试，生成测试报告
   - 分析测试结果

3. **实战作业 2 - 测试套件**：
   - 创建多个测试用例，组成测试套件
   - 每个测试用例测试不同的功能
   - 配置测试的并行执行
   - 生成完整的测试报告

4. **实战作业 3 - 测试数据驱动**：
   - 使用不同的测试数据运行相同的测试
   - 验证测试在不同数据下的表现
   - 处理测试失败的情况

**🤔 思考问题**：
1. 在测试中使用 Midscene 相比传统 Playwright 测试有什么优势？
2. `aiAssert()` 和传统的断言方法（如 `expect()`）有什么区别？
3. 如何编写稳定的 E2E 测试？如何避免测试的脆弱性？
4. Midscene 测试报告包含哪些信息？如何利用报告进行调试？
5. 如何处理测试中的异步操作？如何等待页面加载完成？
6. 测试失败时，如何快速定位问题？有哪些调试技巧？
7. 如何将 Midscene 测试集成到 CI/CD 流程中？

**🚀 扩展问题（附加题）**：
1. **挑战任务 1**：创建一个完整的测试框架：
   - 定义测试规范和标准
   - 实现测试数据管理
   - 实现测试报告生成和分析
   - 集成到 CI/CD 流程

2. **挑战任务 2**：实现测试可视化：
   - 使用 Midscene 的报告功能
   - 创建测试执行的可视化界面
   - 展示测试结果和失败原因
   - 实现测试历史记录

3. **挑战任务 3**：性能测试集成：
   - 使用 Midscene 进行性能测试
   - 测量页面加载时间
   - 分析性能瓶颈
   - 生成性能报告

4. **研究任务**：对比不同的测试方法：
   - 传统 Playwright 测试
   - Midscene AI 测试
   - 混合测试方法
   - 分析各自的适用场景和最佳实践

---

## 📋 第一阶段作业总结

完成第一阶段后，你应该能够：

✅ **环境配置**：
- 能够独立配置 LLM API
- 能够运行连接测试并排查问题
- 了解不同模型的特点和配置方法

✅ **Playwright 基础**：
- 能够使用所有核心 API（aiAction、aiQuery、aiWaitFor 等）
- 能够编写完整的自动化脚本
- 能够处理常见错误和异常情况

✅ **桥接模式**：
- 能够安装和配置 Chrome 扩展
- 能够使用两种连接方式（新标签页、当前标签页）
- 理解桥接模式的优势和使用场景
- 能够在实际场景中应用桥接模式

✅ **YAML 脚本**：
- 能够编写和运行 YAML 脚本
- 理解 YAML 配置结构
- 能够使用 YAML 完成复杂任务

✅ **测试集成**：
- 能够编写 E2E 测试
- 能够生成和分析测试报告
- 理解测试最佳实践

**🎯 完成标准**：
- 完成所有基础作业和至少 2 个实战作业
- 能够回答大部分思考问题
- 尝试完成至少 1 个扩展问题
- 能够独立解决遇到的问题

---

## 🔍 第二阶段：深入理解（2-3周）

### 目标
理解 Midscene 的核心架构和实现原理。

### 学习内容

#### 2.1 核心架构分析

**2.1.1 Agent 类深入理解**
- 阅读 `midscene/packages/core/src/agent/agent.ts`
- 理解 Agent 的职责：
  - 与设备接口交互
  - 管理 Service 和 TaskExecutor
  - 处理缓存和报告
- 分析 Agent 的初始化过程
- 理解 Agent 的各种配置选项

**关键问题**：
- Agent 如何与不同的设备接口（Playwright、Bridge Mode）交互？
- Agent 如何管理任务执行？
- 缓存机制是如何工作的？

**2.1.2 Service 和 UI Context**
- 阅读 `midscene/packages/core/src/service/index.ts`
- 理解 `getUIContext()` 的作用
- 理解截图和 DOM 信息的获取流程
- 分析 UI Context 的数据结构

**关键问题**：
- Service 如何获取页面状态？
- 截图和 DOM 信息如何结合使用？
- 视觉模型如何理解页面？

**2.1.3 TaskExecutor 和任务执行**
- 阅读 `midscene/packages/core/src/agent/tasks.ts`
- 理解 `TaskExecutor` 的工作流程
- 分析任务规划和执行流程
- 理解重试和错误处理机制

**关键问题**：
- 自然语言如何转换为可执行的任务？
- 任务执行失败时如何处理？
- 重试机制是如何实现的？

#### 2.2 AI 模型集成

**2.2.1 LLM 交互机制**
- 阅读 `midscene/packages/core/src/ai-model/` 目录
- 理解 `llm-planning.ts` 中的规划逻辑
- 分析 prompt 工程：
  - `prompt/llm-planning.ts` - 任务规划 prompt
  - `prompt/extraction.ts` - 数据提取 prompt
  - `prompt/assertion.ts` - 断言 prompt
  - `prompt/describe.ts` - 页面描述 prompt
- 理解对话历史管理（`conversation-history.ts`）

**关键问题**：
- Midscene 如何将自然语言指令转换为操作步骤？
- Prompt 是如何设计的？
- 如何优化 prompt 以提高准确性？

**2.2.2 视觉模型定位**
- 阅读 `prompt/ui-tars-locator.ts` 和 `prompt/ui-tars-planning.ts`
- 理解纯视觉定位的原理
- 分析视觉模型如何理解 UI 元素
- 理解定位结果的处理

**关键问题**：
- 如何仅通过截图定位元素？
- 视觉模型如何理解 UI 元素的语义？
- 定位失败时如何处理？

#### 2.3 设备接口抽象

**2.3.1 AbstractInterface**
- 阅读 `midscene/packages/core/src/device/index.ts`
- 理解设备接口的定义
- 分析不同平台的实现：
  - Playwright 实现（`@midscene/web/playwright`）
  - Bridge Mode 实现（`@midscene/web/bridge-mode`）

**关键问题**：
- 如何抽象不同平台的差异？
- 设备接口需要实现哪些方法？
- Playwright 和 Bridge Mode 的实现差异是什么？

**2.3.2 Bridge Mode 实现原理**
- 阅读 `midscene/packages/web-integration/src/bridge-mode/` 目录
- 理解 `AgentOverChromeBridge` 的实现
- 分析 WebSocket 通信机制：
  - `agent-cli-side.ts` - Node.js 端的 Agent 实现
  - `page-browser-side.ts` - 浏览器端的页面代理
  - `io-client.ts` - WebSocket 客户端实现
- 理解 Chrome 扩展与 Node.js 脚本的通信流程
- 分析桥接模式的架构设计

**关键问题**：
- Bridge Mode 如何通过 WebSocket 连接 Node.js 和浏览器？
- Chrome 扩展如何与 Node.js 脚本通信？
- 如何实现远程访问功能？
- Bridge Mode 与 Playwright 模式的主要区别是什么？

**2.3.3 自定义设备接口（可选）**
- 阅读 `custom-interface/src/` 示例
- 理解如何实现自定义设备
- 分析 `SampleDeviceAgent` 的实现

**实践任务**：
- 深入理解 Bridge Mode 的通信机制
- 尝试修改桥接模式的配置选项
- 理解设备接口与 Agent 的交互方式

#### 2.4 缓存机制

- 阅读 `midscene/packages/core/src/agent/task-cache.ts`
- 理解缓存的数据结构
- 分析缓存的存储和读取机制
- 理解缓存如何提高执行效率

**关键问题**：
- 缓存存储了什么信息？
- 如何判断操作是否可缓存？
- 缓存如何与任务执行结合？

#### 2.5 报告和可视化

- 阅读 `midscene/packages/core/src/report.ts`
- 理解报告生成机制
- 分析可视化报告的格式
- 理解如何调试和回放

**关键问题**：
- 报告包含哪些信息？
- 如何利用报告进行调试？
- 可视化如何帮助理解执行过程？

---

## 🛠️ 第三阶段：实现原理（3-4周）

### 目标
从零开始实现 Midscene 的核心功能。

### 学习内容

#### 3.1 最小化实现

**3.1.1 实现基础 Agent 类**
- 设计 Agent 类的接口
- 实现 Agent 的初始化
- 实现基本的设备接口抽象
- 实现简单的 `aiAction()` 方法

**实现目标**：
- 能够接收自然语言指令
- 能够获取页面截图
- 能够调用 LLM 生成操作步骤
- 能够执行基本的点击操作

**3.1.2 实现 Playwright 设备接口**
- 实现 Playwright 的设备接口
- 理解如何将 Agent 操作映射到 Playwright API
- 实现截图获取、DOM 访问等基础功能

**3.1.3 实现 LLM 集成**
- 实现 OpenAI API 调用
- 设计基础的 prompt 模板
- 实现任务规划功能
- 实现操作执行功能

**实现目标**：
- 能够将自然语言转换为操作步骤
- 能够理解页面截图
- 能够生成可执行的操作

**3.1.4 实现视觉定位**
- 实现截图获取
- 实现视觉模型调用
- 实现元素定位功能
- 实现坐标转换

**实现目标**：
- 能够通过视觉模型定位元素
- 能够将定位结果转换为操作坐标
- 能够处理定位失败的情况

#### 3.2 功能扩展

**3.2.1 实现数据提取**
- 实现 `aiQuery()` 方法
- 实现结构化数据提取
- 实现类型转换（Boolean、Number、String）

**3.2.2 实现等待和断言**
- 实现 `aiWaitFor()` 方法
- 实现 `aiAssert()` 方法
- 实现轮询和超时机制

**3.2.3 实现错误处理**
- 实现操作失败检测
- 实现重试机制
- 实现错误恢复策略

#### 3.3 高级功能

**3.3.1 实现缓存机制**
- 设计缓存数据结构
- 实现缓存存储和读取
- 实现缓存匹配逻辑

**3.3.2 实现任务规划优化**
- 实现多步骤任务规划
- 实现任务执行监控
- 实现动态重规划

**3.3.3 实现报告生成**
- 实现执行记录
- 实现报告格式化
- 实现可视化输出

**3.3.4 实现 Bridge Mode（重点）**
- 设计 WebSocket 通信协议
- 实现 Node.js 端的 Bridge Server
- 实现浏览器端的 Bridge Client
- 实现 Chrome 扩展与 Node.js 的通信
- 实现连接管理（新标签页、当前标签页）
- 实现远程访问功能

**实现目标**：
- 能够通过 WebSocket 连接 Node.js 和浏览器
- 能够控制桌面 Chrome 浏览器
- 能够复用浏览器的 cookies 和插件
- 能够支持远程访问配置

---

## 📖 第四阶段：实战项目（2-3周）

### 目标
通过实际项目巩固所学知识。

### 项目建议

#### 项目 1：Web 自动化工具
- 使用 Midscene 实现一个通用的 Web 自动化工具
- 支持多种常见场景：
  - 表单填写
  - 数据抓取
  - 自动化测试
  - 页面监控

#### 项目 2：测试框架集成
- 将 Midscene 深度集成到测试框架中
- 实现测试用例自动生成
- 实现测试报告增强

#### 项目 3：Bridge Mode 增强工具
- 基于 Bridge Mode 实现一个实用的自动化工具
- 支持批量任务执行
- 实现任务队列管理
- 添加远程控制功能

#### 项目 4：简化版 Midscene（包含 Bridge Mode）
- 从零实现一个简化版的 Midscene
- 包含核心功能：
  - Agent 类
  - LLM 集成
  - 视觉定位
  - 基本操作执行

---

## 📝 学习资源

### 官方资源
- [Midscene 官网](https://midscenejs.com/)
- [Midscene 文档](https://midscenejs.com/zh)
- [Midscene GitHub](https://github.com/web-infra-dev/midscene)
- [示例项目](https://github.com/web-infra-dev/midscene-example)

### 代码阅读顺序建议

**第一阶段**：
1. `midscene-example/playwright-demo/demo.ts` - Playwright 基础使用
2. `midscene-example/bridge-mode-demo/` - 桥接模式示例
3. `midscene-example/yaml-scripts-demo/` - YAML 脚本
4. `midscene/packages/core/src/index.ts` - 核心导出

**第二阶段**：
1. `midscene/packages/core/src/agent/agent.ts` - Agent 类
2. `midscene/packages/core/src/service/index.ts` - Service
3. `midscene/packages/core/src/agent/tasks.ts` - TaskExecutor
4. `midscene/packages/core/src/ai-model/llm-planning.ts` - LLM 规划
5. `midscene/packages/core/src/device/index.ts` - 设备接口
6. `midscene/packages/web-integration/src/bridge-mode/` - Bridge Mode 实现（重点）

**第三阶段**：
1. `midscene/packages/core/src/agent/task-builder.ts` - 任务构建
2. `midscene/packages/core/src/ai-model/prompt/` - Prompt 工程
3. `midscene/packages/core/src/agent/task-cache.ts` - 缓存机制
4. `midscene/packages/core/src/report.ts` - 报告生成
5. `midscene/packages/web-integration/src/bridge-mode/agent-cli-side.ts` - Bridge Agent 实现
6. `midscene/packages/web-integration/src/bridge-mode/page-browser-side.ts` - 浏览器端实现
7. `midscene/packages/web-integration/src/bridge-mode/io-client.ts` - WebSocket 通信

### 关键概念理解清单

- Agent 是什么？它的职责是什么？
- Service 的作用是什么？
- UI Context 包含什么信息？
- 任务规划是如何工作的？
- 视觉定位的原理是什么？
- 设备接口如何抽象不同平台（Playwright vs Bridge Mode）？
- Bridge Mode 的通信机制是什么？
- 缓存机制如何提高效率？
- 错误处理和重试如何实现？
- YAML 脚本如何转换为代码执行？

---

## 🎓 学习检查点

### 第一阶段检查点
- 能够独立运行 Playwright 示例
- 能够使用桥接模式控制桌面 Chrome
- 能够编写简单的自动化脚本
- 能够使用 YAML 脚本完成自动化任务
- 理解 Agent 的基本 API
- 理解桥接模式的使用场景和优势

### 第二阶段检查点
- 能够解释 Agent 的工作流程
- 能够理解 LLM 交互机制
- 能够理解视觉定位原理
- 能够理解设备接口抽象（Playwright 和 Bridge Mode）
- 能够理解 Bridge Mode 的通信机制和架构
- 能够阅读和理解核心代码

### 第三阶段检查点
- 能够实现基础的 Agent 功能
- 能够实现 LLM 集成
- 能够实现视觉定位
- 能够实现基本的错误处理
- 能够设计缓存机制
- 能够实现 Bridge Mode 的 WebSocket 通信
- 能够理解并实现桥接模式的完整流程

### 第四阶段检查点
- 完成至少一个实战项目
- 能够解决实际使用中的问题
- 能够扩展 Midscene 功能
- 能够从零实现核心功能

---

## 💡 学习建议

1. **循序渐进**：按照阶段顺序学习，不要跳跃
2. **实践为主**：每学习一个概念，立即编写代码验证
3. **阅读源码**：深入阅读核心代码，理解实现细节
4. **记录笔记**：记录关键概念和实现细节
5. **提问思考**：对每个"关键问题"进行深入思考
6. **参考示例**：充分利用示例代码理解用法
7. **调试实践**：使用报告和可视化工具调试代码
8. **社区交流**：遇到问题及时查阅文档和社区

---

## 📅 时间规划

- **第一阶段**：1-2 周（基础使用）
- **第二阶段**：2-3 周（深入理解）
- **第三阶段**：3-4 周（实现原理）
- **第四阶段**：2-3 周（实战项目）

**总计**：8-12 周（根据个人情况调整）

---

## 🎯 最终目标

完成学习后，你应该能够：

1. ✅ **熟练使用 Midscene**：
   - 能够使用 Playwright 模式完成各种自动化任务
   - 能够熟练使用桥接模式控制桌面 Chrome
   - 能够集成到测试框架中
   - 能够编写 YAML 脚本
   - 能够调试和优化自动化脚本

2. ✅ **深入理解 Midscene**：
   - 理解 Midscene 的架构设计
   - 理解 AI 模型如何驱动自动化
   - 理解视觉定位的原理
   - 理解设备接口抽象（Playwright 和 Bridge Mode）
   - 深入理解桥接模式的实现原理和通信机制

3. ✅ **能够实现类似功能**：
   - 能够从零实现基础的 Agent
   - 能够实现 LLM 集成
   - 能够实现视觉定位
   - 能够实现 Bridge Mode 的 WebSocket 通信
   - 能够扩展和定制功能

---

## 🤖 AI 助手指导原则

当用户在学习 Midscene 时，AI 助手应该：

1. **根据学习阶段提供帮助**：
   - 第一阶段：帮助运行示例、解释 API 用法、解决配置问题、重点讲解桥接模式的使用
   - 第二阶段：解释代码逻辑、回答架构问题、帮助理解核心概念、深入讲解 Bridge Mode 实现原理
   - 第三阶段：指导实现、提供代码示例、帮助调试、重点指导 Bridge Mode 的实现
   - 第四阶段：协助项目开发、提供优化建议、解决实际问题、支持桥接模式相关项目

2. **优先参考学习计划**：
   - 根据用户当前学习阶段，引导到相应的学习内容
   - 提醒用户完成实践任务
   - 帮助用户理解关键问题

3. **提供代码示例和解释**：
   - 当用户询问某个功能时，提供相关代码示例
   - 解释代码的工作原理
   - 指出关键文件和代码位置

4. **鼓励实践和思考**：
   - 鼓励用户动手实践
   - 引导用户思考关键问题
   - 帮助用户完成检查点

5. **参考现有资源**：
   - 优先引导用户查看示例代码
   - 引用官方文档和资源
   - 参考核心代码实现

---
> Source: [wangjs-jacky/jacky-midscene](https://github.com/wangjs-jacky/jacky-midscene) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
