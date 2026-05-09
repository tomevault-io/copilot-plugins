## gsms

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Language Preference
**IMPORTANT: Always respond in Chinese (中文) for this project.** Use Chinese for all explanations, comments, and communications unless specifically requested otherwise.

## Git Commit 约定
**重要：当用户请求 Git 提交时，必须使用 git-commit-assistant agent**

当用户提到以下任一关键词时，**不要直接执行 git 命令**，而是使用 Task tool 调用 git-commit-assistant agent：
- `git commit`
- `commit` + 代码/文件
- `提交代码`
- `committer`（及其拼写变体）

**工作流程：**
1. 识别用户的提交请求
2. 调用：`使用 Task tool 调用 git-commit-assistant agent`
3. Agent 会分析变更并生成提交信息
4. 等待用户确认后再执行提交
5. **不要自动执行 git push**，除非用户明确要求

## 开发协作约定

### 服务启动分工
- **Claude 负责：** 前端服务（`npm run dev`，端口 3000）
- **用户负责：** 后端服务（`mvn spring-boot:run`，端口 8080）
- **特殊情况：** 只有当用户遇到无法解决的后端问题时，才会将后端服务交给 Claude 启动

### 端口约定（固定，不可更改）
- 前端：**3000**
- 后端：**8080**
- ⚠️ 严禁更换其他端口（因 CORS 跨域配置限制）

### 后端热部署最佳实践

**正确做法：使用 Spring Boot DevTools**
```yaml
# application.yml 已配置
spring:
  devtools:
    restart:
      enabled: true  # 启用热部署
      additional-paths: src/main/java
      exclude: static/**,public/**
    livereload:
      enabled: true
```

**优势：**
- ✅ 修改 Java 代码后自动重启应用
- ✅ 无需手动启停服务
- ✅ 高效便捷，开发体验好

**错误做法（严禁）：**
- ❌ 查端口 → 杀进程（效率低）
- ❌ 手动停止 → 重新启动（浪费时间）

**使用方式：**
1. 启动后端：`cd backend && mvn spring-boot:run`
2. **修改 Java 代码后，必须先编译：`mvn compile`**
3. DevTools 检测到 class 文件变化，自动触发重启
4. 查看日志确认重启成功

**⚠️ 关键点：**
- DevTools 监控的是 **编译后的 class 文件**，不是源代码
- 修改代码后**必须执行 `mvn compile`** 才能触发热部署
- 不可跳过编译步骤，否则不会重启

**IDE 支持：**
- IntelliJ IDEA：修改文件后自动编译，无需手动操作
- VS Code：安装 Spring Boot Extension Pack，自动编译
- 命令行（Claude 使用）：手动执行 `mvn compile` 触发热部署

## 快速开始

**环境要求：**
- JDK 8+, Maven 3.6+, Node.js 18+
- MySQL 8.0+ (创建数据库: `CREATE DATABASE gsms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;`)

**启动后端（端口 8080）：**
```bash
cd backend
mvn spring-boot:run
# 或使用环境变量:
DB_USERNAME=root DB_PASSWORD=your_password mvn spring-boot:run
```

**启动前端（端口 3000）：**
```bash
cd frontend
npm install
npm run dev
```

**Docker 快速启动：**
```bash
# 启动所有服务（MySQL + 后端 + 前端）
docker-compose up -d
```

**测试账号：**
- `admin` / `Admin123` - 管理员
- `zhangsan03` / `Admin123` - 普通用户

**API 文档：** http://localhost:8080/swagger-ui.html

---

## 项目概述

TeamMaster（统领工时管理平台）是一个面向研发团队的轻量级工时管理系统。这是一个基于 Spring Boot 的应用，采用标准三层架构结合DTO模式，具有清晰的分层结构。

**核心功能模块：**

1. **工时管理** - 项目、迭代、任务、工时记录的完整管理
2. **用户权限管理（RBAC）** - 基于角色的访问控制系统 ✨
   - 用户管理（CRUD + 角色分配 + 启用/禁用）
   - 角色管理（CRUD + 权限分配）
   - 权限管理（CRUD + 权限查询）
   - 三级权限控制（路由级 + 按钮级 + 数据级）
   - 用户注册流程（默认禁用，需管理员审核）
