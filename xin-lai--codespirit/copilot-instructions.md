## security

> CodeSpirit 安全规范 - 权限系统、审计实体、多租户隔离、数据保护


# 安全规范

> 📖 控制器级别的安全特性（`[Authorize]`、`[AllowAnonymous]`、`[NoAudit]`、`[Audit]`）参见 [controller.mdc](mdc:.cursor/rules/controller.mdc)

## 权限系统架构

### 权限代码格式

权限代码由系统自动生成，格式为：`{module}_{controller}_{action}`

```
exam_questions_getList         // 考试模块-题目控制器-获取列表
identity_users_create          // 身份模块-用户控制器-创建
```

### PermissionAttribute 特性

用于自定义权限名称、描述和继承关系：

```csharp
using CodeSpirit.Core.Attributes;

[HttpPut("{id}")]
[Permission(
    Name = "exam_questions_update",        // 自定义权限代码
    DisplayName = "更新题目",               // 显示名称
    Description = "允许更新题目内容",        // 权限描述
    Parent = "exam_questions",              // 父级权限
    AllowInheritedPermissions = new[] { "exam_questions_manage" }  // 继承权限
)]
[DisplayName("更新题目")]
public async Task<ActionResult<ApiResponse<QuestionDto>>> Update(
    long id, [FromBody] UpdateQuestionDto dto)
{
    // 拥有 exam_questions_manage 权限的用户也可以执行此操作
}
```

### 权限继承机制

`AllowInheritedPermissions` 允许配置权限继承，用户拥有任一继承权限时可访问当前接口：

```csharp
// 方法级别继承
[Permission(AllowInheritedPermissions = new[] { "Question.Manage", "Question.Admin" })]
public async Task<ActionResult> UpdateQuestion(long id) { }
```

### IHasPermissionService

服务层权限检查：

```csharp
public class QuestionService : IScopedDependency
{
    private readonly IHasPermissionService _permissionService;

    public QuestionService(IHasPermissionService permissionService)
    {
        _permissionService = permissionService;
    }

    public async Task<bool> CanManageQuestionAsync()
    {
        // 检查用户是否拥有指定权限
        return _permissionService.HasPermission("exam_questions_manage");
    }
    
    public async Task<bool> CanAccessModuleAsync()
    {
        // 检查导航权限（仅检查一二级权限）
        return _permissionService.HasNavigationPermission("exam_questions");
    }
}
```

## 审计实体

### AuditableEntityBase

实体继承 `AuditableEntityBase<TKey>` 自动记录审计信息：

```csharp
using CodeSpirit.Shared.Entities;

public class Question : AuditableEntityBase<long>, IMultiTenant
{
    // 自动记录以下字段（由框架自动填充）：
    // - CreatedBy (long)      创建人ID
    // - CreatedAt (DateTime)  创建时间
    // - UpdatedBy (long?)     更新人ID
    // - UpdatedAt (DateTime?) 更新时间
    // - IsDeleted (bool)      软删除标记
    // - DeletedBy (long?)     删除人ID
    // - DeletedAt (DateTime?) 删除时间
    
    public string Content { get; set; } = string.Empty;
    public string TenantId { get; set; } = string.Empty;  // 多租户支持
}
```

### 实体审计接口

| 接口 | 说明 | 包含字段 |
|-----|------|---------|
| `ICreatable` | 创建审计 | `CreatedBy`, `CreatedAt` |
| `IUpdatable` | 更新审计 | `UpdatedBy`, `UpdatedAt` |
| `ISoftDelete` | 软删除 | `IsDeleted`, `DeletedBy`, `DeletedAt` |
| `IFullAuditable` | 完整审计 | 以上全部 |

## 多租户数据隔离

### IMultiTenant 接口

实体实现 `IMultiTenant` 接口，自动应用租户数据过滤：

```csharp
using CodeSpirit.Core;

public class Question : AuditableEntityBase<long>, IMultiTenant
{
    public string TenantId { get; set; } = string.Empty;  // 注意：类型为 string
    public string Content { get; set; } = string.Empty;
}
```

### 租户过滤行为

```csharp
// 查询时自动应用租户过滤器，无需手动添加 Where 条件
public async Task<List<QuestionDto>> GetListAsync()
{
    var entities = await _dbContext.Questions
        .ToListAsync();  // ✅ 自动过滤当前租户的数据
    return _mapper.Map<List<QuestionDto>>(entities);
}

// ❌ 禁止：手动添加租户过滤（除非明确需要跨租户查询）
public async Task<List<QuestionDto>> GetListAsync()
{
    var entities = await _dbContext.Questions
        .Where(q => q.TenantId == _currentTenant.Id)  // 不需要
        .ToListAsync();
}
```

## 数据保护

### 排除敏感字段

DTO 中排除不应返回给客户端的字段：

```csharp
using Newtonsoft.Json;
using CodeSpirit.Amis.Attributes.Columns;

public class UserDto
{
    public long Id { get; set; }
    public string Username { get; set; } = string.Empty;
    
    [JsonIgnore]  // 不在 API 响应中返回
    public string PasswordHash { get; set; } = string.Empty;
    
    [IgnoreColumn]  // 不在列表表格中显示
    public string InternalNote { get; set; } = string.Empty;
}
```

### 密码安全存储

密码必须使用哈希存储，禁止明文：

