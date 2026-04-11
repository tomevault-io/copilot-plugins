## unimanage

> Frontend: React + TailwindCSS

Frontend: React + TailwindCSS
luôn ưu tiên dùng hero-ui (from "@heroui/react";) trước khi viết component
luôn dùng tailwindcss
luôn dùng typescript
ưu tiên code truyền thống, rõ ràng, dễ bảo trì

Mục tiêu
Sinh code cho dự án UniManage theo chuẩn CQRS, .NET 9, SQL Server, Dapper (không EF Core), IdentityServer (Duende) với store tự viết, và log4net (log theo ngày & theo API). Ưu tiên code truyền thống, rõ ràng, dễ bảo trì.

**🔥 QUY TẮC QUAN TRỌNG: LUÔN SỬ DỤNG UTILITIES TRƯỚC**
Trước khi viết bất kỳ logic nào, PHẢI kiểm tra UniManage.Core/Utilities/ trước để sử dụng lại:

- ✅ **PasswordHelper**: HashPassword(), VerifyPassword(), GenerateRandomPassword(), IsValidPassword() (Min 8 chars, complexity)
- ✅ **ValidationHelper**: IsValidEmail(), IsValidPhoneNumber(), IsValidCode(), ToFieldErrorModels()
- ✅ **DatabaseHelper**: UserCodeExistsAsync(), ExecuteWithTransactionAsync(), QueryPagingAsync(), CheckTableExistsAsync()
- ✅ **ResponseHelper**: Success(), Error(), NotFound(), Forbidden(), PagedSuccess(), PagedError()
- ✅ **Shared Validation**: Dùng `.All.Contains(status)` từ `CoreCommon.Value` auto-generated cho các danh sách giá trị.
- ✅ **StringHelper**: ToSlug(), ToCamelCase(), RemoveDiacritics(), MaskSensitiveData(), GenerateCode()
- ✅ **DateTimeHelper**: ToVietnamTime(), CalculateAge(), GetRelativeTime(), AddBusinessDays()
- ✅ **FileHelper**: IsValidImageFile(), ValidateFileUpload(), GetMimeType(), GetFileSizeText()
- ✅ **QueryHelper**: BuildOrderByClause(), BuildWhereClause(), EscapeSqlIdentifier() - SQL injection prevention
- ✅ **TranslateHelper**: TranslateAsync(), RemoveDiacritics(), TranslateCommonTerms() - Google Translate + Vietnamese

**🚫 KHÔNG ĐƯỢC:**

- Viết lại logic đã có trong utilities
- Tạo response object thủ công thay vì dùng ResponseHelper
- Hash password bằng cách khác ngoài PasswordHelper.HashPassword()
- Validate email/phone thủ công thay vì dùng ValidationHelper
- Viết transaction logic thay vì dùng DatabaseHelper.ExecuteWithTransactionAsync()

**✅ LUÔN LÀMM:**

- Check utilities trước khi code bất kỳ tính năng nào
- Extend utilities nếu thiếu method cần thiết
- Sử dụng ResponseHelper cho tất cả API responses
- Dùng DatabaseHelper cho tất cả database operations có transaction
- **Validator Resources**: LUÔN dùng `CoreResource` cho tất cả nhãn và thông báo lỗi (mẫu: `string.Format(CoreResource.validation_required, CoreResource.lbl_username)`).
- **Logging Standard**: LUÔN dùng `nameof(request.PropertyName)` cho tên tham số trong `CoreParamModel`.

Tech stack (bắt buộc)
Backend: ASP.NET Core .NET 9

Auth: Duende IdentityServer (tự viết IClientStore, IResourceStore, IPersistedGrantStore, IDeviceFlowStore bằng Dapper)

DB: SQL Server

Data access: **Entity Framework Core 9.0 + Dapper (hybrid)** - EF Core cho CRUD, Dapper cho complex queries

Pattern: CQRS + MediatR

Jobs: Hangfire (SQL Server storage, dashboard port riêng)

Logging: log4net (tách file theo ngày và API)

Frontend: React + TailwindCSS + Next.js (không Angular)

CI/CD: GitHub Actions + Docker

**🔥 RESTful API Standards (BẮT BUỘC)**

**URLs PHẢI dùng nouns (danh từ), KHÔNG dùng verbs (động từ):**

✅ **ĐÚNG:**

- GET /api/v1/users - Get list
- GET /api/v1/users/{id} - Get single
- POST /api/v1/users - Create
- PUT /api/v1/users/{id} - Update
- DELETE /api/v1/users - Delete (có thể batch với body)
- GET /api/v1/menus - Get menu tree (không cần /tree)
- GET /api/v1/users/combobox - Exception: combobox cho UI helper

❌ **SAI:**

- GET /api/v1/users/getList (động từ redundant)
- GET /api/v1/menus/tree (noun redundant, menu đã là resource)
- POST /api/v1/users/create (động từ redundant)
- GET /api/v1/getUsers (động từ, không có resource path)

**HTTP Method Mapping:**

- GET: Retrieve resource(s) - KHÔNG thay đổi data
- POST: Create new resource
- PUT: Update existing resource (full update)
- PATCH: Partial update (optional, ít dùng)
- DELETE: Remove resource(s)

**URL Structure Rules:**

- Plural nouns: /users, /orders, /items (không /user, /order)
- Hierarchical: /departments/{id}/employees
- Query params cho filters: /users?status=active&keyword=john
- Route params cho IDs: /users/{id}, /orders/{code}
- Avoid redundant paths: /menus thay vì /menus/tree
- Exception: Helper endpoints như /combobox, /export, /import

Chuẩn API response (bắt buộc)
JSON camelCase, errors luôn là array.

API thường:

