## rt-thread

> RT-Thread is a real-time operating system (RTOS) for embedded devices. When working with RT-Thread code, please follow these guidelines to ensure high-quality contributions.

# GitHub Copilot Instructions for RT-Thread / RT-Thread GitHub Copilot 指南

## Overview / 概述

RT-Thread is a real-time operating system (RTOS) for embedded devices. When working with RT-Thread code, please follow these guidelines to ensure high-quality contributions.

RT-Thread 是一个面向嵌入式设备的实时操作系统（RTOS）。在处理 RT-Thread 代码时，请遵循以下指南以确保高质量的贡献。

**When reviewing Pull Requests (PRs), you MUST check all items in the PR Review Checklist section and provide feedback according to the PR Review Instructions. / 在审查 Pull Request (PR) 时，必须检查 PR 审查清单部分中的所有项目，并根据 PR 审查指令提供反馈。**

## Code Review Guidelines / 代码审查指南

### Language Requirements / 语言要求

When reviewing code, provide feedback in **both English and Chinese** to ensure accessibility for all contributors.

在审查代码时，请同时使用**英文和中文**提供反馈，以确保所有贡献者都能理解。

### Review Focus Areas / 审查重点领域

1. **Real-Time Performance / 实时性能**
   - Verify interrupt handling efficiency / 验证中断处理效率
2. **Memory Management / 内存管理**
   - Detect memory leaks / 检测内存泄漏
   - Verify proper memory allocation/deallocation / 验证正确的内存分配/释放
   - Check stack usage optimization / 检查栈使用优化
3. **Code Style / 代码风格**
   - Follow RT-Thread coding standards / 遵循 RT-Thread 编码标准
   - Maintain consistent naming conventions / 保持一致的命名约定
   - Ensure proper code comments (not documentation) / 确保适当的代码注释（而非文档）
4. **PR Review Checklist / PR 审查清单**
   - **PR Title Review / PR 标题审查**：
     - Check if PR title has proper prefix format / 检查 PR 标题是否有正确的前缀格式
     - Verify prefix follows pattern: `[module/vendor][subsystem]` or `[module/vendor]` in lowercase / 验证前缀遵循格式：小写的 `[模块/厂商][子系统]` 或 `[模块/厂商]`
     - Verify title describes changes based on modified files / 验证标题基于修改的文件描述变更
     - Check if title is specific enough (avoid vague terms like "fix bug", "optimize code") / 检查标题是否足够具体（避免模糊术语如"修复问题"、"代码优化"）
     - If title lacks prefix or uses incorrect format, suggest: "PR title should follow format: `[module][subsystem] Description`. Example: `[stm32][drivers] Fix UART interrupt handling issue`" / 如果标题缺少前缀或格式错误，建议："PR 标题应遵循格式：`[模块][子系统] 描述`。示例：`[stm32][drivers] Fix UART interrupt handling issue`"
   - **PR Description Review / PR 内容审查**：
     - Check if PR description provides overview of modified files / 检查 PR 描述是否提供了修改文件的总概
     - Verify description explains: What (what changes), Why (why needed), How (which files modified) / 验证描述是否说明：What（做了什么修改）、Why（为什么需要）、How（修改了哪些文件）
     - If description is missing or insufficient, suggest adding description with modified files list / 如果描述缺失或不充分，建议添加包含修改文件列表的描述
   - **PR File Modification Review / PR 修改文件审查**：
     - Check if PR contains multiple unrelated features / 检查 PR 是否包含多个不相关的特性
     - If PR mixes multiple features, suggest splitting into separate PRs / 如果 PR 混杂多个特性，建议拆分为多个 PR
     - Verify all file changes are related to the same feature/bug fix / 验证所有文件修改是否与同一功能/错误修复相关
   - **PR Commit Review / PR Commit 审查**：
     - Check commit message format (should follow PR title format) / 检查 commit 消息格式（应遵循 PR 标题格式）
     - Verify if commits are properly organized / 验证 commit 是否组织得当
     - If routine changes (style fixes, minor bugs, docs, typos) are split into multiple commits, suggest squashing them / 如果常规修改（风格修复、小错误、文档、拼写）被拆分为多个 commit，建议压缩它们
     - Important commits (major features, refactoring, critical bugs) can remain separate / 重要提交（主要功能、重构、关键错误）可以保持独立
     - If commit messages don't follow format, suggest: "Commit message should follow format: `[module][subsystem] Description`. Consider squashing routine changes into one commit." / 如果 commit 消息不符合格式，建议："Commit 消息应遵循格式：`[模块][子系统] 描述`。考虑将常规修改压缩为一个 commit。"

