## diypc

> - **项目名称：** DIY PC Builder（台式主机DIY助手）

# CLAUDE.md

## 项目信息

- **项目名称：** DIY PC Builder（台式主机DIY助手）
- **类型：** Android App
- **框架：** Flutter (Dart)
- **目标用户：** 电脑DIY爱好者

## 规范文档索引

项目相关的标准规范文档，在制定方案和写代码时应参考：

| 文档 | 路径 | 用途 |
|------|------|------|
| 需求规格 | [docs/requirements.md](docs/requirements.md) | 功能和用户需求 |
| 技术规范 | [docs/tech-spec.md](docs/tech-spec.md) | 技术栈、架构、数据模型 |
| 设计规范 | [docs/design-spec.md](docs/design-spec.md) | 色彩、排版、间距、组件 |
| 执行指南 | [docs/implementation-guide.md](docs/implementation-guide.md) | 分阶段执行步骤 |

## 开发日志

每日开发日志记录于 `devlog/YYYY-MM-DD.md`。每次工作结束时：
- 更新当日日志：已完成事项、待办事项、遇到的问题
- 如果日期变更，创建新的日志文件

## 工作原则

### 小步推进
- 严格按照 `implementation-guide.md` 中的阶段顺序执行
- 每个阶段内再拆分为更小的步骤
- 一个步骤完成后才进入下一个

### 每步验证
- 每个阶段有明确的验证标准
- 达标后才进入下一阶段
- 不积压未验证的代码

### 不贪多
- 只做计划内的事情，不额外添加功能
- 用户提出新需求时，先讨论是否纳入当前阶段

### 代码风格
- 简单直接，不过度抽象
- 不做未来才需要的功能
- 只为"为什么这么写"加注释，不为"写了什么"加注释

### 数据层优先
- 先建数据模型和数据库，再写状态管理，最后写UI
- UI组件只负责展示，业务逻辑在 Provider 中

### 安全为本
- 所有文件路径使用 path 库拼接，不拼接字符串
- 数据库操作使用参数化查询
- 用户输入做校验（TXT解析）

---
> Source: [RouJinwin/DiyPC](https://github.com/RouJinwin/DiyPC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