json
Sao chép
Chỉnh sửa
{
"returnCode": 0,
"errors": [],
"message": "Thao tác thành công",
"data": {}
}
API danh sách (phân trang):

json
Sao chép
Chỉnh sửa
{
"returnCode": 0,
"errors": [],
"message": "Lấy danh sách thành công",
"data": {
"items": [],
"paging": {
"pageIndex": 1,
"pageSize": 20,
"totalItems": 100,
"totalPages": 5
}
}
}
Dùng các class:

ApiResponse<T>

PagedResult<T> (Items, Paging)

PagedResponse<T> : ApiResponse<PagedResult<T>>

PagingInfo (PageIndex, PageSize, TotalItems, TotalPages)

Kiến trúc & CQRS (bắt buộc)
Solution layout:

swift
Sao chép
Chỉnh sửa
UniManage.sln
├─ UniManage.API // Controllers mỏng (HTTP → Mediator)
├─ UniManage.Application // Commands, Queries, DTOs, Validators, Behaviors
│ ├─ Commands/{Module}/{UseCase}
│ ├─ Queries/{Module}/{UseCase}
│ ├─ Pipelines (Logging, Validation, Transaction, Caching)
│ └─ Results (ApiResponse, PagedResponse, …)
├─ UniManage.Infrastructure // Dapper Repos (Read/Write), Migrations, log4net
└─ UniManage.IdentityServer // Duende + Dapper stores (no EF)
Nguyên tắc CQRS:

Command = thay đổi dữ liệu → không trả entity đầy đủ, chỉ Id/Result.

Query = chỉ đọc → không mutate, trả DTO đã shape cho UI.

Controller mỏng: chỉ gọi \_mediator.Send(...).

Transaction: chỉ cho Command (qua TransactionBehavior).

Validation: FluentValidation → ValidationBehavior.

Caching (tuỳ): chỉ cho Query.

Không dùng SELECT \*.

Đặt tên:

Command: VerbNounCommand (CreateUserCommand).

Query: Get/List/FindNounQuery (ListUsersQuery).

1 Handler ↔ 1 Command/Query.

**🔥 QUY TẮC BẮT BUỘC: ĐỒNG BỘ INSTRUCTION KHI THAY ĐỔI QUY TẮC**

Khi có bất kỳ thay đổi nào ảnh hưởng đến quy tắc chung (class Common, naming convention, pattern mới, utility mới, cấu trúc project, ...) PHẢI cập nhật đủ cả 3 file instruction:

1. **SKILL.md**: `.agent/skills/unimanage-backend/SKILL.md` — cho Antigravity
2. **GEMINI.md**: `GEMINI.md` — cho Gemini trong VS Code
3. **copilot-instructions.md**: `.github/copilot-instructions.md` — cho GitHub Copilot

> **Lý do**: 3 AI tools cùng lúc, nếu không đồng bộ → mỗi tool sinh code theo pattern khác nhau → không nhất quán, gây lỗi.

**🔥 QUY TẮC KẾ THỪA (BẮT BUỘC):**

**Command PHẢI kế thừa BaseCommand:**

- BaseCommand cung cấp HeaderInfo cho logging context
- HeaderInfo chứa Username, IP, Timestamp, TraceId từ request
- Tự động tích hợp với CoreLogModel để log chi tiết
- Mask sensitive data (password, token, secret) trong logs

```csharp
// ✅ ĐÚNG - Command kế thừa BaseCommand
public sealed class CreateUserCommand : BaseCommand, IRequest<ApiResponse<Response>>
{
    // HeaderInfo inherited from BaseCommand
    public string UserCode { get; init; } = default!;
    public string DisplayName { get; init; } = default!;

    public sealed class Response
    {
        public int Id { get; init; }
    }
}

// ❌ SAI - Thiếu BaseCommand
public sealed class CreateUserCommand : IRequest<ApiResponse<Response>>
{
    // Missing HeaderInfo - không log được context
    public string UserCode { get; init; } = default!;
}
```

**Query PHẢI kế thừa đúng base class:**

- **`BaseListQuery`**: Query trả về `PagedResult` (danh sách + phân trang)
    - Cung cấp `HeaderInfo` + `Keyword` + `PageIndex` + `PageSize` + `Offset` + `SortBy` + `SortDirection` + `SearchFields`
    - PageIndex mặc định = 1, PageSize mặc định = 20
- **`BaseQuery`**: Query đơn (get by id/code, check exists, combobox, permissions...)
    - Chỉ cung cấp `HeaderInfo` cho logging context

**BaseQuery hierarchy (UniManage.Model/Common/BaseModel.cs):**

```
BaseQuery          → chỉ có HeaderInfo
  └── BaseListQuery  → thêm Keyword, SearchFields, PageIndex, PageSize, Offset, SortBy, SortDirection
```

```csharp
// ✅ ĐÚNG - List Query kế thừa BaseListQuery
public sealed class ListUsersQuery : BaseListQuery, IRequest<PagedResponse<UserListItemDto>>
{
    // HeaderInfo, Keyword, PageIndex, PageSize, Offset, SortBy, SortDirection inherited from BaseListQuery
    public byte? Status { get; init; }
    public string? DepartmentCode { get; init; }
}

// ✅ ĐÚNG - Single-item Query kế thừa BaseQuery
public sealed class GetUserByIdQuery : BaseQuery, IRequest<ApiResponse<GetUserByIdQuery.Response>>
{
    // Chỉ HeaderInfo inherited từ BaseQuery
    public long Id { get; init; }
}

// ❌ SAI - List Query thiếu BaseListQuery
public sealed class ListUsersQuery : IRequest<PagedResponse<UserListItemDto>>
{
    public string? Keyword { get; init; }
    public int PageIndex { get; init; } = 1; // Duplicate!
    public int PageSize { get; init; } = 20; // Duplicate!
}
```

