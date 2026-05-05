## code-agents

> This repository contains 86 specialized AI agents for full-stack development across multiple technologies. Use these instructions to leverage the agents effectively with GitHub Copilot.

# GitHub Copilot Instructions

This repository contains 86 specialized AI agents for full-stack development across multiple technologies. Use these instructions to leverage the agents effectively with GitHub Copilot.

## Repository Structure

```
agents/
├── frontend-ai-agents/          # 13 Frontend agents (React, Angular, Vue, etc.)
├── backend-ai-agents/           # 48 Backend agents (Python, Go, Java, Node.js, Ruby, Rust)
├── dotnet-ai-agents/backend/    # 11 .NET backend agents
└── flutter-ai-agent/            # 14 Flutter development agents
```

**Total: 86 Specialized Professional Agents**

## Using with GitHub Copilot

### Inline Comments for Context

When working on code, use comments to reference specific agents:

```typescript
// Following react-component-designer patterns for accessible components
// Using frontend-performance-optimizer best practices for Core Web Vitals
```

```python
# Following python-clean-architecture patterns for domain layer
# Using python-security-specialist best practices for input validation
```

```csharp
// Following dotnet-api-designer patterns for RESTful API
// Using dotnet-security-implementer best practices
```

```dart
// Following flutter-use-cases pattern with validation
// Using flutter-ui guidelines - UI only invokes Cubit
```

### Workspace Instructions

Add this to your `.vscode/settings.json`:

```json
{
  "github.copilot.advanced": {
    "instructionsFile": "agents/.github/copilot-instructions.md"
  }
}
```

## Agent Patterns by Language

### Frontend (React, Angular, Vue)

#### React Architecture (`react-architect`)
```typescript
// Pattern: Feature-based structure with Clean Architecture
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── api/
│   └── dashboard/
├── shared/
└── lib/
```

#### React Components (`react-component-designer`)
```typescript
// Pattern: Accessible compound components
interface SelectProps {
  value: string
  onChange: (value: string) => void
  children: React.ReactNode
}

export function Select({ value, onChange, children }: SelectProps) {
  return (
    <div role="combobox" aria-expanded={isOpen}>
      {children}
    </div>
  )
}
```

#### State Management (`react-state-manager`)
```typescript
// Pattern: Zustand for simple state
import { create } from 'zustand'

interface Store {
  count: number
  increment: () => void
}

export const useStore = create<Store>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}))

// Pattern: TanStack Query for server state
const { data, isLoading } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
})
```

#### Tailwind CSS (`tailwind-specialist`)
```typescript
// Pattern: Responsive design with custom variants
<div className="
  p-4 md:p-6 lg:p-8
  bg-white dark:bg-gray-800
  rounded-lg shadow-md
  hover:shadow-lg transition-shadow
">
  {/* Content */}
</div>
```

#### Performance Optimization (`frontend-performance-optimizer`)
```typescript
// Pattern: Virtual scrolling for large lists
const virtualizer = useVirtualizer({
  count: items.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 50,
  overscan: 5,
})

// Pattern: Lazy loading routes
const Dashboard = lazy(() => import('./pages/Dashboard'))
```

#### Testing (`frontend-tester`, `e2e-tester`)
```typescript
// Pattern: Testing Library for components
it('should submit form with valid data', async () => {
  render(<LoginForm />)

  await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com')
  await userEvent.click(screen.getByRole('button', { name: /login/i }))

  expect(await screen.findByText(/welcome/i)).toBeInTheDocument()
})

// Pattern: Playwright for E2E
test('user can complete checkout flow', async ({ page }) => {
  await page.goto('/products')
  await page.click('text=Add to cart')
  await page.click('text=Checkout')
  await expect(page).toHaveURL(/checkout/)
})
```

### Backend (Python, Go, Java, Node.js, Ruby, Rust)

#### Python FastAPI (`python-fastapi-developer`, `python-clean-architecture`)
```python
# Pattern: Clean Architecture with dependency injection
class CreateUserUseCase:
    def __init__(self, user_repository: UserRepository):
        self._repository = user_repository

    async def execute(self, name: str, email: str) -> User:
        # Validation
        if not email or '@' not in email:
            raise ValidationError("Invalid email")

        # Business logic
        user = User(name=name, email=email)
        return await self._repository.create(user)

# Pattern: FastAPI endpoint with validation
@router.post("/users", response_model=UserResponse, status_code=201)
async def create_user(
    user_data: UserCreate,
    use_case: CreateUserUseCase = Depends(get_create_user_use_case)
):
    user = await use_case.execute(name=user_data.name, email=user_data.email)
    return UserResponse.from_entity(user)
```

