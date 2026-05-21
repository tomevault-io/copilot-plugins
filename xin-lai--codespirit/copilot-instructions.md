## dependency-injection

> CodeSpirit 依赖注入规范 - Scrutor自动注册、生命周期管理


# 依赖注入规范（Scrutor 自动注册）

## 概述

项目使用 Scrutor 库实现基于标记接口的自动依赖注入，无需手动注册服务。

## 标记接口

位于 `CodeSpirit.Core.DependencyInjection` 命名空间：

| 接口 | 生命周期 | 适用场景 |
|-----|---------|---------|
| `IScopedDependency` | Scoped | 业务服务、数据库操作、请求相关 |
| `ITransientDependency` | Transient | 无状态工具类、轻量操作 |
| `ISingletonDependency` | Singleton | 配置服务、缓存、ID生成器 |

### 生命周期说明

```csharp
// IScopedDependency - 作用域注入
// 同一个请求中是同一个实例，不同请求是不同实例
// 推荐：大多数业务服务、DbContext 相关操作

// ITransientDependency - 瞬时注入  
// 每次注入都创建新实例
// 推荐：无状态工具类、不持有资源的服务

// ISingletonDependency - 单例注入
// 整个应用生命周期只有一个实例
// 推荐：配置服务、缓存管理、ID生成器
```

## 标记方式

### 方式一：接口继承标记接口（推荐）

接口继承标记接口，实现类无需再次标记：

```csharp
// 接口定义 - 继承 IScopedDependency
public interface IAuthService : IScopedDependency
{
    Task<AuthResultDto> LoginAsync(LoginDto input);
    Task<bool> LogoutAsync(long userId);
}

// 实现类 - 无需标记接口
public class AuthService : IAuthService
{
    private readonly IRepository<User> _userRepository;
    
    public AuthService(IRepository<User> userRepository)
    {
        _userRepository = userRepository;
    }
    
    public async Task<AuthResultDto> LoginAsync(LoginDto input)
    {
        // 实现逻辑
    }
}
```

### 方式二：实现类标记接口

适用于无业务接口的服务类：

```csharp
// 无接口的服务类，直接实现标记接口
public class SeederService : IScopedDependency
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<SeederService> _logger;

    public SeederService(IServiceProvider serviceProvider, ILogger<SeederService> logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
    }

    public async Task SeedAsync()
    {
        // 初始化种子数据
    }
}
```

### 方式三：同时实现业务接口和标记接口

适用于需要明确指定生命周期的服务：

```csharp
public interface IUserService : IBaseCRUDService<User, long, CreateUserDto, UpdateUserDto, UserQueryDto>
{
    Task<UserDto> GetByUsernameAsync(string username);
}

public class UserService : BaseCRUDService<User, long, CreateUserDto, UpdateUserDto, UserQueryDto>, 
    IUserService, IScopedDependency
{
    public async Task<UserDto> GetByUsernameAsync(string username)
    {
        // 实现逻辑
    }
}
```

## 生命周期选择指南

### IScopedDependency（作用域 - 最常用）

```csharp
// ✅ 业务服务
public interface IQuestionService : IScopedDependency
{
    Task<QuestionDto> GetByIdAsync(long id);
    Task CreateAsync(CreateQuestionDto dto);
}

// ✅ 数据访问服务
public interface IExamRepository : IScopedDependency
{
    Task<Exam> GetWithQuestionsAsync(long examId);
}

// ✅ 种子数据服务
public class TenantSeeder : IScopedDependency
{
    public async Task SeedAsync() { }
}
```

### ISingletonDependency（单例）

```csharp
// ✅ ID 生成器
public interface IIdGenerator : ISingletonDependency
{
    long NewId();
}

// ✅ 缓存服务
public interface IConfigCacheService : ISingletonDependency
{
    Task<string> GetAsync(string key);
    Task SetAsync(string key, string value, TimeSpan? expiry = null);
}

// ✅ 端点扫描器（应用启动时扫描一次）
public class AiFormFillEndpointScanner : ISingletonDependency
{
    public void ScanAssemblies(params Assembly[] assemblies) { }
}

// ✅ 本地化设置初始化器
public class LocalizationSettingsInitializer : ISingletonDependency
{
    public void Initialize() { }
}
```

### ITransientDependency（瞬时）

```csharp
// ✅ 无状态工具类
public interface IPasswordHasher : ITransientDependency
{
    string HashPassword(string password);
    bool VerifyPassword(string password, string hash);
}

// ✅ 存储提供器工厂（每次创建新实例）
public interface IStorageProviderFactory : ITransientDependency
{
    IStorageProvider CreateProvider(string providerType);
}

// ✅ 配置变更通知器
public interface IConfigChangeNotifier : ITransientDependency
{
    Task NotifyChangeAsync(string configKey);
}
```

## Scrutor 自动注册扩展方法

### 基础注册方法

```csharp
// 位于 CodeSpirit.Shared.DependencyInjection.ServiceCollectionExtensions

// 自动扫描并注册标记接口的服务
services.AddDependencyInjectionWithScrutor(Assembly.GetExecutingAssembly());

// 可同时扫描多个程序集
services.AddDependencyInjectionWithScrutor(
    Assembly.GetExecutingAssembly(),
    typeof(SharedService).Assembly);
```

### 高级注册方法

```csharp
// 按命名约定自动注册（Service、Repository 后缀）
services.AddAdvancedDependencyInjection(Assembly.GetExecutingAssembly());
```