**BaseCommand/BaseQuery/BaseListQuery location:**

- Namespace: `UniManage.Model.Common`
- File: `UniManage.Model/Common/BaseModel.cs`
- `BaseCommand` extends `BaseModel` (has HeaderInfo with [JsonIgnore])
- `BaseQuery` chỉ có HeaderInfo (cho get by id/code, check exists, combobox)
- `BaseListQuery` kế thừa `BaseQuery` + thêm pagination/search/sort properties

**HeaderInfo usage in Handlers:**

```csharp
public async Task<ApiResponse<Response>> Handle(
    CreateUserCommand request,
    CancellationToken ct)
{
    var log = new CoreLogModel
    {
        Username = request.HeaderInfo?.Username ?? "Anonymous",
        IpAddress = request.HeaderInfo?.IpAddress,
        TraceId = request.HeaderInfo?.TraceId,
        Parameters = JsonConvert.SerializeObject(request, _maskSettings)
    };

    try
    {
        // Handler logic...
        log.Result = JsonConvert.SerializeObject(response);
        return response;
    }
    catch (Exception ex)
    {
        log.ErrorMessage = ex.Message;
        throw;
    }
    finally
    {
        _logger.Info(JsonConvert.SerializeObject(log));
    }
}
```

Database conventions (SQL Server)
PK: Id (INT IDENTITY hoặc BIGINT).

Text TV: NVARCHAR; thời gian: DATETIME2(3).

Concurrency: ROWVERSION → cột DataRowVersion (map byte[]).

Cột audit (mọi bảng): CreatedBy, CreatedAt, UpdatedBy, UpdatedAt, DataRowVersion.

Hậu tố cột: Amount/Number/Name/Code/Qty/Date/Rate.

Index: IX*{Table}*{Column}.

Phân trang SQL:

sql
Sao chép
Chỉnh sửa
... ORDER BY SortColumn, Id
OFFSET (@PageIndex - 1) \* @PageSize ROWS FETCH NEXT @PageSize ROWS ONLY;

Database access patterns (bắt buộc)

**🔥 HYBRID APPROACH: Entity Framework Core 9.0 + Dapper**

- **EF Core cho CRUD operations**: Create, Read by Id, Update, Delete
- **Dapper cho complex queries**: Joins, aggregations, raw SQL optimization
- **DbContext tự động discover entities** từ UniManage.Model.Entities namespace

**Using statements (bắt buộc):**

```csharp
using Microsoft.EntityFrameworkCore;
using DbContext = UniManage.Core.Database.DbContext; // Alias để tránh conflict
using UniManage.Model.Entities; // Entity classes
```

**Command pattern (EF Core với transaction):**

```csharp
using (var dbContext = new DbContext(openTransaction: true))
{
    try
    {
        // EF Core CRUD operations
        var user = new sy_users
        {
            Username = request.Username,
            Email = request.Email,
            Password = passwordHash,
            CreatedBy = request.HeaderInfo?.Username ?? "SYSTEM",
            CreatedAt = DateTime.Now,
            RowVersion = new byte[8]
        };

        dbContext.Set<sy_users>().Add(user);
        await dbContext.SaveChangesAsync(ct);
        await dbContext.CommitAsync(ct);

        return user.Id; // EF Core auto-populates Id
    }
    catch
    {
        await dbContext.RollbackAsync(ct);
        throw;
    }
}
```

**Query pattern (EF Core LINQ, không transaction):**

```csharp
using (var dbContext = new DbContext())
{
    var query = dbContext.Set<sy_users>().AsQueryable();

    // Apply filters with LINQ
    if (!string.IsNullOrEmpty(keyword))
    {
        query = query.Where(u => u.Username.Contains(keyword));
    }

    // Pagination
    var totalItems = await query.CountAsync(ct);
    var items = await query
        .OrderByDescending(u => u.CreatedAt)
        .Skip((pageIndex - 1) * pageSize)
        .Take(pageSize)
        .Select(u => new UserDto { Id = u.Id, Username = u.Username })
        .ToListAsync(ct);

    return new PagedResult<UserDto> { Items = items, Paging = new PagingInfo { ... } };
}
```

**Optimistic Concurrency (RowVersion):**

```csharp
// Update with RowVersion check
var user = await dbContext.Set<sy_users>().FirstOrDefaultAsync(u => u.Id == id, ct);

if (request.RowVersion != null && !user.RowVersion.SequenceEqual(request.RowVersion))
{
    return ResponseHelper.Error<Response>("Data has been modified by another user");
}

user.Email = request.Email;
user.UpdatedAt = DateTime.Now;

try
{
    await dbContext.SaveChangesAsync(ct); // EF Core checks RowVersion automatically
}
catch (DbUpdateConcurrencyException)
{
    return ResponseHelper.Error<Response>("Concurrency conflict");
}
```

**Package Dependencies (Core project)**

```xml
<PackageReference Include="BCrypt.Net-Next" Version="4.0.3" />
<PackageReference Include="Dapper" Version="2.1.66" />
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="9.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="9.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="9.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="9.0.0" />
<PackageReference Include="FluentValidation" Version="12.0.0" />
<PackageReference Include="log4net" Version="3.1.0" />
<PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
<PackageReference Include="System.Data.SqlClient" Version="4.8.6" />
```

**Model Architecture (chuẩn hóa)**

