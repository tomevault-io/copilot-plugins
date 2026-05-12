## eazymeny-main

> EazyMenu is a SaaS platform for restaurant management built with **Clean Architecture** principles using **.NET 9.0**.

# GitHub Copilot Instructions - EazyMenu Project

## 🎯 Project Overview
EazyMenu is a SaaS platform for restaurant management built with **Clean Architecture** principles using **.NET 9.0**.

**Tech Stack:**
- Framework: ASP.NET Core 9.0 MVC (No Razor Pages)
- Database: SQL Server 2022 + Entity Framework Core 9.0
- Authentication: ASP.NET Core Identity
- Patterns: CQRS (MediatR), Repository Pattern, Unit of Work
- External Services: Zarinpal (Payment), Kavenegar (SMS), QRCoder

## 🏗️ Architecture Layers

### 1. Domain Layer (`EazyMenu.Domain`)
**Purpose:** Core business logic and entities - NO external dependencies

**Rules:**
- ✅ Contains only Entities, Enums, and Domain logic
- ✅ All entities inherit from `BaseEntity` (Id, CreatedAt, UpdatedAt, IsDeleted)
- ✅ Use `IAggregateRoot` marker for aggregate roots
- ✅ NO references to Infrastructure, Application, or any external packages
- ✅ Soft delete by default (IsDeleted flag)

