## tiandaowenhua-xcx

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

**天道文化小程序** - 基于 UniApp + 腾讯云开发的跨平台应用，用于孙膑道天道文化课程的线上销售、学习管理和推广。主要目标平台为微信小程序。

仓库包含三个部分：
- `universal-cloudbase-uniapp-template/` - 前端主项目（UniApp + 云开发）
- `prototype-tdesign/` - 原型设计（静态 HTML/CSS，TDesign 规范）
- `zuowei/` - 独立项目（座位管理系统），与本仓库无架构关联

**必读文档**：`docs/database/数据库详细信息.md` — 数据库字段名/类型的唯一真实来源，每次新任务前必须读取。

## 常用命令

```bash
cd universal-cloudbase-uniapp-template

# 安装依赖
npm install

# H5 开发
npm run dev:h5

# 微信小程序开发
npm run dev:mp-weixin

# 构建
npm run build:h5        # H5 生产版本
npm run build:mp-weixin # 微信小程序

# 类型检查
npm run type-check
```

原型设计：`cd prototype-tdesign && python -m http.server 8000`

## 技术栈

- **框架**: UniApp (Vue 3 Composition API) + TypeScript + Vite
- **后端**: 腾讯云开发 CloudBase — 云函数(Node.js) + MySQL 关系型数据库
- **双 SDK**: `wx-server-sdk`（微信支付/订阅消息/小程序码）+ `@cloudbase/node-sdk`（Supabase 风格 Query Builder）
- **管理后台**: 纯静态 HTML + Vue 3 CDN + TDesign，部署在 CloudBase 静态托管

## 项目架构

### 页面路由（主包 + 5 个分包）

**TabBar（4 页）**: 首页 `/pages/index`、商城 `/pages/mall`、商学院 `/pages/academy`、我的 `/pages/mine`

**分包**:
- `pages/auth` — 登录、完善资料（独立分包，控制主包体积）
- `pages/course` — 课程详情、我的课程、排课、预约确认、签到
- `pages/order` — 下单、支付、待付款/已取消/已退款、退款申请/状态
- `pages/ambassador` — 等级、申请、团队、二维码、功德分/现金积分、提现、合约签署
- `pages/common` — 公告、用户协议、隐私政策

### 云函数（6 个）

| 函数 | 职责 | 关键能力 |
|------|------|----------|
| `user` | 用户管理 | 登录、资料、推荐人、积分、提现 |
| `order` | 订单支付 | 微信支付、退款、商城兑换 |
| `course` | 课程排课 | 课程 CRUD、预约、QR 签到、商学院 |
| `ambassador` | 大使系统 | 等级、合约、二维码、活动记录 |
| `system` | 系统管理 | Banner、公告、配置、管理员认证 |
| `callbacks` | 支付回调 | 微信支付回调处理 |

**路由模式**: `index.js` 按 `action` 字段路由到 `handlers/{public|client|admin}/` 下对应模块。

**三层认证**:
- **Public**: 无需认证（课程列表、支付回调）
- **Client**: `checkClientAuth(OPENID)` 验证用户存在于 `users` 表
- **Admin**: JWT token（Web 后台）或 OPENID 查 `admin_users` 表

**共享层**: 每个云函数都有独立的 `common/` 和 `business-logic/` 副本。修改共享代码后**必须重新部署所有云函数**。

### 数据库概览（MySQL，`tiandao_culture`，35 张表）

核心表：`users`（用户）、`courses`/`user_courses`（课程）、`orders`（订单）、`appointments`/`class_records`（排课签到）、`ambassadors`/`ambassador_levels`/`ambassador_contracts`（大使体系）、`merit_points_records`/`cash_points_records`（双积分系统）、`academy_*`（商学院）

**重要**: 任何 INSERT/UPDATE 前必须先 `SHOW COLUMNS` 确认字段名。

## 关键模式

