## onenet-csharp

> 此文件为 Claude Code (claude.ai/code) 在本仓库中工作时提供指导。

# CLAUDE.md

此文件为 Claude Code (claude.ai/code) 在本仓库中工作时提供指导。

## 项目概述

中国移动 **OneNET 物联网平台** API (`iot-api.heclouds.com`) 的 C# SDK。以 NuGet 包形式发布，目标框架为 **netstandard2.0**，具备广泛的兼容性。

## 构建命令

```bash
# 还原依赖
dotnet restore OneNet-CSharp/OneNet-CSharp.csproj

# 构建
dotnet build OneNet-CSharp/OneNet-CSharp.csproj --configuration Release

# 打包为 NuGet 包
dotnet pack OneNet-CSharp/OneNet-CSharp.csproj --configuration Release --output ./nupkgs
```

SDK 需要 .NET SDK 8.0+（在 `global.json` 中配置了 `rollForward: latestMinor`）。CI 使用 .NET 10.0.x 进行构建。

## 架构

单项目解决方案（`OneNet-CSharp/`），目前没有测试项目。

### 入口

`OneNetClient` — 通过 `productId` 和 `accessKey` 实例化，以属性方式暴露各领域子客户端（如 `client.Device`）。

### 鉴权

`OneNetAuthHelper` 为 OneNET API 生成 HMAC-SHA1 签名 Token，包含两个方法：
- `GenerateToken()` — 产品级鉴权（版本 `2022-05-01`）
- `GenerateClientToken()` — 设备级鉴权（版本 `2018-10-31`）

Token 在每次 API 调用时作为 `Authorization` 请求头传入。

### 模块结构

每个 API 领域遵循以下模式：
```
OneNet/
├── {Domain}/
│   ├── {Domain}Client.cs      # HTTP 调用逻辑
│   ├── Request/               # 请求模型 (JsonProperty 映射 snake_case)
│   └── Response/              # 响应模型
├── Response/
│   └── OneNetCommResponse.cs  # 统一响应信封 (code, msg, request_id, data)
├── OneNetClient.cs            # SDK 入口
└── OneNetAuthHelper.cs        # 鉴权 Token 生成
```

目前仅实现了 **Device（设备）** 领域（列表查询、详情查询、创建设备）。

### 关键约定

- JSON 序列化统一使用 **Newtonsoft.Json**，通过 `[JsonProperty("snake_case")]` 特性映射字段
- API 响应统一包装在 `OneNetCommResponse<T>` 中（包含 code、msg、request_id、data）
- API 基础地址：`https://iot-api.heclouds.com`
- 根命名空间：`OneNet_CSharp`

## 代码风格

- 代码注释必须使用中文
- 公开模型属性需添加 XML 文档注释，描述 OneNET API 字段的含义

## CI/CD

GitHub Actions 工作流（`.github/workflows/publish.yml`）— 手动触发（`workflow_dispatch`），执行构建、打包并推送到 NuGet。支持可选的版本后缀（如 alpha、beta）。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bbhxwl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bbhxwl)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