### 装饰器模式

```csharp
// 使用装饰器包装现有服务
services.AddDecorator<IUserService, CachingUserServiceDecorator>();
services.AddDecorator<ILogger<UserService>, AuditLoggerDecorator<UserService>>();
```

## 注册行为

Scrutor 自动完成以下注册：

1. **接口注册**：服务注册为其实现的业务接口
2. **自身注册**：服务同时注册为自身类型（可直接注入具体类）

```csharp
// 给定服务类
public class UserService : IUserService, IScopedDependency { }

// Scrutor 自动注册：
// services.AddScoped<IUserService, UserService>();  // 接口注册
// services.AddScoped<UserService>();                 // 自身注册

// 两种方式都可以注入：
public class UserController
{
    public UserController(
        IUserService userService,      // ✅ 接口注入
        UserService userServiceImpl)   // ✅ 具体类注入
    { }
}
```

## API 配置类中的服务注册

### 自动注册（BaseApiConfiguration 已处理）

```csharp
public class ExamApiConfiguration : BaseApiConfiguration
{
    public override void ConfigureServices(IServiceCollection services, IConfiguration configuration)
    {
        base.ConfigureServices(services, configuration);
        
        // Scrutor 自动注册已在 BaseApiConfiguration 中完成
        // 无需再调用 AddDependencyInjectionWithScrutor
    }
}
```

### 手动注册特殊服务

```csharp
public override void ConfigureServices(IServiceCollection services, IConfiguration configuration)
{
    base.ConfigureServices(services, configuration);
    
    // 手动注册：特殊配置、外部库服务、条件注册
    services.AddScoped<ISpecialService>(sp => 
        new SpecialService(sp.GetRequiredService<IOptions<SpecialOptions>>()));
    
    // 注册外部库服务
    services.AddHttpClient<IExternalApiClient, ExternalApiClient>();
    
    // 条件注册
    if (configuration.GetValue<bool>("Features:EnableNewFeature"))
    {
        services.AddScoped<INewFeatureService, NewFeatureService>();
    }
}
```

## 依赖注入最佳实践

### ✅ 推荐做法

```csharp
// 1. 构造函数注入（推荐）
public class QuestionService : IQuestionService, IScopedDependency
{
    private readonly IRepository<Question> _repository;
    private readonly IMapper _mapper;
    
    public QuestionService(IRepository<Question> repository, IMapper mapper)
    {
        _repository = repository;
        _mapper = mapper;
    }
}

// 2. 接口定义继承标记接口
public interface IExamService : IScopedDependency
{
    Task<ExamDto> GetByIdAsync(long id);
}

// 3. 使用 IServiceProvider 延迟解析（避免循环依赖）
public class CrudDialogHandler : IScopedDependency
{
    private readonly IServiceProvider _serviceProvider;
    
    public CrudDialogHandler(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    private ColumnHelper ColumnHelper => _serviceProvider.GetRequiredService<ColumnHelper>();
}
```

### ❌ 禁止做法

```csharp
// 1. 不要手动注册标记接口的服务（会重复注册）
services.AddScoped<IUserService, UserService>(); // ❌ Scrutor 已自动注册

// 2. 不要使用服务定位器反模式
public class BadService
{
    public void DoWork()
    {
        var service = ServiceLocator.GetService<IOtherService>(); // ❌
    }
}

// 3. 不要在 Singleton 服务中注入 Scoped 服务
public class BadSingletonService : ISingletonDependency
{
    private readonly IUserService _userService; // ❌ Scoped 服务不能注入到 Singleton
}

// 4. 不要创建多余的包装接口
public interface IUserServiceWrapper : IUserService { } // ❌ 不必要
```

## 常见问题

### 循环依赖处理

```csharp
// 使用 IServiceProvider 延迟解析
public class ServiceA : IScopedDependency
{
    private readonly IServiceProvider _serviceProvider;
    
    public ServiceA(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    // 延迟获取依赖
    private IServiceB ServiceB => _serviceProvider.GetRequiredService<IServiceB>();
}
```

### 条件服务注册

```csharp
// 在 API 配置类中进行条件注册
public override void ConfigureServices(IServiceCollection services, IConfiguration configuration)
{
    base.ConfigureServices(services, configuration);
    
    var storageType = configuration["Storage:Type"];
    services.AddScoped<IStorageService>(sp => storageType switch
    {
        "S3" => sp.GetRequiredService<S3StorageService>(),
        "Azure" => sp.GetRequiredService<AzureBlobStorageService>(),
        _ => sp.GetRequiredService<LocalStorageService>()
    });
}
```

### 多实现注册

```csharp
// 注册多个实现
services.AddScoped<INotificationService, EmailNotificationService>();
services.AddScoped<INotificationService, SmsNotificationService>();

// 注入所有实现
public class NotificationManager
{
    private readonly IEnumerable<INotificationService> _notificationServices;
    
    public NotificationManager(IEnumerable<INotificationService> notificationServices)
    {
        _notificationServices = notificationServices;
    }
    
    public async Task NotifyAllAsync(string message)
    {
        foreach (var service in _notificationServices)
        {
            await service.SendAsync(message);
        }
    }
}
```

## 命名空间引用

```csharp
using CodeSpirit.Core.DependencyInjection;  // 标记接口
using CodeSpirit.Shared.DependencyInjection; // 扩展方法
```

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
