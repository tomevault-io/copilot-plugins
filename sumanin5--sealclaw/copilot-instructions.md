## sealclaw

> 本项目 (`SealClaw`) 的核心目的是：**通过自主重构的方式，深度学习和吸收优秀 Agent 框架和工具箱的核心原理与架构设计**。我们将借助 Rust 语言将其核心逻辑重新实现，以此达到透彻理解其底层原理的目标。

# SealClaw 项目开发准则与重构目标

## 1. 项目大局观与重构学习目标

本项目 (`SealClaw`) 的核心目的是：**通过自主重构的方式，深度学习和吸收优秀 Agent 框架和工具箱的核心原理与架构设计**。我们将借助 Rust 语言将其核心逻辑重新实现，以此达到透彻理解其底层原理的目标。

> **📚 重要参考资料：`REFERENCES.md`**
> 本项目根目录下存在一个 `REFERENCES.md` 文件。在深挖下面两个参考项目的源码、理解其架构上下文和设计理念时，**必须优先阅读和参考该文件**，它提供了对这两个项目非常有价值的背景补充建议和架构摘要。

我们需要解构并重写以下两个参考项目：

- 🌟 **主要参考项目 (Python，重点剖析对象)**:
  `Path: /home/tomy/projects/python/aiProjects/nanobot`
  由于 Python 版本逻辑相对清晰易懂，我们将重点对这个项目的源码进行剖析。将其核心逻辑用地道的 Rust 重新实现，通过代码的“翻译”和“重构”来真正掌握 Agent 的运行机制。

- 🧱 **进阶参考项目 (Rust，难度较大)**:
  `Path: /home/tomy/projects/rust-project/ironclaw`
  作为一个更高阶、难度更大的参考，我们将用它来对标和学习 Rust 在复杂系统、高并发或者更底层模块设计上的优秀实践。

## 2. 协作重构工作流 (剖析 -> 设计 -> 实现)

在为 `SealClaw` 编写或者迁移任何代码前，必须严格遵守以下工作流：

1. **源码潜水 (深度剖析)**：
   必须先使用工具（如 `view_file`, `grep_search` 等）深入阅读 `nanobot` 中相关核心模块的源码。梳理出核心的类继承、函数调用链路和数据流转状态。
2. **逻辑重塑与方案设计 (Idiomatic Rust)**：
   严禁生搬硬套旧代码或直接一对一翻译语法。必须先向 USER 彻底解释旧代码的核心机制（例如：它的核心状态流转是怎么做的？工具分发是怎么路由的？）。随后，提出在 `SealClaw` 中如何利用 Rust 的优势（如 `Trait`, 模式匹配, 所有权模型）进行更加优雅的设计方案。
3. **结对编程，渐进式重写**：
   采用“看一段 -> 聊一段 -> 写一段”的结对编程模式。确保 USER 真正理解并掌握了每一部分的设计意图。在编写关键逻辑时，多用注释详细说明设计思路。

## 3. 严格遵守全局准则

请始终牢记并严格遵循全局配置的 《Antigravity 开发协作准则》：

- 贯彻**“先行后做”原则 (Analyze Before Action)**。
- 所有的沟通、架构分析、任务规划、代码注释和 Git 提交记录，必须使用**中文**。
- 在调整代码或解决 BUG 时，先追溯根本原因，再给出包含具体文件和思路的详尽修改方案。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sumanin5)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sumanin5)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
