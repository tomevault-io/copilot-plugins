## southhome

> 此文件为 Claude Code (claude.ai/code) 在此代码库中工作时提供指导。

# CLAUDE.md

此文件为 Claude Code (claude.ai/code) 在此代码库中工作时提供指导。

## 项目概述

SouthHome 是一个前后端分离的个人博客系统：
- **后端**: ASP.NET Core 10.0 Web API + PostgreSQL + Entity Framework Core
- **前端**: ASP.NET Core 10.0 + Blazor WebAssembly + Microsoft Fluent UI
- **共享**: SouthHome.Shared - 前后端共用的 HTTP 模型和枚举

## 常用命令

```bash
# 构建共享项目
dotnet build SouthHome.Shared/SouthHome.Shared.csproj

# 构建后端
dotnet build SouthHome.Backend/SouthHome.Backend.csproj

# 运行后端 API
dotnet run --project SouthHome.Backend/SouthHome.Backend.csproj

# 构建前端
dotnet build SouthHome.Frontend.Fluent/SouthHome.Frontend.Fluent.csproj

# 运行前端
dotnet run --project SouthHome.Frontend.Fluent/SouthHome.Frontend.Fluent.csproj

# 安装前端依赖
dotnet restore SouthHome.Frontend.Fluent/SouthHome.Frontend.Fluent.csproj

# 添加 EF 迁移（配置好 DbContext 后）
dotnet ef migrations add 迁移名称 --project SouthHome.Backend

# 更新数据库
dotnet ef database update --project SouthHome.Backend
```

## 项目结构

```
SouthHome/
├── SouthHome.Shared/          # 共享项目（HTTP 模型、枚举）
│   ├── Common/
│   │   └── Enums/
│   │       ├── EditType.cs
│   │       └── PostStatus.cs
│   ├── Http/
│   │   ├── Auth/
│   │   │   ├── LoginRequest.cs
│   │   │   ├── LoginResponse.cs
│   │   │   └── InitSiteOwnerRequest.cs
│   │   └── BlogPosts/
│   │       ├── BlogPostDto.cs
│   │       ├── CreateBlogPostRequest.cs
│   │       ├── UpdateBlogPostRequest.cs
│   │       ├── GetPostsQuery.cs
│   │       └── PagedResult.cs
│   └── _Imports.cs
├── SouthHome.Backend/         # 后端 API
│   ├── Common/
│   │   ├── Enums/
│   │   ├── Http/
│   │   └── Tokens/
│   ├── Controllers/
│   ├── DatabaseContext/
│   ├── Entities/
│   ├── Services/
│   └── Migrations/
└── SouthHome.Frontend.Fluent/ # 前端 Blazor
    ├── Components/
    ├── Layout/
    ├── Pages/
    └── wwwroot/
```

## 架构设计

### 实体层模式

所有实体继承自 `EntityBase` 并遵循以下约定：

- **ID 生成**: 使用 `Util.GenerateUniqueGuid()` 的雪花算法（起始时间：2025-07-01）
- **软删除**: `IsDeleted` 标志 + `DeletedAt` 时间戳
- **审计字段**: `CreateAt`、`UpdateAt`（均为 UTC 时间）
- **工厂方法**: 所有实体使用 protected 构造函数，通过公共静态工厂方法创建（`CreateInstance`、`Create` 等）
- **表名**: 通过 `[Table("table_name")]` 属性显式指定 snake_case 命名

**实体模式示例：**
```csharp
[Table("my_entity")]
public class MyEntity : EntityBase
{
    protected MyEntity() { }  // EF Core 无参构造函数

    public static MyEntity CreateInstance(string name)
    {
        return new MyEntity { Name = name };
    }
}
```

### JWT 认证

通过 `TokenBase` / `UserToken` 实现的自定义 JWT：