### PR Review Instructions / PR 审查指令

**When reviewing a PR, you MUST systematically check the following items and provide feedback for any violations / 审查 PR 时，必须系统性地检查以下项目，并对任何违规行为提供反馈：**

#### Step 1: PR Title Check / 步骤 1：PR 标题检查

1. **Check prefix format / 检查前缀格式**:
   - Extract the PR title from the PR / 从 PR 中提取 PR 标题
   - Verify if it starts with `[module][subsystem]` or `[module]` pattern in lowercase / 验证是否以小写的 `[模块][子系统]` 或 `[模块]` 格式开头
   - Check if prefix matches the modified files (e.g., if files are in `bsp/stm32/drivers/`, prefix should be `[stm32][drivers]` or similar) / 检查前缀是否与修改的文件匹配（例如，如果文件在 `bsp/stm32/drivers/`，前缀应为 `[stm32][drivers]` 或类似）
   - If prefix is missing, incorrect case (e.g., `[STM32]`), or doesn't match files, provide feedback / 如果前缀缺失、大小写错误（如 `[STM32]`）或不匹配文件，提供反馈
   - **Feedback template / 反馈模板**:
     ```
     🟡 [PR Title/PR 标题]: Missing or incorrect prefix format / 缺少或错误的前缀格式
     
     English: PR title should follow format: `[module][subsystem] Description` in lowercase. 
     Current title: `{current_title}`. 
     Based on modified files, suggested title: `{suggested_title}`.
     
     中文：PR 标题应遵循格式：小写的 `[模块][子系统] 描述`。
     当前标题：`{current_title}`。
     基于修改的文件，建议标题：`{suggested_title}`。
     ```

2. **Check title specificity / 检查标题具体性**:
   - Analyze modified files to understand what changes were made / 分析修改的文件以了解所做的更改
   - Verify if title accurately describes changes based on modified files / 验证标题是否基于修改的文件准确描述更改
   - Check for vague terms: "fix bug", "optimize code", "update", "modify", etc. / 检查模糊术语："修复问题"、"代码优化"、"更新"、"修改"等
   - If title is vague or doesn't match modified files, suggest a more specific title / 如果标题模糊或不匹配修改的文件，建议更具体的标题
   - **Feedback template / 反馈模板**:
     ```
     🟡 [PR Title/PR 标题]: Title is too vague or doesn't match modified files / 标题过于模糊或不匹配修改的文件
     
     English: PR title should specifically describe changes based on modified files. 
     Current title: `{current_title}`. 
     Suggested: `{suggested_title}` based on files: {list_modified_files}.
     
     中文：PR 标题应基于修改的文件具体描述更改。
     当前标题：`{current_title}`。
     建议：基于文件 {list_modified_files} 的 `{suggested_title}`。
     ```

#### Step 2: PR Description Check / 步骤 2：PR 内容检查

