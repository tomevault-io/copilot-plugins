## truwit-starter-template

> Here's a comprehensive `.cursorrules` file for your Angular + .NET application that follows best practices for modularity, extensibility, and maintainability:

Here's a comprehensive `.cursorrules` file for your Angular + .NET application that follows best practices for modularity, extensibility, and maintainability:

```markdown
# Cursor Rules for Angular + .NET Application

## Project Overview
This is a full-stack application with Angular frontend and .NET backend services. Follow these rules to maintain consistency, quality, and best practices across the codebase.

---

## General Principles

### Code Quality
- Write clean, readable, and self-documenting code
- Follow SOLID principles
- Apply DRY (Don't Repeat Yourself) principle
- Use meaningful names for variables, methods, and classes
- Keep functions small and focused on a single responsibility
- Prefer composition over inheritance
- Write code that is testable and maintainable

### Architecture
- Maintain clear separation of concerns
- Follow modular architecture patterns
- Keep business logic separate from presentation logic
- Use dependency injection consistently
- Implement proper error handling and logging
- Design for scalability and extensibility

---

## Angular Frontend Rules

### Project Structure
```
src/
├── app/
│   ├── core/                 # Singleton services, guards, interceptors
│   │   ├── services/
│   │   ├── guards/
│   │   ├── interceptors/
│   │   └── models/
│   ├── shared/               # Shared components, directives, pipes
│   │   ├── components/
│   │   ├── directives/
│   │   ├── pipes/
│   │   └── utils/
│   ├── features/             # Feature modules (lazy-loaded)
│   │   └── feature-name/
│   │       ├── components/
│   │       ├── services/
│   │       ├── models/
│   │       └── feature-name.module.ts
│   ├── layout/               # Layout components (header, footer, sidebar)
│   └── app.config.ts
├── assets/
├── environments/
└── styles/
```

### Module Organization
- Create feature modules for distinct functionality
- Implement lazy loading for feature modules
- Keep CoreModule for singleton services (import only in AppModule)
- Keep SharedModule for reusable components (import in feature modules)
- Never import SharedModule in CoreModule
- Use standalone components for new development when appropriate

### Component Best Practices
- Use OnPush change detection strategy by default
- Implement OnDestroy and unsubscribe from observables
- Keep components focused on presentation logic
- Delegate business logic to services
- Use smart/container and dumb/presentational component pattern
- Limit template complexity; use pipes and computed properties
- Use trackBy functions in *ngFor loops
- Prefer async pipe over manual subscriptions

```typescript
// Good: Smart Component
@Component({
  selector: 'app-user-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  standalone: true,
  imports: [CommonModule, UserCardComponent]
})
export class UserListComponent implements OnInit, OnDestroy {
  users$ = this.userService.getUsers();
  
  constructor(private userService: UserService) {}
  
  trackByUserId(index: number, user: User): number {
    return user.id;
  }
}

// Good: Dumb Component
@Component({
  selector: 'app-user-card',
  changeDetection: ChangeDetectionStrategy.OnPush,
  standalone: true
})
export class UserCardComponent {
  @Input({ required: true }) user!: User;
  @Output() userSelected = new EventEmitter<User>();
}
```

### Service Best Practices
- Create services for business logic and data management
- Use providedIn: 'root' for singleton services
- Implement proper error handling
- Return Observables from HTTP calls
- Use RxJS operators effectively
- Create facade services for complex state management

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private readonly apiUrl = environment.apiUrl;
  private usersSubject = new BehaviorSubject<User[]>([]);
  
  users$ = this.usersSubject.asObservable();
  
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(`${this.apiUrl}/users`).pipe(
      tap(users => this.usersSubject.next(users)),
      catchError(this.handleError)
    );
  }
  
  private handleError(error: HttpErrorResponse): Observable<never> {
    // Log error and return user-friendly message
    return throwError(() => new Error('Failed to fetch users'));
  }
}
```

### State Management
- Use signals for reactive state (Angular 16+)
- Consider NgRx or Akita for complex state management
- Keep state immutable
- Use selectors for derived state
- Implement optimistic updates where appropriate

