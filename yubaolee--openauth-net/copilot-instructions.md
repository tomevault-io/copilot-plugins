## openauth-net

> OpenAuth.Net是一个基于.NET 9的企业级权限管理和快速开发框架，采用Martin Fowler企业级应用开发思想，集成了最新的技术栈。

# OpenAuth.Net Cursor Rules

## 项目概述
OpenAuth.Net是一个基于.NET 9的企业级权限管理和快速开发框架，采用Martin Fowler企业级应用开发思想，集成了最新的技术栈。

## 技术栈
- **后端**: .NET 9, ASP.NET Core WebAPI
- **ORM**: SqlSugar (主要) + Entity Framework Core (兼容)
- **依赖注入**: Autofac
- **数据库**: 支持SqlServer、MySQL、Oracle、PostgreSQL
- **定时任务**: Quartz.NET
- **缓存**: Redis, MemoryCache
- **前端**: Vue2 + Element-UI
- **测试**: NUnit
- **文档**: Swagger

## 项目架构
```
📦OpenAuth.Net
 ┣ 📂Infrastructure       # 基础设施层 - 工具类、扩展方法、帮助类
 ┣ 📂OpenAuth.Repository  # 数据访问层 - 实体定义、数据访问
 ┣ 📂OpenAuth.App         # 应用服务层 - 业务逻辑
 ┣ 📂OpenAuth.WebApi      # 表示层 - WebAPI控制器
 ┣ 📂OpenAuth.Identity    # 身份认证服务 - IdentityServer4
 ┣ 📂Vue2                 # 前端项目
 ┗ 📂数据库脚本           # 数据库初始化脚本
```

## 编码规范

### 通用规范
- 使用C# 9+语法特性
- 遵循Microsoft C#编码规范
- 类名使用PascalCase，方法名使用PascalCase
- 私有字段使用_camelCase，公共属性使用PascalCase
- 常量使用UPPER_CASE
- 异步方法必须添加Async后缀
- 修改完代码后不需要编译

### 命名规范
- **Controller**: 以Controller结尾，如`UsersController`
- **Service/App**: 以App结尾，如`UserManagerApp`
- **Repository**: 以Repository结尾，如`UserRepository`
- **Entity**: 实体类直接使用名词，如`SysUser`
- **Request**: 请求类以Req结尾，如`QueryUserListReq`
- **Response**: 响应类以Resp结尾，如`PagedListDataResp`
- **DTO**: 视图模型以View结尾，如`UserView`

### 注释规范
- 所有公共方法必须添加XML注释
- 复杂业务逻辑必须添加行内注释
- 使用/// <summary>标记方法说明
- 参数使用/// <param name="参数名">说明</param>
- 返回值使用/// <returns>说明</returns>

## 分层架构规则

### Infrastructure层 (基础设施层)
- **职责**: 提供通用工具类、扩展方法、帮助类、常量定义
- **规则**: 
  - 不依赖其他业务层
  - 提供可复用的基础功能
  - 包含缓存、配置、工具类等
- **文件组织**: 
  - `/Cache` - 缓存相关
  - `/Extensions` - 扩展方法
  - `/Helpers` - 帮助类
  - `/Const` - 常量定义

### Repository层 (数据访问层)
- **职责**: 数据访问、实体定义、数据库操作
- **规则**:
  - 继承自`BaseRepository<T,TContext>`
  - 实体类继承自`StringEntity`、`LongEntity`或`IntAutoGenEntity`
  - 使用SqlSugar或EF Core进行数据访问
  - 包含DbContext配置
- **基类使用**:
  - `StringEntity`: 字符串主键实体
  - `LongEntity`: 长整型主键实体  
  - `IntAutoGenEntity`: 自增整型主键实体