```csharp
// Base API Response
public class ApiResponse<T>
{
    public int ReturnCode { get; set; }
    public string Message { get; set; } = string.Empty;
    public T? Data { get; set; }
    public List<string> Errors { get; set; } = new();
}

// Paged Response Structure
public class PagedResponse<T> : ApiResponse<PagedResult<T>> { }
public class PagedResult<T>
{
    public List<T> Items { get; set; } = new();
    public PagingInfo Paging { get; set; } = new();
}

public class PagingInfo
{
    public int PageIndex { get; set; } = 1;
    public int PageSize { get; set; } = 20;
    public int TotalItems { get; set; }
    public int TotalPages => (int)Math.Ceiling((double)TotalItems / PageSize);
}

// Validation Error Model
public class FieldErrorModel
{
    public string Field { get; set; } = string.Empty;
    public List<string> Messages { get; set; } = new();
}
```

**Core Project Structure (sau cleanup)**

```
UniManage.Core/
├── Database/           # DbContext, IDbContext, UniManageDbContext, Generator/
├── Logging/           # UniLogger, UniLogManager
├── Models/            # ApiResponse, PagedResponse, PagingInfo, FieldErrorModel, Entities/
├── Security/          # ConfigEncryption
└── Utilities/         # 9 comprehensive utility classes
```

IdentityServer (không EF)
Lưu Clients, IdentityResources, ApiResources, ApiScopes, PersistedGrants, DeviceCodes, Keys trong SQL Server (schema thủ công).

Implement stores bằng Dapper: IClientStore, IResourceStore, IPersistedGrantStore, IDeviceFlowStore (+ IServerSideSessionStore nếu bật).

Flow mặc định: Authorization Code + PKCE (không ROPC).

IdentityServer: job Hangfire 15 phút xóa PersistedGrants/DeviceCodes hết hạn.

🔥 **Security Note**: Login handler PHẢI check `Status == "Active"` để chặn user bị vô hiệu hóa.

Logging (log4net) — theo ngày & theo API
Mỗi request gắn api = {controller}-{action} → log vào file:

logs/yyyy-MM-dd/{api}.log (INFO+)

logs/yyyy-MM-dd/error-{api}.log (ERROR/FATAL)

Mọi log có: timestamp, CorrelationId (cid), user, method, path, status, ms.

Mask dữ liệu nhạy cảm: password|token|secret|authorization.

Không log raw body > 64KB.

**Logging Pattern trong Handler:**

```csharp
// Khởi tạo log với HeaderInfo từ BaseCommand/BaseQuery
var log = new CoreLogModel(request.HeaderInfo)
{
    Parameter = new List<CoreParamModel>
    {
        new CoreParamModel(nameof(request.Username), request.Username),
        new CoreParamModel(nameof(request.Email), request.Email)
    }
};

try
{
    // Handler logic...
    var response = ResponseHelper.Success(data, "Operation successful");
    log.Result = response;
    log.ReturnCode = response.ReturnCode;
    log.Message = response.Message;
    return response;
}
catch (Exception ex)
{
    log.IsException = 1;
    log.Message = ex.Message;
    log.ReturnCode = CoreApiReturnCode.ExceptionOccurred;
    return ResponseHelper.Error<T>("Error message");
}
finally
{
    UniLogManager.WriteApiLog(log);
}
```

Yêu cầu Copilot khi sinh code:

Tạo ApiLogContextMiddleware gắn api theo {controller}-{action}.

Tạo CorrelationIdMiddleware (header X-Correlation-Id).

Tạo HttpLoggingMiddleware để log request/response + st, ms.

Cấu hình log4net bằng SiftingAppender trỏ tới logs/%date{yyyy-MM-dd}/%property{api}.log.

Controller & Handler (mẫu tối thiểu)

**🔥 CONTROLLER FORMATTING RULES (BẮT BUỘC):**

1. **Kế thừa BaseController** (KHÔNG dùng ControllerBase)
2. **Parameters PHẢI viết cùng dòng với method signature** - KHÔNG xuống dòng
3. **Body method dùng explicit blocks** - KHÔNG dùng expression-bodied members
4. **Luôn gán HeaderInfo** trước khi Send

**🔥 RESPONSEHELPER FORMATTING RULES (BẮT BUỘC):**

1. **Parameters PHẢI viết cùng dòng** - KHÔNG xuống dòng
2. **Áp dụng cho TẤT CẢ methods**: Success, Error, NotFound, Forbidden, etc.
3. **Nếu dòng quá dài (>120 chars)**: Tách object creation ra biến riêng trước

```csharp
// ✅ ĐÚNG - Parameters cùng dòng
var response = ResponseHelper.Success(new CreateUserCommand.Response
{
    Id = id
},
CoreResource.User_msg_CreateSuccess);

// ✅ ĐÚNG - Object phức tạp tách ra biến
var responseData = new CreateUserCommand.Response { Id = id, UserCode = userCode };
var response = ResponseHelper.Success(responseData, CoreResource.User_msg_CreateSuccess);

// ❌ SAI - Xuống dòng parameters
var response = ResponseHelper.Success(
    new CreateUserCommand.Response { Id = id },
    CoreResource.User_msg_CreateSuccess);
```

Controller (mỏng):

```csharp
[ApiController]
[Route("api/v1/users")]
public class UsersController : BaseController
{
    // ...
    [HttpPost("change-password")]
    public async Task<ActionResult<ApiResponse<bool>>> ChangePassword([FromBody] ChangePasswordCommand command, CancellationToken ct)
    {
        command.HeaderInfo = HeaderInfo;
        command.Username = this.Username; // 🔥 BẮT BUỘC: Lấy Username từ JWT để tránh IDOR
        var result = await _mediator.Send(command, ct);
        return Ok(result);
    }
}
    private readonly IMediator _mediator;

    public UsersController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    public async Task<ActionResult<ApiResponse<CreateUserCommand.Response>>> Create([FromBody] CreateUserCommand command, CancellationToken ct)
    {
        command.HeaderInfo = HeaderInfo;  // PHẢI gán HeaderInfo
        var result = await _mediator.Send(command, ct);
        return Ok(result);
    }

    [HttpGet]
    public async Task<ActionResult<ApiResponse<PagedResult<GetUserListQuery.Response>>>> List([FromQuery] GetUserListQuery query, CancellationToken ct)
    {
        query ??= new GetUserListQuery();
        query.HeaderInfo = HeaderInfo;  // PHẢI gán HeaderInfo
        var result = await _mediator.Send(query, ct);
        return Ok(result);
    }
}
```