#### Go REST API (`go-rest-api-developer`, `go-clean-architecture`)
```go
// Pattern: Repository pattern with interfaces
type UserRepository interface {
    Create(ctx context.Context, user *User) error
    FindByID(ctx context.Context, id string) (*User, error)
}

type userRepository struct {
    db *sql.DB
}

// Pattern: Handler with proper error handling
func (h *UserHandler) CreateUser(c *gin.Context) {
    var req CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    user, err := h.userService.Create(c.Request.Context(), req.Name, req.Email)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusCreated, user)
}
```

#### Java Spring Boot (`spring-boot-developer`, `spring-architect`)
```java
// Pattern: Layered architecture with DTOs
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public UserResponse createUser(CreateUserRequest request) {
        // Validation
        if (request.getEmail() == null || !request.getEmail().contains("@")) {
            throw new ValidationException("Invalid email");
        }

        // Business logic
        User user = new User();
        user.setName(request.getName());
        user.setEmail(request.getEmail());

        User savedUser = userRepository.save(user);
        return UserResponse.fromEntity(savedUser);
    }
}

// Pattern: REST controller with proper status codes
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public UserResponse createUser(@Valid @RequestBody CreateUserRequest request) {
        return userService.createUser(request);
    }
}
```

#### Node.js NestJS (`nestjs-developer`, `nestjs-architect`)
```typescript
// Pattern: Module-based architecture with DI
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
  ) {}

  async createUser(createUserDto: CreateUserDto): Promise<User> {
    // Validation
    if (!createUserDto.email.includes('@')) {
      throw new BadRequestException('Invalid email');
    }

    // Business logic
    const user = this.userRepository.create(createUserDto);
    return await this.userRepository.save(user);
  }
}

// Pattern: Controller with validation pipes
@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(@Body() createUserDto: CreateUserDto): Promise<User> {
    return await this.userService.createUser(createUserDto);
  }
}
```

#### Ruby on Rails (`rails-developer`, `rails-architect`)
```ruby
# Pattern: Service objects for business logic
class CreateUserService
  def initialize(user_params)
    @user_params = user_params
  end

  def call
    # Validation
    raise ValidationError, 'Invalid email' unless valid_email?

    # Business logic
    user = User.new(@user_params)
    user.save!
    user
  end

  private

  def valid_email?
    @user_params[:email].include?('@')
  end
end

# Pattern: Controller with service objects
class UsersController < ApplicationController
  def create
    service = CreateUserService.new(user_params)
    @user = service.call

    render json: @user, status: :created
  rescue ValidationError => e
    render json: { error: e.message }, status: :unprocessable_entity
  end

  private

  def user_params
    params.require(:user).permit(:name, :email)
  end
end
```

#### Rust Actix (`rust-actix-developer`, `rust-clean-architecture`)
```rust
// Pattern: Domain-driven design with Result types
pub struct CreateUserUseCase {
    repository: Arc<dyn UserRepository>,
}

impl CreateUserUseCase {
    pub async fn execute(&self, name: String, email: String) -> Result<User, AppError> {
        // Validation
        if !email.contains('@') {
            return Err(AppError::Validation("Invalid email".to_string()));
        }

        // Business logic
        let user = User { name, email };
        self.repository.create(user).await
    }
}

// Pattern: Actix handler with proper error handling
#[post("/users")]
async fn create_user(
    user_data: web::Json<CreateUserRequest>,
    use_case: web::Data<CreateUserUseCase>,
) -> Result<HttpResponse, AppError> {
    let user = use_case
        .execute(user_data.name.clone(), user_data.email.clone())
        .await?;

    Ok(HttpResponse::Created().json(user))
}
```

### .NET Backend

#### API Design (`dotnet-api-designer`)
```csharp
// Pattern: RESTful endpoints with proper status codes
[HttpGet("{id}")]
[ProducesResponseType(typeof(UserResponse), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<ActionResult<UserResponse>> GetUser(int id)
```

