## panix-incremental

> When working on tasks, automatically read and follow these files based on context:

# Copilot Instructions

## Workflow Files

When working on tasks, automatically read and follow these files based on context:

### Task Management
Before starting any work, check for these files and follow their instructions:

1. `.tasks/task-list-[feature-name].md` - Current task list, always check this first
2. `.tasks/prd-[feature-name].md` - Product requirements document

### Workflow Rules
- When I say "next task" → Read `.ai/process-task-list.md` and follow it, if there is a next sub-task in the current task list, if all prior sub-tasks are complete, move the task list and the prd to `.ai/tasks/completed/`
- When I say "create prd" → Read `.ai/create-prd.md` and follow it, the generated prd should be saved to `.tasks/prd-[feature-name].md`
- When I say "generate tasks from prd" → Read `.ai/generate-tasks.md` and follow it, the generated task list should be saved to `.tasks/task-list-[feature-name].md`

### Git Branch Strategy

#### Phase-Based Development
- **Create a phase branch at the start of each phase** using format: `phase/phase[N]`
  - Example: `phase/phase1`, `phase/phase2`, `phase/phase3`
  - This branch serves as the integration branch for all tasks within that phase
  
- **Create task branches for individual sub-tasks** using format: `task/phase[N]-task[X.Y]-[short-description]`
  - Example: `task/phase1-task1.1-fix-scrolling`, `task/phase2-task3.2-add-settings-ui`
  - Always branch FROM the current phase branch, not from main
  
#### Workflow for Each Task
1. **Start task:** Create task branch from phase branch
   ```bash
   git checkout phase/phase1
   git checkout -b task/phase1-task1.1-fix-scrolling
   ```
2. **Complete work:** Make changes, verify no errors
3. **Before committing:**
   - Run `npm run test` or equivalent to verify NO ERRORS in code
   - Verify at least 70% of tests pass (if tests exist)
   - Check with `get_errors` tool to ensure no linting/compilation errors
   - Only commit when code is error-free and tests pass
4. **Commit:** Use conventional commit format
   - `feat(scope): description` for features
   - `fix(scope): description` for bugfixes
   - `test(scope): description` for tests
   - `refactor(scope): description` for refactoring
5. **Merge back:** Merge task branch into phase branch
   ```bash
   git checkout phase/phase1
   git merge task/phase1-task1.1-fix-scrolling --no-ff
   ```
6. **Stay on phase branch** for next task

#### After Completing Parent Task (e.g., Task 1.0 with all sub-tasks)
- **Provide overview of completed work:**
  - List all sub-tasks completed
  - Summarize changes made
  - List files modified
  - Confirm all tests pass and no errors

#### After Completing Entire Phase
- **Wait for user approval** ("phase complete", "ready to release", etc.)
- **Merge to main and create release:**
  ```bash
  git checkout main
  git merge phase/phase1 --no-ff
  git push origin main
  git tag v[X.Y.Z] -m "Phase [N] Complete: [Description]"
  git push origin v[X.Y.Z]
  ```
- **Never push broken code** - always verify tests and error-free status first

## About This Project
[Describe what your project does]

---

## Tech Stack

### Frontend
- React (with hooks, functional components)
- Angular (standalone components preferred)
- TypeScript (strict mode)

### Backend
- Python 3.11+
- FastAPI
- SQLAlchemy (async where possible)
- Pydantic for validation

### Databases
- PostgreSQL / MySQL (via SQLAlchemy ORM)
- MongoDB / other NoSQL databases

---

## Core Principles

### KISS (Keep It Simple, Stupid)
- Write the simplest solution that works
- Avoid clever or overly abstract code
- Prefer readability over brevity
- If code needs extensive comments to explain, simplify it

### YAGNI (You Aren't Gonna Need It)
- Don't implement features "just in case"
- Don't over-engineer for hypothetical future requirements
- Build for current requirements, refactor when needed
- Avoid premature optimization

### DRY (Don't Repeat Yourself)
- Extract repeated code into reusable functions/components
- Use shared utilities for common operations
- BUT: don't over-abstract — duplication is better than wrong abstraction
- Rule of three: refactor after third occurrence

### SRP (Single Responsibility Principle)
- Each function does ONE thing well
- Each class/module has ONE reason to change
- Split large functions into smaller, focused ones
- Separate concerns: data fetching, business logic, presentation

