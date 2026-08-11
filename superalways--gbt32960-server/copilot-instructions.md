## gbt32960-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

GBT32960-server是中国国标新能源汽车远程监控协议网关的实现，支持GBT32960-2016和GBT32960-2025两个版本，基于自定义Netty框架构建。

## 开发命令

### 构建与运行
```bash
# 编译整个项目
mvn clean compile

# 打包应用
mvn clean package

# 运行服务器
cd GBT32960-server && mvn spring-boot:run

# 运行测试（默认跳过）
mvn test -DskipTests=false
```

### 应用端口
- **HTTP服务**: 8100 (REST API, 文档)
- **TCP协议服务**: 7100 (GBT32960协议)

## 架构概述

### 多模块Maven结构
- **commons** - 共享工具类、缓存、JSON序列化
- **GBT32960-netmc** - 基于Netty的自定义MVC框架，处理TCP协议
- **GBT32960-protostar** - 注解驱动的字节流序列化框架
- **GBT32960-protocol** - GBT32960消息定义和编解码器
- **GBT32960-server** - 主Spring Boot应用，包含REST API

### 核心技术栈
- **Spring Boot 3.3.10** (Java 17)
- **Netty 4.1.119** TCP通信
- **MyBatis Plus 3.5.10** 配合MariaDB/H2
- **Project Reactor** 响应式编程
- **Caffeine** 缓存

## 核心架构模式

### 协议消息处理
协议消息使用注解驱动开发：
- `@Message` - 定义协议消息结构
- `@Field` - 映射字段到字节位置
- `@Endpoint` - 标记协议处理器类
- `@Mapping` - 路由消息到处理方法

### 消息流程
1. TCP消息通过Netty在7100端口接收
2. 使用`GBTMessageDecoder`解码
3. 通过`@Mapping`注解路由到`@Endpoint`处理器
4. 在处理方法中执行业务逻辑
5. 通过`GBTMessageEncoder`编码响应

### 会话管理
自定义会话管理处理：
- 设备注册和认证
- 连接生命周期管理和心跳（#25 #25消息）
- 支持2016版本（#23 #23）和2025版本（#24 #24）协议

## 开发指南

### 框架使用
- 使用netmc框架处理所有TCP协议
- 使用protostar注解进行消息序列化
- 遵循JT808架构模式保持一致性
- 支持GBT32960双版本兼容

### 关键类
- `GBTMessage` - 所有协议消息的基类
- `Session` - 管理设备连接和状态
- `SchemaManager` - 处理多版本协议架构
- `MessageManager` - 协调请求/响应流程

### 性能考虑
- 使用Netty PooledByteBufAllocator进行内存管理
- 通过`@AsyncBatch`注解实现响应式编程
- TCP长连接配合心跳监控
- 通过protostar框架实现零拷贝序列化

## 测试与调试

### 可用测试工具
- `Elucidator.java` - 协议调试的十六进制转储分析器
- `StressTest.java` - 性能验证
- 外部工具`发包工具.exe`用于端到端协议测试

### 关键URL
- API文档: http://127.0.0.1:8100/doc.html
- 实时监控: http://127.0.0.1:8100/ws.html
- 协议连接: TCP 127.0.0.1:7100

### 协议版本
在application properties中配置协议版本：
- 2016版本使用帧标记 #23 #23
- 2025版本使用帧标记 #24 #24

---
> Source: [SuperAlways/gbt32960-server](https://github.com/SuperAlways/gbt32960-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