3. **统计报表** - 首页看板、项目统计、用户统计、部门统计

**项目结构（Monorepo）：**

```
gsms/
├── backend/          # Spring Boot 后端（端口 8080）
├── frontend/         # Vue 3 前端（端口 3000）
├── docs/            # 项目文档
├── deployment/      # 部署配置
└── TODO.md          # 待办事项
```

**技术栈：**

**后端：**

- Java 8 + Spring Boot 2.7.0
- MyBatis-Plus 3.5.3.1 + MySQL 8.0
- JWT 认证 (jjwt 0.9.1)
- Maven 构建管理
- SpringDoc OpenAPI 1.7.0 API文档
- Flyway 数据库版本管理
- PageHelper 分页插件
- Java 8 Time API (LocalDateTime, LocalDate)

**前端：**

- Vue 3.4 + TypeScript 5.3 + Vite 5.0
- Element Plus 2.5 UI 组件库
- Vue Router 4 + Pinia 2.1
- Axios 1.6 HTTP 客户端

## 常用命令

### 后端构建和运行

```bash
cd backend

# 构建项目
mvn clean install

# 运行应用（开发环境）
mvn spring-boot:run

# 使用指定配置文件运行
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# 使用环境变量覆盖数据库配置
DB_USERNAME=root DB_PASSWORD=your_password mvn spring-boot:run

# 打包
mvn clean package
```

### 前端构建和运行

```bash
cd frontend

# 安装依赖
npm install

# 运行开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览生产构建
npm run preview

# 代码检查
npm run lint

# 代码格式化
npm run format
```

### Docker 部署

```bash
# 启动所有服务（MySQL、后端、前端）
docker-compose up -d

# 查看日志
docker-compose logs -f backend

# 停止服务
docker-compose down

# 重新构建并启动
docker-compose up -d --build
```

### 测试

```bash
# 运行所有测试
mvn test

# 运行指定测试类
mvn test -Dtest=UserControllerTest

# 运行指定测试方法
mvn test -Dtest=UserControllerTest#testGetUserById

# 运行所有Controller测试
mvn test -Dtest=*ControllerTest
```

### 数据库

```bash
# Flyway数据库迁移（启动时自动执行）
# 手动执行迁移：
mvn flyway:migrate

# 查看Flyway状态
mvn flyway:info

# 修复失败的迁移
mvn flyway:repair
```

## 架构设计

### 分层结构（标准三层架构）

```
controller/         # REST API层 - 处理HTTP请求、参数校验
├── dto/           # 数据传输对象（独立包）
├── service/       # 业务逻辑层
│   └── impl/      # 服务实现类
├── model/         # 领域层（注意：不是 domain/）
│   ├── entity/    # JPA实体（数据库模型）
│   ├── enums/     # 业务枚举（包括错误码）
│   └── errorcode/ # 错误码枚举
├── repository/    # 数据访问层（MyBatis Mapper接口）
└── infra/         # 基础设施层
    ├── common/    # 通用组件（Result、PageResult）
    ├── config/    # Spring配置类
    ├── exception/ # 自定义异常
    └── utils/     # 工具类
```

**⚠️ 重要：目录命名差异**

本项目使用 `model/` 而非 `domain/`：
- 实体位于：`model/entity/` （不是 `domain/entity/`）
- 枚举位于：`model/enums/` （不是 `domain/enums/`）
- 错误码位于：`model/errorcode/` （不是 `domain/errorcode/`）

**数据流：**

1. **Controller** 接收HTTP请求，使用 `@Valid` 校验，调用Service
2. **DTO** 对象在Controller和Service层之间传输数据
3. **Service** 实现业务逻辑，使用 `UserContext.getCurrentUserId()` 获取当前用户
4. **Repository**（MyBatis Mapper）处理数据库操作
5. **Model** 实体表示数据库表结构（位于 `model/entity/` 目录）
6. **Result<T>** 包装所有API响应为统一格式
7. **GlobalExceptionHandler** 统一异常处理，返回标准错误响应

### 架构模式

**标准三层架构 + DTO模式：**

