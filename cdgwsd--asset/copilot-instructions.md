## asset

> 这是一个自用的记账/资产记录 PWA 应用，运行在 iPhone Safari（添加到主屏幕）。Vue 3 + Pinia + Dexie (IndexedDB) + ECharts，无后端、无路由。

# CLAUDE.md — AI 协作规则

## 项目概述

这是一个自用的记账/资产记录 PWA 应用，运行在 iPhone Safari（添加到主屏幕）。Vue 3 + Pinia + Dexie (IndexedDB) + ECharts，无后端、无路由。

## 常用命令

```bash
npm run dev      # 启动开发服务器（局域网可访问）
npm run build    # 类型检查 + 生产构建
npm run preview  # 预览生产构建
```

## 开发规范

### 代码组织
- 类型定义放 `src/types/`
- 数据库操作放 `src/repositories/`（纯 CRUD）
- 业务逻辑放 `src/services/`（含快照触发、数据校验等）
- UI 状态放 `src/stores/`
- 工具函数放 `src/utils/`
- 常量定义放 `src/constants/`
- 组件样式优先使用 scoped style，逐步从 global.css 迁移

### 关键架构约束
- **没有 Vue Router**，所有导航通过 BottomSheet 弹窗叠加
- 弹窗状态由 `uiStore.activeSheet` 统一管理
- 所有弹窗必须使用 BottomSheet 组件作为容器
- 危险操作（删除、清空、导入覆盖）必须使用 ConfirmDialog 二次确认

### 数据结构约束
- 金额统一用 `number` 存储，通过 `normalizeMoney()` 保证 2 位小数精度
- 时间戳统一用 ISO 8601 字符串（`new Date().toISOString()`）
- 日期统一用 `YYYY-MM-DD` 格式
- 账户删除是软删除（`isDeleted = true, isActive = false`）
- 资产快照以日期为唯一键（`asset_snapshot_{date}`）

### 移动端约束
- 所有输入框 `font-size` 不小于 16px（防止 iOS 自动缩放）
- 底部元素必须包含 `var(--safe-bottom)` 安全区域间距
- 弹窗高度固定为 80vh（通过 BottomSheet 组件保证）
- 弹窗内容在弹窗内部滚动，不能导致背景页面滚动
- 禁止使用 `overflow-x: auto` 在 body 级别，避免横向滚动

## 禁止事项

- **禁止**引入 Vue Router 或其他路由库
- **禁止**修改已有数据表结构而不提供迁移方案
- **禁止**直接删除账户记录（必须软删除）
- **禁止**在组件中直接操作 `db`（必须通过 repository 或 service 层）
- **禁止**引入后端框架、数据库、或任何服务端依赖
- **禁止**一次性大范围重构，每次只改一个明确功能
- **禁止**修改 `src/db/defaults.ts` 中已有系统账户类型的 `id` 字段
- **禁止**在未确认的情况下修改业务代码、样式代码、配置文件
- **禁止**仅考虑桌面浏览器效果，必须以 iPhone Safari PWA 场景为优先
- **禁止**引入超过 50KB（gzip）的新依赖，除非明确说明原因并得到确认

## 修改前必须确认

在修改任何代码前，先输出：
1. 要修改的文件
2. 修改原因
3. 修改方案
4. 可能影响的功能
5. 风险点
6. 验证步骤

---
> Source: [cdgwsd/asset](https://github.com/cdgwsd/asset) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
