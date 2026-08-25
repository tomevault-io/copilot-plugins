## network-device-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.







## 项目概述

网络设备只读查询 MCP 接口 — 通过 MCP 协议为 AI 助手提供安全的网络设备只读查询能力。支持 Cisco、Huawei、H3C 交换机/路由器。

**核心原则**：安全第一。系统绝不允许执行任何配置变更命令，宁可拒绝合法查询也不放过危险命令。

## 当前状态

- **阶段**：架构设计完成初稿，尚未开始开发
- **PRD 文档**：`docs/product/PRD-网络设备只读查询接口.md`（v1.4）
- **架构文档**：`docs/architecture/整体架构设计.md`（v0.2 草稿）

## 系统职责分工

**在开启对话前，必须明确当前对话所属角色。**

1. **产品经理 (Product Manager)**

   - **核心职责**：负责业务逻辑的顶层设计，定义 MCP 工具的功能边界与用户交互场景。

   - **权限边界**：具备claude.md的读写权限 和 `/docs/product/` 目录下设计文档的完整读写权限。

   - **禁制规范**：严禁干涉具体代码实现及系统架构细节，不具备任何环境操作或指令执行权限。

2. **系统架构师 (System Architect)**

   - **核心职责**：基于产品需求进行技术建模与安全性设计。负责制定 API 规范、拦截逻辑及并发模型。

   - **权限边界**：具备 claude.md 的读写权限、`/docs/architecture/` 目录的读写权限，以及对 `/docs/product/` 的只读参考权限。

   - **禁制规范**：仅负责设计方案的产出，禁止直接编写或修改 `/src/` 目录下的业务逻辑代码。

3. **Python** 开发工程师 (Software Engineer)

   - **核心职责**：根据架构与需求文档进行代码落地。负责异步 SSH 逻辑封装、MCP 接口对接及单元测试编写。

   - **权限边界**：具备 claude.md 的读写权限、`/src/` 目录的全量操作权限，以及对 `/docs/` 全量文档的只读参考权限。

   - **禁制规范**：必须严格遵守既定设计方案，禁止擅自修改 `/docs/` 目录下的产品与架构定义。

4. **Debug与安全审计员 **  (Audit & QA)

   - **核心职责**：负责分析系统运行日志、捕获异常堆栈并进行代码层面的安全审计。

   - **权限边界**：具备docs/debug/的读写权限，具备对其他全量项目文件的【只读】权限和claude.md的读写权限，允许读取源码与日志进行诊断。

   - **禁制规范**：仅限产出诊断报告与修复建议，**严禁执行任何文件修改或系统配置变更**指令。

服务端保持轻量，不做自然语言理解和结果分析。

## 四个 MCP 工具

1. **单设备查询** — AI 传入 `command_id` + `params`，服务端从 YAML 查命令并执行
2. **批量查询** — 对多台设备并发执行同一条命令（并发上限 50）
3. **查看命令菜单** — 返回指定设备类型的可用命令列表（含 command_id、描述、参数定义）
4. **多命令单连接执行** — 一次 SSH 连接执行多条命令，适用于巡检场景，支持单台/多台设备

## 命令配置化

- AI **不能**自由输入命令字符串，只能从 `config/commands.yaml` 的封闭集合中选择 command_id
- 主命令锁死在配置文件中不可变，AI 只能补充参数
- 命令拼装由服务端完成，AI 不参与
- 扩展命令只需编辑 YAML，不改代码
- 命令列表参考 [ntc-templates](https://github.com/networktocode/ntc-templates) 项目

## 安全防护

采用封闭集合模型：

1. **command_id 校验** — 不在 YAML 配置中的命令直接拒绝（取代白名单）
2. **参数校验** — 类型检查 + 注入字符拦截（攻击面仅限参数值）
3. **兜底黑名单** — 拼装后命令仍检查危险关键词（防御纵深）

默认拒绝：校验在 SSH 连接建立之前完成。

## 凭据管理

- 服务端 `.env` 中维护一组默认凭据：`SSH_USERNAME` / `SSH_PASSWORD`
- 客户端可通过 `username` + `password` 参数覆盖默认凭据
- 优先级：客户端传入 > 服务端默认值
- 仅支持密码认证

## 部署与传输

- MCP 服务部署在**远程服务器**上，通过 **SSE（Server-Sent Events）** 传输
- 首期暂不做接入认证，直接 HTTP 通信
- 客户端通过 URL 连接 MCP 服务

## 目录结构

```
docs/product/         — PRD 等产品文档
docs/architecture/    — 技术架构文档
config/commands.yaml  — 命令配置文件（封闭集合）
src/                  — 源代码（待开发）
logs/audit.log        — 审计日志
.env                  — 默认凭据（不入版本控制）
.env.example          — 凭据格式示例
```

## 技术约束

- 语言：Python
- 协议：MCP（SSE 传输，远程部署）
- SSH 库：netmiko（设备类型：cisco_ios / huawei / hp_comware）
- SSH 仅密码认证，单设备超时 30 秒
- 批量并发上限 50 台（asyncio.Semaphore）
- 命令配置在 `config/commands.yaml` 中维护，扩展不改代码
- 审计日志写入 `logs/audit.log`，每行一条 JSON

## 待确认项

- 批量查询是否允许混合设备类型
- 不同厂商的参数拼接格式差异处理

---
> Source: [gj-hat/network-device-mcp](https://github.com/gj-hat/network-device-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