- **Controller层**：处理HTTP请求和响应，参数校验
- **Service层**：实现业务逻辑和事务管理
- **Repository/Mapper层**：数据持久化操作
- **DTO模式**：Controller和Service之间使用DTO传输数据
- **Model层**（而非Domain）：实体类（entity）和枚举（enums），使用贫血模型模式
  - `model/entity/` - JPA实体（数据库模型）
  - `model/enums/` - 业务枚举
  - `model/errorcode/` - 错误码枚举
- 这种架构适合中小型项目，职责清晰，易于理解和维护

### 关键设计模式

**DTO模式：**

- **CreateReq** DTO用于创建实体（排除`id`、`createTime`等字段）
- **UpdateReq** DTO用于更新（包含`id`，字段可选）
- **InfoResp** DTO用于API响应（排除敏感字段）
- **QueryReq** DTO用于搜索/过滤和分页（继承`BasePageQuery`）
- `dto/*/converter/`中的转换器处理 Entity ↔ DTO 映射

**Service接口模式：**

- 所有服务在`service/`中有接口，在`service/impl/`中有实现
- 方法返回Entity（内部使用）或DTO（API响应）
- **重要：** Service方法现在使用DTO参数（如`create(UserCreateReq)`）而非Entity

**认证和授权：**

- 通过`JwtInterceptor`实现基于JWT Token的无状态认证
- `UserContext.getCurrentUserId()`获取已认证用户ID
- `AuthService`提供权限检查：
  - `checkProjectAccess(userId, projectId)` - 项目成员检查
  - `checkTaskAccess(userId, taskId)` - 任务可见性检查
  - `canViewAllProjects(userId)` - 全局项目查看权限
- 基于角色/权限表的访问控制

**分页：**

- MyBatis查询使用`PageHelper.startPage(pageNum, pageSize)`
- `PageResult<T>`包装分页结果及元数据
- 请求DTO继承`BasePageQuery`（默认：pageNum=1, pageSize=10）

**前后端集成（CORS 配置）：**

- 后端 CORS 配置在 `WebConfig.java` 中，允许 `localhost:3000` 跨域访问
- 前端 Vite 代理配置在 `vite.config.ts` 中，将 `/api/*` 请求转发到 `http://localhost:8080/api/*`
- 开发时需要同时启动后端（8080）和前端（3000）
- JWT Token 存储在前端，通过 `Authorization: Bearer <token>` 请求头传递

## 关键实现细节

### 用户上下文管理

始终使用`executeWithUserContext`包装需要用户上下文的操作：

```java
executeWithUserContext(userId, () -> {
    // 需要 UserContext.getCurrentUserId() 才能正常工作的代码
    return result;
});
```

这在测试的`@BeforeEach`设置中创建测试数据时特别重要。

### 实体审计字段

大多数实体具有以下审计字段（自动管理）：

- `createTime` / `updateTime` - 时间戳
- `createUserId` / `updateUserId` - 用户追踪
- `isDeleted` - 软删除标志（0=有效，1=已删除）

**重要：** 始终在Service的create方法中设置`createUserId`和`updateUserId`：

```java
workHour.setCreateUserId(currentUserId);
workHour.setUpdateUserId(currentUserId);
```

### 错误处理

- 自定义异常继承`BusinessException`
- 错误码定义在`model/enums/errorcode/`包中
- `GlobalExceptionHandler`捕获所有异常并返回标准`Result`格式
- 常见错误码：`UNAUTHORIZED(1401)`、`FORBIDDEN(1403)`、`NOT_FOUND`系列

### 参数校验

- 在DTO上使用JSR-303注解：`@NotBlank`、`@NotNull`、`@Email`等
- Controller方法在请求体参数上使用`@Valid`或`@Validated`
- Service层的自定义校验规则抛出`BusinessException`

### MyBatis Mapper XML

位于`src/main/resources/mapper/`。

- 使用`<resultMap>`进行复杂实体映射
- 使用`<sql id="selectAllFields">`避免重复列列表
- 使用`typeHandler="com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler"`处理枚举类型

### 日期时间处理

- **使用 Java 8 Time API**：所有日期字段使用 `LocalDateTime`（日期+时间）或 `LocalDate`（仅日期）
- **Jackson 配置**（`JacksonConfig.java`）：
  - 禁用 ISO 8601 格式输出
  - `LocalDateTime` 序列化为 `yyyy-MM-dd HH:mm:ss`
  - 时区设置为 `Asia/Shanghai`
