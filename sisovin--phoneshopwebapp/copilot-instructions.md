## phoneshopwebapp

> >


# ASP.NET Full-Stack & Backend Services — Professional Skill Document

**Version:** .NET 8 LTS (Core) | .NET Framework 4.8 (MVC 5)
**Official Repositories**
- ASP.NET Core: https://github.com/dotnet/aspnetcore
- ASP.NET MVC 5 (AspNetWebStack): https://github.com/aspnet/AspNetWebStack

---

## Reference File Index

Load the section file that matches the user's need. Multiple files may apply.

| Section | Topic | Reference File |
|---|---|---|
| **A** | Architecture & Framework (MVC, API, Minimal APIs, Routing, DI, Middleware, Filters, Swagger) | `references/A-architecture.md` |
| **B** | Backend Service Skills (Web API, REST endpoints, JSON, Auth, CORS, OWASP security) | `references/B-backend.md` |
| **C** | Database Skills (EF Core, ADO.NET, MySQL, Migrations, Repository pattern, Query optimization) | `references/C-database.md` |
| **D** | Full-Stack Development (HTML5, CSS3, JS, Tailwind, jQuery, AJAX, Fetch API) | `references/D-frontend.md` |
| **E** | System Architecture (Clean Arch, Layered Arch, DDD, SOLID, folder conventions) | `references/E-architecture-patterns.md` |
| **F** | Cross-Platform & Cloud (Linux/macOS/Win, Kestrel, Nginx, Docker, Azure/AWS/GCP, CI/CD) | `references/F-cloud-deploy.md` |
| **G** | GitHub Workflow (Git Flow, PRs, issue tracking, releases) | `references/G-github.md` |
| **H** | Best Practices (coding standards, logging, error handling, performance, security hardening) | `references/H-best-practices.md` |

---

## Technology Overview

### Framework Selection Matrix

| Need | Recommended Framework | .NET Package |
|---|---|---|
| Server-rendered web app | ASP.NET Core MVC | `Microsoft.AspNetCore.Mvc` |
| RESTful API (full features) | ASP.NET Core Web API | `Microsoft.AspNetCore.Mvc` + `[ApiController]` |
| Lightweight API / microservice | Minimal APIs (.NET 6+) | `WebApplication.MapGet/Post/...` |
| Legacy Windows-only app | ASP.NET MVC 5 | `System.Web.Mvc` |

### Core vs. MVC 5 — Critical Differences

| Dimension | ASP.NET Core | ASP.NET MVC 5 |
|---|---|---|
| Runtime | .NET 8 (cross-platform) | .NET Framework 4.x (Windows) |
| DI Container | Built-in `IServiceCollection` | Third-party (Autofac, Unity) |
| Middleware | `IMiddleware` / `Use/Map/Run` | HTTP Modules + HTTP Handlers |
| Configuration | `appsettings.json` + env vars | `Web.config` (XML) |
| Hosting | Kestrel + reverse proxy | IIS (HttpApplication pipeline) |
| Docker | First-class support | Windows containers only |
| Tag Helpers | ✅ Native | ❌ Not available |
| Minimal APIs | ✅ .NET 6+ | ❌ Not available |
| Razor Pages | ✅ Available | ❌ Not available |
| Recommendation | **All new projects** | Maintenance / migration only |

---

## Quick Reference — Program.cs (ASP.NET Core)

The single entry point for all Core applications (replaces `Startup.cs`):

```csharp
var builder = WebApplication.CreateBuilder(args);

// ── Service Registration ──────────────────────────────────────
builder.Services.AddControllersWithViews();           // MVC
builder.Services.AddControllers();                    // Web API only
// builder.Services.AddRazorPages();                 // Razor Pages (optional)

builder.Services.AddDbContext<AppDbContext>(opt =>
    opt.UseMySql(builder.Configuration.GetConnectionString("Default"),
        ServerVersion.AutoDetect(builder.Configuration.GetConnectionString("Default"))));

builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddScoped<IProductService, ProductService>();

builder.Services.AddAuthentication().AddJwtBearer();
builder.Services.AddAuthorization();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// ── Middleware Pipeline (order is critical) ───────────────────
if (app.Environment.IsDevelopment()) { app.UseSwagger(); app.UseSwaggerUI(); }
else { app.UseExceptionHandler("/Home/Error"); app.UseHsts(); }

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();    // Before Authorization
app.UseAuthorization();

app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}"); // MVC
app.MapControllers();                                                          // API

// Minimal API endpoints
app.MapGet("/api/health", () => Results.Ok(new { status = "healthy" }))
   .WithName("HealthCheck")
   .WithOpenApi();

app.Run();
```

---

## Skill Authorship Notes

- All code targets **C# 12 / .NET 8** unless marked `[MVC5]` (C# 7.3 / .NET Framework 4.8)
- MySQL provider: **Pomelo.EntityFrameworkCore.MySql** (open-source, async-first)
- Test framework: **xUnit + Moq + FluentAssertions**
- Logging: **Serilog** (structured) over built-in providers for production
- Architecture default: **Clean Architecture** with layered fallback for CRUD apps

---
> Source: [sisovin/PhoneShopWebApp](https://github.com/sisovin/PhoneShopWebApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