### HTTP and API Communication
- Create a base HTTP service with common functionality
- Use interceptors for authentication, error handling, and logging
- Implement retry logic for failed requests
- Type all HTTP responses
- Use environment files for API URLs

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    
    if (token) {
      req = req.clone({
        setHeaders: { Authorization: `Bearer ${token}` }
      });
    }
    
    return next.handle(req);
  }
}
```

### Routing
- Implement lazy loading for feature modules
- Use route guards for authentication and authorization
- Implement route resolvers for data preloading
- Use typed route parameters
- Implement proper 404 handling

### Forms
- Use reactive forms over template-driven forms
- Create reusable form controls
- Implement custom validators
- Use FormBuilder for cleaner code
- Handle form errors gracefully

```typescript
export class UserFormComponent {
  userForm = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(3)]],
    email: ['', [Validators.required, Validators.email]],
    age: [null, [Validators.required, Validators.min(18)]]
  });
  
  constructor(private fb: FormBuilder) {}
  
  onSubmit(): void {
    if (this.userForm.valid) {
      // Handle submission
    }
  }
}
```

### Styling
- Use SCSS for styling
- Follow BEM or similar naming convention
- Create reusable style mixins and variables
- Use CSS custom properties for theming
- Implement responsive design with mobile-first approach
- Avoid inline styles

### TypeScript
- Enable strict mode in tsconfig.json
- Use interfaces for data models
- Prefer const over let
- Use type guards and discriminated unions
- Avoid any type; use unknown if type is uncertain
- Use enums for fixed sets of values

```typescript
// Good: Strongly typed models
export interface User {
  id: number;
  name: string;
  email: string;
  role: UserRole;
}

export enum UserRole {
  Admin = 'ADMIN',
  User = 'USER',
  Guest = 'GUEST'
}

export type ApiResponse<T> = {
  data: T;
  message: string;
  status: number;
};
```

### Testing
- Write unit tests for components, services, and pipes
- Aim for 80%+ code coverage
- Use TestBed for component testing
- Mock dependencies in tests
- Test user interactions and edge cases
- Use descriptive test names

---

## .NET Backend Rules

### Project Structure
```
Solution/
├── src/
│   ├── API/                          # Presentation layer
│   │   ├── Controllers/
│   │   ├── Filters/
│   │   ├── Middleware/
│   │   └── Program.cs
│   ├── Application/                   # Business logic layer
│   │   ├── Services/
│   │   ├── Interfaces/
│   │   ├── DTOs/
│   │   ├── Validators/
│   │   └── Mappings/
│   ├── Domain/                        # Domain models
│   │   ├── Entities/
│   │   ├── ValueObjects/
│   │   ├── Enums/
│   │   └── Interfaces/
│   ├── Infrastructure/                # Data access & external services
│   │   ├── Data/
│   │   ├── Repositories/
│   │   └── ExternalServices/
├── tests/
│   ├── UnitTests/
│   └── IntegrationTests/
```

### Architecture Patterns
- Use Clean Architecture or Onion Architecture
- Implement Repository pattern for data access
- Use Unit of Work pattern for transaction management
- Apply CQRS for complex domains
- Use MediatR for command/query handling
- Implement Specification pattern for complex queries

### API Best Practices
- Use RESTful conventions
- Version your API (e.g., /api/v1/users)
- Return appropriate HTTP status codes
- Implement proper error handling
- Use async/await consistently
- Implement pagination for list endpoints
- Use DTOs for request/response models

```csharp
[ApiController]
[Route("api/v1/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UsersController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet]
    [ProducesResponseType(typeof(PagedResult<UserDto>), StatusCodes.Status200OK)]
    public async Task<ActionResult<PagedResult<UserDto>>> GetUsers(
        [FromQuery] PaginationParams paginationParams)
    {
        var users = await _userService.GetUsersAsync(paginationParams);
        return Ok(users);
    }
    
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(UserDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<UserDto>> GetUser(int id)
    {
        var user = await _userService.GetUserByIdAsync(id);
        
        if (user == null)
            return NotFound();
        
        return Ok(user);
    }
}
```

### Dependency Injection
- Register services in Program.cs or use extension methods
- Use appropriate service lifetimes (Singleton, Scoped, Transient)
- Inject interfaces, not concrete implementations
- Create service collection extensions for clean registration

```csharp
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddApplicationServices(
        this IServiceCollection services)
    {
        services.AddScoped<IUserService, UserService>();
        services.AddScoped<IUserRepository, UserRepository>();
        return services;
    }
}
```

### Data Access
- Use Entity Framework Core with repository pattern
- Implement database migrations
- Use async methods for database operations
- Implement soft deletes where appropriate
- Use navigation properties effectively
- Create database indexes for frequently queried columns

```csharp
public class ApplicationDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(ApplicationDbContext).Assembly);
        
        // Global query filters
        modelBuilder.Entity<User>().HasQueryFilter(u => !u.IsDeleted);
    }
}

