## naming-conventions

> CodeSpirit 命名约定规范 - 实体、DTO、服务、控制器等命名规则


# 实体类 (Entity)
- 命名格式：`{EntityName}` (如 `User`, `Question`)
- 继承：`AuditableEntityBase<TKey>` 或 `EntityBase<TKey>`
- 多租户：需租户隔离的实体必须实现 `IMultiTenant` 接口

# DTO 类
- **创建**: `Create{EntityName}Dto` (如 `CreateUserDto`)
- **更新**: `Update{EntityName}Dto` (如 `UpdateUserDto`)
- **查询**: `{EntityName}Dto` 或 `{EntityName}QueryDto`
- **列表项**: `{EntityName}ListItemDto`
- **批量导入**: `{EntityName}BatchImportItemDto`

# 服务类
- **业务服务**: `{EntityName}Service` (如 `UserService`)
- **接口**: `I{EntityName}Service`
- **查询服务**: `{EntityName}QueryService`
- **基础 CRUD**: 继承 `BaseCRUDService<TEntity, TKey, TCreateDto, TUpdateDto, TQueryDto>`
- **生命周期标记**: 实现 `IScopedDependency` / `ITransientDependency` / `ISingletonDependency`

示例：
```csharp
public interface IUserService : IBaseCRUDService<User, long, CreateUserDto, UpdateUserDto, UserQueryDto>
{
}

public class UserService : BaseCRUDService<User, long, CreateUserDto, UpdateUserDto, UserQueryDto>, 
    IUserService, IScopedDependency
{
}
```

# 控制器
- 命名：`{EntityName}Controller` (如 `UsersController`)，使用复数形式
- 路由：`[Route("/[service]/api/[controller]")]` 或具体路由如 `[Route("/exam/api/Questions")]`
- 继承：项目中的 `ApiControllerBase`

# 配置类
- API配置：`{ApiName}Configuration` (如 `ExamApiConfiguration`)
- 实体配置：`{EntityName}Configuration` (如 `UserConfiguration`)
- 位置：`Configuration` 文件夹

# 资源文件
- **共享资源**: `SharedResources`, `SharedResources.resx`
- **显示资源**: `DisplayResources`, `Display.resx`
- **验证资源**: `ValidationResources`, `Validation.resx`
- **服务特定**: `{Service}DisplayResources`, `{Service}Display.resx`
- **多语言**: `*.en.resx` (英文), `*.zh-CN.resx` (中文)

# 常量
- 类名：`{Module}Constants` (如 `AuthConstants`)
- 字段：全大写，下划线分隔 (如 `MAX_LOGIN_ATTEMPTS`)

# 事件
- 命名：`{EntityName}{Action}Event` (如 `UserCreatedEvent`, `OrderCancelledEvent`)
- 租户感知：继承 `TenantAwareEventBase`

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
