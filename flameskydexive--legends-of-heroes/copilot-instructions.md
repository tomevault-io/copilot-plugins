## legends-of-heroes

> 服务发现包，负责服务注册、注销、查询、变更通知，以及进程级 `ServiceDiscoveryAgent` 到当前服务发现主节点的转发。

# cn.etetet.servicediscovery

## 概述

服务发现包，负责服务注册、注销、查询、变更通知，以及进程级 `ServiceDiscoveryAgent` 到当前服务发现主节点的转发。

## 详细文档

- **代码规范**：请查看 `/et-code` skill
- **异步规范**：请查看 `/et-async` skill
- **测试规范**：请查看 `/et-test-write` skill

## 核心目录

| 路径 | 说明 |
|------|------|
| `Scripts/Model/Server` | 服务发现组件、运行状态、启动配置 singleton |
| `Scripts/Hotfix/Server` | 初始化、注册、查询、心跳与主租约逻辑 |

---
> Source: [FlameskyDexive/Legends-Of-Heroes](https://github.com/FlameskyDexive/Legends-Of-Heroes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->