1. **Check description completeness / 检查描述完整性**:
   - Read the PR description / 阅读 PR 描述
   - Verify if it includes: / 验证是否包含：
     - Overview of modified files / 修改文件的总概
     - What changes were made / 做了什么修改
     - Why changes are needed / 为什么需要这些修改
     - List of modified files (optional but recommended) / 修改文件列表（可选但推荐）
   - If description is missing, empty, or insufficient, provide feedback / 如果描述缺失、为空或不充分，提供反馈
   - **Feedback template / 反馈模板**:
     ```
     🟢 [PR Description/PR 描述]: Missing or insufficient description / 缺少或不充分的描述
     
     English: PR description should include: (1) Overview of modified files, (2) What changes were made, (3) Why changes are needed, (4) List of modified files (optional). 
     Please add/modify the PR description.
     
     中文：PR 描述应包含：(1) 修改文件的总概，(2) 做了什么修改，(3) 为什么需要这些修改，(4) 修改文件列表（可选）。
     请添加/修改 PR 描述。
     
     Example format / 示例格式:
     ## Description / 描述
     This PR fixes the UART interrupt handling issue in STM32 serial driver.
     本次 PR 修复了 STM32 串口驱动中的中断处理问题。
     
     ## Modified Files / 修改文件
     - `bsp/stm32/drivers/drv_usart.c`: Fixed interrupt handler logic
     - `bsp/stm32/drivers/drv_usart.h`: Updated function declarations
     ```

#### Step 3: PR File Modification Check / 步骤 3：PR 修改文件检查

1. **Check feature separation / 检查特性分离**:
   - List all modified files in the PR / 列出 PR 中的所有修改文件
   - Group files by feature/functionality / 按特性/功能对文件进行分组
   - Identify if multiple unrelated features are mixed / 识别是否混杂了多个不相关的特性
   - Unrelated features include: different drivers, different subsystems, unrelated bug fixes, etc. / 不相关的特性包括：不同的驱动、不同的子系统、不相关的错误修复等
   - If multiple unrelated features are found, provide feedback with specific suggestions / 如果发现多个不相关的特性，提供具体建议的反馈
   - **Feedback template / 反馈模板**:
     ```
     🟡 [PR Structure/PR 结构]: Multiple unrelated features in one PR / 一个 PR 中包含多个不相关的特性
     
     English: This PR contains multiple unrelated features: {list_features}. 
     Please split into separate PRs, each focusing on one feature. 
     Suggested PRs:
     - PR 1: `[module1][subsystem1] {feature1_description}` (files: {list_files1})
     - PR 2: `[module2][subsystem2] {feature2_description}` (files: {list_files2})
     
     中文：此 PR 包含多个不相关的特性：{list_features}。
     请拆分为多个 PR，每个专注于一个特性。
     建议的 PR：
     - PR 1: `[模块1][子系统1] {特性1描述}` (文件: {list_files1})
     - PR 2: `[模块2][子系统2] {特性2描述}` (文件: {list_files2})
     ```

#### Step 4: PR Commit Check / 步骤 4：PR Commit 检查

1. **Check commit message format / 检查 commit 消息格式**:
   - Review all commit messages in the PR / 审查 PR 中的所有 commit 消息
   - Verify if each commit message follows format: `[module][subsystem] Description` / 验证每个 commit 消息是否遵循格式：`[module][subsystem] 描述`
   - Check if commit message prefix matches PR title prefix / 检查 commit 消息前缀是否与 PR 标题前缀匹配
   - If commit messages don't follow format, provide feedback / 如果 commit 消息不符合格式，提供反馈
   - **Feedback template / 反馈模板**:
     ```
     🟡 [Commit Message/Commit 消息]: Commit message format violation / Commit 消息格式违规
     
     English: Commit message should follow format: `[module][subsystem] Description`. 
     Invalid commits: {list_invalid_commits}. 
     Example: `[stm32][drivers] Fix UART interrupt handling issue`.
     
     中文：Commit 消息应遵循格式：`[模块][子系统] 描述`。
     无效的 commit：{list_invalid_commits}。
     示例：`[stm32][drivers] Fix UART interrupt handling issue`。
     ```

