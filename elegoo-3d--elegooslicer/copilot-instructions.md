## dev-log

> 当用户说"记录"时创建/更新开发日志（仅在用户主动触发时记录）


# 开发日志规则

## 核心原则
- 简洁：几句话说清楚，不要写成技术文档
- 要点化：用列表，不要长篇大论
- 被动触发：仅在用户说"记录"时才记录，不要自动记录

## 目录和命名
- 目录：`dev_log/`
- 命名规则：
  - **较大功能/新功能/重构**：创建 `module-name.md`
  - **日常修正/优化/小改动**：追加到 `YYYY-MM-DD.md`
- 禁止一个功能创建多个文件（如 test-checklist、final-status 等）

## 记录内容
必须：日期、需求（1-2句话）、实现方案（3-5要点）、新增/修改文件、状态
可选：修复问题、待测试事项、注意事项、已完成TODO、待完成TODO（较大功能）

## 禁止
- 详细代码示例、完整架构图、详细测试步骤
- 没有用户触发就自动记录

---
> Source: [ELEGOO-3D/ElegooSlicer](https://github.com/ELEGOO-3D/ElegooSlicer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
