## api-design

> CodeSpirit API 设计规范 - RESTful、路由、响应格式等


# RESTful 约定

| HTTP 方法 | 用途 | 幂等性 |
|----------|------|-------|
| **GET** | 查询资源（列表或单个） | ✅ |
| **POST** | 创建资源 | ❌ |
| **PUT** | 完整更新资源 | ✅ |
| **PATCH** | 部分更新资源 | ✅ |
| **DELETE** | 删除资源 | ✅ |

# HTTP 状态码

| 状态码 | 场景 | 响应方式 |
|-------|------|---------|
| 200 OK | GET/PUT/PATCH/DELETE 成功 | `SuccessResponse(data)` |
| 201 Created | POST 创建成功 | `SuccessResponseWithCreate()` |
| 400 Bad Request | 参数验证失败、业务规则错误 | `BadResponse()` 或抛出异常 |
| 401 Unauthorized | 未认证 | 框架自动处理 |
| 403 Forbidden | 无权限 | 框架自动处理 |
| 404 Not Found | 资源不存在 | 抛出 `BusinessException` |

# 路由规范

- 使用复数形式：`/api/users` 而非 `/api/user`
- 服务前缀：`/{service-name}/api/{controller}` (如 `/exam/api/Questions`)
- 版本控制：`/api/v{version}/{controller}` (未来需要时)

示例：
```csharp
using CodeSpirit.Core.Attributes;
using CodeSpirit.Navigation.Resources;

[Route("/exam/api/[controller]")]
[DisplayName("题目管理")]
[Navigation(Icon = "fa-solid fa-book", 
    TitleResourceKey = "Controller.Questions", 
    TitleResourceType = typeof(NavigationResources),
    PlatformType = PlatformType.Tenant)]
public class QuestionsController : ApiControllerBase
{
    [HttpGet]
    [DisplayName("获取题目列表")]
    public async Task<ActionResult<ApiResponse<PageList<QuestionDto>>>> GetList(
        [FromQuery] QuestionQueryDto query)
    {
        var result = await _service.GetPagedListAsync(query);
        return SuccessResponse(result);
    }
    
    [HttpGet("{id}")]
    [DisplayName("获取题目详情")]
    public async Task<ActionResult<ApiResponse<QuestionDto>>> GetById(long id)
    {
        var result = await _service.GetByIdAsync(id);
        return SuccessResponse(result);
    }
}
```

# Action 命名约定

| HTTP 方法 | 方法命名 | 示例 |
|----------|---------|------|
| GET (列表) | `GetList` / `Get{Entity}s` | `GetRoles()` |
| GET (单个) | `GetById` / `Get{Entity}` | `GetRole(long id)` |
| POST | `Create` | `Create(CreateDto dto)` |
| PUT | `Update` | `Update(long id, UpdateDto dto)` |
| DELETE | `Delete` | `Delete(long id)` |
| DELETE (批量) | `BatchDelete` | `BatchDelete(long[] ids)` |

# 响应格式

## 基类响应方法

```csharp
// 成功响应（带数据）- 返回 200
return SuccessResponse(data);  // { status: 0, msg: "操作成功！", data: {...} }

// 成功响应（无数据）- 返回 200
return SuccessResponse();

// 创建成功响应 - 返回 201
return SuccessResponseWithCreate<RoleDto>(nameof(GetRole), roleDto);

// 失败响应 - 返回指定状态码（默认 400）
return BadResponse("操作失败", code: 1, statusCode: 400);
```

## 分页响应

使用 `PageList<T>` 作为分页响应类型：

```csharp
public async Task<ActionResult<ApiResponse<PageList<RoleDto>>>> GetRoles(
    [FromQuery] RoleQueryDto queryDto)
{
    PageList<RoleDto> result = await _roleService.GetRolesAsync(queryDto);
    return SuccessResponse(result);
}
```

## 文件下载

```csharp
// Excel 文件下载
return DownloadExcelFile(fileBytes, "导出数据.xlsx");

// CSV 文件下载
return DownloadCsvFile(fileBytes, "导出数据.csv");

// 通用文件下载
return DownloadFile(fileBytes, "文件名.zip", "application/zip");

// 流式文件下载
return DownloadFile(fileStream, "大文件.pdf", "application/pdf");
```

# 认证授权

- 默认所有控制器需要认证（继承自基类配置）
- 匿名访问：`[AllowAnonymous]`
- 显式认证：`[Authorize]`

```csharp
// 无需登录的控制器
[AllowAnonymous]
[Navigation(Hidden = true)]
[NoAudit("授权控制器不需要审计")]
public class AuthController : ApiControllerBase { }

// 显式要求认证
[Authorize]
public class ApiKeysController : ApiControllerBase { }
```

