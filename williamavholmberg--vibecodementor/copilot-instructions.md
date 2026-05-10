## backend

> Everytime working with backend related stuff

# Backend Architecture - Vertical Slices + CQRS

## 🎯 Philosophy: PRAGMATIC FIRST
- **Speed over purity** - Ship working features, refactor later
- **MediatR for consistency** - Predictable command/query patterns
- **ASP.NET Core Identity** - Use framework features, avoid custom auth
- **Result pattern** - Explicit success/failure handling
- **ALWAYS RETURN TYPES IN CONTROLLERS** - In order for the swagger schema generator to work, (frontend uses orval) WE NEED TO RETURN ACTUAL TYPES; IF NOT, FRONTEND TYPES WILL NOT WORK!! THIS IS SUPER IMPORTANT!

## 📁 Structure
```
api/Source/
├── Features/           # 🎯 VERTICAL SLICES
│   ├── Users/         # User management slice
│   │   ├── Commands/  # CreateUser, UpdateUser, DeleteUser
│   │   ├── Queries/   # GetUser, GetAllUsers
│   │   ├── Controllers/ # UsersController (API endpoints)
│   │   ├── Events/    # UserCreated (domain events)
│   │   ├── EventHandlers/ # SendWelcomeEmail, etc.
│   │   └── Models/    # User entity, DTOs
│   └── [Feature]/     # Self-contained feature slices
├── Infrastructure/    # Shared services, DB context
├── Shared/           # CQRS interfaces, Result pattern
```

## 🏗️ Feature Slice Pattern
```
Features/FeatureName/
├── Commands/          # State-changing operations
├── Queries/          # Data retrieval operations  
├── Controllers/      # API endpoints
├── Events/          # Domain events (past tense)
├── EventHandlers/   # React to domain events
└── Models/          # Entities, DTOs, requests
```

## 🔧 Tech Stack
- **.NET 9** + **ASP.NET Core** + **MediatR** + **EF Core**
- **PostgreSQL** with Code First migrations
- **ASP.NET Core Identity** + **JWT** for auth
- **Hangfire** for background jobs
- **SignalR** for real-time communication

## 📋 Core Patterns

### Commands (State Changes)
```csharp
// Command with Result pattern
public record CreateUserCommand(string Email, string Password) : ICommand<Result<CreateUserResponse>>;

// Handler
public class CreateUserCommandHandler : ICommandHandler<CreateUserCommand, Result<CreateUserResponse>>
{
    public async Task<Result<CreateUserResponse>> Handle(CreateUserCommand request, CancellationToken cancellationToken)
    {
        // Validation
        var existingUser = await _userManager.FindByEmailAsync(request.Email);
        if (existingUser != null)
            return Result.Failure<CreateUserResponse>("User already exists");

        // Business logic
        var user = new User { UserName = request.Email, Email = request.Email };
        var result = await _userManager.CreateAsync(user, request.Password);
        
        if (!result.Succeeded)
            return Result.Failure<CreateUserResponse>("Creation failed");

        // Success
        return Result.Success(new CreateUserResponse(user.Id, user.Email));
    }
}
```

### Queries (Data Retrieval)
```csharp
public record GetUserQuery(string UserId) : IQuery<Result<UserResponse>>;

public class GetUserQueryHandler : IQueryHandler<GetUserQuery, Result<UserResponse>>
{
    // Implementation
}
```

### Controllers (API Endpoints)
```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize] // JWT auth required
public class UsersController : ControllerBase
{
    private readonly IMediator _mediator;

    [HttpPost]
    public async Task<IActionResult> CreateUser([FromBody] CreateUserRequest request)
    {
        var command = new CreateUserCommand(request.Email, request.Password);
        var result = await _mediator.Send(command);

        if (!result.IsSuccess)
            return BadRequest(new { error = result.Error });

        return CreatedAtAction(nameof(GetUser), new { id = result.Value.UserId }, result.Value);
    }
}
```

### Domain Events
```csharp
// Event (past tense)
public record UserCreated(string UserId, string Email, DateTime OccurredAt) : IDomainEvent;

// Event Handler
public class SendWelcomeEmailHandler : IEventHandler<UserCreated>
{
    public async Task Handle(UserCreated notification, CancellationToken cancellationToken)
    {
        await _emailService.SendEmailAsync(notification.Email, "Welcome!", "Welcome message");
    }
}
```

## 🔐 Authentication Pattern

### ASP.NET Core Identity Setup
- **User Entity**: Custom User class inheriting IdentityUser
- **JWT Tokens**: Generated via JwtTokenService
- **OTP Flow**: Identity token providers for email verification
- **Claims**: Standard + custom claims in JWT