**🔥 REGION FORMATTING (BẮT BUỘC):**

**1. Command/Query files PHẢI có 3 regions**: Command/Query, Validator, Handler

```csharp
#region Command
// Command definition
#endregion

#region Validator
// Validator definition
#endregion

#region Handler
// Handler definition
#endregion
```

**2. Controller files PHẢI có regions cho Properties và mỗi endpoint**:

```csharp
[ApiController]
[Route("api/v1/users")]
public class UsersController : BaseController
{
    #region Properties

    private readonly IMediator _mediator;

    public UsersController(IMediator mediator)
    {
        _mediator = mediator;
    }

    #endregion

    #region GET: /api/v1/users

    [HttpGet]
    public async Task<ActionResult<ApiResponse<PagedResult<Response>>>> GetList([FromQuery] GetUserListQuery query, CancellationToken ct)
    {
        query ??= new GetUserListQuery();
        query.HeaderInfo = HeaderInfo;
        var result = await _mediator.Send(query, ct);
        return Ok(result);
    }

    #endregion

    #region POST: /api/v1/users

    [HttpPost]
    public async Task<ActionResult<ApiResponse<CreateUserCommand.Response>>> Create([FromBody] CreateUserCommand command, CancellationToken ct)
    {
        command.HeaderInfo = HeaderInfo;
        var result = await _mediator.Send(command, ct);
        return Ok(result);
    }

    #endregion

    #region PUT: /api/v1/users/{id}

    [HttpPut("{id}")]
    public async Task<ActionResult<ApiResponse<Response>>> Update([FromRoute] long id, [FromBody] UpdateUserCommand command, CancellationToken ct)
    {
        command ??= new UpdateUserCommand();
        command.Id = id;
        command.HeaderInfo = HeaderInfo;
        var result = await _mediator.Send(command, ct);
        return Ok(result);
    }

    #endregion

    #region DELETE: /api/v1/users/{id}

    [HttpDelete("{id}")]
    public async Task<ActionResult<ApiResponse<bool>>> Delete(long id, CancellationToken ct)
    {
        var command = new DeleteUserCommand { Id = id, HeaderInfo = HeaderInfo };
        var result = await _mediator.Send(command, ct);
        return Ok(result);
    }

    #endregion
}
```

Command mẫu (kèm Validator và Handler):

```csharp
#region Command

// Command (PHẢI kế thừa BaseCommand)
public sealed class CreateUserCommand : BaseCommand, IRequest<ApiResponse<CreateUserCommand.Response>>
{
    public string Username { get; init; } = default!;
    public string Email { get; init; } = default!;
    public string? EmployeeCode { get; init; }
    public string Password { get; init; } = default!;
    public string Status { get; init; } = default!;
    public List<string> RoleCode { get; init; } = default!;

    public sealed class Response
    {
        public bool Success => Id > 0;
        public long Id { get; init; }
    }
}

#endregion

#region Validator

// Validator (có thể có static async methods để check DB với EF Core)
public sealed class CreateUserValidator : AbstractValidator<CreateUserCommand>
{
    public CreateUserValidator()
    {
        RuleFor(x => x.Username)
            .Cascade(CascadeMode.Stop)
            .NotEmpty().WithMessage(string.Format(CoreResource.validation_required, CoreResource.lbl_username))
            .Length(3, 50).WithMessage(string.Format(CoreResource.validation_range, CoreResource.lbl_username, 3, 50))
            .Must(ValidationHelper.IsValidUserCode).WithMessage(CoreResource.validation_alphanumericOnly)
            .MustAsync(async (username, cancel) => !await IsUsernameExistsAsync(username))
            .WithMessage(string.Format(CoreResource.validation_alreadyExists, CoreResource.lbl_username));

        RuleFor(x => x.Email)
            .NotEmpty().WithMessage(string.Format(CoreResource.validation_required, CoreResource.lbl_email))
            .Must(ValidationHelper.IsValidEmail).WithMessage(CoreResource.validation_invalidEmail)
            .MustAsync(async (email, cancel) => !await IsEmailExistsAsync(email))
            .WithMessage(string.Format(CoreResource.validation_alreadyExists, CoreResource.lbl_email));

        RuleFor(x => x.Password)
            .Password(CoreResource.lbl_password);
    }

    private static async Task<bool> IsUsernameExistsAsync(string username)
    {
        using (var dbContext = new DbContext())
        {
            // Use EF Core LINQ instead of raw SQL
            return await dbContext.Set<sy_users>().AnyAsync(u => u.Username == username);
        }
    }

    private static async Task<bool> IsEmailExistsAsync(string email)
    {
        using (var dbContext = new DbContext())
        {
            // Use EF Core LINQ instead of raw SQL
            return await dbContext.Set<sy_users>().AnyAsync(u => u.Email == email);
        }
    }
}

#endregion

#region Handler

// Handler (KHÔNG inject ILogger, dùng UniLogManager)
// **Handler method parameters PHẢI cùng dòng với signature**
public sealed class CreateUserCommandHandler : IRequestHandler<CreateUserCommand, ApiResponse<CreateUserCommand.Response>>
{
    public async Task<ApiResponse<CreateUserCommand.Response>> Handle(CreateUserCommand request, CancellationToken ct)
    {
        // Initialize log với HeaderInfo từ BaseCommand
        var log = new CoreLogModel(request.HeaderInfo)
        {
            Parameter = new List<CoreParamModel>
            {
                new CoreParamModel(nameof(request.Username), request.Username),
                new CoreParamModel(nameof(request.Email), request.Email)
            }
        };

        using (var dbContext = new DbContext(openTransaction: true))
        {
            try
            {
                // Hash password bằng PasswordHelper
                var passwordHash = PasswordHelper.HashPassword(request.Password);

                // Create new user entity with EF Core
                var newUser = new sy_users
                {
                    Username = request.Username,
                    Password = passwordHash,
                    Email = request.Email,
                    EmployeeCode = request.EmployeeCode,
                    RoleCode = request.RoleCode?.FirstOrDefault() ?? "USER",
                    Status = request.Status,
                    CreatedBy = request.HeaderInfo?.Username ?? "SYSTEM",
                    CreatedAt = DateTime.Now,
                    RowVersion = new byte[8] // SQL Server manages via rowversion
                };

                // Add to DbContext
                dbContext.Set<sy_users>().Add(newUser);

                // Save changes - EF Core auto-populates Id
                await dbContext.SaveChangesAsync(ct);
                await dbContext.CommitAsync(ct);

                var response = ResponseHelper.Success(
                    new CreateUserCommand.Response { Id = newUser.Id },
                    "User created successfully");

                log.Result = response;
                log.ReturnCode = response.ReturnCode;
                log.Message = response.Message;
                UniLogManager.WriteApiLog(log);

                return response;
            }
            catch (Exception ex)
            {
                await dbContext.RollbackAsync(ct);

                log.IsException = 1;
                log.Message = ex.Message;
                log.ReturnCode = CoreApiReturnCode.ExceptionOccurred;
                UniLogManager.WriteApiLog(log);

                return ResponseHelper.Error<CreateUserCommand.Response>("Error occurred");
            }
        }
    }
}

#endregion
```

