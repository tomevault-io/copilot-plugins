## vpm

> - 此项目只使用一个服务器文件server.js

# 项目规范配置

## 基础配置
- 项目后端使用nodejs
- 使用mcp服务工具
- 此项目只使用一个服务器文件server.js
- 使用pm2管理服务

## 数据库配置
- MongoD数据库 (本地)
- 连接信息：
  - username: vpm
  - password: Dusk0616
  - database: vpm

  -主要使用PG数据库

## 前端规范
- 使用统一的sidebar.vue左侧栏
- 前端使用VUE最新版本
- 不使用echo命令
- 不使用php
- 不使用&&命令
- 不使用&命令

## 文件管理
- 项目信息存储于README.md中
- README.md随项目更新和修正
- 测试功能时可以新建文件
- 测试完成后自动删除测试文件及功能
- 防止垃圾文件累积
- 网页子页面放到子目录/pages/中
- 网页子页面文件名不能用index.html命名
## 数据库管理
- 不允许未经指令创建数据库
- 防止覆盖已存在的数据库

## 工具使用必
- 优先使用MCP服务工具
- 可以直接操作文件
- 可以执行命令

## 命令执行规则
- 禁止使用&&连接多个命令
- 禁止使用&后台执行命令
- 禁止使用echo命令
- MongoDB安装必须按以下步骤分步执行：
  1. 导入MongoDB GPG密钥
  2. 添加MongoDB源
  3. 更新软件包列表
  4. 安装MongoDB
  5. 启动MongoDB服务
- 每个步骤使用单独的命令执行
- 命令执行前必须确认上一步骤完成

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VK0118) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