### Additional Principles
- Composition over inheritance: Prefer composing behaviors over deep inheritance hierarchies
- Fail fast: Validate inputs early, throw errors immediately when something is wrong
- Explicit over implicit: Be clear about intent, avoid magic

---

## Naming Conventions

### Python

```python
# Variables and functions: snake_case
user_name = "john"
def get_user_by_id(user_id: int) -> User:
    pass

# Classes: PascalCase
class UserService:
    pass

# Constants: UPPER_SNAKE_CASE
MAX_RETRY_COUNT = 3
DEFAULT_TIMEOUT = 30

# Private/internal: prefix with underscore
_internal_cache = {}
def _helper_function():
    pass

# Boolean variables: use is_, has_, can_, should_ prefix
is_active = True
has_permission = False
can_edit = True
```

### JavaScript / TypeScript

```typescript
// Variables and functions: camelCase
const userName = "john";
function getUserById(userId: number): User {}

// React components: PascalCase
function UserProfile() {}
const UserCard: React.FC = () => {};

// Angular components: PascalCase
@Component({})
export class UserProfileComponent {}

// Classes and interfaces: PascalCase
interface UserResponse {}
class ApiService {}

// Constants: UPPER_SNAKE_CASE or camelCase (project preference)
const MAX_RETRY_COUNT = 3;
const apiEndpoints = {};

// Boolean variables: use is, has, can, should prefix
const isLoading = true;
const hasError = false;
const canSubmit = true;

// Event handlers: handle prefix
const handleClick = () => {};
const handleSubmit = () => {};

// Private class members: prefix with underscore or use #
private _internalState = {};
#privateField = {};
```

### Files and Folders

```
# Python
user_service.py          # snake_case
test_user_service.py     # test_ prefix

# React
UserProfile.tsx          # PascalCase for components
userService.ts           # camelCase for utilities
useUserData.ts           # hooks start with "use"
user.types.ts            # type definitions

# Angular
user-profile.component.ts    # kebab-case
user.service.ts
user.model.ts
user.guard.ts
```

---

## Python / FastAPI Guidelines

### Type Hints

```python
# Always use type hints
def create_user(name: str, age: int) -> User:
    pass

# Use Optional for nullable values
from typing import Optional
def get_user(user_id: int) -> Optional[User]:
    pass

# Use modern syntax (Python 3.10+)
def process_items(items: list[str]) -> dict[str, int]:
    pass
```

### FastAPI Patterns

```python
# Use Pydantic models for request/response
class UserCreate(BaseModel):
    name: str
    email: EmailStr
    age: int = Field(ge=0, le=150)

class UserResponse(BaseModel):
    id: int
    name: str
    email: str

    class Config:
        from_attributes = True  # for ORM compatibility

# Use dependency injection
async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session() as session:
        yield session

@router.get("/users/{user_id}")
async def get_user(
    user_id: int,
    db: AsyncSession = Depends(get_db)
) -> UserResponse:
    pass

# Use HTTPException for errors
from fastapi import HTTPException, status

if not user:
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail="User not found"
    )

# Group routes with APIRouter
router = APIRouter(prefix="/users", tags=["users"])
```

### SQLAlchemy Patterns

```python
# Modern SQLAlchemy 2.0 style
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100))
    email: Mapped[str] = mapped_column(String(255), unique=True)
    is_active: Mapped[bool] = mapped_column(default=True)
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)

# Use async sessions
from sqlalchemy.ext.asyncio import AsyncSession

async def get_user_by_id(db: AsyncSession, user_id: int) -> User | None:
    result = await db.execute(select(User).where(User.id == user_id))
    return result.scalar_one_or_none()
```

### Python Error Handling

```python
# Use specific exceptions
class UserNotFoundError(Exception):
    pass

class ValidationError(Exception):
    pass

# Use early returns
def process_user(user: User | None) -> dict:
    if user is None:
        raise UserNotFoundError("User not found")
    
    if not user.is_active:
        raise ValidationError("User is inactive")
    
    return {"id": user.id, "name": user.name}

# Context managers for resources
async with async_session() as session:
    async with session.begin():
        # transaction logic
        pass
```

---

## React Guidelines

### Component Structure