public class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.HasKey(u => u.Id);
        builder.Property(u => u.Email).IsRequired().HasMaxLength(255);
        builder.HasIndex(u => u.Email).IsUnique();
    }
}
```

### Error Handling
- Use global exception handling middleware
- Create custom exception types
- Log errors appropriately
- Return consistent error responses
- Don't expose sensitive information in error messages

```csharp
public class GlobalExceptionHandlerMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionHandlerMiddleware> _logger;
    
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An unhandled exception occurred");
            await HandleExceptionAsync(context, ex);
        }
    }
    
    private static Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        var response = exception switch
        {
            NotFoundException => (StatusCodes.Status404NotFound, "Resource not found"),
            ValidationException => (StatusCodes.Status400BadRequest, exception.Message),
            _ => (StatusCodes.Status500InternalServerError, "An error occurred")
        };
        
        context.Response.StatusCode = response.Item1;
        return context.Response.WriteAsJsonAsync(new { error = response.Item2 });
    }
}
```

### Validation
- Use FluentValidation for complex validation
- Validate at the API boundary
- Create reusable validators
- Return detailed validation errors

```csharp
public class CreateUserDtoValidator : AbstractValidator<CreateUserDto>
{
    public CreateUserDtoValidator()
    {
        RuleFor(x => x.Email)
            .NotEmpty()
            .EmailAddress()
            .MaximumLength(255);
        
        RuleFor(x => x.Name)
            .NotEmpty()
            .MinimumLength(3)
            .MaximumLength(100);
    }
}
```

### Authentication & Authorization
- Use JWT tokens for authentication
- Implement role-based or policy-based authorization
- Secure sensitive endpoints
- Use HTTPS only
- Implement refresh tokens
- Store passwords securely (hashed with salt)

### Configuration
- Use appsettings.json and environment variables
- Use IOptions pattern for strongly typed configuration
- Don't commit secrets to source control
- Use Azure Key Vault or similar for production secrets

```csharp
public class JwtSettings
{
    public string Secret { get; set; }
    public int ExpirationMinutes { get; set; }
}

// In Program.cs
services.Configure<JwtSettings>(configuration.GetSection("JwtSettings"));
```

### Logging
- Use structured logging (Serilog recommended)
- Log at appropriate levels (Information, Warning, Error)
- Include correlation IDs for request tracking
- Don't log sensitive information
- Use log enrichers for additional context

### Testing
- Write unit tests for business logic
- Write integration tests for API endpoints
- Use xUnit or NUnit
- Mock dependencies using Moq or NSubstitute
- Aim for 80%+ code coverage
- Test edge cases and error scenarios

```csharp
public class UserServiceTests
{
    private readonly Mock<IUserRepository> _mockRepository;
    private readonly UserService _service;
    
    public UserServiceTests()
    {
        _mockRepository = new Mock<IUserRepository>();
        _service = new UserService(_mockRepository.Object);
    }
    
    [Fact]
    public async Task GetUserByIdAsync_WhenUserExists_ReturnsUser()
    {
        // Arrange
        var expectedUser = new User { Id = 1, Name = "Test" };
        _mockRepository.Setup(r => r.GetByIdAsync(1))
            .ReturnsAsync(expectedUser);
        
        // Act
        var result = await _service.GetUserByIdAsync(1);
        
        // Assert
        Assert.NotNull(result);
        Assert.Equal(expectedUser.Id, result.Id);
    }
}
```

### Performance
- Use asynchronous programming
- Implement caching where appropriate (Redis, memory cache)
- Use pagination for large datasets
- Optimize database queries (use includes, projections)
- Implement response compression
- Use HTTP/2

---

## Cross-Cutting Concerns

### Security
- Implement CORS properly
- Validate all inputs
- Use parameterized queries to prevent SQL injection
- Implement rate limiting
- Keep dependencies updated
- Regular security audits

### Documentation
- Use XML comments for C# code
- Use JSDoc for TypeScript code
- Maintain API documentation (Swagger/OpenAPI)
- Document complex business logic
- Keep README files updated

### Version Control
- Use meaningful commit messages (conventional commits)
- Create feature branches
- Use pull requests for code review
- Keep branches short-lived
- Don't commit secrets, build artifacts, or node_modules

### CI/CD
- Implement automated testing in pipeline
- Use linting and code analysis tools
- Automate deployment process
- Implement environment-specific configurations
- Use semantic versioning

---

## Code Review Checklist

Before submitting code for review, ensure:
- [ ] Code follows established patterns and conventions
- [ ] All tests pass
- [ ] Code is properly documented
- [ ] No console.log or debugging code remains
- [ ] Error handling is implemented
- [ ] No security vulnerabilities introduced
- [ ] Performance considerations addressed
- [ ] Code is DRY and follows SOLID principles
- [ ] Accessibility requirements met (frontend)
- [ ] API contracts are backward compatible

---

## Common Pitfalls to Avoid

### Angular
- Don't forget to unsubscribe from observables
- Don't mutate state directly
- Avoid logic in templates
- Don't inject services in shared modules
- Don't use ElementRef for DOM manipulation without sanitization

### .NET
- Don't block async code with .Result or .Wait()
- Don't catch exceptions without proper handling
- Don't return IQueryable from repositories
- Don't use Select N+1 queries
- Don't forget to dispose of disposable resources

---

## When to Ask for Help
- When implementing a new architectural pattern
- When unsure about security implications
- When performance optimization is needed
- When dealing with complex business logic
- When external integrations are required
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rvadapally) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
