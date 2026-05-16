## dw-admin

> This file provides guidance to Qoder (qoder.com) when working with code in this repository.

# AGENTS.md

This file provides guidance to Qoder (qoder.com) when working with code in this repository.

## 项目概述

DW Admin 是一个基于 Spring Boot 3.5.6 的后台管理系统,使用 Java 21 开发。项目采用模块化设计,支持用户认证授权(JWT + RBAC)、双重云存储(阿里云OSS/腾讯云COS)、接口限流、权限管理等功能。

## 常用命令

### 构建和运行
```bash
# 清理项目
mvn clean

# 编译项目
mvn compile

# 运行测试
mvn test

# 运行单个测试类
mvn test -Dtest=UserServiceTest

# 运行单个测试方法
mvn test -Dtest=UserServiceTest#testLogin

# 打包项目(跳过测试)
mvn install -Dmaven.test.skip=true

# 启动项目(开发模式)
mvn spring-boot:run

# 运行打包后的JAR文件
java -jar target/dw-admin-1.0.0.jar

# 停止项目
./stop.sh
```

### Docker 相关
```bash
# 构建镜像
docker build -t dw-admin:1.0.0 .

# 运行容器
docker run -d -p 8020:8020 -v ./logs:/app/logs -e TZ=Asia/Shanghai -e SPRING_PROFILES_ACTIVE=dev --name dw-admin dw-admin:1.0.0
```

## 核心架构

### 包结构说明

```
com.dw.admin/
├── DwAdminApp.java          # 启动类,启用了异步和定时任务(@EnableAsync, @EnableScheduling)
├── common/                  # 通用模块
│   ├── constant/           # 常量定义
│   ├── entity/             # 通用实体(Response<T>, PageInfo等)
│   ├── enums/              # 枚举类
│   ├── exception/          # 统一异常处理(GlobalExceptionHandler)
│   └── utils/              # 工具类
├── components/              # 业务组件(核心组件实现)
│   ├── auth/               # 认证组件: JWT生成验证、Token缓存(DB/Redis双实现)
│   ├── cache/              # 缓存组件: 统一缓存抽象
│   ├── generator/          # MyBatis-Plus代码生成器
│   ├── limiter/            # 限流组件: 基于AOP的接口限流
│   ├── log/                # 日志组件: 操作日志记录(AOP)
│   ├── permission/         # 权限组件: 基于注解的权限校验(AOP)
│   ├── redis/              # Redis配置和分布式锁
│   └── storage/            # 存储服务: 工厂模式实现多云存储(OSS/COS/本地)
├── config/                 # Spring配置类
├── controller/             # 控制器层(只处理HTTP请求响应)
├── dao/                    # Mapper接口(只处理单表操作)
├── model/                  # 数据模型
│   ├── entity/            # 数据库实体(对应表结构)
│   ├── param/             # 请求参数(Controller入参)
│   └── vo/                # 视图对象(Controller返回值)
└── service/               # 业务逻辑层
    └── impl/              # 服务实现
```

### 关键设计模式

1. **认证授权架构** (`components/auth/`)
   - JWT无状态认证: `JwtUtils` 生成和验证token
   - Token缓存双实现: `TokenDBCacheHelper`(数据库)、`TokenRedisCacheHelper`(Redis),通过配置 `dwa.auth.cache-type` 切换
   - AOP拦截: `AuthAspect` 拦截 `@Auth` 注解的接口进行认证
   - 用户上下文: `UserContextHolder` 使用 TransmittableThreadLocal 管理当前用户信息

2. **存储服务架构** (`components/storage/`)
   - 工厂模式: 根据 `dwa.storage.provider` 配置动态选择存储实现
   - 统一接口: `FileStorageService` 定义标准操作(upload/download/delete/preview)
   - 三种实现:
     - `oss/OssService` - 阿里云对象存储
     - `cos/CosService` - 腾讯云对象存储
     - `local/LocalService` - 本地文件存储
   - 在 `StorageProperties` 中管理所有存储配置

3. **权限管理** (`components/permission/`)
   - `@Permission` 注解标记需要权限的接口
   - `PermissionAspect` 通过AOP拦截并校验权限
   - 支持权限缓存优化性能

4. **限流机制** (`components/limiter/`)
   - `@Limiter` 注解配置限流规则
   - `LimiterAspect` 实现限流拦截

### 核心技术栈

- **Java 21** + **Spring Boot 3.5.6**
- **MyBatis-Plus 3.5.14**: ORM框架,Mapper位于 `com.dw.admin.dao`
- **MySQL 8.0+**: 数据库,表前缀 `dwa_`
- **Redis**: 缓存和分布式锁
- **JWT (jjwt 0.12.6)**: 身份认证
- **Lombok**: 减少样板代码
- **FastJSON2**: JSON序列化
- **Hutool + Guava**: 工具库

### 配置管理

- 主配置: `application.yml` (默认 profile: dev)
- 环境配置: `application-dev.yml`, `application-prod.yml`
- 关键配置项:
  - `dwa.auth.cache-type`: Token缓存类型 (DB/REDIS)
  - `dwa.auth.secret`: JWT密钥
  - `dwa.storage.provider`: 存储提供商 (aliyun-oss/tencent-cos/local)
  - `dwa.redis.url`: Redis地址
- 日志配置: `logback/logback-spring.xml`
- 不要读取 `application-prod.yml`

### 开发规范

1. **分层规范**:
   - Controller: 只处理HTTP请求响应,不写业务逻辑
   - Service: 业务逻辑层,可调用多个Mapper
   - Dao/Mapper: 只处理单表数据访问

2. **命名规范**:
   - 数据库表: `dwa_` 前缀
   - 实体类: 对应表名(如 DwaUser)
   - Mapper: 表名 + Mapper (如 UserMapper)
   - Service: 业务名 + Service (如 UserService)
   - 配置类: `*Properties` 或 `*Config` 后缀

3. **注解使用**:
   - `@Auth`: 需要登录的接口
   - `@Permission`: 需要特定权限的接口
   - `@Limiter`: 需要限流的接口
   - `@Log`: 需要记录操作日志的接口

4. **统一响应格式**: 所有接口返回 `Response<T>` 类型,由 `GlobalExceptionHandler` 统一处理异常

### 数据库初始化

```bash
# 执行SQL脚本
mysql -u root -p < docs/sql/init_ddl.sql
```

### 重要注意事项

1. **配置敏感信息**: 生产环境的数据库密码、Redis密码、云存储密钥等需要通过环境变量或配置中心管理
2. **云存储切换**: 修改 `dwa.storage.provider` 后需要重启应用
3. **Token缓存切换**: 修改 `dwa.auth.cache-type` 后需要重启应用,且两种方式的token不互通
4. **文件上传限制**: 默认最大文件 5MB,最大请求 10MB,可在配置中调整
5. **日志级别**: Mapper层默认debug级别,生产环境建议调整为info

---
> Source: [dawei1898/dw-admin](https://github.com/dawei1898/dw-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
