## codespirit

> description: CodeSpirit 包管理规范 - 集中式包管理、版本统一、冗余引用处理

﻿---
description: CodeSpirit 包管理规范 - 集中式包管理、版本统一、冗余引用处理
globs: ["*.csproj", "Directory.Packages.props"]
alwaysApply: false
---

# CodeSpirit 包管理规范

## 集中式包管理（Central Package Management）

项目已启用集中式包管理，所有包版本在 `Directory.Packages.props` 文件中统一管理。

### 工作原理

1. **版本定义**: 在 `Directory.Packages.props` 中使用 `<PackageVersion>` 定义所有包版本
2. **项目引用**: 在项目文件中使用 `<PackageReference Include="PackageName" />`，**不指定 Version**
3. **自动应用**: MSBuild 自动从 `Directory.Packages.props` 获取版本

### 添加新包

**步骤 1**: 在 `Directory.Packages.props` 中添加包版本

```xml
<ItemGroup>
  <!-- 按功能分类添加 -->
  <PackageVersion Include="NewPackage" Version="1.0.0" />
</ItemGroup>
```

**步骤 2**: 在项目文件中添加包引用（不指定版本）

```xml
<ItemGroup>
  <PackageReference Include="NewPackage" />
</ItemGroup>
```

### 版本覆盖（特殊情况）

仅在必要时使用 `VersionOverride`，并添加注释说明原因：

```xml
<!-- 注意：Aspire.Microsoft.EntityFrameworkCore.SqlServer 13.0.2 需要 EF Core 10.0.0 -->
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" VersionOverride="10.0.0" />
```

**使用 VersionOverride 的场景**：
- Aspire EF Core 集成包需要特定版本的 EF Core
- 特定项目需要不同版本的包（应尽量避免）

## 包版本策略

### 版本更新原则

1. **补丁版本更新**（推荐）
   - 如：`9.0.9` → `9.0.10`
   - 通常只包含 bug 修复，风险较低

2. **小版本更新**（谨慎）
   - 如：`9.0.x` → `9.1.x`
   - 可能包含新功能和 API 变化，需要测试

3. **大版本更新**（充分测试）
   - 如：`9.x` → `10.x`
   - 通常包含破坏性更改，需要全面测试

### 当前统一版本

主要包的统一版本（详见 `Directory.Packages.props`）：

- **Aspire 包**: `13.0.2`（部分使用 `13.1.0`）
- **Entity Framework Core**: `9.0.9`（使用 Aspire EF Core 集成的项目使用 `10.0.0`）
- **Microsoft Extensions**: `10.0.0`
- **OpenTelemetry**: `1.11.1`
- **StackExchange.Redis**: `2.9.32`
- **Newtonsoft.Json**: `13.0.3`
- **AutoMapper**: `13.0.1`
- **FluentValidation**: `11.11.0`

## 传递依赖处理

### 识别传递依赖

使用以下命令查看传递依赖：

```bash
dotnet list package --include-transitive
```

### 常见传递依赖

以下包通过项目引用传递可用，通常不需要显式引用：

#### 通过 `CodeSpirit.Shared` 传递

- `Newtonsoft.Json`
- `Microsoft.EntityFrameworkCore.SqlServer`
- `Microsoft.EntityFrameworkCore`
- `Pomelo.EntityFrameworkCore.MySql`
- `LinqKit.Core`
- `RabbitMQ.Client`
- `Aspire.RabbitMQ.Client`
- `StackExchange.Redis`
- `ClosedXML`
- `Polly`
- `Scrutor`
- `System.Linq.Dynamic.Core`

#### 通过 `CodeSpirit.ServiceDefaults` 传递

- `Aspire.Seq`
- `Aspire.StackExchange.Redis.DistributedCaching`
- `Microsoft.Extensions.Diagnostics.HealthChecks`
- `Microsoft.Extensions.Http.Resilience`
- `Microsoft.Extensions.ServiceDiscovery`
- `OpenTelemetry.*` 相关包

#### 通过 `FrameworkReference` 提供（.NET 10）

- `Microsoft.AspNetCore.Http.Abstractions`
- `Microsoft.AspNetCore.Mvc.Core`
- 大部分 `Microsoft.AspNetCore.*` 包

