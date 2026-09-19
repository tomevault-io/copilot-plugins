## zwift-offline

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个Zwift离线服务器的Python实现，作为官方Zwift服务器的部分实现，允许用户在离线状态下使用Zwift。项目支持单用户和多用户模式，包含虚拟骑行、幽灵竞赛、机器人等功能。

## 核心架构

### 主要文件结构

- `standalone.py` - 独立服务器入口，处理基本的网络服务和UDP通信
- `zwift_offline.py` - 核心Flask web应用，包含所有主要API端点和业务逻辑 
- `profiles.py` - 用户配置文件管理
- `discord_bot.py` - Discord集成机器人
- `tokens.py` - JWT token处理和验证
- `online_sync.py` - 在线同步功能

### 目录结构

- `protobuf/` - Protocol Buffers定义文件和生成的Python代码
- `scripts/` - 各种辅助脚本（获取游戏信息、Strava认证等）
- `storage/` - 用户数据存储目录
- `ssl/` - SSL证书文件
- `cdn/` - CDN代理和游戏资源
- `data/` - 游戏相关数据文件

## 开发命令

### 启动服务器

```bash
# 开发模式（从源码运行）
python standalone.py

# 或在Linux/macOS上需要sudo权限（绑定到80和443端口）
sudo python standalone.py
```

### 构建Protocol Buffers

```bash
# 在protobuf目录中
cd protobuf
make
```

### Docker部署

```bash
# 使用docker-compose
docker-compose up -d

# 或使用Docker直接运行
docker run -p 443:443 -p 80:80 -p 3024:3024/udp -p 3025:3025 -p 53:53/udp zoffline/zoffline
```

### 依赖管理

```bash
# 安装依赖
pip install -r requirements.txt

# 可选依赖（用于特定功能）
pip install garth  # Garmin Connect集成
pip install discord.py  # Discord机器人
```

## 技术架构要点

### 网络通信
- HTTP/HTTPS服务器：处理Web API和用户界面
- UDP服务器：处理游戏内的实时通信（端口3024）
- TCP服务器：处理游戏连接（端口3025）
- DNS服务：可选的假DNS服务器（端口53）

### 数据存储
- 使用SQLite数据库（通过Flask-SQLAlchemy）
- 二进制文件存储用户配置文件和活动数据
- Protocol Buffers序列化/反序列化游戏数据

### 身份验证
- JWT token验证
- 支持多用户注册和登录
- 密码哈希存储

### 核心功能模块
- 用户配置文件管理和自定义
- 虚拟世界和路线管理
- 活动跟踪和统计
- 幽灵竞赛和机器人系统
- 外部服务集成（Strava, Garmin Connect, Intervals.icu）

### 配置文件系统
项目通过在`storage`目录中的各种文本文件来控制功能：
- `multiplayer.txt` - 启用多用户模式
- `enable_ghosts.txt` - 启用幽灵功能
- `enable_bots.txt` - 启用机器人
- `server-ip.txt` - 配置服务器IP地址
- `discord.cfg` - Discord机器人配置

## 开发注意事项

- 项目主要使用Python 3，依赖Flask web框架
- 需要理解Protocol Buffers协议来处理游戏数据
- 网络编程涉及多线程和异步处理（使用gevent）
- SSL/TLS证书管理对于HTTPS通信至关重要
- 游戏客户端需要特殊的DNS重定向配置才能连接到离线服务器

---
> Source: [wmzhai/zwift-offline](https://github.com/wmzhai/zwift-offline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