- **枚举类型序列化**：所有业务枚举使用 `@JsonValue` 注解 `toString()` 方法，返回枚举名称（如 `NORMAL`）
  - 数据库存储：使用 `@EnumValue` 标记的 `code` 字段（Integer）
  - JSON 序列化：返回枚举的 `name()`（String）

### 数据库设计原则

项目采用严格的表分类和审计策略（详见 `docs/DATABASE_OPTIMIZATION.md`）：

**1. 核心业务表（9个）** - 完整审计字段：

- 表：`sys_user`, `sys_department`, `role`, `permission`, `gsms_project`, `gsms_iteration`, `gsms_task`, `gsms_work_hour`, `gsms_project_member`
- 审计字段：`create_user_id`, `create_time`, `update_user_id`, `update_time`, `is_deleted`
- 注意：`gsms_project` 和 `gsms_iteration` 的 `create_user_id` 通过关联表间接获取

**2. 关联表（4个）** - 简化审计字段：

- 表：`gsms_project_member`, `role_permission`, `user_role`, `department_user`
- 仅需：`update_time`, `update_user_id`, `is_deleted`
- 原则：关联表专注于表达关系，不需要创建人信息

**3. 枚举表（0个）** - 使用 Java 枚举：

- 所有状态/类型字段使用 Java 枚举 + MyBatis-Plus 枚举处理器
- 不在数据库中维护枚举表

**设计原则**：

- 数据表专注于核心职责
- 关联表用于表达关系，审计由专门日志系统完成
- 外键约束确保数据完整性
- 性能优化索引（查询常用字段）

## 测试策略

### 测试结构

```
src/test/java/com/gsms/gsms/
├── config/
│   └── TestMyBatisConfig.java       # MyBatis测试配置
├── controller/
│   ├── BaseControllerTest.java      # Controller测试基类
│   └── *ControllerTest.java         # 各Controller的集成测试
├── service/
│   └── *ServiceTest.java            # Service层测试
└── repository/
    └── *MapperTest.java             # Mapper层测试
```

### Controller集成测试

- 继承`BaseControllerTest`（使用`@SpringBootTest`和真实Service、MySQL测试数据库）
- 在`@BeforeEach`中创建测试用户并使用`JwtUtil`生成JWT Token
- 使用`executeWithUserContext()`创建测试数据以设置UserContext
- 使用`objectMapper.writeValueAsString()`将DTO序列化为JSON
- 测试成功和失败场景（401、403、404）

### Service层测试

- 对依赖项使用`@MockBean`（其他Service、Mapper）
- 测试业务逻辑和错误处理
- 验证与Mapper的交互

### 单元测试最佳实践

- 使用MySQL测试数据库（gsms_test）进行集成测试
- 必要时在`@AfterEach`中清理测试数据
- 避免硬编码ID - 使用创建的测试数据的ID
- 使用描述性测试方法名：`test{方法名}_{场景}_{预期结果}`

## API文档

Swagger UI地址：`http://localhost:8080/swagger-ui.html`

**常用端点：**

- POST `/api/users/login` - 获取JWT Token（设置到`Authorization: Bearer <token>`请求头）
- GET `/api/users` - 用户列表（分页）
- GET `/api/projects` - 项目列表（需要项目成员权限）
- GET `/api/tasks/search` - 按条件搜索任务
- POST `/api/work-hours` - 创建工时记录
- GET `/api/statistics/dashboard` - 首页看板统计数据

**认证：**
- 除 `/api/users/login` 外，所有接口需要 `Authorization: Bearer <token>` 请求头
- Token 有效期：24小时（可在 `application.yml` 中配置）

## 配置文件

- `application.yml` - 主配置（数据库、MyBatis、PageHelper、日志）
- `application-dev.yml` - 开发环境配置覆盖
- `application-prod.yml` - 生产环境配置覆盖
- `logback-spring.xml` - 日志配置（输出到`logs/`目录）
- `src/main/resources/mapper/`中的Mapper XML - MyBatis SQL映射

**环境变量：**
- `DB_USERNAME` - 数据库用户名（默认：root）
- `DB_PASSWORD` - 数据库密码（默认：root）

