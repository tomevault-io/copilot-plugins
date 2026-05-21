## smart-schedule-corev2

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

智能排课系统（Smart Schedule Core V2）是一个基于 Spring Boot 3.5.9 的智能排课核心服务，使用 PostgreSQL 数据库。系统采用 MyBatis Plus 进行数据访问，使用 JSONB 字段存储复杂数据关系。

## 构建和运行命令

```bash
# 清理并编译项目
mvn clean compile

# 运行测试
mvn test

# 运行单个测试类
mvn test -Dtest=ClassName

# 打包项目
mvn clean package

# 跳过测试打包
mvn clean package -DskipTests

# 运行应用
mvn spring-boot:run

# 或者运行打包后的 JAR
java -jar target/smart-schedule-coreV2-0.0.1-SNAPSHOT.jar
```

### 开发环境
- Java 版本：17
- Maven：自动管理依赖
- 数据库：PostgreSQL
- Redis：用于缓存（spring-boot-starter-data-redis）

## 核心架构

### 三层架构设计

项目采用 MyBatis Plus 标准三层架构：

1. **DO 层（Model）**：数据对象
   - 位置：`model/entity/`
   - 命名：`*DO.java`
   - 使用 `@TableName`、`@TableId`、`@TableField` 注解
   - 所有 DO 使用 `@Data` 和 `@Accessors(chain = true)` 支持链式调用

2. **Mapper 层**：数据访问接口
   - 位置：`mapper/`
   - 命名：`*Mapper.java`
   - 继承 `BaseMapper<*DO>`，使用 `@Mapper` 注解

3. **DAO 层**：数据访问服务
   - 位置：`dao/`
   - 命名：`*DAO.java`
   - 继承 `ServiceImpl<*Mapper, *DO>` 并实现 `IService<*DO>`
   - 使用 `@Slf4j` 和 `@Repository` 注解

### 数据库设计

1. **UUID 主键**：所有表使用 32 位 varchar 类型的 UUID 作为主键
2. **关联表设计**：多对多关系使用独立关联表（如 `sc_course_qualification`、`sc_teaching_class_class`）
3. **JSONB 存储**：仅用于 `sc_schedule.weeks_json` 等特定场景
4. **冗余设计**：`sc_schedule` 表冗余存储课程、教师、教室信息以减少 JOIN

### 核心业务概念

**行政班级 vs 教学班**：
- **行政班级**（`sc_class`）：学生的固定组织单位（如"计科2101班"）
- **教学班**（`sc_teaching_class`）：临时的上课组织单位（如"高等数学-张老师-计科2101+2102"）

## API 规范

### 接口路径

- 基础路径前缀：`/v1/`（如 `/v1/home/dashboard`、`/v1/dify/chat`）
- 认证方式：Bearer Token（请求头 `Authorization: Bearer <token>`）

### 响应格式

项目配置了 Jackson `property-naming-strategy: SNAKE_CASE`，所有 JSON 响应字段使用下划线命名：

```json
{
  "output": "Success",
  "message": "操作成功",
  "data": {
    "user_info": { ... },
    "created_at": "2025-03-25T10:00:00"
  }
}
```

### 权限控制

使用 `@RequireRole` 注解控制接口权限：

```java
@RequireRole({UserType.ACADEMIC_ADMIN, UserType.SYSTEM_ADMIN})
public void someMethod() { ... }
```

用户类型定义在 `UserType` 枚举中：`STUDENT`、`TEACHER`、`ACADEMIC_ADMIN`、`SYSTEM_ADMIN`

## 数据库初始化

项目包含自动数据库初始化功能，配置前缀 `database.init`：

| 配置项 | 说明 |
|--------|------|
| `enabled` | 是否启用初始化 |
| `mode` | 模式：`FULL`/`DEMO`/`MINIMAL` |
| `drop-and-create` | 是否强制重建表（生产环境禁用） |

**初始化模式**：
- `FULL`：初始化所有数据（含排课、公告、活动日志、统计快照）
- `DEMO`：初始化到行政班级（不含排课）
- `MINIMAL`：只初始化基础数据

### 初始化器

位置：`config/database/*Initializer.java`

- `BaseDataInitializer` - 学期、学院、专业、教学楼
- `PersonnelInitializer` - 教师、学生、教务管理员
- `CourseResourceInitializer` - 课程、教室
- `TeachingClassInitializer` - 教学班
- `ScheduleInitializer` - 排课记录
- `ScheduleConflictInitializer` - 排课冲突检测
- `HomeDataInitializer` - 首页数据（公告、活动日志、统计快照）

## MCP 工具集成

项目集成了 Spring AI MCP Server，允许外部 AI 应用（如 Dify）通过 MCP 协议调用排课功能。

### 工具列表

**QueryTools**：
- `queryConflicts(semesterUuid, limit)` - 查询排课冲突记录
- `queryTeacherSchedule(teacherName, semesterUuid)` - 查询教师课表
- `queryClassroomOccupancy(classroomName, semesterUuid)` - 查询教室占用

**EduScheduleTool**：
- `checkAndPreview(teacher, time, room)` - 检测排课冲突

### 开发指南

添加新的 MCP 工具：
1. 在 `mcp/tools/` 目录下创建工具类
2. 使用 `@Service` 和 `@Tool` 注解标注方法
3. 在 `McpConfig.toolProvider()` 中注册工具对象

## Dify 智能调课助手

项目集成了 Dify 工作流对话型应用，提供智能调课助手功能。

### 配置

```yaml
dify:
  base-url: http://xxx/v1
  api-key: app-xxx
  connect-timeout: 5000
  read-timeout: 60000
```

### 接口

位置：`controller/DifyChatController.java`，路径：`/v1/dify/chat`

| 接口 | 说明 |
|------|------|
| `POST /message` | 发送消息 |
| `GET /message/stream` | 流式发送消息（SSE） |
| `GET /conversations` | 获取会话列表 |
| `GET /conversations/{id}/messages` | 获取会话消息 |
| `DELETE /conversations/{id}` | 删除会话 |
| `PUT /conversations/{id}/name` | 重命名会话 |

> 仅限系统管理员和教务管理员访问

## 遗传算法模块

项目使用遗传算法实现自动排课功能，位于 `algorithm/` 包。

### 核心组件

| 类 | 说明 |
|----|------|
| `GeneticAlgorithm` | 遗传算法主类，实现选择、交叉、变异 |
| `FitnessCalculator` | 适应度计算，评估排课方案质量 |
| `ConflictDetector` | 冲突检测，识别硬约束和软约束 |
| `TimeSlotGenerator` | 时间槽生成器 |

### 算法参数

- 种群大小：100
- 最大迭代：500 代
- 交叉概率：0.8
- 变异概率：0.2
- 精英保留：10 个

### 硬约束类型

教师时间冲突、教室时间冲突、班级时间冲突、教室容量约束、教室类型匹配、教师资格约束

## 依赖说明

项目使用 `com.x-lf.utility:general-utils` 库，已包含 MyBatis Plus、JSONB 类型处理器等功能，无需单独添加 MyBatis Plus 依赖。

## 开发规范

1. **DO 类必须包含中文注释**
2. **多对多关系使用独立关联表**，而非 JSONB 字段
3. **主键统一使用 String 类型存储 32 位 UUID**
4. **JSON 响应字段使用 snake_case**（由 Jackson 配置自动转换）
5. **DTO/DO 使用链式调用**（`@Accessors(chain = true)`）

---
> Source: [FLASHLACK1314/smart-schedule-coreV2](https://github.com/FLASHLACK1314/smart-schedule-coreV2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
