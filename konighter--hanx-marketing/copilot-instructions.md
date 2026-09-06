## hanx-marketing

> **hzapp-erplus** 是一套面向跨境电商的 ERP 系统，采用前后端分离的微服务/模块化架构。后端基于 Java (Spring Boot)，前端基于 Vue 3 (`hzapp-ui-admin-vue3-tiny`)。

# Coding Agent Guide for hzapp-erplus

## Project Overview

**hzapp-erplus** 是一套面向跨境电商的 ERP 系统，采用前后端分离的微服务/模块化架构。后端基于 Java (Spring Boot)，前端基于 Vue 3 (`hzapp-ui-admin-vue3-tiny`)。

---

## 🏗 Project Structure (项目结构)

### Backend (`hzapp-module-erplus`)
后端按业务领域拆分了多个子模块：
- `hzapp-module-erplus-api`: 存放全局常量（如 `ErpMetricConstants.java`）、DTO 和共用接口定义。
- `hzapp-module-erplus-biz`: 核心基础业务逻辑（订单、仓储等）。
- `hzapp-module-erplus-adv`: 广告相关业务逻辑（预算、看板、融合模型等）。
- `hzapp-module-erplus-amz`: 亚马逊平台集成通用模块。
- `hzapp-module-erplus-spapi`: 针对 Amazon SP-API 的专属数据拉取与同步模块。

### Frontend (`hzapp-ui/hzapp-ui-admin-vue3-tiny`)
前端采用 Vue 3 + `<script setup>` 组合式 API 架构。

---

## ⚠️ Development Conventions (核心开发规约)

### 1. 前端架构边界 (Front-end Module Boundary) **[CRITICAL]**
- **严格按模块化开发**：所有具体的业务页面、组件和独立逻辑，必须放在 `src/app/xx` 目录下（例如：`src/app/erplus/views/adv/budget`）。
- **禁止污染公共区**：绝不允许为了业务需求，去修改或污染 `src/app` 目录外的所有公共文件（包括 `src/views` 下的基础页面等）。公共目录仅限核心框架级改动。
- **公共组件复用 (Component Reuse)**：在 `erplus` 模块开发前端页面时，遇到通用的业务组件（如选店铺、选平台等），必须优先引入 `src/app/erplus/compononts/` 目录下的已有组件（例如直接使用 `<ShopCascaderSelect />`），禁止在页面中重复造轮子。

### 2. 后端开发规约 (Backend Conventions)
- **精准落位**：新功能必须落在对应的业务子模块内（如广告功能写在 `-adv` 模块，常量补充在 `-api` 模块）。
- **代码一致性**：遵循现有的 Spring Boot 分层规范 (Controller -> Service -> Mapper/Repository)。
- **最小化变更**：只修改被用户明确要求变更的逻辑。保留现有注释、格式、和无关的常数。

### 3. 可视化与原型设计 (Visualization & Prototyping)
- 在开发复杂看板或数据展示类页面前，应优先利用 `brainstorming` 技能及 Visual Companion 在浏览器中快速输出 HTML 布局供用户确认。
- **重构分离**：图表、看板底层数据的流速判定和累计期望算法应尽早解耦，保持“后端产出算法/偏差，前端只做响应式渲染”的原则。

---

## Development Commands (开发命令提示)

*(此部分可根据本地项目的实际打包、启动脚本进行扩展)*

| Command | Purpose |
|---------|---------|
| `cd hzapp-ui/hzapp-ui-admin-vue3-tiny && npm run dev` | 启动前端本地开发服务器 |
| `./gradlew clean build` | 编译构建后端多模块项目 |

## Operational Guidelines (AI Agent 操作指南)
- **先思考后执行 (Plan before code) [CRITICAL]**: 拿到用户指令后，需要先思考，给出推荐做法和原因，经过用户确认后进行实施，绝不要直接开始写代码。
- **Think Before Coding**: 在跨模块新建类和前端视图前，先探查目录结构，如果不确定具体模块位置，先与用户对齐。
- **No Speculative Abstractions**: 采用直给的解决方案，不要过度封装用户未要求的基础工具类。
- **Verify Execution**: 执行移动文件 (`mv`)、创建目录等 shell 命令前，务必检查源路径和目标路径的有效性。

---
> Source: [konighter/hanx-marketing](https://github.com/konighter/hanx-marketing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