## 常见陷阱

1. **目录结构差异**：代码使用 `model/` 而非 `domain/`

   - 实体位于：`model/entity/`
   - 枚举位于：`model/enums/`
   - 错误码位于：`model/errorcode/`

2. **Service方法签名已变更：** 许多Service现在接受DTO而非Entity

   - 旧版：`createUser(User user)`
   - 新版：`create(UserCreateReq req)`

3. **缺少审计字段：** 忘记设置`createUserId`/`updateUserId`会导致SQL错误

   - 核心业务表需要设置：`createUserId` 和 `updateUserId`
   - 关联表（如 `project_member`）只需要 `updateUserId`

4. **权限检查：** Controller通过`AuthService`检查权限，因此测试必须：

   - 创建具有适当成员关系的测试项目
   - 使用具有相应权限的用户
   - 权限检查在存在性检查之前失败时，期望403（而非404）

5. **查询参数绑定：** 测试带查询参数的GET请求时，使用`.param("key", "value")`而非`.queryParam()`

6. **JWT Token必需：** 除`/api/users/login`和`/hello`外的所有端点都需要`Authorization: Bearer <token>`请求头

7. **PageHelper使用：** 必须在执行查询**之前**调用`PageHelper.startPage()`，且只对之后执行的**第一个**查询生效

8. **枚举序列化差异**：

   - GET 请求枚举参数支持两种形式：数字码（`?status=1`）或枚举名（`?status=NOT_STARTED`）
   - JSON 响应中枚举输出为字符串（枚举名），如 `"status": "NORMAL"`
   - 数据库存储为整数（code），如 `status = 1`

9. **日期类型迁移**：项目已从 `java.util.Date` 迁移到 Java 8 Time API

   - 新代码应使用 `LocalDateTime` 或 `LocalDate`
   - 测试中使用 `LocalDateTime.now()` 而非 `new Date()`
   - Jackson 自动格式化为 `yyyy-MM-dd HH:mm:ss`

10. **Debug 模式死锁问题**：已升级 ClassGraph 到 4.8.172 修复

    - 详见：`docs/DEBUG_MODE_HANG_ISSUE.md`
    - 如果遇到 Spring Boot Debug 模式启动卡住，检查 ClassGraph 版本

11. **CORS 预检请求**：JWT拦截器必须处理OPTIONS请求

    - JWT拦截器需要在 `preHandle` 方法开头放行 OPTIONS 请求

## 参考文档

- **RBAC 权限系统**：`docs/RBAC_IMPLEMENTATION.md` - 用户、角色、权限管理系统完整实现文档（2026-01-12）
- **缓存技术决策**：`docs/caching-technical-decisions.md` - 缓存方案对比（ConcurrentHashMap vs Caffeine vs Redis）、Spring 单例原理
- **数据库优化**：`docs/DATABASE_OPTIMIZATION.md` - 表分类、审计字段、外键约束设计
- **调试指南**：`docs/DEBUG_PROCESS_WALKTHROUGH.md` - 远程调试、断点、变量查看
- **前后端联调**：`docs/development/frontend-backend-setup.md` - CORS、代理、认证配置
- **API 文档**：`docs/api-docs.md` - REST API 接口文档
- **重构原则**：`docs/refactoring-principles.md` - 代码重构最佳实践
- **待办事项**：`TODO.md` - API 文档优化、枚举类型标准化、后续扩展功能

## 文件命名规范

- 实体：`User.java`、`Project.java`、`WorkHour.java`
- DTO：`UserCreateReq.java`、`UserInfoResp.java`、`UserQueryReq.java`
- Controller：`UserController.java`
- Service：`UserService.java`（接口）、`UserServiceImpl.java`（实现）
- Mapper：`UserMapper.java`（接口）、`UserMapper.xml`（SQL）
- 测试：`UserControllerTest.java`、`UserServiceTest.java`

## 开发工作流

### 新增功能的标准步骤