2. **Check commit organization / 检查 commit 组织**:
   - Identify routine changes: style fixes, minor bugs, documentation updates, typo corrections / 识别常规修改：风格修复、小错误、文档更新、拼写错误修正
   - Identify important changes: major features, significant refactoring, critical bug fixes / 识别重要更改：主要功能、重大重构、关键错误修复
   - Check if routine changes are split into multiple commits / 检查常规修改是否被拆分为多个 commit
   - If routine changes are split, suggest squashing them / 如果常规修改被拆分，建议压缩它们
   - **Feedback template / 反馈模板**:
     ```
     🟢 [Commit Organization/Commit 组织]: Routine changes should be squashed / 常规修改应压缩
     
     English: Routine changes (style fixes, minor bugs, docs, typos) should be squashed into one commit. 
     Commits to squash: {list_commits_to_squash}. 
     Please use `git rebase -i` to squash these commits.
     
     中文：常规修改（风格修复、小错误、文档、拼写）应压缩为一个 commit。
     要压缩的 commit：{list_commits_to_squash}。
     请使用 `git rebase -i` 压缩这些 commit。
     ```

### Review Comment Format / 审查评论格式

When providing review comments, use the following format:

提供审查评论时，请使用以下格式：

```
[Category/类别]: Brief description / 简要描述

English: Detailed explanation of the issue and suggested improvement.
中文：问题的详细说明和改进建议。

Example/示例:
```c
// Your code example here / 你的代码示例
```
```

**For PR-related issues, use severity level 🟡 Minor or 🟢 Suggestion / 对于 PR 相关的问题，使用严重程度级别 🟡 Minor 或 🟢 Suggestion**
### Common Issues to Check / 常见问题检查

1. **Resource Management / 资源管理**
   - Unclosed file handles / 未关闭的文件句柄
   - Unreleased semaphores / 未释放的信号量
   - Memory not freed after malloc / malloc 后未释放内存

2. **Error Handling / 错误处理**
   - Missing error checks / 缺少错误检查
   - Improper error propagation / 不当的错误传播
   - Silent failures / 静默失败

3. **Performance Concerns / 性能问题**
   - Unnecessary polling / 不必要的轮询
   - Inefficient algorithms in ISRs / ISR 中的低效算法
   - Excessive context switching / 过度的上下文切换

### Severity Levels / 严重程度级别

- **🔴 Critical/严重**: Issues that may cause system crashes or data corruption / 可能导致系统崩溃或数据损坏的问题
- **🟠 Major/主要**: Significant bugs or performance issues / 重大错误或性能问题
- **🟡 Minor/次要**: Code style or minor optimization opportunities / 代码风格或次要优化机会
- **🟢 Suggestion/建议**: Best practices or enhancement ideas / 最佳实践或增强建议

## RT-Thread Specific Guidelines / RT-Thread 特定指南

### Kernel Components / 内核组件

When reviewing kernel-related code:
审查内核相关代码时：

- Verify rt_thread structure usage / 验证 rt_thread 结构使用

### Device Drivers / 设备驱动

For device driver reviews:
对于设备驱动审查：

- Ensure proper device registration / 确保正确的设备注册
- Verify I/O operation handling / 验证 I/O 操作处理

### Network Stack / 网络协议栈

When reviewing network code:
审查网络代码时：

- Validate SAL (Socket Abstraction Layer) usage / 验证 SAL（套接字抽象层）使用
- Check protocol implementations / 检查协议实现
- Ensure proper buffer management / 确保正确的缓冲区管理

## Coding Standards / 编码标准

### Object-Oriented Design in C / C语言面向对象设计

1. **Inheritance Pattern / 继承模式**
   - First member should be base struct / 第一个成员希望是基类结构体
   - Use pointer casting for type conversion / 通过指针强制转换实现类型转换

2. **Polymorphism via ops / 通过ops实现多态**
   - Define ops struct with function pointers / 定义包含函数指针的ops结构体
   - Share single ops table across instances / 多个实例共享同一ops表

