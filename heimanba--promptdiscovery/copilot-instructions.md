## promptdiscovery

> - [src/index.ts](mdc:src/index.ts): 项目入口文件，包含MCP服务器配置和工具注册


# 项目结构指南

## 主要文件

- [src/index.ts](mdc:src/index.ts): 项目入口文件，包含MCP服务器配置和工具注册
- [src/services/NacosService.ts](mdc:src/services/NacosService.ts): Nacos服务实现，处理配置管理和提示词操作

## 目录结构

- `src/`: 源代码目录
  - `services/`: 服务组件目录，包含与外部系统交互的服务实现
- `.cache/`: 本地缓存目录，存储Nacos配置的本地副本

## 关键组件

### MCP服务器

MCP服务器是项目的核心，负责注册和处理工具请求。服务器使用`@modelcontextprotocol/sdk`库实现，通过标准输入/输出与客户端通信。

### Nacos服务

NacosService负责与Nacos配置中心交互，提供以下功能：

- 获取和管理提示词配置
- 本地缓存配置数据
- 监听配置变更
- 提供提示词查询和搜索功能

## 配置管理

项目使用Nacos作为配置中心，配置数据以JSON格式存储。当Nacos服务不可用时，系统会使用本地缓存的配置数据，确保服务的可用性。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/heimanba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