```csharp
// ✅ 正确：存储密码哈希
user.PasswordHash = _passwordHasher.HashPassword(user, password);

// ❌ 禁止：明文存储密码
user.Password = password;
```

## SQL 注入防护

使用 EF Core 参数化查询（默认安全）：

```csharp
// ✅ 安全：EF Core 自动参数化
public async Task<Question?> GetByCodeAsync(string code)
{
    return await _dbContext.Questions
        .FirstOrDefaultAsync(q => q.Code == code);
}

// ✅ 安全：使用参数化 SQL
public async Task<List<Question>> SearchAsync(string keyword)
{
    return await _dbContext.Questions
        .FromSqlInterpolated($"SELECT * FROM Questions WHERE Content LIKE {$"%{keyword}%"}")
        .ToListAsync();
}

// ❌ 危险：字符串拼接（禁止）
public async Task<Question?> GetByCodeAsync(string code)
{
    var sql = $"SELECT * FROM Questions WHERE Code = '{code}'";  // SQL 注入风险！
    return await _dbContext.Questions.FromSqlRaw(sql).FirstOrDefaultAsync();
}
```

## 日志安全

### 敏感信息脱敏

日志中禁止记录敏感信息：

```csharp
// ✅ 安全：只记录必要信息
_logger.LogInformation("用户登录: {Username}, IP: {IpAddress}", 
    username, 
    ipAddress);

// ✅ 安全：使用脱敏字符串
_logger.LogInformation("API密钥验证: {MaskedKey}", 
    $"{apiKey[..4]}****{apiKey[^4..]}");

// ❌ 危险：记录密码（禁止）
_logger.LogInformation("用户登录: {Username}, Password: {Password}", 
    username, password);

// ❌ 危险：记录完整 Token（禁止）
_logger.LogInformation("Token: {Token}", token);
```

### 异常日志处理

```csharp
try
{
    await _service.ProcessAsync(data);
}
catch (Exception ex)
{
    // ✅ 记录异常但不暴露敏感上下文
    _logger.LogError(ex, "处理数据失败: EntityId={EntityId}", data.Id);
    throw;  // 重新抛出，由统一异常处理器处理
}
```

## CORS 配置

框架统一配置 CORS，API 项目无需单独配置：

```csharp
// CommonApiServiceExtensions 已配置
app.UseCors("AllowSpecificOriginsWithCredentials");
```

自定义配置（仅在特殊需求时）：

```csharp
// appsettings.json
{
  "Cors": {
    "AllowedOrigins": ["https://yourdomain.com"],
    "AllowCredentials": true
  }
}
```

## CSRF/XSS 防护

- **CSRF**: API 使用 JWT 认证，天然防御 CSRF 攻击
- **XSS**: AMIS 框架自动转义输出，后端无需手动处理

## 安全配置

### JWT 配置

```json
{
  "Jwt": {
    "SecretKey": "your-secret-key-at-least-32-characters",
    "Issuer": "CodeSpirit",
    "Audience": "CodeSpirit",
    "ExpireMinutes": 120,
    "RefreshExpireMinutes": 10080
  }
}
```

### 审计配置

```json
{
  "Audit": {
    "Enabled": true,
    "LogRequestParams": true,
    "LogResponseData": false,
    "SensitiveData": {
      "Enabled": true,
      "SensitiveFieldPatterns": ["password", "token", "apiKey", "secret"],
      "ExcludedFields": ["password", "newPassword", "confirmPassword"]
    }
  }
}
```

## 禁止事项

| 禁止 | 原因 | 正确做法 |
|-----|------|---------|
| 明文存储密码 | 安全风险 | 使用密码哈希 |
| SQL 字符串拼接 | SQL 注入风险 | EF Core 参数化查询 |
| 日志记录密码/Token | 信息泄露 | 只记录必要标识信息 |
| 手动租户过滤 | 可能遗漏 | 使用 `IMultiTenant` 自动过滤 |
| DTO 返回密码字段 | 信息泄露 | 使用 `[JsonIgnore]` 排除 |

## 安全检查清单

- [ ] 实体继承 `AuditableEntityBase` 实现自动审计
- [ ] 多租户实体实现 `IMultiTenant` 接口
- [ ] DTO 中使用 `[JsonIgnore]` 排除敏感字段
- [ ] 密码使用哈希存储
- [ ] 数据库查询使用 EF Core 参数化（避免 `FromSqlRaw` 拼接）
- [ ] 日志中不记录密码、Token 等敏感信息
- [ ] 敏感操作配置权限检查
- [ ] 定期审查权限配置和审计日志

## 参考文件

- 权限特性: [PermissionAttribute.cs](mdc:Src/CodeSpirit.Core/Attributes/PermissionAttribute.cs)
- 权限服务: [IHasPermissionService.cs](mdc:Src/CodeSpirit.Core/Authorization/IHasPermissionService.cs)
- 审计实体基类: [AuditableEntityBase.cs](mdc:Src/CodeSpirit.Shared/Entities/AuditableEntityBase.cs)
- 多租户接口: [IMultiTenant.cs](mdc:Src/CodeSpirit.Core/IMultiTenant.cs)
- 审计组件: [CodeSpirit.Audit/README.md](mdc:Src/Components/CodeSpirit.Audit/README.md)
- 控制器规范: [controller.mdc](mdc:.cursor/rules/controller.mdc)

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