- **Claims**: 序列化为 JSON 存储在 JWT payload 的 `claims` 字段中（非标准 JWT claims）
- **验证**: HS256 签名，默认 24 小时过期，零时钟偏移
- **密钥**: 从 `appsettings.json` 的 `SecretString` 读取
- **用户名**: 固定为 "south"

**重要说明**：创建或验证 token 时，`Claims` 字典会在 payload 内部序列化/反序列化为 JSON。

### 密码加密

使用 HMAC-SHA256 进行密码哈希：
- **盐值**: 32 字节随机生成的盐
- **哈希**: 密码 + 盐通过 HMAC-SHA256 计算
- **存储**: `PasswordHash` 和 `PasswordSalt` 均为 `byte[]` 类型

### 数据库上下文

`PgSqlContext` 包含所有实体的 DbSet：

```csharp
public DbSet<BlogPost> BlogPosts => Set<BlogPost>();
public DbSet<Tag> Tags => Set<Tag>();
public DbSet<BlogPostTag> BlogPostTags => Set<BlogPostTag>();
public DbSet<PostCategory> PostCategories => Set<PostCategory>();
public DbSet<PortfolioProject> PortfolioProjects => Set<PortfolioProject>();
public DbSet<SiteOwner> SiteOwners => Set<SiteOwner>();
public DbSet<ProtfolioProjectTag> PortfolioProjectTags => Set<ProtfolioProjectTag>();
```

### 服务层

后端使用服务层模式，所有服务通过依赖注入注册：
- `IAuthService` - 登录认证和密码哈希
- `ISeedDataService` - 数据库初始化
- `IBlogPostService` - 博客文章 CRUD

### 前端技术栈