Query mẫu (paging với EF Core LINQ):

```csharp
#region Query

// Query (PHẢI kế thừa BaseQuery - có sẵn Keyword, PageIndex, PageSize, SortBy, SortDirection)
public sealed class GetUserListQuery : BaseQuery, IRequest<ApiResponse<PagedResult<GetUserListQuery.Response>>>
{
    public string? Status { get; set; }

    public sealed record Response
    {
        public long Id { get; set; }
        public string Username { get; set; } = default!;
        public string EmployeeCode { get; set; } = default!;
        public string Email { get; set; } = default!;
        public string Status { get; set; } = default!;
        public DateTime CreatedAt { get; set; }
    }
}

#endregion

#region Validator

// Validator (validate PageIndex/PageSize từ BaseQuery)
public sealed class GetUserListQueryValidator : AbstractValidator<GetUserListQuery>
{
    public GetUserListQueryValidator()
    {
        RuleFor(x => x.PageIndex)
            .GreaterThan(0).WithMessage("Page index must be greater than 0");

        RuleFor(x => x.PageSize)
            .InclusiveBetween(1, 100).WithMessage("Page size must be between 1 and 100");
    }
}

#endregion

#region Handler

// Handler (KHÔNG inject ILogger, dùng EF Core LINQ)
public sealed class GetUserListQueryHandler : IRequestHandler<GetUserListQuery, ApiResponse<PagedResult<GetUserListQuery.Response>>>
{
    public async Task<ApiResponse<PagedResult<GetUserListQuery.Response>>> Handle(
        GetUserListQuery request,
        CancellationToken ct)
    {
        // Initialize log với HeaderInfo từ BaseQuery
        var log = new CoreLogModel(request.HeaderInfo)
        {
            Parameter = new List<CoreParamModel>
            {
                new CoreParamModel(nameof(request.Keyword), request.Keyword),
                new CoreParamModel(nameof(request.Status), request.Status),
                new CoreParamModel(nameof(request.PageIndex), request.PageIndex),
                new CoreParamModel(nameof(request.PageSize), request.PageSize)
            }
        };

        using (var dbContext = new DbContext()) // Query KHÔNG cần transaction
        {
            try
            {
                // Build query with EF Core LINQ
                var query = dbContext.Set<sy_users>().AsQueryable();

                // Apply filters
                if (!string.IsNullOrEmpty(request.Status))
                {
                    query = query.Where(u => u.Status == request.Status);
                }

                if (!string.IsNullOrEmpty(request.Keyword))
                {
                    var keyword = request.Keyword.Trim().ToLower();
                    query = query.Where(u =>
                        u.Username.ToLower().Contains(keyword) ||
                        u.Email.ToLower().Contains(keyword) ||
                        (u.EmployeeCode != null && u.EmployeeCode.ToLower().Contains(keyword)));
                }

                // Get total count before pagination
                var totalItems = await query.CountAsync(ct);

                // Apply sorting (default: CreatedAt DESC)
                query = query.OrderByDescending(u => u.CreatedAt);

                // Apply pagination with projection
                var items = await query
                    .Skip((request.PageIndex - 1) * request.PageSize)
                    .Take(request.PageSize)
                    .Select(u => new GetUserListQuery.Response
                    {
                        Id = u.Id,
                        Username = u.Username,
                        EmployeeCode = u.EmployeeCode ?? string.Empty,
                        Email = u.Email,
                        Status = u.Status,
                        CreatedAt = u.CreatedAt
                    })
                    .ToListAsync(ct);

                // Build paged result
                var result = new PagedResult<GetUserListQuery.Response>
                {
                    Items = items,
                    Paging = new PagingInfo
                    {
                        PageIndex = request.PageIndex,
                        PageSize = request.PageSize,
                        TotalItems = totalItems
                    }
                };

                var response = ResponseHelper.Success(result, "Users retrieved successfully");

                log.Result = result;
                log.ReturnCode = response.ReturnCode;
                UniLogManager.WriteApiLog(log);

                return response;
            }
            catch (Exception ex)
            {
                UniLogger.Error($"Error retrieving users: {ex.Message}", ex);

                var response = ResponseHelper.Error<PagedResult<GetUserListQuery.Response>>("Error occurred");
                log.IsException = 1;
                log.Message = ex.Message;
                log.ReturnCode = response.ReturnCode;
                UniLogManager.WriteApiLog(log);

                return response;
            }
        }
    }
}

#endregion
```