#### Security (`dotnet-security-implementer`)
```csharp
// Pattern: Input validation with allowlist approach
public bool IsValidUsername(string username)
{
    return Regex.IsMatch(username, @"^[a-zA-Z0-9_]{3,20}$");
}
```

#### EF Core (`dotnet-ef-specialist`)
```csharp
// Pattern: Parameterized queries
var users = await context.Users
    .Where(u => u.Username == username)
    .ToListAsync();
```

### Flutter

#### Domain Layer (`flutter-domain`, `flutter-use-cases`)
```dart
// Pattern: Entity with Equatable
class UserEntity extends Equatable {
  final String id;
  final String name;

  const UserEntity({required this.id, required this.name});

  @override
  List<Object?> get props => [id, name];
}

// Pattern: UseCase with validation (ONLY layer that validates!)
class CreateUserUseCase {
  Future<Either<Failure, UserEntity>> call({
    required String name,
    required String email,
  }) async {
    // Validate first
    if (name.trim().isEmpty) {
      return const Left(ValidationFailure(message: 'Name is required'));
    }
    // Then call repository
    return await _repository.createUser(name: name, email: email);
  }
}
```

#### Data Layer (`flutter-data-layer`, `flutter-repository`)
```dart
// Pattern: Model extends Entity
class UserModel extends UserEntity {
  const UserModel({required super.id, required super.name});

  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'] as String,
      name: json['name'] as String,
    );
  }
}

// Pattern: Repository with Either
Future<Either<Failure, List<UserEntity>>> getUsers() async {
  try {
    final models = await _remoteDataSource.getUsers();
    return Right(models.map((m) => m.toEntity()).toList());
  } on ServerException catch (e) {
    return Left(ServerFailure(message: e.message));
  }
}
```

#### Presentation Layer (`flutter-state`, `flutter-ui`)
```dart
// Pattern: Cubit with specific states
abstract class UsersState extends Equatable {
  const UsersState();
}

class UsersLoading extends UsersState {
  const UsersLoading();
}

class UsersLoaded extends UsersState {
  final List<UserEntity> users;
  const UsersLoaded({required this.users});
  @override
  List<Object?> get props => [users];
}

// Pattern: UI only invokes Cubit (NEVER UseCase directly!)
ElevatedButton(
  onPressed: () {
    context.read<UsersCubit>().createUser(
      name: nameController.text,
      email: emailController.text,
    );
  },
  child: const Text('Save'),
)
```

#### Design System (`flutter-design-system`)
```dart
// Pattern: Always use design tokens
Container(
  padding: EdgeInsets.all(AppSpacing.md),
  decoration: BoxDecoration(
    color: AppColors.surface,
    borderRadius: BorderRadius.circular(AppBorderRadius.md),
  ),
  child: Text(
    'Hello',
    style: AppTypography.bodyLarge,
  ),
)
```

## Critical Rules for Copilot

### Frontend
- ✅ Use TypeScript strict mode
- ✅ Implement proper accessibility (WCAG 2.1 AA)
- ✅ Optimize for Core Web Vitals (LCP < 2.5s, FID < 100ms, CLS < 0.1)
- ✅ Use lazy loading for routes and heavy components
- ✅ Implement proper error boundaries
- ✅ Use semantic HTML elements
- ✅ Follow mobile-first responsive design
- ✅ Avoid prop drilling (use Context or state management)
- ✅ Write comprehensive tests (unit + integration + E2E)
- ✅ Use CSS Modules or Tailwind for styling (avoid inline styles)

### Backend (All Languages)
- ✅ Use parameterized queries (NEVER string concatenation in SQL)
- ✅ Validate ALL user input with allowlist approach
- ✅ Use async/await for I/O operations
- ✅ Return proper HTTP status codes
- ✅ Implement proper error handling and logging
- ✅ Hash passwords with bcrypt/Argon2 (NEVER plain text)
- ✅ Implement rate limiting
- ✅ Use environment variables for configuration
- ✅ Write comprehensive tests (unit + integration + E2E)
- ✅ Follow Clean Architecture / Hexagonal Architecture

### .NET
- ✅ Use parameterized queries (NEVER string concatenation in SQL)
- ✅ Validate input with allowlist approach
- ✅ Use async/await for I/O operations
- ✅ Return proper HTTP status codes
- ✅ Implement security headers