- **框架**: ASP.NET Core 10.0 + Blazor WebAssembly
- **UI 组件库**: Microsoft Fluent UI (v4.14.0)
- **图标库**: Fluent UI Icons (@microsoft/fluentui-system-icons)
- **配色主题**: 黑红配色 (#1a1a1a → #dc2626)
- **Markdown 渲染**: Blazorise.Markdown (v1.8.8)

### 配置要求

`appsettings.Development.json` 需要配置以下键值：

| 键 | 用途 |
|-----|------|
| `SecretString` | JWT 签名密钥 |
| `PublicFolder` | 公共文件存储路径 |
| `PublicUrl` | 公共资源基础 URL |
| `RoomId` | 雪花算法工作者 ID 的一部分（3位） |
| `MachineId` | 雪花算法工作者 ID 的一部分（3位） |
| `ConnectionStrings:DefaultConnection` | PostgreSQL 连接字符串 |

### 错误处理

`ExceptionMiddleware` 包装所有异常并返回 `ServiceResponse<T>`：
- 始终返回 HTTP 200 状态码
- 自定义状态码在响应体 `Status` 字段中
- 错误信息在 `Message` 字段中

### 工具函数

- `Util.GenerateUniqueGuid()`: 生成雪花算法 ID
- `Util.GenerateAccount()`: 基于时间的账号编号（预留）
- `Util.GetImageFolder()` / `Util.GetImageUrl()`: 公共资源路径

## 前端项目结构

```
SouthHome.Frontend.Fluent/
├── Components/
│   ├── App.razor              # 应用根组件
│   └── _Imports.razor          # 全局导入
├── Layout/
│   └── MainLayout.razor   # 主布局
├── Pages/
│   ├── Home.razor        # 主页
│   ├── Blog.razor        # 博客列表页（含标签筛选）
│   └── Post.razor        # 文章详情页（含目录功能）
├── Program.cs                   # 应用入口
└── wwwroot/
    ├── css/                   # 样式文件
    ├── content/               # Markdown 文章文件
    └── images/                # 图片资源
```

## 前端开发规范

### 命名约定
- 使用 Fluent UI 组件时使用 PascalCase（如 `FluentStack`、`FluentButton`）
- 自定义类名使用 kebab-case（如 `social-card`、`post-card`）
- 图标使用 Fluent UI Icons（如 `Icons.Regular.Size24.Home`）

### 配色方案

| 用途 | 颜色值 |
|------|---------|
| 主红色 | #dc2626 |
| 悬停红色 | #e53935 |
| 黑色 | #1a1a1a |
| 深灰色 | #2c3e50 |
| 暗红色 | #e74c3c |
| 背景 | #f5f5f7 |
| 卡片背景 | white |
| 边框 | #e5e5e5 |

### Fluent UI 常用组件

- `FluentStack` - 布局容器
- `FluentButton` - 按钮
- `FluentCard` - 卡片
- `FluentText` - 文本
- `FluentBadge` - 徽章

## 前端功能模块

### 主页 (Home.razor)
- 圆形头像显示
- 四个功能卡片：项目、博客、游戏、联系方式
- 点击博客卡片可跳转到博客列表页

### 博客列表页 (Blog.razor)
- 标签筛选区（多选标签）
- 清除选中标签按钮
- 左侧时间线（按月份分组展示）
- 时间线月份可折叠/展开，对齐文章位置
- 博客文章卡片列表
- 点击卡片跳转到文章详情页

### 文章详情页 (Post.razor)
- 文章头部：标题、作者、日期、标签
- Markdown 内容渲染（从文件加载）
- 目录功能：
  - 右上角固定显示
  - 点击目录项跳转到对应章节
  - 滚动监听自动高亮当前章节
- 回到顶部按钮（滚动超过 300px 后显示）

### Markdown 文件加载
- 文章内容存放在 `wwwroot/content/post-{id}.md`
- 使用 `HttpClient` 异步加载文件内容
- 自定义 Markdown 解析器（支持标题、代码块、列表、表格等）

## 重要注意事项

### Blazor 字符串插值与花括号

当需要在 C# 字符串插值中输出包含花括号的 JavaScript 对象时：

```csharp
// ❌ 错误 - 与插值语法冲突
$"document.getElementById('{id}')?.scrollIntoView({{ behavior: 'smooth' }})";

// ✅ 正确 - 使用原始字符串字面量
$@"document.getElementById('{id}')?.scrollIntoView({{
    behavior: 'smooth',
    block: 'start'
}});";
```

`@""` 告诉编译器不要解析字符串中的任何特殊字符，直接按字面量处理。

### HTTP 客户端配置

`Program.cs` 中已注册 `HttpClient`：
```csharp
builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });
```

### Shared 项目

所有 HTTP 模型和枚举定义在 `SouthHome.Shared` 项目中，供前后端共用：
- DTOs: `BlogPostDto`, `LoginResponse`, `PagedResult<T>`
- Requests: `LoginRequest`, `CreateBlogPostRequest`, `UpdateBlogPostRequest`, etc.
- Enums: `PostStatus`, `EditType`, `Role`

## 表命名约定

数据库表使用 `[Table]` 属性指定的 snake_case 命名：

- `site_owners`
- `blog_posts`
- `blog_post_tags`
- `post_categories`
- `tags`
- `portfolio_projects`
- `protfolio_project_tags`（注意：原拼写有误）

## API 端点

### 认证相关

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/api/auth/login` | 登录获取 JWT Token |
| POST | `/api/seed/site-owner` | 初始化站点所有者 |

### 博客文章相关

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/posts` | 获取博客列表（分页、筛选） | 无 |
| GET | `/api/posts/{id}` | 获取单篇博客 | 无 |
| POST | `/api/posts` | 创建博客 | SiteOwnerOnly |
| PUT | `/api/posts/{id}` | 更新博客 | SiteOwnerOnly |
| DELETE | `/api/posts/{id}` | 删除博客 | SiteOwnerOnly |
| GET | `/api/posts/tags` | 获取所有标签 | 无 |

### 认证说明

使用 JWT Bearer Token 进行认证：
- 请求头格式: `Authorization: Bearer {token}`
- 用户名固定为 "south"
- 密码在初始化站点所有者时设置

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/southpyj)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/southpyj)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