1. **数据库迁移**：在 `src/main/resources/db/migration/` 创建新的 Flyway 脚本
2. **创建实体类**：在 `model/entity/` 创建实体，添加枚举到 `model/enums/`
3. **创建 Mapper**：创建 `Mapper.java` 接口和 `mapper/Mapper.xml` SQL 映射
4. **创建 DTO**：在 `dto/` 下创建包，定义 CreateReq、UpdateReq、InfoResp、QueryReq
5. **创建 Converter**：在 `dto/` 下创建 converter 包，实现 Entity ↔ DTO 转换
6. **实现 Service**：创建 Service 接口和 Impl，包含业务逻辑和权限检查
7. **实现 Controller**：创建 REST 端点，使用 `@Valid` 校验，调用 Service
8. **编写测试**：创建 ControllerTest、ServiceTest，覆盖成功和失败场景
9. **更新文档**：如需要，更新 API 文档（Swagger 自动生成）

### 代码审查要点

- [ ] 审计字段是否正确设置（createUserId/updateUserId）
- [ ] 权限检查是否完善（使用 AuthService）
- [ ] DTO 参数校验是否完整（@Valid、@NotNull 等）
- [ ] 异常处理是否恰当（BusinessException、错误码）
- [ ] 测试覆盖率是否足够（成功/失败场景）
- [ ] 枚举序列化是否正确（@JsonValue、@EnumValue）
- [ ] 日志输出是否合理（INFO、DEBUG、ERROR）

---

## 前端开发规范

基于项目实践总结的前端开发规范，涵盖布局、样式、组件设计等最佳实践。

### 前端项目结构

```
frontend/
├── src/
│   ├── api/              # API 接口模块
│   │   ├── request.ts   # Axios 实例和拦截器
│   │   └── *.ts         # 各业务模块 API
│   ├── components/      # 公共组件
│   ├── router/          # 路由配置
│   ├── stores/          # Pinia 状态管理
│   │   └── auth.ts      # 认证状态 Store
│   ├── views/           # 页面组件
│   │   ├── auth/        # 认证相关页面
│   │   ├── project/     # 项目相关页面
│   │   ├── task/        # 任务相关页面
│   │   ├── iteration/   # 迭代相关页面
│   │   ├── workhour/    # 工时相关页面
│   │   └── dashboard/   # 首页看板
│   ├── App.vue          # 根组件
│   └── main.ts          # 应用入口
├── public/              # 公共静态资源
├── index.html           # HTML 模板
├── vite.config.ts       # Vite 配置
└── package.json         # 项目依赖
```

### 状态管理规范

#### 使用 Pinia Store

**必须使用 auth store 管理认证状态：**

```typescript
import { useAuthStore } from '@/stores/auth'

const authStore = useAuthStore()

// ✅ 正确：从 store 获取用户ID
const userId = authStore.getCurrentUserId()

// ❌ 错误：直接从 localStorage 获取
const userId = parseInt(localStorage.getItem('userId') || '0')
```

**认证信息管理：**

```typescript
// 登录时设置认证信息
authStore.setAuth(token, username)

// 退出登录时清除认证信息
authStore.clearAuth()

// 检查是否已登录
if (authStore.isAuthenticated) {
  // 已登录逻辑
}

// 获取当前用户信息
const { id, username } = authStore.currentUser
```

### API 调用规范

**响应拦截器支持多种 code 格式：**
- `code: 0` - 成功（登录接口使用）
- `code: 200` - 成功（其他接口使用）
- 其他 code 值视为错误

**统一错误处理：**
```typescript
try {
  const res = await getProjectList(searchForm)
  list.value = res.list || []
  total.value = res.total || 0
} catch (error) {
  console.error('获取数据失败:', error)
  ElMessage.error('获取数据失败')
}
```

### 页面组件设计规范

#### 组件文件结构

**标准模板：**