**EF Core Update Pattern (Optimistic Concurrency):**

```csharp
public sealed class UpdateUserCommandHandler : IRequestHandler<UpdateUserCommand, ApiResponse<Response>>
{
    public async Task<ApiResponse<Response>> Handle(UpdateUserCommand request, CancellationToken ct)
    {
        using (var dbContext = new DbContext(openTransaction: true))
        {
            try
            {
                // Find user by Id
                var user = await dbContext.Set<sy_users>()
                    .FirstOrDefaultAsync(u => u.Id == request.Id, ct);

                if (user == null)
                {
                    await dbContext.RollbackAsync(ct);
                    return ResponseHelper.NotFound<Response>("User not found");
                }

                // Check RowVersion for optimistic concurrency
                if (request.RowVersion != null && !user.RowVersion.SequenceEqual(request.RowVersion))
                {
                    await dbContext.RollbackAsync(ct);
                    return ResponseHelper.Error<Response>("User has been modified by another process");
                }

                // Update properties
                user.Email = request.Email;
                user.Status = request.Status;
                user.UpdatedBy = request.HeaderInfo?.Username ?? "SYSTEM";
                user.UpdatedAt = DateTime.Now;

                // Save changes - EF Core handles RowVersion concurrency check
                await dbContext.SaveChangesAsync(ct);
                await dbContext.CommitAsync(ct);

                return ResponseHelper.Success(new Response { Id = user.Id }, "Updated successfully");
            }
            catch (DbUpdateConcurrencyException)
            {
                await dbContext.RollbackAsync(ct);
                return ResponseHelper.Error<Response>("Concurrency conflict");
            }
        }
    }
}
```

**EF Core Delete Pattern (Soft Delete):**

```csharp
public sealed class DeleteUserCommandHandler : IRequestHandler<DeleteUserCommand, ApiResponse<Response>>
{
    public async Task<ApiResponse<Response>> Handle(DeleteUserCommand request, CancellationToken ct)
    {
        using (var dbContext = new DbContext(openTransaction: true))
        {
            try
            {
                // Find users by Ids using EF Core
                var users = await dbContext.Set<sy_users>()
                    .Where(u => request.Ids.Contains(u.Id))
                    .ToListAsync(ct);

                if (users.Count == 0)
                {
                    await dbContext.RollbackAsync(ct);
                    return ResponseHelper.NotFound<Response>("No users found to delete");
                }

                // Soft delete: Set Status to "Inactive"
                foreach (var user in users)
                {
                    user.Status = "Inactive";
                    user.UpdatedBy = request.HeaderInfo?.Username ?? "SYSTEM";
                    user.UpdatedAt = DateTime.Now;
                }

                await dbContext.SaveChangesAsync(ct);
                await dbContext.CommitAsync(ct);

                return ResponseHelper.Success(
                    new Response { DeletedIds = users.Select(u => u.Id).ToList(), Count = users.Count },
                    $"{users.Count} user(s) deleted successfully");
            }
            catch (Exception ex)
            {
                await dbContext.RollbackAsync(ct);
                return ResponseHelper.Error<Response>(ex.Message);
            }
        }
    }
}
```

MediatR Pipelines (bắt buộc)
Thứ tự: Logging → Validation → Authorization (tuỳ) → Transaction (Command) → Caching (Query)

LoggingBehavior: log handler name + thời gian.

ValidationBehavior: gom lỗi FluentValidation → ApiResponse.Fail(errors).

TransactionBehavior (Command): mở SqlConnection + BeginTransaction, truyền IDbTransaction xuống repo, commit/rollback.

CachingBehavior (Query, tuỳ): key = TypeName + hash(params).

JSON camelCase & chuẩn hóa
csharp
Sao chép
Chỉnh sửa
builder.Services.AddControllers()
.AddJsonOptions(o => o.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase);
❌ Không SELECT \*.

✔️ Có <summary> cho mọi public type/method/prop.

**Query & Filter Utilities (tùy chọn - dùng cho complex queries)**

**✅ EF Core LINQ là ưu tiên cho CRUD:**

- `Where()` cho filters
- `OrderBy()` / `OrderByDescending()` cho sorting
- `Skip()` / `Take()` cho pagination
- `CountAsync()` cho total count
- `Select()` cho projection

**💡 DatabaseHelper & QueryHelper dùng cho complex queries (Dapper):**

Khi cần raw SQL cho performance hoặc complex joins:

```csharp
// 1. Build dynamic WHERE clause
var filters = new Dictionary<string, object?>
{
    { "Username", QueryHelper.SanitizeSearchTerm(keyword) },
    { "Status", status }
};
var (whereClause, parameters) = DatabaseHelper.BuildWhereClause(filters);

// 2. Build dynamic ORDER BY
var columnMappings = new Dictionary<string, string>
{
    { "default", "CreatedAt" },
    { "username", "Username" }
};
var (orderBy, _) = QueryHelper.BuildOrderByClause(
    sortBy,
    sortDirection ?? "DESC",
    columnMappings);

// 3. Execute paginated query (Dapper)
var result = await DatabaseHelper.QueryPagingAsync(
    dbContext,
    baseQuery,
    whereClause,
    orderBy,
    parameters,
    pageIndex,
    pageSize);
```

