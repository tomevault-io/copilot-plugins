## annote

> 岁记（Annote）是一个个人财务与日程管理工具，统一管理存款余额、预算、记账流水与日程安排，并支持与伴侣协作共享计划书。

# AGENTS.md

## 项目概述

岁记（Annote）是一个个人财务与日程管理工具，统一管理存款余额、预算、记账流水与日程安排，并支持与伴侣协作共享计划书。

## 技术栈

| 层级 | 技术 |
|------|------|
| 前端 | UmiJS 4 + React 18 + Ant Design 5 + Motion (动画) |
| 后端 | uniCloud 阿里云（支付宝云服务空间） |
| 认证 | uniID 账号密码登录 |
| 数据库 | uniCloud 云数据库（MongoDB 兼容） |
| 测试 | 前端 Vitest + Testing Library，后端 Jest |

## 项目结构

```
Annote/
├── frontend/                  # UmiJS 前端应用
│   ├── src/
│   │   ├── .umi/              # Umi 自动生成（不提交）
│   │   ├── .umi-production/   # 生产构建产物（不提交）
│   │   ├── components/        # 通用组件（MotionComponents 等）
│   │   ├── layouts/           # 布局组件（BasicLayout）
│   │   ├── models/            # 数据模型与 API 封装
│   │   ├── pages/             # 页面组件
│   │   │   ├── login/         # 登录/注册
│   │   │   ├── dashboard/     # 仪表盘首页
│   │   │   ├── accounts/      # 存款账户管理
│   │   │   ├── transactions/  # 记账流水
│   │   │   ├── budget/        # 预算管理
│   │   │   ├── schedule/      # 日程与计划书
│   │   │   └── share/         # 分享预览
│   │   ├── styles/            # 全局样式
│   │   └── test/              # 前端测试
│   ├── .umirc.ts              # Umi 配置
│   ├── vitest.config.ts       # Vitest 配置
│   └── package.json
├── backend/                   # uniCloud 后端
│   ├── uniCloud-alipay/
│   │   ├── cloudfunctions/    # 云对象（auth/account/transaction/budget/schedule/share/dashboard）
│   │   └── database/          # 数据库集合 schema
│   └── test/                  # 后端测试
└── .trae/specs/               # 项目 Spec 文档
```

## 前置条件

- Node.js 18+
- HBuilderX（用于上传云函数/云对象到 uniCloud 阿里云）
- uniCloud 阿里云服务空间已绑定

## 本地开发

```bash
# 前端
cd frontend
npm install
npm run dev          # 启动开发服务器，默认 http://localhost:8000

# 前端测试
npm test             # 运行一次测试
npm run test:watch   # 监听模式
npm run test:coverage # 覆盖率报告

# 后端测试
cd backend
npm install
npm test             # 运行一次测试
npm run test:watch   # 监听模式
npm run test:coverage # 覆盖率报告
```

## 部署

### 前端

```bash
cd frontend
npm run build        # 输出到 frontend/dist/
```

### 后端（云函数/云对象）

在 HBuilderX 中右键 `backend/uniCloud-alipay/cloudfunctions/` 目录，选择「上传所有云函数/云对象」，然后在 uniCloud 控制台为每个云对象启用 URL 化（路径名与云对象同名）。

## 架构设计

### 数据流

```
用户 → UmiJS 前端 → uniCloud SDK → 云对象（URL化） → 云数据库
                                        ↓
                                   uniID 认证（token 校验）
```

### 认证

- 所有业务接口通过 uniID token 校验身份
- 数据按 `user_id` 严格隔离
- 共享数据通过协作者列表或分享 token 显式授权

### 核心业务联动

- **记账 ↔ 账户**：支出/收入自动更新账户余额；删除流水回滚余额
- **记账 ↔ 预算**：支出自动累计当月类别预算已用；删除流水回滚
- **记账 ↔ 日程**：支出可关联日程；日程下录入实际支出自动生成流水
- **日程 → 预算**：计划书计划预算按日期月份自动汇入对应类别预算
- **分享**：双模式 — 实时协作（邀请伴侣）和链接分享（token 生成）

## 设计规范

遵循 Apple 设计原则：
- 交互：pointer-down 即时反馈，弹簧动画，可中断手势
- 视觉：毛玻璃半透明材质（backdrop-filter），层次阴影
- 排版：光学字号字体，合理字重/字距/行高
- 无障碍：`prefers-reduced-motion` 时降级动画

## 编码约定

- 路由路径使用 kebab-case（如 `/accounts`、`/transactions`）
- 页面组件使用 PascalCase 目录名，入口文件为 `index.tsx`
- API 调用统一封装在 `src/models/` 中
- 云对象使用 `index.obj.js` 命名，通过 URL 化暴露 HTTP 接口
- 中文注释与 UI 文案
- 提交前确保 `npm test` 通过（前后端）

---
> Source: [ZMK2017/annote](https://github.com/ZMK2017/annote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
