## generic-dev-sop

> 把项目从零到交付作为可追踪闭环：搭建底座 → 规划任务 → 单功能闭环 × N → 完整交付。每项功能把「计划、契约、UI、实现、测试、证据」同步交付，禁止只做底层逻辑、接口或 UI Mock 后宣称完成。纯文案/无行为变化的单行修改可跳过步骤；无 UI 的后端功能可跳过设计稿，但必须说明原因。


# 通用项目开发 SOP（generic-dev-sop）

把项目从零到交付作为可追踪闭环：搭建底座 → 规划任务 → 单功能闭环 × N → 完整交付。每项功能把「计划、契约、UI、实现、测试、证据」同步交付，禁止只做底层逻辑、接口或 UI Mock 后宣称完成。纯文案/无行为变化的单行修改可跳过步骤；无 UI 的后端功能可跳过设计稿，但必须说明原因。

状态仅使用：`未开始`、`分析中`、`设计确认`、`开发中`、`审查测试中`、`已完成`、`阻塞`。

## 生命周期

1. 项目搭建：确认工程边界 → 确认技术栈 → 输出架构决策 → 输出任务计划 → 搭质量底座 → 建开发记录机制。
2. 详细任务计划：执行原则、交付定义（真实闭环，不是「页面都能点」）、范围/估算/关键路径、阶段总览、详细任务（每条含验收条件）、功能矩阵（P0/P1/P2）、质量门禁、团队边界、第一批执行顺序。
3. 单功能开发闭环（每个功能循环）：确认计划进度 → 需求分析 → 最小方案 → 写开发记录 → 设计稿（有 UI 先出默认/加载/空/错误/禁用/无权限/离线状态）→ 数据结构（先定模型/校验/迁移）→ 开发（复用现有模式）→ 审查测试（按审查清单 + 留最小可运行检查）→ 回写进度（证据一致才标记完成）。
4. 完整交付：发布候选门禁 → 灰度与回滚 → 文档同步 → 项目交付检查（P0 功能矩阵全绿，无「页面有数据假」「接口有客户端不可达」）。

## 技术栈确认

启动前先与用户确认技术栈：用户指定则以用户为准；未指定或直接回车 = 同意使用默认。默认组合（已验证）：Web/桌面 = Electron + React + Vite + MobX + Ant Design + i18next（单构建同时出 Web 与桌面）；跨端移动 = Flutter + Riverpod + GoRouter + Dio + just_audio；服务端 = NestJS + PostgreSQL/PostGIS + TypeORM + Redis + Swagger + JWT/RBAC + Winston。

## 核心规则

- 单一业务真相：主库是唯一事实来源，缓存/实时事件可丢失且可恢复。
- 有 UI 先出覆盖各状态的静态稿再写代码；无 UI 要说明依据。
- 非平凡逻辑留一个最小可运行检查；日常跑变更测试 + 直接调用链 + typecheck + 受影响 lint，全量门禁在阶段出口、重构、发布前执行。
- 跨模块用路径别名，禁止 `../../../`；所有声明模块正上方写多行中文注释。
- 密钥只存环境变量/安全存储，日志脱敏；写接口做归属校验；外部输入先校验再进业务代码。
- 平台差异用运行时适配器判断，禁止构建分叉。
- 交付 = 计划、契约、设计稿、实现、测试、文档一致。

## 模板与清单

完整模板在本仓库 `skills/generic-dev-sop/references/`：architecture-template.md、task-plan-template.md、development-record-template.md、coding-standards.md、review-checklist.md、default-stack.md。单功能记录复制 development-record-template.md 到 `references/features/<功能短名>.md` 并全程回写。

---
> Source: [vien-meng/vien-skills](https://github.com/vien-meng/vien-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
