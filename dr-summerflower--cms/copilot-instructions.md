## cms

> 本仓库为演唱会/票务管理系统的单体仓库，包含前端 `cms_client` 与后端 `cms_server`。AI 在本项目中工作时，应优先阅读当前工作目录及其父级目录下的 AGENTS.md，以遵循该区域的约定。

# 演唱会管理系统 (Concert Management System) - AI Agent 指南

## 项目概述

本仓库为演唱会/票务管理系统的单体仓库，包含前端 `cms_client` 与后端 `cms_server`。AI 在本项目中工作时，应优先阅读当前工作目录及其父级目录下的 AGENTS.md，以遵循该区域的约定。

## 仓库结构

- **cms_client/** — 前端应用（React + Vite + TypeScript）
- **cms_server/** — 后端服务（NestJS + MongoDB）
- **docker-compose.yml** / **Dockerfile** — 容器化部署

## 构建/测试/Lint 命令

### 前端 (cms_client)

```bash
cd cms_client
npm run dev          # 开发模式
npm run build        # 生产构建
npm run lint         # ESLint 检查
npm run preview      # 预览构建结果
```

### 后端 (cms_server)

```bash
cd cms_server
npm run start:dev    # 开发模式（热重载）
npm run build        # 生产构建
npm run start:prod   # 生产模式运行
npm run lint         # ESLint + Prettier 自动修复
npm run format       # Prettier 格式化
npm run test         # 运行所有单元测试
npm run test:watch   # 监听模式运行测试
npm run test:cov     # 生成测试覆盖率报告
npm run test:e2e     # 运行端到端测试
```

### 运行单个测试文件

```bash
# 运行指定测试文件
npx jest path/to/file.spec.ts

# 运行匹配测试名称的单个测试
npx jest -t "测试名称关键字"
```

### 类型检查

```bash
# 前端
cd cms_client && npx tsc --noEmit

# 后端
cd cms_server && npx tsc --noEmit
```

## 代码风格指南

### 通用约定

- **语言**：注释、提交说明、文档使用**简体中文**
- **行尾**：统一使用 LF (`\n`)
- **引号**：后端使用单引号 `'`，前端遵循 ESLint 配置
- **分号**：后端使用分号，前端可选
- **尾随逗号**：后端启用尾随逗号（`trailingComma: "all"`）

### 命名约定

- **文件/目录**：kebab-case（如 `user-service.ts`）或 snake_case（模块目录）
- **类名**：PascalCase（如 `UserService`、`AuthService`）
- **函数/变量**：camelCase（如 `getUserInfo`、`userId`）
- **常量**：UPPER_SNAKE_CASE（如 `MAX_RETRY_COUNT`）
- **接口/类型**：PascalCase（如 `UserDTO`、`CreateTicketInput`）
- **DTO 后缀**：使用 `Dto` 后缀（如 `CreateUserDto`、`LoginResponseDto`）

### 导入约定

- **顺序**：第三方库 → 项目内部模块 → 相对路径
- **分组**：不同组之间用空行分隔
- **默认导出**：优先使用命名导出，便于 tree-shaking
- **绝对路径**：后端使用 `baseUrl` 配置导入 `src/` 下文件

### TypeScript 规范

- **严格模式**：启用 `strictNullChecks`、`forceConsistentCasingInFileNames`
- **任何类型**：避免使用 `any`，后端已配置为警告
- **类型推断**：简单类型允许推断，复杂类型需显式声明
- **泛型**：工具函数和公共方法优先使用泛型
- **装饰器**：NestJS 装饰器遵循官方文档规范

### 代码组织

- **类结构**：属性 → 构造函数 → 公共方法 → 私有方法
- **函数长度**：单一函数不超过 50 行，超长需拆分
- **文件长度**：单一文件不超过 500 行，超长需拆分模块
- **注释**：复杂逻辑需添加中文注释，公共 API 使用 JSDoc/TSDoc

### 错误处理

- **后端**：使用 NestJS 内置 `HttpException` 或自定义异常类
- **前端**：统一在 `src/utils/http.ts` 中处理 HTTP 错误
- **异常消息**：使用中文错误消息，便于用户理解
- **日志**：后端使用 NestJS Logger，禁止 console.log

### NestJS 规范（后端）

- **Controller**：仅处理参数校验、调用 Service、返回响应
- **Service**：包含所有业务逻辑
- **DTO**：使用 `class-validator` 装饰器进行参数校验
- **Guard/Filter/Interceptor**：复用全局实现，特殊逻辑在模块内实现
- **依赖注入**：优先使用构造函数注入

### React 规范（前端）

- **组件**：使用函数组件 + Hooks，禁止 class 组件
- **Props**：使用接口定义 Props 类型
- **状态管理**：简单状态用 useState，跨组件用 Zustand
- **效果**：useEffect 需正确清理副作用
- **事件处理**：使用 `handle` 前缀（如 `handleClick`）

## 代码导航与重构

**始终使用 LSP 进行代码导航和重构，禁止使用文本搜索（grep）：**

- ✅ `typescript_find_references("SymbolName")` — 查找所有引用
- ✅ `typescript_get_type_definition("symbol")` — 跳转到类型定义
- ✅ `typescript_rename_symbol("oldName", "newName")` — 安全重命名
- ✅ `typescript_find_implementations("InterfaceName")` — 查找实现
- ✅ `typescript_get_diagnostics()` — 检查类型错误

## 相关 AGENTS.md

- 前端约定与结构见：`cms_client/AGENTS.md`、`cms_client/src/AGENTS.md` 及各子目录
- 后端约定与模块见：`cms_server/AGENTS.md`、`cms_server/src/AGENTS.md` 及各模块根目录

## AI规则

- AI 助手应遵循本 AGENTS.md 及各子目录 AGENTS.md 中的所有约定
- 操作时注意专一性，比如，在编写项目的业务代码时禁止编写测试代码，在编写测试代码时禁止修改业务代码

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dr-SummerFlower)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Dr-SummerFlower)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