### Flutter
- ✅ Clean Architecture flow: UI → Cubit → UseCase → Repository → DataSource
- ✅ Validation ONLY in UseCases (never in UI or Cubit)
- ✅ UI ONLY invokes Cubit methods
- ✅ Use Either<Failure, Success> for error handling
- ✅ All new pages MUST have routes in GoRouter
- ✅ All new classes MUST be registered in DI
- ✅ Use design tokens (AppColors, AppSpacing, etc) - NO hardcoded values
- ✅ Every UI action must be functional (NO empty onPressed)

## Quality Checks

### Frontend
```bash
# TypeScript check
npm run type-check

# Linting
npm run lint

# Testing
npm run test
npm run test:e2e

# Build check
npm run build

# Bundle analysis
npm run build:analyze
```

### Backend

#### Python
```bash
# Type checking
mypy .

# Linting
ruff check .

# Testing
pytest --cov

# Security scan
bandit -r .
```

#### Go
```bash
# Format check
gofmt -l .

# Linting
golangci-lint run

# Testing
go test ./... -cover

# Security scan
gosec ./...
```

#### Java
```bash
# Build
./mvnw clean package

# Testing
./mvnw test

# Code quality
./mvnw checkstyle:check
```

#### Node.js
```bash
# Type checking
npm run type-check

# Linting
npm run lint

# Testing
npm run test

# Security audit
npm audit
```

#### Ruby
```bash
# Linting
rubocop

# Testing
bundle exec rspec

# Security scan
bundle audit
```

#### Rust
```bash
# Format check
cargo fmt --check

# Linting
cargo clippy -- -D warnings

# Testing
cargo test

# Security audit
cargo audit
```

### .NET
```bash
dotnet build
dotnet test
dotnet format --verify-no-changes
```

### Flutter
```bash
flutter analyze  # Must show 0 issues
flutter test
flutter format .
```

## Example Workflows

### Frontend: Create New React Feature
1. Reference `react-architect` for feature structure
2. Use `react-component-designer` for accessible components
3. Use `react-state-manager` for state management
4. Use `tailwind-specialist` or `css-modules-specialist` for styling
5. Use `typescript-specialist` for type definitions
6. Use `frontend-tester` for unit/integration tests
7. Use `e2e-tester` for E2E tests
8. Use `frontend-performance-optimizer` to optimize performance

### Backend: Create New REST API (Python Example)
1. Reference `python-clean-architecture` for architecture
2. Use `python-fastapi-developer` for API implementation
3. Use `python-security-specialist` for security (validation, auth)
4. Use `sql-database-specialist` or `nosql-database-specialist` for database
5. Use `python-tester` for comprehensive tests
6. Use `python-performance-optimizer` for optimization

### Backend: Create New REST API (Go Example)
1. Reference `go-clean-architecture` for architecture
2. Use `go-rest-api-developer` for API implementation
3. Use `go-security-specialist` for security
4. Use `sql-database-specialist` for database
5. Use `go-tester` for tests
6. Use `go-performance-optimizer` for optimization

### Backend: Create New REST API (Java Example)
1. Reference `spring-architect` for architecture
2. Use `spring-boot-developer` for API implementation
3. Use `spring-security-specialist` for security
4. Use `sql-database-specialist` for database
5. Use `spring-tester` for tests
6. Use `spring-performance-optimizer` for optimization

### .NET: Create New API Endpoint
1. Reference `dotnet-api-designer` for design
2. Use `dotnet-code-implementer` for implementation
3. Apply `dotnet-security-implementer` for security
4. Document with `dotnet-documentation-writer`

### Flutter: Create New Feature
1. Reference `flutter-feature-planner` to analyze codebase
2. Use `flutter-domain` for entities
3. Use `flutter-use-cases` for business logic (with validation!)
4. Use `flutter-data-layer` for data access
5. Use `flutter-repository` for repository
6. Use `flutter-state` for state management
7. Use `flutter-ui` for UI (only invoke Cubit!)
8. Use `flutter-router` to register routes
9. Use `flutter-di` to register in DI
10. Use `flutter-tester` for tests

## Model Information
All agents use: **Claude Sonnet 4.5** (`claude-sonnet-4-5-20250929`)

---
> Source: [gabrielscr/code-agents](https://github.com/gabrielscr/code-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
