## chat-server

> 基于 C++17 + Muduo + MySQL + Redis 的组织内部即时通讯平台后端，配套 Qt6 图形客户端。面向校招求职项目展示。

# ChatServer - 组织内部即时通讯平台

## 项目定位
基于 C++17 + Muduo + MySQL + Redis 的组织内部即时通讯平台后端，配套 Qt6 图形客户端。面向校招求职项目展示。

## 当前状态
- 功能：注册登录、单聊群聊、离线消息、历史查询、Redis 跨服转发
- 协议：JSON + \0 分帧（Proto 文件已定义，双协议兼容待完成）
- 构建：CMake，Ubuntu 24.04+
- 客户端：Qt6 Widgets 图形客户端 + 命令行客户端

## 进行中的优化计划

### 主线：性能优化闭环（目标：面试10问全覆盖）

10 项任务，预计 14 天：

1. 改造压测脚本建立性能基线
2. MySQL 连接池
3. shared_mutex 替换 mutex
4. 群聊在线状态批量查询
5. Valgrind 内存泄漏排查 + RAII 修复
6. 密码哈希 bcrypt/SHA-256
7. 故障模拟 + 降级逻辑（MySQL/Redis 宕机 + 限流）
8. Docker Compose 一键部署
9. Protobuf + JSON 双协议共存
10. 压测数据整理 + 架构图绘制

### 面试叙事主线
> 从单机 100 并发优化到 10000 并发，P99 延迟从 500ms 降到 50ms。每项优化有数据对比。

## 技术栈
- C++17, CMake
- Muduo（多 Reactor TCP 网络库）
- MySQL 8.x + libmysqlclient（持久化）
- Redis 6.x/7.x + hiredis（跨服务器 Pub/Sub）
- Protobuf（待完成协议迁移）
- nlohmann/json（当前线缆协议）
- Qt6 Widgets + QTcpSocket（图形客户端）

## 目录结构
```
├── CMakeLists.txt
├── conf/chatserver.json    # 服务端配置
├── include/server/         # 头文件
├── src/server/             # 服务端源码（main, chatservice, db, model, redis）
├── proto/chat.proto        # Protobuf 定义
├── qt_client/              # Qt 图形客户端（独立 CMake 项目）
├── script/                 # 构建/压测脚本
├── sql/                    # 数据库初始化 SQL
└── docs/                   # 文档
```

---
> Source: [su416/Chat_Server](https://github.com/su416/Chat_Server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