### 何时需要显式引用

以下情况需要显式引用传递依赖：

1. **需要特定版本**
   - 使用 `VersionOverride` 覆盖传递的版本

2. **需要特定 API**
   - 如果项目直接使用包的公共 API，建议显式引用

3. **避免版本冲突**
   - 当传递依赖版本与项目需求不匹配时

4. **工具包**
   - 如 `Microsoft.EntityFrameworkCore.Design`、`Microsoft.EntityFrameworkCore.Tools`
   - 这些包需要显式引用并设置 `PrivateAssets`

## 包分类组织

`Directory.Packages.props` 中的包按以下分类组织：

1. **Aspire 包** - Aspire 框架相关
2. **Entity Framework Core** - EF Core 相关
3. **Microsoft Extensions** - Microsoft.Extensions.* 包
4. **Microsoft ASP.NET Core** - Microsoft.AspNetCore.* 包
5. **OpenTelemetry** - 监控和追踪
6. **缓存和消息队列** - Redis、RabbitMQ
7. **工具和库** - 第三方工具包
8. **FluentValidation** - 验证框架
9. **其他工具包** - 各种工具库
10. **云服务 SDK** - 云服务集成
11. **开发工具** - 开发时使用的工具
12. **测试框架包** - 测试相关包

## 最佳实践

### 添加新包

1. ✅ 先检查 `Directory.Packages.props` 是否已存在该包
2. ✅ 如果存在，直接使用现有版本
3. ✅ 如果不存在，添加到 `Directory.Packages.props` 的相应分类
4. ✅ 在项目文件中只引用包名，不指定版本
5. ✅ 添加注释说明包的用途（如需要）

### 更新包版本

1. ✅ 在 `Directory.Packages.props` 中更新版本
2. ✅ 运行 `dotnet restore` 验证兼容性
3. ✅ 运行 `dotnet build` 确保构建成功
4. ✅ 运行 `aspire run` 验证运行时功能

### 移除包引用

1. ✅ 检查是否通过传递依赖可用
2. ✅ 使用 `dotnet list package --include-transitive` 确认
3. ✅ 移除后立即构建验证
4. ✅ 如果编译失败，说明需要显式引用

### 版本冲突处理

1. ✅ 优先统一版本到最新补丁版本
2. ✅ 必要时使用 `VersionOverride`（添加注释）
3. ✅ 考虑升级相关包到兼容版本
4. ✅ 记录版本冲突原因和解决方案

## 工具和命令

### 检查过时的包

```bash
dotnet list package --outdated
```

### 查看传递依赖

```bash
dotnet list package --include-transitive
```

### 查看特定项目的包

```bash
dotnet list package --project Src/ApiServices/CodeSpirit.ExamApi/CodeSpirit.ExamApi.csproj
```

### 还原包

```bash
dotnet restore
```

### 构建验证

```bash
dotnet build
```

## 常见问题

### Q: 如何知道一个包是否通过传递依赖可用？

A: 使用 `dotnet list package --include-transitive` 查看。如果包出现在传递依赖列表中，通常不需要显式引用。

### Q: 为什么有些包显示"可能不需要"的警告？

A: NuGet 的静态分析可能误判。如果项目确实使用了包的 API，应保留引用。建议通过代码搜索验证实际使用情况。

### Q: 如何更新所有包到最新版本？

A: 
1. 使用 `dotnet outdated-tool` 分析
2. 在 `Directory.Packages.props` 中逐个更新版本
3. 每次更新后构建验证
4. 优先更新补丁版本，谨慎更新小版本和大版本

### Q: 版本冲突如何解决？

A:
1. 优先统一到最新补丁版本
2. 必要时使用 `VersionOverride`（添加注释说明）
3. 考虑升级相关包到兼容版本
4. 记录冲突原因和解决方案

## 参考文档

- [NuGet Central Package Management](https://learn.microsoft.com/nuget/consume-packages/central-package-management)
- [Directory.Packages.props 文件](../../Directory.Packages.props)
- [包管理优化报告](../../Docs/PackageOptimizationReport.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xin-lai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/xin-lai)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