### Naming Conventions / 命名规范

- **Structures / 结构体**: `rt_[name]`
- **Public Functions / 公开函数**: `rt_[class]_[action]`
- **Static Functions / 静态函数**: `_[class]_[action]`
- **Hardware Functions / 硬件函数**: `rt_hw_`
- **Macros / 宏定义**: UPPERCASE (except for local function/variable macros)
- **Error Codes / 错误码**: `RT_` + POSIX error code, `RT_EOK` for success

### Object Lifecycle / 对象生命周期

- Provide dual APIs / 提供双模式API:
  - `init/detach` for static objects / 用于静态对象
  - `create/delete` for dynamic objects / 用于动态对象
- Use reference counting / 使用引用计数
- Return unified error codes / 返回统一错误码

### Code Format / 代码格式

- 4 spaces indentation, no tabs / 4空格缩进，不使用tab
- Braces on separate lines / 大括号独占一行
- Align parameters on line breaks / 参数换行时对齐

## Documentation Standards / 文档标准

### Language and Format / 语言和格式

- Use English for code comments / 所有代码注释使用英文
- Markdown format for documentation / 文档使用Markdown格式
- Prefer Mermaid for diagrams, or PlantUML (hide footbox in sequence diagrams) / 优先使用Mermaid绘图，或PlantUML（时序图隐藏footbox）

### Document Structure / 文档结构

1. **Main Level / 主干层**: Overall overview / 整体概述
2. **Branch Level / 分支层**: Module introduction / 子模块介绍
3. **Node Level / 节点层**: Detailed knowledge points / 知识点详解

### Documentation Principles / 文档原则

- Keep structure flat / 保持扁平结构
- Modular organization / 模块化组织
- Clear and concise content / 内容简洁直接
- Complete executable examples / 完整可执行示例

## Best Practices / 最佳实践

1. **Always consider embedded constraints** / **始终考虑嵌入式约束**
   - Limited RAM and ROM / 有限的 RAM 和 ROM
   - Power consumption / 功耗
   - Real-time requirements / 实时要求
   - Prefer static allocation / 优先静态分配
   - Use memory alignment / 使用内存对齐

2. **Verify on real hardware or at least QEMU** / **尽可能在真实硬件上验证，或至少在QEMU上验证**
   - Test on actual hardware when available / 有条件时在真实硬件上测试
   - Use QEMU simulation as minimum verification / 至少使用QEMU仿真进行验证
   - Consider various BSP configurations / 考虑各种 BSP 配置

3. **Document hardware dependencies** / **记录硬件依赖**
   - Specify required peripherals / 指定所需外设
   - Note timing constraints / 注意时序约束
   - List supported MCU/MPU families / 列出支持的 MCU/MPU 系列

4. **Code Optimization / 代码优化**
   - Use `rt_inline` for simple functions / 简单函数使用rt_inline
   - Avoid deep nesting / 避免深层嵌套
   - Single responsibility per function / 函数功能单一
   - Minimize code duplication / 减少代码重复

## Contributing / 贡献

When suggesting improvements:
提出改进建议时：

1. Provide clear, actionable feedback / 提供清晰、可操作的反馈
2. Include code examples when possible / 尽可能包含代码示例
3. Reference RT-Thread documentation / 引用 RT-Thread 文档
4. Consider backward compatibility / 考虑向后兼容性

## References / 参考资料

- [RT-Thread Documentation](https://www.rt-thread.io/document/site/)
- [RT-Thread Coding Style Guide](https://github.com/RT-Thread/rt-thread/blob/master/documentation/coding_style_en.md)
- [RT-Thread 文档中心](https://www.rt-thread.org/document/site/)
- [RT-Thread 编码规范](https://github.com/RT-Thread/rt-thread/blob/master/documentation/coding_style_cn.md)
```

---
> Source: [RT-Thread/rt-thread](https://github.com/RT-Thread/rt-thread) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