# 审计支持

- **启用审计**：`[Audit]` - 控制器或方法级别
- **禁用审计**：`[NoAudit]` - 敏感操作或高频接口

```csharp
// 启用审计并配置详细日志
[Audit(EntityName = nameof(Department), LogRequestParams = true, LogResponseData = true)]
public class DepartmentsController : ApiControllerBase { }

// 禁用审计
[NoAudit("授权控制器不需要审计")]
public class AuthController : ApiControllerBase { }
```

# 操作特性

> 📖 详细操作特性配置参见 [controller.mdc](mdc:.cursor/rules/controller.mdc)

操作特性用于定义前端操作按钮，必须包含：
- `DisplayName`: 操作显示名称（控制器方法）
- `Icon`: Font Awesome 图标类名

常用操作特性：
- **Operation**: 基础操作特性
- **HeaderOperation**: 表头操作（新增、导入、导出等）
- **RowOperation**: 行操作（编辑、删除等）
- **BatchOperation**: 批量操作

示例：
```csharp
[HttpPut("{id}/unlock")]
[Operation("解锁", "ajax", null, "确定要解除用户锁定吗？", "lockoutEnd != null",
    LabelResourceKey = "Operations.Unlock",
    LabelResourceType = typeof(OperationsResources),
    Icon = "fa-solid fa-unlock")]
[DisplayName("解锁用户")]
public async Task<ActionResult<ApiResponse>> UnlockUser(long id)
{
    await _service.UnlockAsync(id);
    return SuccessResponse();
}
```

# 参数绑定

| 来源 | 特性 | 示例 |
|-----|------|------|
| 查询字符串 | `[FromQuery]` | `GetList([FromQuery] QueryDto query)` |
| 请求体 | `[FromBody]` | `Create([FromBody] CreateDto dto)` |
| 路由参数 | `[FromRoute]` 或省略 | `GetById(long id)` |
| 表单数据 | `[FromForm]` | `Upload([FromForm] IFormFile file)` |

> 💡 Action 方法如果存在多个参数，请使用 DTO 模型替代。

# 批量操作路由

```csharp
// 批量删除
[HttpDelete("batch")]
[DisplayName("批量删除")]
public async Task<ActionResult<ApiResponse>> BatchDelete([FromBody] long[] ids)
{
    await _service.BatchDeleteAsync(ids);
    return SuccessResponse();
}

// 批量导入
[HttpPost("batch-import")]
[DisplayName("批量导入")]
public async Task<ActionResult<ApiResponse<BatchImportResult>>> BatchImport(
    [FromForm] IFormFile file)
{
    var result = await _service.BatchImportAsync(file);
    return SuccessResponse(result);
}

// 导出
[HttpGet("export")]
[DisplayName("导出数据")]
public async Task<ActionResult> Export([FromQuery] ExportQueryDto query)
{
    var bytes = await _service.ExportAsync(query);
    return DownloadExcelFile(bytes, "导出数据.xlsx");
}
```

# 异常处理

- 不在 Action 中捕获异常
- 使用统一异常过滤器 `HttpResponseExceptionFilter`
- 抛出业务异常：`throw new BusinessException("错误消息")`
- 本地化异常：`throw new BusinessException("Errors.InvalidStartTime")`

```csharp
// ❌ 错误：在 Action 中捕获异常
public async Task<ActionResult<ApiResponse>> Create(CreateDto dto)
{
    try
    {
        await _service.CreateAsync(dto);
        return SuccessResponse();
    }
    catch (Exception ex)
    {
        return BadResponse(ex.Message);  // 不推荐
    }
}

// ✅ 正确：让异常过滤器处理
public async Task<ActionResult<ApiResponse>> Create(CreateDto dto)
{
    await _service.CreateAsync(dto);  // 异常会被过滤器捕获
    return SuccessResponse();
}
```

# 参考文件

- API 响应类: [ApiResponse.cs](mdc:Src/CodeSpirit.Core/ApiResponse.cs)
- 控制器基类: [ApiControllerBase.cs](mdc:Src/CodeSpirit.Shared/Controllers/ApiControllerBase.cs)
- 异常过滤器: [HttpResponseExceptionFilter.cs](mdc:Src/CodeSpirit.Shared/Filters/HttpResponseExceptionFilter.cs)
- 控制器规范: [controller.mdc](mdc:.cursor/rules/controller.mdc)
- 安全规范: [security.mdc](mdc:.cursor/rules/security.mdc)

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