### Auth Flow
```csharp
// Login with OTP
[HttpPost("send-otp")]
public async Task<IActionResult> SendOtp([FromBody] SendOtpRequest request)
{
    var command = new SendOtpCommand(request.Email);
    var result = await _mediator.Send(command);
    return Ok(result.Value);
}

[HttpPost("verify-otp")]
public async Task<IActionResult> VerifyOtp([FromBody] VerifyOtpRequest request)
{
    var command = new VerifyOtpCommand(request.Email, request.Otp);
    var result = await _mediator.Send(command);
    
    if (!result.IsSuccess)
        return BadRequest(new { error = result.Error });

    // Return JWT token
    return Ok(result.Value);
}
```

## 🗄️ Database Patterns

### EF Core Setup
```csharp
public class ApplicationDbContext : IdentityDbContext<User>
{
    public DbSet<ChatMessage> ChatMessages { get; set; }
    
    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder); // ASP.NET Core Identity tables
        
        // Configure entities
        builder.Entity<ChatMessage>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.HasIndex(e => e.Timestamp);
            entity.Property(e => e.Timestamp)
                .HasColumnType("timestamp with time zone")
                .HasDefaultValueSql("CURRENT_TIMESTAMP AT TIME ZONE 'UTC'");
        });
    }
}
```

### Migrations
```bash
dotnet ef migrations add [MigrationName]
dotnet ef database update
```

## 🔄 Real-time with SignalR

### Hub Pattern
```csharp
[Authorize] // JWT auth
public class ChatHub : Hub
{
    public async Task JoinGroup(string groupName)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, groupName);
    }

    public async Task SendMessage(string message)
    {
        var userName = Context.User?.Identity?.Name;
        await Clients.All.SendAsync("ReceiveMessage", userName, message);
    }
}
```

## ⚙️ Background Jobs (Hangfire)

### Job Service Pattern
```csharp
public class EmailBackgroundService
{
    public async Task SendWelcomeEmailJob(string userId, string email)
    {
        // Heavy email processing
        await _emailService.SendTemplatedEmailAsync(email, "welcome-template");
    }
}

// Trigger from event handler
public class UserCreatedHandler : IEventHandler<UserCreated>
{
    public async Task Handle(UserCreated notification, CancellationToken cancellationToken)
    {
        // Queue background job
        BackgroundJob.Enqueue<EmailBackgroundService>(x => 
            x.SendWelcomeEmailJob(notification.UserId, notification.Email));
    }
}
```

## 🚀 Development Patterns

### Feature Development Flow
1. **Commands/Queries**: Define operations
2. **Handlers**: Implement business logic
3. **Controllers**: Add API endpoints
4. **Events**: Define domain events (if needed)
5. **EventHandlers**: React to events
6. **Tests**: Unit test handlers

### Error Handling
```csharp
// Result pattern - no exceptions for business logic
if (user == null)
    return Result.Failure<UserResponse>("User not found");

// Use exceptions only for technical errors
try 
{
    await _dbContext.SaveChangesAsync();
}
catch (DbUpdateException ex)
{
    _logger.LogError(ex, "Database error saving user");
    return Result.Failure<UserResponse>("Database error occurred");
}
```

## 🎯 Key Rules
- ✅ Use **Result<T>** pattern, avoid exceptions for business logic
- ✅ Commands for state changes, Queries for data retrieval
- ✅ Controllers are thin - just mediate to handlers
- ✅ Domain events (past tense) for cross-feature communication
- ✅ ASP.NET Core Identity for auth - don't reinvent
- ✅ Background jobs for heavy operations
- ✅ Features CAN reference each other via events
- ✅ Keep handlers focused and single-purpose

## 🚨 Critical Gotchas
- **EF migrations** - Always test locally before deploy
- **JWT expiry** - Handle gracefully in frontend
- **Domain events** - Use for async cross-feature communication
- **Background jobs** - Don't block HTTP requests
- **UTC timestamps** - Always store/return UTC
- **Result pattern** - Check IsSuccess before accessing Value

## 📡 External Services
- **Resend**: Email delivery via IEmailService
- **Cloudflare R2**: File storage via IFileStorageService  
- **Hangfire**: Background job processing
- **SignalR**: Real-time communication

This architecture ensures predictable, maintainable, and AI-friendly backend development! 🚀 

---
> Source: [WilliamAvHolmberg/vibecodementor](https://github.com/WilliamAvHolmberg/vibecodementor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
