## hermes-book

> 本书基于 Hermes Agent 源码，以"self-improving personal agent 的工程实现需要解决哪些问题"为叙事主线。使用 mdbook 组织和渲染。

# Hermes Agent 源码与设计 — 写作指南

## 项目概述

本书基于 Hermes Agent 源码，以"self-improving personal agent 的工程实现需要解决哪些问题"为叙事主线。使用 mdbook 组织和渲染。

- **大纲**: [outline-v2.md](./outline-v2.md) — 全书结构、章节划分、写作顺序的权威参考
- **章节 specs**: `specs/` 目录下的 `.spec.md` 文件 — 每章的验收标准
- **源码目录**: 上级目录 `../` 即 hermes-agent 项目根目录

## 写作原则

1. **理念驱动，不是文件驱动** — 每章以"为什么需要这个机制"开头，不以"这个模块有哪些文件"开头
2. **代码走读服务于设计理解** — 引用源码是为了说明设计选择，不追求行覆盖率
3. **中文写作** — 正文使用中文，代码和专有名词保留英文原文
4. **Mermaid 流程图** — 流程图使用 Mermaid 格式，mdbook 可直接渲染
5. **源码引用格式** — 使用 `文件名:行号` 格式引用源码，如 `run_agent.py:416`

## 章节结构模板

每章应包含以下结构：

```markdown
# 章节标题

> **本章核心源码**: `file1.py`, `file2.py`, `file3.py`

## 为什么需要这个机制

（从产品需求或工程问题出发，解释这个子系统存在的理由）

## 设计选择

（Hermes 做了什么选择，为什么这么选，有什么 trade-off）

## 源码走读

（关键代码的逐步解读，用代码块 + 解说交替）

## 设计启示

（这个子系统的设计对构建类似系统有什么可借鉴的经验）
```

## 四个设计赌注（全书分析框架）

所有章节都应该能回扣到 Hermes 的四个核心设计赌注：

1. **Learning Loop** — agent 从经验中创建技能、自我改进、主动持久化知识
2. **CLI-First** — 终端是第一等公民，不是 Web UI 的附庸
3. **Personal Long-Term** — 跨会话记忆、用户建模、越用越懂你
4. **Run Anywhere** — 从 $5 VPS 到 GPU 集群，从本地到 Telegram

## 技术约定

- mdbook 配置在 `book.toml`
- 章节文件在 `src/` 下按 part 分组
- 目录结构在 `src/SUMMARY.md`
- 构建命令: `mdbook build`（输出到 `book/`）
- 预览命令: `mdbook serve`（本地 HTTP 预览）

## 写作顺序

参见 outline-v2.md "建议写作顺序" 部分。先写主线（ch02-04），再写能力层，再写状态层，最后写平台层和工程基础。导论（ch01）最后写。

## 稳定性专题

稳定性内容分布在多个章节，形成"四层防御"叙事：
- **第 1 层 · 进程生命周期**（Ch20）：信号处理、优雅关停、Process Registry、清理守护线程
- **第 2 层 · 运行时防御**（Ch21）：SafeWriter、重试策略、Graceful degradation、断线重连
- **第 3 层 · 数据安全**（Ch10）：SQLite WAL、jitter retry、事务管理
- **第 4 层 · 架构防御**（Ch19）：async→sync 桥接、线程隔离、IterationBudget

## Spec 工作流

每章在开始写作前，检查对应的 `specs/ch-XX-*.spec.md` 验收标准。写完后用 spec 自查：
- 是否覆盖了所有核心源码文件
- 是否回扣了至少一个设计赌注
- 是否包含了流程图或架构图
- 异常路径和设计 trade-off 是否被讨论

---
> Source: [ZhangHanDong/hermes-book](https://github.com/ZhangHanDong/hermes-book) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