**Entities:**
- `ApplicationUser` - Base user (not IdentityUser - that's in Infrastructure)
- `Restaurant`, `Subscription`, `Category`, `Product`
- `Order`, `OrderItem`, `Payment`, `Reservation`, `Notification`

**Example Entity:**
```csharp
public class Restaurant : BaseEntity, IAggregateRoot
{
    public string Name { get; set; } = string.Empty;
    public Guid OwnerId { get; set; }
    public virtual ApplicationUser Owner { get; set; } = null!;
    // ...
}
```

### 2. Application Layer (`EazyMenu.Application`)
**Purpose:** Use cases, CQRS commands/queries, interfaces

**Rules:**
- ✅ Define interfaces for repositories and services
- ✅ Use CQRS pattern with MediatR (Commands for writes, Queries for reads)
- ✅ DTOs and ViewModels go here
- ✅ AutoMapper profiles for mappings
- ✅ FluentValidation for input validation
- ✅ NO direct database access - only interfaces

**Structure:**
```
Application/
├── Common/
│   ├── Interfaces/ (IRepository, IUnitOfWork, ISmsService, etc.)
│   ├── Models/ (DTOs, ViewModels)
│   └── Mappings/ (AutoMapper profiles)
├── Features/
│   ├── Restaurants/
│   │   ├── Commands/ (CreateRestaurant, UpdateRestaurant)
│   │   └── Queries/ (GetRestaurantById, GetAllRestaurants)
│   ├── Products/
│   ├── Orders/
│   └── ...
```

**Example Command:**
```csharp
public class CreateRestaurantCommand : IRequest<Guid>
{
    public string Name { get; set; } = string.Empty;
    public string PhoneNumber { get; set; } = string.Empty;
    // ...
}

public class CreateRestaurantCommandHandler : IRequestHandler<CreateRestaurantCommand, Guid>
{
    private readonly IRepository<Restaurant> _repository;
    private readonly IUnitOfWork _unitOfWork;
    
    public async Task<Guid> Handle(CreateRestaurantCommand request, CancellationToken cancellationToken)
    {
        var restaurant = new Restaurant { /* map from request */ };
        await _repository.AddAsync(restaurant, cancellationToken);
        await _unitOfWork.SaveChangesAsync(cancellationToken);
        return restaurant.Id;
    }
}
```

### 3. Infrastructure Layer (`EazyMenu.Infrastructure`)
**Purpose:** Data access, external services implementation

**Rules:**
- ✅ `ApplicationDbContext` extends `IdentityDbContext<ApplicationIdentityUser, IdentityRole<Guid>, Guid>`
- ✅ Use Fluent API for entity configurations (IEntityTypeConfiguration)
- ✅ Implement repositories and UnitOfWork
- ✅ External service implementations (SMS, Payment, QRCode)
- ✅ Global query filters for soft delete
- ✅ Auto-set CreatedAt/UpdatedAt in SaveChangesAsync

**Key Files:**
- `Data/ApplicationDbContext.cs` - Main DbContext
- `Data/Configurations/` - Entity configurations
- `Repositories/Repository.cs` - Generic repository
- `Repositories/UnitOfWork.cs` - Transaction management
- `Services/` - External service implementations
- `Identity/ApplicationIdentityUser.cs` - Identity user (extends IdentityUser)

**Example Configuration:**
```csharp
public class RestaurantConfiguration : IEntityTypeConfiguration<Restaurant>
{
    public void Configure(EntityTypeBuilder<Restaurant> builder)
    {
        builder.HasKey(r => r.Id);
        builder.Property(r => r.Name).IsRequired().HasMaxLength(200);
        builder.HasIndex(r => r.Slug).IsUnique();
        
        builder.HasOne(r => r.Owner)
            .WithMany(u => u.Restaurants)
            .HasForeignKey(r => r.OwnerId)
            .OnDelete(DeleteBehavior.Restrict);
    }
}
```

### 4. Web Layer (`EazyMenu.Web`)
**Purpose:** Presentation layer - MVC controllers and views

**Rules:**
- ✅ MVC only (NO Razor Pages as per PRD)
- ✅ Use Areas for logical separation (Admin, Restaurant)
- ✅ Controllers are thin - delegate to MediatR
- ✅ Mobile-first responsive design
- ✅ NO inline styles or scripts
- ✅ Persian (RTL) + English (LTR) support

**Structure:**
```
Web/
├── Areas/
│   ├── Admin/ (System administration)
│   └── Restaurant/ (Restaurant owner panel)
├── Controllers/ (Public controllers)
├── Views/
│   ├── Shared/ (_Layout.cshtml, _AdminLayout.cshtml)
│   └── ...
├── wwwroot/
│   ├── css/
│   ├── js/
│   └── qrcodes/
```

**Example Controller:**
```csharp
[Area("Admin")]
[Authorize(Roles = "Admin")]
public class RestaurantController : Controller
{
    private readonly IMediator _mediator;
    
    public RestaurantController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    public async Task<IActionResult> Index()
    {
        var restaurants = await _mediator.Send(new GetAllRestaurantsQuery());
        return View(restaurants);
    }
    
    [HttpPost]
    public async Task<IActionResult> Create(CreateRestaurantCommand command)
    {
        var id = await _mediator.Send(command);
        return RedirectToAction(nameof(Index));
    }
}
```

## 📝 Coding Standards

### Naming Conventions
- **Classes/Interfaces:** PascalCase (`RestaurantService`, `IRepository`)
- **Methods:** PascalCase (`GetByIdAsync`, `CreateRestaurant`)
- **Variables/Parameters:** camelCase (`restaurantId`, `userName`)
- **Private fields:** `_camelCase` (`_context`, `_repository`)
- **Async methods:** Must end with `Async` suffix
- **Interfaces:** Must start with `I` prefix

### Language Usage
- **Code:** English (classes, methods, variables)
- **Comments:** Persian (فارسی) for business logic explanations
- **User-facing:** Persian (UI, messages, validation)
- **Database:** English column names

### File Organization
```
Feature/
├── FeatureCommand.cs (Command definition)
├── FeatureCommandHandler.cs (Handler)
├── FeatureCommandValidator.cs (FluentValidation)
└── FeatureDto.cs (Data Transfer Object)
```

## 🔒 Security & Best Practices

### Authentication & Authorization
- Use ASP.NET Core Identity for user management
- Role-based authorization: `Admin`, `RestaurantOwner`, `Customer`
- JWT tokens for API (if needed later)
- Strong password requirements (6+ chars, upper, lower, digit)

### Database
- **Always use async/await** for database operations
- Use `CancellationToken` in all async methods
- Implement soft delete (never hard delete)
- Use transactions for multi-table operations (UnitOfWork)
- Apply migrations: `dotnet ef migrations add MigrationName --startup-project ../EazyMenu.Web`

### Error Handling
- Use try-catch in service layer
- Return Result pattern or throw custom exceptions
- Log errors appropriately
- User-friendly Persian error messages

### Performance
- Use `.AsNoTracking()` for read-only queries
- Implement pagination for lists
- Use eager loading (`.Include()`) wisely
- Cache frequently accessed data

## 🧪 Testing Guidelines

### Unit Tests
- Test business logic in Domain/Application layers
- Mock dependencies (IRepository, IUnitOfWork)
- Use xUnit or NUnit
- Follow Arrange-Act-Assert pattern

### Integration Tests
- Test full request pipeline
- Use in-memory database or test database
- Test authentication/authorization

## 🚀 Development Workflow

### Adding a New Feature
1. **Define Entity** in Domain layer (if needed)
2. **Create Command/Query** in Application layer
3. **Create Handler** with business logic
4. **Add Validator** using FluentValidation
5. **Update DbContext** configuration (if new entity)
6. **Create Migration**: `dotnet ef migrations add FeatureName`
7. **Create Controller** in Web layer
8. **Create Views** with proper layout
9. **Test** thoroughly

### Running the Project
```powershell
# Build
dotnet build EazyMenu.sln

# Run migrations
cd src/EazyMenu.Infrastructure
dotnet ef database update --startup-project ../EazyMenu.Web

# Run application
cd ../EazyMenu.Web
dotnet run
```

## 📦 Package Management

### Common Packages
- **Domain:** None (pure .NET)
- **Application:** MediatR, AutoMapper, FluentValidation
- **Infrastructure:** EF Core, Identity, QRCoder, HttpClient
- **Web:** No additional (ASP.NET Core MVC included)

### Adding Packages
```powershell
cd src/EazyMenu.LayerName
dotnet add package PackageName
```

## 🌐 External Services

### Zarinpal (Payment)
- API: `https://api.zarinpal.com/pg/v4/`
- Config: `appsettings.json` → `Payment:Zarinpal:MerchantId`
- Service: `IPaymentService` → `ZarinpalPaymentService`

### Kavenegar (SMS)
- API: `https://api.kavenegar.com/v1/{ApiKey}/`
- Config: `appsettings.json` → `SMS:Kavenegar:ApiKey`
- Service: `ISmsService` → `KavenegarSmsService`

### QR Code
- Library: QRCoder
- Service: `IQRCodeService` → `QRCodeService`
- Storage: `wwwroot/qrcodes/{restaurantId}/`

## ⚠️ Important Notes

1. **Never break Clean Architecture rules:** Domain must not reference other layers
2. **Always use interfaces:** Controllers → MediatR → Interfaces → Implementations
3. **Persian comments:** Explain business logic in Persian for better understanding
4. **Async all the way:** Use async/await consistently
5. **Validation:** Validate inputs at Application layer (FluentValidation)
6. **Soft delete:** Use `IsDeleted` flag, don't hard delete records
7. **CQRS:** Separate read (Query) from write (Command) operations
8. **Connection String:** Update in `appsettings.json` before first run

## 📚 Key Files to Reference

- `NEXT_STEPS.md` - Implementation roadmap
- `README.md` - Project setup guide
- `Docs/PRD.MD` - Product requirements
- `Docs/UserStories/` - Feature specifications

## 🎨 UI/UX Guidelines

- **Mobile-first** responsive design
- **RTL** for Persian content
- **Bootstrap 5** or Tailwind CSS
- **No inline styles** - use CSS files
- **No inline scripts** - use JS files
- **Accessible** - ARIA labels, semantic HTML
- **Persian fonts** - IRANSans or Vazir

---

**Last Updated:** October 2, 2025
**Version:** 1.0
**Status:** ✅ Clean Architecture Base Complete

---
> Source: [Merkousha/EazyMeny-Main](https://github.com/Merkousha/EazyMeny-Main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
