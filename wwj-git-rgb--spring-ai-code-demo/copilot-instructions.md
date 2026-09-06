## java-mcp-server

> - 项目名称: Java MCP Server

# 项目开发规则

## 1. 项目概览

### 1.1 项目基本信息（核心）
- 项目名称: Java MCP Server
- 版本: 1.0.0
- 技术栈: Spring Boot 3.4.x + JDK 17 + Spring AI 1.1.0-SNAPSHOT
- 功能: MCP服务器（WEBMVC/SSE/STDIO）



### 1.2 目标架构（核心）
- 基于MCP，集成Spring AI
- 企业级分层：web/service/model/infrastructure
- 强类型API接口

## 2. 核心开发原则

### 2.1 无Map策略 (Map-Free Policy)（强制）
规则：禁止 `Map<String, Object>`
- 允许：强类型DTO/Entity/VO
- 替代：Lombok Builder 强类型结构

```java
// ❌ 禁止
public Map<String, Object> getResult() {
    Map<String, Object> result = new HashMap<>();
    result.put("success", true);
    return result;
}

// ✅ 推荐
@Data
@Builder
public class ApiResponse<T> {
    private boolean success;
    private T data;
    private String message;
}
```

### 2.2 分层架构规范（核心目录）
web/controller|config|interceptor|exception
service/auth|mcp|persistence
model/dto|entity|vo|enums
infrastructure/client|repository|util|validator

### 2.3 命名约定（核心）
- 控制器: `XxxController`
- 服务类: `XxxService`
- 配置类: `XxxConfig`
- DTO: `XxxRequest`/`XxxResponse`
- VO: `XxxInfo`/`XxxData`

## 3. 技术栈规范

### 3.1 依赖版本
```xml
<spring-boot.version>3.4.1</spring-boot.version>
<spring-ai.version>1.1.0-SNAPSHOT</spring-ai.version>
<java.version>17</java.version>
<hutool.version>5.8.25</hutool.version>
```

### 3.2 必要依赖（核心）
- spring-boot-starter-web
- spring-ai-starter-mcp-server-webmvc
- spring-boot-starter-validation
- hutool-all
- lombok

### 3.3 禁用依赖（核心）
- 非必要数据库依赖
- 默认Web安全自动配置（使用自定义）

## 4. API设计规范

### 4.1 RESTful API规范（核心）
基础路径 `/api/v1.0`；MCP `/api/v1.0/mcp/*`；健康检查 `/health`

### 4.2 响应格式
```java
// 成功响应
@Data
@Builder
public class ApiResponse<T> {
    private boolean success = true;
    private T data;
    private String message;
    private Long timestamp;
}

// 错误响应
@Data
@Builder
public class ApiErrorResponse {
    private boolean success = false;
    private String error;
    private String message;
    private Integer code;
    private Long timestamp;
}
```# 项目开发规则

## 1. 项目概览

### 1.1 项目基本信息
- **项目名称**: Java MCP Server
- **版本**: 1.0.0
- **技术栈**: Spring Boot 3.4.x + JDK 17 + Spring AI 1.0.0
- **主要功能**: MCP协议服务器，支持OAuth2和OpenAPI双认证模式

### 1.2 目标架构
- 基于MCP 
- 支持Spring AI集成
- 企业级分层架构设计
- 类型安全的API接口

## 2. 核心开发原则

### 2.1 无Map策略 (Map-Free Policy) ⭐ **已100%实现**
**强制规则**: 代码中禁止使用`Map<String, Object>`类型
- ✅ **允许**: 强类型DTO、Entity、VO类
- ❌ **禁止**: Map传参、Map返回值、Map存储
- **替代方案**: 使用Lombok Builder模式创建类型安全的数据结构

**✅ 已实现的核心类型**:
- `InputSchema` - 工具输入模式强类型定义
- `ToolDefinition` - 工具定义强类型版本
- `ToolExecutionResult` - 工具执行结果强类型版本
- `ToolHandler<T>` - 强类型工具处理器接口

**🔧 重构成果**: 从15个文件使用Map优化为0个文件，100%实现强类型设计

```java
// ❌ 禁止
public Map<String, Object> getResult() {
    Map<String, Object> result = new HashMap<>();
    result.put("success", true);
    return result;
}

// ✅ 推荐
@Data
@Builder
public class ApiResponse<T> {
    private boolean success;
    private T data;
    private String message;
}
```

### 2.2 分层架构规范
```
src/main/java/com/wwj/mcp/server/
├── web/           # Web层
│   ├── controller/    # REST控制器
│   ├── config/       # 配置类
│   ├── interceptor/  # 拦截器
│   └── exception/    # 异常处理
├── service/       # 服务层
│   ├── auth/         # 认证服务
│   ├── mcp/          # MCP服务
│   └── persistence/  # 持久化服务
├── model/         # 数据模型层
│   ├── dto/          # 数据传输对象
│   ├── entity/       # 实体类
│   └── vo/           # 值对象
└── infrastructure/ # 基础设施层
    ├── client/       # 外部客户端
    ├── repository/   # 数据访问
    └── util/         # 工具类
```

### 2.3 命名约定
- **控制器**: `XxxController`
- **服务类**: `XxxService`
- **配置类**: `XxxConfig`
- **DTO类**: `XxxRequest`, `XxxResponse`
- **实体类**: 业务名称实体
- **VO类**: `XxxInfo`, `XxxData`

## 3. 技术栈规范

### 3.1 依赖版本
```xml
<spring-boot.version>3.4.1</spring-boot.version>
<spring-ai.version>1.1.0-SNAPSHOT</spring-ai.version>
<java.version>17</java.version>
<hutool.version>5.8.25</hutool.version>
```

### 3.2 必要依赖
- **Spring Boot Starter Web**: Web应用基础
- **Spring AI MCP Server**: MCP协议支持
- **Spring Security**: 安全认证
- **Hutool**: 工具库
- **Lombok**: 代码简化

### 3.3 禁用依赖
- 避免引入不必要的数据库依赖（除非明确需要）
- 禁用自动配置的Web安全（使用自定义配置）

## 4. API设计规范

### 4.1 RESTful API规范
```
基础路径: /api/v1.0
MCP协议: /api/v1.0/mcp/*
OAuth认证: /api/v1.0/oauth/*
配置管理: /config/*
健康检查: /health
```

### 4.2 响应格式
成功响应 `ApiResponse<T>`；错误响应 `ApiErrorResponse`

---
> Source: [wwj-git-rgb/spring-ai-code-demo](https://github.com/wwj-git-rgb/spring-ai-code-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