```tsx
// Functional components only, no class components
// Keep components small and focused

interface UserProfileProps {
  userId: string;
  onUpdate?: (user: User) => void;
}

export function UserProfile({ userId, onUpdate }: UserProfileProps) {
  // 1. Hooks at the top
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  
  // 2. Effects
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  // 3. Event handlers
  const handleUpdate = useCallback(() => {
    onUpdate?.(user);
  }, [user, onUpdate]);
  
  // 4. Early returns for loading/error states
  if (isLoading) return <Spinner />;
  if (!user) return <NotFound />;
  
  // 5. Main render
  return (
    <div className="user-profile">
      <h1>{user.name}</h1>
    </div>
  );
}
```

### React Best Practices

```tsx
// Use custom hooks for reusable logic
function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    // fetch logic
  }, [userId]);
  
  return { user, isLoading, error };
}

// Prefer named exports
export function UserCard() {}
export function UserList() {}

// Use React.memo sparingly, only when needed
export const ExpensiveComponent = memo(function ExpensiveComponent() {});

// Prop spreading: be explicit
// Bad: <Component {...props} />
// Good: <Component id={props.id} name={props.name} />
```

---

## Angular Guidelines

### Component Structure

```typescript
// Prefer standalone components
@Component({
  selector: 'app-user-profile',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `...`,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserProfileComponent {
  // 1. Injections
  private readonly userService = inject(UserService);
  private readonly router = inject(Router);
  
  // 2. Inputs/Outputs
  @Input({ required: true }) userId!: string;
  @Output() userUpdated = new EventEmitter<User>();
  
  // 3. Signals (Angular 16+)
  user = signal<User | null>(null);
  isLoading = signal(true);
  
  // 4. Computed values
  displayName = computed(() => this.user()?.name ?? 'Unknown');
  
  // 5. Methods
  async loadUser(): Promise<void> {
    // implementation
  }
}
```

### Angular Best Practices

```typescript
// Use inject() function over constructor injection
// Good
private readonly http = inject(HttpClient);

// Bad
constructor(private http: HttpClient) {}

// Use signals for state management (Angular 16+)
count = signal(0);
doubleCount = computed(() => this.count() * 2);

// Use OnPush change detection
changeDetection: ChangeDetectionStrategy.OnPush

// Prefer reactive forms over template-driven
form = new FormGroup({
  name: new FormControl('', Validators.required),
  email: new FormControl('', [Validators.required, Validators.email])
});

// Unsubscribe using takeUntilDestroyed
private destroyRef = inject(DestroyRef);

ngOnInit() {
  this.userService.getUsers().pipe(
    takeUntilDestroyed(this.destroyRef)
  ).subscribe();
}
```

---

## NoSQL / MongoDB Guidelines

```typescript
// Use clear schema definitions even for schemaless DBs
interface UserDocument {
  _id: ObjectId;
  name: string;
  email: string;
  profile: {
    avatar?: string;
    bio?: string;
  };
  tags: string[];
  createdAt: Date;
  updatedAt: Date;
}

// Index frequently queried fields
// Document this in code comments or separate schema file

// Prefer embedding for 1:1 and 1:few relationships
// Use references for 1:many and many:many

// Handle ObjectId conversion explicitly
function toObjectId(id: string): ObjectId {
  if (!ObjectId.isValid(id)) {
    throw new ValidationError('Invalid ID format');
  }
  return new ObjectId(id);
}
```

```python
# Python with Motor (async MongoDB)
from motor.motor_asyncio import AsyncIOMotorClient
from bson import ObjectId

async def get_user(user_id: str) -> dict | None:
    if not ObjectId.is_valid(user_id):
        raise ValueError("Invalid user ID")
    
    return await db.users.find_one({"_id": ObjectId(user_id)})

# Use Pydantic for validation with MongoDB
class MongoBaseModel(BaseModel):
    id: str = Field(alias="_id")
    
    class Config:
        populate_by_name = True
        json_encoders = {ObjectId: str}
```

---

## Error Handling

### General Pattern

```typescript
// TypeScript: Use Result pattern for expected errors
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

async function getUser(id: string): Promise<Result<User>> {
  try {
    const user = await fetchUser(id);
    return { success: true, data: user };
  } catch (error) {
    return { success: false, error: error as Error };
  }
}
```

```python
# Python: Use specific exceptions
class AppError(Exception):
    """Base application error"""
    pass

class NotFoundError(AppError):
    pass

class ValidationError(AppError):
    pass

# FastAPI exception handlers
@app.exception_handler(NotFoundError)
async def not_found_handler(request: Request, exc: NotFoundError):
    return JSONResponse(
        status_code=404,
        content={"detail": str(exc)}
    )
```