### API 调用链
```
前端 callCloudFunction({name, action, data})
  → 微信: wx.cloud.callFunction() / H5: app.callFunction()
  → 云函数 index.js 按 action 路由
  → handler 认证 + 业务逻辑 + Query Builder
  → response.success()/error() 标准化返回
  → 前端 responseInterceptor 全局处理 401/403/404/422
```

### 字段命名约定
- **前端 API 参数**: camelCase（`realName`, `courseId`）
- **数据库字段**: snake_case（`real_name`, `course_id`）
- **云函数返回**: snake_case（与数据库一致）

### 认证状态管理
- 无 Vuex/Pinia，使用 `uni.getStorageSync('userInfo')` + 每页 reactive state
- **渐进式访问**: 浏览 → 登录 → 完善资料 → 全功能（满足微信审核要求，不强制登录）
- `preview-guard.ts` 路由守卫控制需要登录的页面
- `EventBus` + `uni.$emit('tabBarSync')` 用于跨组件通信

### 云存储模式
- 数据库存 `cloud://` fileID
- **禁止使用 `getTempFileURL`**（鉴权失败），使用 `cloudFileIDToURL()` 直接构造 CDN URL
- `cloud://{env}.{bucket}/{path}` → `https://{bucket}.tcb.qcloud.la/{path}`

### 日期时间格式
- MySQL 日期字段必须使用 `formatDateTime(new Date())` 返回 `YYYY-MM-DD HH:MM:SS`
- **禁止** `toISOString()`（T 和 Z 字符导致 MySQL Error 1292）、`toLocaleString()`、原始 `Date` 对象

### 订单状态机
- 类型: COURSE(1), RETRAIN(2), UPGRADE(4)
- 状态流: PENDING(0) → PAID(1)，或 → CANCELLED(2)/CLOSED(3)/REFUNDED(4)
- 混合支付: 功德分优先 → 现金积分补充（`mixed-payment-calculator.ts`）

### 大使等级
0=普通用户, 1=实习, 2=青鸾, 3=鸿鹄

## 开发规则

### 云函数开发
- **部署**: 使用 MCP `updateCloudFunction`，禁止使用 tcb CLI
- **调试**: 使用 MCP `getFunctionLogs`
- **定时触发器**: `cloudfunction.json` 中定义但不会随代码同步，需单独用 MCP `manageFunctionTriggers` 部署。`create` 操作会覆盖所有触发器，必须一次性传入全部触发器
- **callbacks 函数的数据库导入**: 必须从 `../common/db` 导入，禁止从其他函数目录导入（避免 `getDb is not a function`）

### 前端开发
- 使用 `src/styles/components/` 下的 `.t-*` 类名和 `$td-xxx` SCSS 变量，禁止裸写 hex 颜色
- 管理后台 favicon 防止、侧边栏导航注册、AdminAPI 注册、CDN 缓存刷新（`?v=YYYYMMDD`）

### 文档同步
- 功能开发或 bug 修复完成后，必须更新 `需求文档-V2.md` 和 `后端API接口文档.md` 才算"完成"
- 集成测试文档 `admin/INTEGRATION-TEST-README.md` 也需同步更新
- 数据库 schema 变更后必须更新 `数据库详细信息.md`

### 代码注释
**新增代码必须添加注释**：文件头注释、JSDoc、业务逻辑注释、Vue 组件标注。

### 文档创建
创建任何独立文档（.md/.txt/README 等）前须向用户确认。例外：用户明确要求、项目配置文件、代码注释/JSDoc。

## 设计规范

```css
--td-brand-color: #0052D9;          /* 主品牌色 */
--td-brand-color-light: #266FE8;    /* 品牌色-浅 */
--td-bg-color-page: #F5F5F5;        /* 页面背景 */
--td-bg-color-container: #FFFFFF;   /* 容器背景 */
--td-warning-color: #D4AF37;        /* 金色(价格/徽章) */
```

间距：页面边距 16px，卡片间距 12px，区块间距 24px

---
> Source: [richenHe/tiandaowenhua_xcx](https://github.com/richenHe/tiandaowenhua_xcx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