```vue
<template>
  <div class="{module-list}">
    <!-- 页面头部 -->
    <div class="page-header">
      <div class="header-left">
        <h2 class="page-title">{模块标题}</h2>
        <el-button type="primary" :icon="Plus" @click="handleCreate">
          新建{模块}
        </el-button>
      </div>
      <div class="header-right">
        <!-- 搜索、筛选、视图切换 -->
      </div>
    </div>

    <!-- 主内容区域 -->
    <div class="{view-mode}-view">
      <!-- 表格/看板/列表 -->
    </div>

    <!-- 分页 -->
    <div class="pagination" v-if="total > 0">
      <el-pagination />
    </div>

    <!-- 对话框 -->
    <el-dialog v-model="dialogVisible" title="{标题}">
      <el-form :model="formData" :rules="formRules">
        <!-- 表单字段 -->
      </el-form>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, computed } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage, ElMessageBox, type FormInstance, type FormRules } from 'element-plus'
import { useAuthStore } from '@/stores/auth'
import { getModuleList } from '@/api/module'

const router = useRouter()
const authStore = useAuthStore()
const formRef = ref<FormInstance>()

// 数据定义
const list = ref<any[]>([])
const total = ref(0)

// 搜索表单
const searchForm = reactive({
  name: '',
  status: undefined as string | undefined,
  pageNum: 1,
  pageSize: 10
})

// 对话框
const dialogVisible = ref(false)
const formData = reactive({...})
const formRules: FormRules = {...}

// 生命周期
onMounted(() => {
  fetchData()
})

// 方法定义
const fetchData = async () => {
  try {
    const res = await getModuleList(searchForm)
    list.value = res.list || []
    total.value = res.total || 0
  } catch (error) {
    console.error('获取数据失败:', error)
  }
}

const handleCreate = () => {
  dialogTitle.value = '新建{模块}'
  resetForm()
  dialogVisible.value = true
}

// ...
</script>

<style scoped>
.{module-list} {
  min-height: calc(100vh - 160px);
}

.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
  padding: 20px;
  background: #fff;
  border-radius: 4px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.03);
}

.header-left {
  display: flex;
  align-items: center;
  gap: 16px;
}

.page-title {
  margin: 0;
  font-size: 20px;
  font-weight: 500;
  color: #333;
}

.header-right {
  display: flex;
  align-items: center;
}

/* 主内容样式 */
.{view-mode}-view {
  margin-bottom: 24px;
}

/* 分页样式 */
.pagination {
  display: flex;
  justify-content: flex-end;
  padding: 20px;
  background: #fff;
  border-radius: 4px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.03);
}
</style>
```

### 看板视图设计规范

#### 看板布局结构

**标准看板模板（基于项目列表实践）：**

```vue
<template>
  <div class="kanban-view">
    <el-row :gutter="16">
      <el-col v-for="status in statuses" :key="status.value" :xs="24" :sm="12" :md="6">
        <div
          class="kanban-column"
          :data-status="status.value"
          @dragover.prevent="handleDragOver"
          @dragleave="handleDragLeave"
          @drop="handleDrop"
        >
          <!-- 列头 -->
          <div class="column-header">
            <div class="column-title">
              <div class="status-dot" :style="{ backgroundColor: status.color }"></div>
              <span>{{ status.label }}</span>
              <span class="count">{{ getItemsByStatus(status.value).length }}</span>
            </div>
          </div>

          <!-- 列体 -->
          <div class="column-body">
            <div
              v-for="item in getItemsByStatus(status.value)"
              :key="item.id"
              class="item-card"
              draggable="true"
              @dragstart="handleDragStart(item, $event)"
              @click="handleView(item)"
            >
              <!-- 卡片内容 -->
            </div>
            <el-empty v-if="getItemsByStatus(status.value).length === 0" />
          </div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script setup lang="ts">
// 使用 computed 优化性能
const status1Items = computed(() => list.value.filter(item => item.status === 'STATUS_1'))
const status2Items = computed(() => list.value.filter(item => item.status === 'STATUS_2'))

// 辅助函数
const getItemsByStatus = (status: string) => {
  const map: Record<string, any[]> = {
    'STATUS_1': status1Items.value,
    'STATUS_2': status2Items.value
  }
  return map[status] || []
}

// 拖拽处理
const draggedItem = ref<any>(null)

const handleDragStart = (item: any, event: DragEvent) => {
  draggedItem.value = item
  if (event.dataTransfer) {
    event.dataTransfer.setData('text/plain', item.id.toString())
    event.dataTransfer.effectAllowed = 'move'
  }
  ;(event.target as HTMLElement).classList.add('dragging')
}

const handleDrop = async (event: DragEvent) => {
  event.preventDefault()
  const targetStatus = (event.currentTarget as HTMLElement).getAttribute('data-status')

  // 移除拖拽样式
  const draggingElements = document.querySelectorAll('.dragging')
  draggingElements.forEach(el => el.classList.remove('dragging'))

  if (draggedItem.value && targetStatus && draggedItem.value.status !== targetStatus) {
    // 更新状态
    await updateItem({
      id: draggedItem.value.id,
      status: targetStatus
    })
    ElMessage.success('状态已更新')
    fetchData()
  }

  draggedItem.value = null
}
</script>

<style scoped>
.kanban-column {
  background: #f5f5f5;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 16px;
  transition: all 0.3s;
}

.kanban-column.drag-over {
  background: #e6f7ff;
  box-shadow: 0 0 0 2px #1890ff inset;
}

.item-card.dragging {
  opacity: 0.5;
  cursor: move;
}
</style>
```