### App层 (应用服务层)
- **职责**: 业务逻辑处理、数据传输对象、业务规则
- **规则**:
  - 继承自`SqlSugarBaseApp<T>`、`BaseStringApp<T>`、`BaseLongApp<T>`等基类
  - 通过构造函数注入依赖
  - 实现业务逻辑，不直接操作数据库
  - 使用Request/Response模式
- **依赖注入**:
  ```csharp
  public UserManagerApp(ISqlSugarClient client, RevelanceManagerApp app, IAuth auth) 
      : base(client, auth)
  ```

### WebApi层 (表示层)
- **职责**: HTTP请求处理、参数验证、响应格式化
- **规则**:
  - 继承自`ControllerBase`
  - 使用`[ApiController]`特性
  - 统一返回`Response<T>`格式
  - 进行参数验证
  - 添加Swagger文档注释

## 数据库操作规范

### SqlSugar使用规范
```csharp
// 查询
var users = SugarClient.Queryable<SysUser>()
    .Where(u => u.Status == 1)
    .ToList();

// 分页查询
var result = new PagedListDataResp<Role>();
var objs = SugarClient.Queryable<Role>();
if (!string.IsNullOrEmpty(request.key))
{
    objs = objs.Where(u => u.Name.Contains(request.key));
}

result.Data = await objs.OrderBy(u => u.Name)
    .Skip((request.page - 1) * request.limit)
    .Take(request.limit).ToListAsync();
result.Count = await objs.CountAsync();
return result;

// 联表查询
 var result = SugarClient.Queryable<Relevance>()
                .LeftJoin<SysOrg>((u, o) => u.SecondId == o.Id)
                .Where((u, o) => u.FirstId == userId && u.RelKey == Define.USERORG)
                .Select((u, o) => o);
    return result.ToList();
```

### 事务处理
```csharp
SugarClient.Ado.BeginTran();
try
{
    // 数据库操作
    SugarClient.Ado.CommitTran();
}
catch
{
    SugarClient.Ado.RollbackTran();
    throw;
}
```

## 权限和安全规范

### 权限验证
- 使用`IAuth`接口获取当前用户信息
- 通过`_auth.GetCurrentUser()`获取登录用户
- 数据权限通过机构级联控制

## API设计规范

### 控制器设计
```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly UserManagerApp _app;
    
    public UsersController(UserManagerApp app)
    {
        _app = app;
    }
    
    /// <summary>
    /// 获取用户列表
    /// </summary>
    [HttpGet]
    public async Task<Response<PagedListDataResp<UserView>>> Get([FromQuery] QueryUserListReq request)
    {
        var result = await _app.Load(request);
        return Response.Ok(result);
    }
}
```

## 依赖注入规范

### Autofac配置
- 在`AutofacExt.cs`中配置依赖注入
- 实现`IDependency`接口的类自动注册
- 使用构造函数注入

## 测试规范

### 单元测试
- 使用NUnit框架
- 继承自`TestBase`基类
- 测试类以Test结尾
- 测试方法使用描述性名称

### 集成测试
- 使用`AutofacWebApplicationFactory`
- 模拟HTTP请求
- 验证完整的业务流程

## 工作流规范

### 流程定义
- 使用`FlowScheme`定义流程模板
- 通过`FlowInstance`管理流程实例
- `FlowNode`表示流程节点

### 流程引擎
- 使用`FlowRuntime`执行流程
- 支持顺序、并行、条件等流程类型
- 流程状态通过`FlowInstanceStatus`管理

## 定时任务规范

### Quartz使用
- 继承自`IJob`接口
- 在`QuartzService`中注册任务
- 支持Cron表达式配置

## 文档规范

### 代码文档
- 重要业务逻辑添加注释
- 复杂算法添加说明
- 配置文件添加说明注释

## 配置管理

### 配置文件
- `appsettings.json`: 基础配置
- `appsettings.Development.json`: 开发环境配置
- `appsettings.Production.json`: 生产环境配置

---
> Source: [yubaolee/OpenAuth.Net](https://github.com/yubaolee/OpenAuth.Net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
