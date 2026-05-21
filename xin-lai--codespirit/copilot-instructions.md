## database

> CodeSpirit 数据库与 EF Core 迁移规范 - 多数据库支持、DbContext 设计、迁移命令


# 数据库与 EF Core 迁移规范

## 📋 目录

1. [多数据库架构](#多数据库架构)
2. [DbContext 设计模式](#dbcontext-设计模式)
3. [迁移命令规范](#迁移命令规范)
4. [实体 ID 配置](#实体-id-配置)
5. [实体配置规范](#实体配置规范)

---

## 多数据库架构

CodeSpirit 支持多数据库（SQL Server / MySQL），每个 API 服务需要定义：

```
Data/
├── {Service}DbContext.cs              # 基础 DbContext（运行时使用）
├── SqlServer{Service}DbContext.cs     # SQL Server 专用 DbContext
├── SqlServer{Service}DbContextFactory.cs  # SQL Server 设计时工厂
├── MySql{Service}DbContext.cs         # MySQL 专用 DbContext
├── MySql{Service}DbContextFactory.cs  # MySQL 设计时工厂
├── DatabaseSpecificConfigurations.cs  # 数据库特定配置
├── Configurations/                    # 实体配置
│   └── {Entity}Configuration.cs
└── Migrations/
    ├── SqlServer/                     # SQL Server 迁移
    │   └── {timestamp}_{MigrationName}.cs
    └── MySql/                         # MySQL 迁移
        └── {timestamp}_{MigrationName}.cs
```

---

## DbContext 设计模式

### 基础 DbContext

运行时使用的 DbContext，继承自 `MultiDatabaseDbContextBase`：

```csharp
public class MallDbContext : MultiDatabaseDbContextBase
{
    public MallDbContext(
        DbContextOptions options,
        IServiceProvider serviceProvider,
        ICurrentUser currentUser,
        IHttpContextAccessor httpContextAccessor) 
        : base(options, serviceProvider, currentUser, httpContextAccessor)
    {
    }

    // DbSet 属性
    public DbSet<Product> Products => Set<Product>();
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(MallDbContext).Assembly);
    }
}
```

### 数据库特定 DbContext

**SQL Server 版本**：

```csharp
/// <summary>
/// SQL Server 特定的数据库上下文（用于迁移）
/// </summary>
public class SqlServerMallDbContext : MallDbContext
{
    public SqlServerMallDbContext(
        DbContextOptions<SqlServerMallDbContext> options,
        IServiceProvider serviceProvider,
        ICurrentUser currentUser,
        IHttpContextAccessor httpContextAccessor) 
        : base((DbContextOptions)options, serviceProvider, currentUser, httpContextAccessor)
    {
    }

    protected override void ApplyDatabaseSpecificConfigurations(ModelBuilder modelBuilder)
    {
        DatabaseSpecificConfigurations.ApplySqlServerConfigurations(modelBuilder);
    }
}
```

**MySQL 版本**：

```csharp
/// <summary>
/// MySQL 特定的数据库上下文（用于迁移）
/// </summary>
public class MySqlMallDbContext : MallDbContext
{
    public MySqlMallDbContext(
        DbContextOptions<MySqlMallDbContext> options,
        IServiceProvider serviceProvider,
        ICurrentUser currentUser,
        IHttpContextAccessor httpContextAccessor) 
        : base((DbContextOptions)options, serviceProvider, currentUser, httpContextAccessor)
    {
    }

    protected override void ApplyDatabaseSpecificConfigurations(ModelBuilder modelBuilder)
    {
        DatabaseSpecificConfigurations.ApplyMySqlConfigurations(modelBuilder);
    }
}
```

### 设计时工厂

用于 `dotnet ef` 命令的设计时 DbContext 创建：

**SQL Server**:

```csharp
public class SqlServerMallDbContextFactory : IDesignTimeDbContextFactory<SqlServerMallDbContext>
{
    public SqlServerMallDbContext CreateDbContext(string[] args)
    {
        var optionsBuilder = new DbContextOptionsBuilder<SqlServerMallDbContext>();
        
        optionsBuilder.UseSqlServer(
            "Server=localhost;Database=Mall;User Id=sa;Password=Password123!;TrustServerCertificate=True;",
            options => options.MigrationsHistoryTable("__EFMigrationsHistory", "mall")
        );

        var services = new ServiceCollection();
        var serviceProvider = services.BuildServiceProvider();
        var currentUser = new DesignTimeCurrentUser();
        var httpContextAccessor = new HttpContextAccessor();

        return new SqlServerMallDbContext(optionsBuilder.Options, serviceProvider, currentUser, httpContextAccessor);
    }
}
```

**MySQL**:

```csharp
public class MySqlMallDbContextFactory : IDesignTimeDbContextFactory<MySqlMallDbContext>
{
    public MySqlMallDbContext CreateDbContext(string[] args)
    {
        var optionsBuilder = new DbContextOptionsBuilder<MySqlMallDbContext>();
        
        optionsBuilder.UseMySql(
            "Server=localhost;Database=Mall;User=root;Password=password;",
            new MySqlServerVersion(new Version(8, 0, 21)),
            options => options.MigrationsHistoryTable("__EFMigrationsHistory")
        );

        var services = new ServiceCollection();
        var serviceProvider = services.BuildServiceProvider();
        var currentUser = new DesignTimeCurrentUser();
        var httpContextAccessor = new HttpContextAccessor();

        return new MySqlMallDbContext(optionsBuilder.Options, serviceProvider, currentUser, httpContextAccessor);
    }
}
```

---

## 迁移命令规范

### ⚠️ 重要：必须使用数据库特定的 DbContext

**❌ 错误**：使用基础 DbContext
```bash
dotnet ef migrations add InitialCreate --context MallDbContext
```

**✅ 正确**：使用数据库特定的 DbContext

#### SQL Server 迁移

```bash
# 创建迁移
dotnet ef migrations add InitialCreate --context SqlServerMallDbContext --output-dir Data/Migrations/SqlServer

# 更新数据库
dotnet ef database update --context SqlServerMallDbContext

# 删除最后一次迁移
dotnet ef migrations remove --context SqlServerMallDbContext
```

#### MySQL 迁移

```bash
# 创建迁移
dotnet ef migrations add InitialCreate --context MySqlMallDbContext --output-dir Data/Migrations/MySql

# 更新数据库
dotnet ef database update --context MySqlMallDbContext

# 删除最后一次迁移
dotnet ef migrations remove --context MySqlMallDbContext
```

### 迁移命名规范

| 场景 | 命名示例 |
|------|---------|
| 初始创建 | `InitialCreate` |
| 添加实体 | `Add{EntityName}` |
| 添加字段 | `Add{FieldName}To{EntityName}` |
| 修改字段 | `Update{FieldName}In{EntityName}` |
| 删除字段 | `Remove{FieldName}From{EntityName}` |
| 添加索引 | `AddIndexTo{EntityName}` |

---

## 实体 ID 配置

### 雪花 ID 配置

当实体使用应用层生成的雪花 ID（通过 `IIdGenerator`）时，**必须**在实体配置中添加 `ValueGeneratedNever()`：

```csharp
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.ToTable("Products");
        builder.HasKey(x => x.Id);
        
        // ✅ 必须：禁用数据库自动生成 ID
        builder.Property(x => x.Id).ValueGeneratedNever();
        
        // 其他配置...
    }
}
```

### 常见错误

**❌ 缺少 `ValueGeneratedNever()` 导致的错误**：

```
当 IDENTITY_INSERT 设置为 OFF 时，不能为表 'Products' 中的标识列插入显式值
```

或 MySQL：

```
Cannot insert explicit value for identity column in table 'Products' when IDENTITY_INSERT is set to OFF
```

**✅ 解决方案**：

1. 在实体配置中添加 `ValueGeneratedNever()`
2. 删除现有迁移
3. 使用正确的 DbContext 重新生成迁移

### 需要配置 ValueGeneratedNever 的场景

| 场景 | 需要配置 | 说明 |
|------|---------|------|
| 使用 `IIdGenerator` 生成 ID | ✅ 是 | 应用层生成雪花 ID |
| DemoDataService 中设置显式 ID | ✅ 是 | 测试数据生成 |
| 数据导入时保留原始 ID | ✅ 是 | 数据迁移 |
| 使用数据库自增 ID | ❌ 否 | 默认行为 |

### 检查清单

创建新实体时，确认以下事项：

- [ ] 实体是否继承 `AuditableEntityBase<long>` 或类似基类？
- [ ] 是否在代码中使用 `_idGenerator.NewId()` 设置 ID？
- [ ] 实体配置中是否添加了 `ValueGeneratedNever()`？

---

## 实体配置规范

### 配置文件位置

```
Data/Configurations/{EntityName}Configuration.cs
```

### 标准配置模板

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

namespace CodeSpirit.{Service}Api.Data.Configurations;

/// <summary>
/// {EntityName} 实体配置
/// </summary>
public class {EntityName}Configuration : IEntityTypeConfiguration<{EntityName}>
{
    public void Configure(EntityTypeBuilder<{EntityName}> builder)
    {
        // 表名
        builder.ToTable("{TableName}");
        
        // 主键
        builder.HasKey(x => x.Id);
        builder.Property(x => x.Id).ValueGeneratedNever(); // 雪花 ID
        
        // 租户 ID（多租户实体必须）
        builder.Property(x => x.TenantId).IsRequired().HasMaxLength(50);
        
        // 必填字符串字段
        builder.Property(x => x.Name).IsRequired().HasMaxLength(100);
        
        // 可选字符串字段
        builder.Property(x => x.Description).HasMaxLength(500);
        
        // 金额字段
        builder.Property(x => x.Amount).HasColumnType("decimal(18,2)");
        
        // 长文本字段
        builder.Property(x => x.Content).HasColumnType("text");
        
        // 带默认值的字段
        builder.Property(x => x.IsEnabled).HasDefaultValue(true);
        builder.Property(x => x.SortOrder).HasDefaultValue(0);
        
        // 索引
        builder.HasIndex(x => new { x.TenantId, x.Code })
            .IsUnique()
            .HasDatabaseName("IX_{TableName}_TenantId_Code");
        
        // 关系配置
        builder.HasOne(x => x.Parent)
            .WithMany(x => x.Children)
            .HasForeignKey(x => x.ParentId)
            .OnDelete(DeleteBehavior.Restrict);
    }
}
```

---

## 注意事项

1. **始终使用数据库特定的 DbContext 进行迁移操作**
2. **SQL Server 和 MySQL 的迁移必须分别生成**
3. **使用雪花 ID 的实体必须配置 `ValueGeneratedNever()`**
4. **迁移文件按数据库类型分目录存放**
5. **设计时工厂中的连接字符串仅用于迁移生成，不用于运行时**

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
