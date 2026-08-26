## huasunsecproj

> Golang CLI 网络资产测绘工具：输入 IP 网段与端口范围，输出该范围内 mDNS 协议资产信息。

# AGENTS.md

## 项目概述

Golang CLI 网络资产测绘工具：输入 IP 网段与端口范围，输出该范围内 mDNS 协议资产信息。

- 功能规格唯一来源：`需求.md`（务必以中文阅读并遵循其中的示例输出格式）
- 当前仓库尚无代码，`go.mod`、构建配置等尚未创建；首个任务为从零搭建项目

## 核心需求（来自 需求.md）

- CLI 输入：IP 网段、端口范围
- 输出：mDNS 资产信息，至少包含 `ip`、`port`、`host`、`banner`
- **banner 深度识别**是验收重点：输出深度必须达到 `需求.md` 中示例的层级，包括：
  - 服务块（如 `9/tcp workstation`、`5000/tcp http`、`445/tcp smb`、`5000/tcp qdiscover`、`548/tcp afpovertcp`），每块含 `Name`、`IPv4`、`IPv6`、`Hostname`、`TTL`
  - 设备信息（`device-info` 块的 `model` 等字段）
  - qdiscover 深度字段：`accessType`、`accessPort`、`model`、`displayModel`、`fwVer`、`fwBuildNum`
  - `answers` 部分：`PTR` 记录（如 `_workstation._tcp.local`）

## 约束

- 使用 Golang 开发，注意 mDNS 使用 UDP 组播（224.0.0.251:5353）实现时的包选择
- 开发完成后需提交至公开 GitHub 仓库

---
> Source: [zklmiao/HuaSunSecProj](https://github.com/zklmiao/HuaSunSecProj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