---

## Testing Conventions

### File Naming

```
# Python
tests/
├── test_user_service.py
├── test_user_api.py
└── conftest.py              # shared fixtures

# TypeScript
__tests__/
├── UserService.test.ts
├── UserProfile.test.tsx
└── setup.ts
```

### Test Structure

```python
# Python: Use pytest, arrange-act-assert pattern
class TestUserService:
    async def test_create_user_with_valid_data_returns_user(
        self,
        db_session: AsyncSession
    ):
        # Arrange
        user_data = UserCreate(name="John", email="john@example.com")
        
        # Act
        result = await create_user(db_session, user_data)
        
        # Assert
        assert result.name == "John"
        assert result.email == "john@example.com"
    
    async def test_create_user_with_duplicate_email_raises_error(self):
        # ...
```

```typescript
// TypeScript: Use descriptive test names
describe('UserService', () => {
  describe('createUser', () => {
    it('should return user when given valid data', async () => {
      // Arrange
      const userData = { name: 'John', email: 'john@example.com' };
      
      // Act
      const result = await userService.createUser(userData);
      
      // Assert
      expect(result.name).toBe('John');
    });
    
    it('should throw ValidationError when email is invalid', async () => {
      // ...
    });
  });
});
```

---

## Code Structure

### API Layer Separation

```
# Keep these concerns separate:

1. Routes/Controllers  → HTTP handling, request/response
2. Services           → Business logic
3. Repositories       → Data access
4. Models/Schemas     → Data structures
```

```
# Example structure for FastAPI
app/
├── api/
│   ├── routes/
│   │   ├── users.py
│   │   └── products.py
│   └── dependencies.py
├── services/
│   ├── user_service.py
│   └── product_service.py
├── repositories/
│   ├── user_repository.py
│   └── product_repository.py
├── models/
│   ├── user.py           # SQLAlchemy models
│   └── product.py
├── schemas/
│   ├── user.py           # Pydantic schemas
│   └── product.py
└── core/
    ├── config.py
    └── database.py
```

---

## Things to Avoid

### General
- No `any` type in TypeScript (use `unknown` if needed)
- No console.log in production code (use proper logging)
- No magic numbers/strings (use named constants)
- No deeply nested code (max 3 levels)
- No functions longer than 50 lines
- No commented-out code in commits
- No ignoring errors silently

### Python Specific
- No mutable default arguments: `def foo(items=[])`
- No bare `except:` clauses
- No `from module import *`
- No global variables for state

### TypeScript/JavaScript Specific
- No `var` (use `const` or `let`)
- No `==` (use `===`)
- No async functions without error handling
- No direct DOM manipulation in React/Angular

---

## Git Commit Messages

```
# Format: <type>(<scope>): <description>

feat(auth): add JWT refresh token support
fix(users): handle null email in profile update
refactor(api): extract validation to middleware
docs(readme): update installation instructions
test(users): add integration tests for user creation

# Types: feat, fix, refactor, docs, test, chore, style
```

---

## Comments and Documentation

```python
# Python: Use docstrings for public functions
def calculate_discount(price: float, percentage: float) -> float:
    """
    Calculate discounted price.
    
    Args:
        price: Original price in dollars
        percentage: Discount percentage (0-100)
    
    Returns:
        Discounted price
    
    Raises:
        ValueError: If percentage is not between 0 and 100
    """
    if not 0 <= percentage <= 100:
        raise ValueError("Percentage must be between 0 and 100")
    
    return price * (1 - percentage / 100)
```

```typescript
// TypeScript: Use JSDoc for public APIs
/**
 * Calculate discounted price
 * @param price - Original price in dollars
 * @param percentage - Discount percentage (0-100)
 * @returns Discounted price
 * @throws {ValidationError} If percentage is invalid
 */
function calculateDiscount(price: number, percentage: number): number {
  // implementation
}
```

---

## When Generating Code

1. Always include proper error handling
2. Add type hints/annotations
3. Follow the naming conventions above
4. Keep functions small and focused (SRP)
5. Prefer early returns over nested conditionals
6. Include basic input validation
7. Use async/await over callbacks or .then()
8. Add TODO comments for incomplete implementations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Panix-Develop) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
