## zr-admin-net

> > 本文件是 ZR.Admin.NET 项目的 AI Coding Agent 开发规范。

# AGENTS.md

# ZR.Admin 项目 AI 开发规范

> 本文件是 ZR.Admin.NET 项目的 AI Coding Agent 开发规范。
> 所有 AI Agent 在分析、修改、创建或删除代码之前，都必须优先阅读并遵守本文件。
>
> 适用范围：
> - ZR.Admin.NET
> - ZR.Admin.Vue3
> - Workflow 工作流引擎
> - LogicFlow 流程设计器
> - 管理后台
> - API
> - 数据库
> - 多租户
> - 定时任务Quartz.net
> - 第三方服务集成

---


# 后端编码强制规范（C# / ZrAdminNet）

- **Service 实现类新增公共方法时，必须同步在对应接口声明。**
  - ZRAdmin 的 Service 通过接口注入（`Service/IService/IxxxService.cs`），Controller 只依赖接口。
  - 仅改实现类（`Service/XxxService.cs`）而不在 `IService/IXxxService.cs` 补同签名声明，会编译失败（`CS1061`），且因 DLL 未生成，运行时仍跑旧逻辑，表现为“改了代码却没生效”。
  - **自查清单**：每给 `*Service.cs` 加一个 `public` 方法，先到 `IService/I*Service.cs` 补一行同签名声明。
- 改完后端务必 `dotnet build`（建议直接构建 `ZR.Admin.WebApi/ZR.Admin.WebApi.csproj`）确认 0 错误再交付。
- 排障原则：“现象不变”优先怀疑后端未重新编译，而非前端逻辑问题。

# 一、项目基本原则

## 1.1 第一原则：优先复用现有代码

修改项目时必须优先寻找已有实现。

禁止：

- 已经存在相同功能却重新实现
- 已经存在公共方法却重新写一份
- 已经存在组件却重新创建组件
- 已经存在 Service / Repository 模式却引入新的架构
- 为了一个小功能引入新的框架
- 为了一个简单问题增加复杂设计

优先：

1. 查找现有代码
2. 找到最接近的实现
3. 复用现有模式
4. 在现有模式上扩展
5. 只有现有设计无法满足需求时才新增结构

---

# 二、项目技术栈

## 2.1 Backend

主要技术：

- .NET
- ASP.NET Core
- SqlSugar
- SQL Server
- FluentValidation
- Mapster
- NLog
- Swagger / OpenAPI
- MiniExcel

除非明确要求，否则不要随意引入新的 ORM、Web Framework 或基础设施。

---

## 2.2 Frontend

主要技术：

- Vue 3
- Vite
- JavaScript
- Element Plus
- Pinia
- Vue Router
- Axios / 项目现有 request 封装

---

## 2.3 Workflow

工作流相关：

- LogicFlow
- 自定义 Workflow Engine
- FlowJSON
- SQL Server
- SqlSugar

LogicFlow 主要负责：

- 流程设计
- 流程可视化
- 流程预览

Workflow Engine 负责：

- 流程实例
- 节点执行
- 审批任务
- 条件判断
- 抄送
- 并发
- 任务抢占
- 状态流转

不要把 LogicFlow 当成工作流引擎。

---

# 三、AI Agent 工作方式

## 3.1 修改代码前必须先分析

AI 不应该看到需求后立即修改代码。

必须按照：

```text
需求
 ↓
分析项目结构
 ↓
搜索相关代码
 ↓
寻找已有实现
 ↓
确认数据结构
 ↓
确认调用链
 ↓
制定最小修改方案
 ↓
修改代码
 ↓
编译 / 测试
 ↓
检查修改结果

---
> Source: [izhaorui/Zr.Admin.NET](https://github.com/izhaorui/Zr.Admin.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