**✅ Luôn sử dụng:**

- EF Core LINQ cho CRUD operations (Create, Read, Update, Delete)
- `AsQueryable()` cho dynamic filters
- `ToListAsync()` / `FirstOrDefaultAsync()` / `AnyAsync()` cho async operations

**🚫 Không được:**

- Viết raw SQL cho CRUD operations (dùng EF Core)
- Concatenate SQL strings với user input (SQL injection risk)
- `SELECT *` trong raw SQL (specify columns)
- Implement pagination logic thủ công (dùng Skip/Take)

Few-shot gợi ý cho Copilot
"Sinh CreateUserCommand kế thừa BaseCommand + Handler + Validator, repo Dapper (Insert), trả ApiResponse theo chuẩn, có TransactionBehavior cho Command, log với CoreLogModel(HeaderInfo)."

"Sinh GetUserListQuery kế thừa BaseQuery + Handler + Validator, sử dụng DatabaseHelper.QueryPagingAsync, QueryHelper.BuildOrderByClause, trả ApiResponse<PagedResult>, log với CoreLogModel(HeaderInfo), sử dụng properties từ BaseQuery (Keyword, PageIndex, PageSize, SortBy, SortDirection)."

“Sinh Dapper IClientStore cho IdentityServer lấy client theo ClientId, có RedirectUris, PostLogoutUris, Scopes từ các bảng riêng.”

“Tạo log4net SiftingAppender để log theo logs/%date{yyyy-MM-dd}/%property{api}.log và error-%property{api}.log (ERROR+).”

Chất lượng & review checklist
Controller mỏng, mọi nghiệp vụ nằm trong Handler

Command/Query tách rõ, 1–1 Handler

**Command PHẢI kế thừa BaseCommand, Query PHẢI kế thừa BaseQuery**

Có Validator; lỗi hợp nhất theo ApiResponse

Command có TransactionBehavior; Query không

**EF Core LINQ cho CRUD, Dapper cho complex queries**

**Validator dùng AnyAsync/FirstOrDefaultAsync thay vì ExecuteScalarAsync**

Update/Delete kiểm tra RowVersion cho optimistic concurrency

Query trả DTO đã shape + paging chuẩn với EF Core LINQ

Log per-day/per-api, có cid/user/method/path/status/ms, mask secrets

**Handler phải sử dụng CoreLogModel(request.HeaderInfo) với Parameter list**

**Handler KHÔNG inject ILogger, dùng UniLogManager.WriteApiLog()**

**Validator có thể có static async methods để check DB với EF Core LINQ**

**Query handler Ư u tiên EF Core LINQ, chỉ dùng DatabaseHelper/QueryHelper cho complex queries**

**Dùng ResponseHelper.Success/Error thay vì tạo response thủ công**

**Entity classes trong UniManage.Model.Entities namespace**

**DbContext alias: using DbContext = UniManage.Core.Database.DbContext (tránh conflict với EF Core)**

**Cleanup & Code Quality Rules (từ experience)**

- **Resource Standardization**: 100% Validators phải sử dụng `CoreResource` cho field labels và error messages.
- **Type-safe Logging**: Luôn sử dụng `nameof(request.PropertyName)` cho `CoreParamModel`.
- **Unified Security**: Sử dụng `.Password()` extension và check `Status == "Active"` khi login.
  ✅ **Utilities-First Approach:**

- Luôn check UniManage.Core/Utilities/ trước khi viết logic mới
- Extend utilities thay vì duplicate code
- Utilities phải có XML documentation đầy đủ

✅ **Package Dependencies:**

- BCrypt.Net-Next cho password hashing (không tự implement)
- FluentValidation cho validation rules
- **Entity Framework Core 9.0 cho CRUD operations**
- **Dapper cho complex queries (optional)**
- Microsoft.EntityFrameworkCore.SqlServer cho SQL Server
- System.Data.SqlClient cho raw SQL connection

✅ **Model Structure:**

- ApiResponse<T> cho tất cả API responses
- PagedResponse<T> : ApiResponse<PagedResult<T>> cho pagination
- FieldErrorModel cho validation errors từ FluentValidation

✅ **Database Patterns:**

- using statements để đảm bảo disposal
- Write operations có transaction (openTransaction: true)
- Read operations không cần transaction
- **EF Core cho CRUD: Add, FirstOrDefaultAsync, SaveChangesAsync**
- **Dapper cho complex queries: QueryAsync, ExecuteAsync**
- Async methods với CancellationToken
- **Optimistic concurrency với RowVersion (byte[])**

✅ **File Organization:**

- Không duplicate folders (tránh Helpers/ và Utilities/ cùng tồn tại)
- Xóa files không sử dụng (Database/Test/, empty handlers)
- Tổ chức theo responsibility: Database/, Models/, Utilities/, Security/

🚫 **Anti-patterns:**

- Không tạo response objects thủ công
- Không hash passwords bằng MD5/SHA
- **Không dùng raw SQL cho CRUD operations (dùng EF Core)**
- Không viết SQL injection vulnerable queries
- Không để files duplicate giữa các folders
- Không để external dependencies trong Core project
- **Không dùng ExecuteScalarAsync cho validation (dùng AnyAsync)**
- **Không dùng DbContext trực tiếp (dùng alias: using DbContext = UniManage.Core.Database.DbContext)**

Nguyên tắc: ít phép màu, ưu tiên rõ ràng, đúng “truyền thống” để team onboard nhanh, debug khỏe.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tchiphuong)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tchiphuong)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