### 模块联动规范

**实现页面间跳转联动：**

```typescript
import { useRouter } from 'vue-router'

const router = useRouter()

// 跳转到项目详情
const handleProjectClick = (projectId: number) => {
  router.push(`/projects/${projectId}`)
}

// 跳转到任务详情
const handleTaskClick = (taskId: number) => {
  router.push(`/tasks/${taskId}`)
}

// 跳转到列表页并带状态过滤
const handleViewAll = (status: string) => {
  router.push({
    path: '/tasks',
    query: { status }
  })
}
```

**路由参数接收：**

```typescript
import { useRoute } from 'vue-router'

const route = useRoute()

// 获取查询参数
const status = route.query.status as string | undefined

if (status) {
  searchForm.status = status
}
```

### 前端最佳实践总结

#### 1. 组件设计

**✅ DO（推荐）：**

- 使用 Composition API (`<script setup>`)
- 使用 TypeScript 类型定义
- 组件文件命名：`{Module}List.vue`, `{Module}Detail.vue`
- 单一职责原则，每个组件只负责一个功能模块
- Props 和 Emits 明确定义
- 使用 ref 和 reactive 管理响应式状态

**❌ DON'T（避免）：**

- 组件过大时，拆分为多个子组件
- 硬编码数据和配置
- 在组件中直接访问 localStorage（使用 auth store）
- 混用 Options API 和 Composition API

#### 2. 状态管理

**✅ DO（推荐）：**

- 使用 Pinia Store 管理全局状态（认证、用户信息等）
- 使用 local state 管理组件私有状态
- 认证信息统一从 auth store 获取
- 使用 computed 缓存计算结果

**❌ DON'T（避免）：**

- 在组件中直接操作 localStorage
- 在多个组件中重复获取相同数据
- 将大量状态存储在全局 store（只在必要时使用）

#### 3. 样式编写

**✅ DO（推荐）：**

- 使用 `<style scoped>` 限制样式作用域
- 遵循 BEM 命名规范
- 使用 CSS 变量定义颜色和尺寸
- 响应式设计考虑（断点、布局）
- 过渡动画使用 Vue transition

**❌ DON'T（避免）：**

- 内联样式（style="..."）
- !important 声明（优先级问题）
- 硬编码颜色和尺寸
- 过度嵌套选择器（性能问题）

#### 4. API 调用

**✅ DO（推荐）：**

- 每个 API 模块定义 TypeScript 接口
- 统一的错误处理（try-catch + ElMessage）
- 使用 async/await 处理异步
- 请求前验证权限和数据
- loading 状态管理

**❌ DON'T（避免）：**

- 在组件中直接调用 axios（使用 API 封装）
- 重复的 API 调用
- 忽略错误处理
- 忘记设置 loading 状态

#### 5. 路由设计

**✅ DO（推荐）：**

- 路由懒加载（动态 import）
- 路由元信息定义（title, requiresAuth 等）
- 编程式导航（router.push）
- 路由参数校验

**❌ DON'T（避免）：**

- 硬编码路径
- 重复定义路由
- 忘记处理 404 路由

---

## 前端技术决策

**详细的架构和设计决策：**

- **前端架构文档**：`docs/development/frontend-architecture.md`
- **前后端联调**：`docs/development/frontend-backend-setup.md`
- **模块联动分析**：`docs/frontend-module-linkage-analysis.md`

---
> Source: [dedatech/gsms](https://github.com/dedatech/gsms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
