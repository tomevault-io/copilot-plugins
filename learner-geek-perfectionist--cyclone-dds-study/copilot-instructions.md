## cyclone-dds-study

> 回答关于 Cyclone DDS 源码机制的问题时，必须遵循以下分场景规则。

# Cyclone DDS 学习项目规则

## 问答规则：源码事实优先

回答关于 Cyclone DDS 源码机制的问题时，必须遵循以下分场景规则。

### 必须先查代码再回答

以下场景禁止凭推测回答，必须先用 Read/Grep/Agent 查看源码，引用具体文件和行号后再给出结论：

- 解释某个机制"为什么这样设计"（如引用计数为什么用偏置）
- 解释某个变量、参数、常量的作用（如 RDATA_BIAS 的用途）
- 描述函数的调用流程或数据流向（如 rdata 经过哪些 reorder）
- 涉及并发安全、生命周期、引用计数的结论
- 描述多个模块之间的交互关系

违反时的正确做法：说"我需要查代码确认"，然后去查，而不是给出推测性结论。

### 可以直接回答（无需查代码）

- 解释当前上下文中已经读过的代码
- 解释 UML 图、Mermaid 图、时序图的阅读方法
- 通用计算机科学概念（如 bump allocator、引用计数、原子操作）
- 解释 DDS/RTPS 协议的通用概念（如 Writer、Reader、Topic）
- 对已查证过的代码做进一步的类比或简化解释

### 回答质量要求

- 如果对某个结论不确定，必须明确标注"这是我的推测，需要查代码验证"
- 不得在同一个问题上给出前后矛盾的结论。如果需要修正之前的回答，必须明确说明"之前的回答有误"并给出基于代码的修正
- 涉及源码的结论应尽量附带文件名和行号（如 ddsi_radmin.c:539）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Learner-Geek-Perfectionist)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Learner-Geek-Perfectionist)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
