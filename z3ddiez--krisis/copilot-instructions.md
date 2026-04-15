## krisis

> **Engineer**: Zawadi MC Nyachiya

# Cursor Config

**Engineer**: Zawadi MC Nyachiya  
**Last Updated**: 2026-02-15  
**Scope**: Global configuration for all projects

---

## 1. Core Engineering Identity

### Technical Stack Foundations

**Scope**: Global Engineering Standards (encompassing Enterprise & SaaS)

**Primary Languages**:

- **TypeScript** (Strict, Shared Types across Web/Mobile/Edge)
- **C#** (Enterprise Backend Systems)
- **Python** (Cloud Functions for AI/Heavy Lifting)
- **SQL** (PostgreSQL via Drizzle/EF Core)

**Backend Architectures**:

- **Option A (SaaS/Startup)**: **Supabase Serverless** (Auth, Database, Realtime, Edge Functions)
- **Option B (Enterprise)**: **ASP.NET Core (.NET 8+)** + Entity Framework Core
- **Option C (Event-Driven)**: Node.js/Python Cloud Functions (GCP/Azure)
- **Legacy**: Node.js/Express (Deprecated)

**Frontend Ecosystem**:

- **Web**: React + Vite (SPA)
- **Mobile**: React Native + Expo (Managed Workflow)
- **State Management**: Zustand (Client), TanStack Query (Server), Nanostores
- **Styling**: NativeWind (Tailwind CSS v4) or twrnd
- **Motion**: Reanimated (Mobile), Framer Motion (Web)

**Infrastructure & "External Brain"**:

- **Platforms**: Supabase, Google Cloud Platform (GCP), Azure App Services
- **Databases**: Supabase PostgreSQL (RLS Enforced), Firestore
- **Queues**: Upstash QStash (Serverless), Google Pub/Sub
- **Compute**: Supabase Edge Functions (Deno), Google Cloud Functions (Python)

### Architectural Philosophy

Build production-grade systems with clear separation of concerns (Thick Client + Serverless):

- **Domain layer remains pure**: Zero framework dependencies, isolated business logic in `packages/shared-types`
- **Security boundaries are explicit**: RLS (Row Level Security) is the primary firewall
- **Observability is first-class**: Sentry for error tracking, PostHog for analytics
- **Cost optimization matters**: Scale-to-zero infrastructure (Serverless/Edge)

---

## 2. AI Agent Collaboration Protocol

### Read-First, Verify-Always Approach

**Before any code modification**:

1. Read relevant source files using `view_file` or equivalent
2. Check project-specific `agents.md` (if present) for tactical constraints
3. Review existing architecture documentation (`docs/spec/`, `docs/architecture/`, `docs/`)
4. Understand the change's impact on domain boundaries and security posture
5. **Verify edge cases against Production Readiness Framework (Section 9)**

**Never**:

- Run destructive commands without explicit user approval
- Introduce framework dependencies into domain layers
- Skip test execution after logic changes
- Assume implicit requirements—ask for clarification
- Proceed without considering the 28 critical production areas (Section 9)

### Communication Standards

- **Blocked on ambiguity**: Use `notify_user` with `BlockedOnUser: true` immediately
- **Phase completion**: Request explicit review before proceeding to next phase
- **Tradeoff decisions**: Present architectural options with clear pros/cons, cost implications, security considerations, and impact on production readiness areas
- **Error reporting**: Provide specific error messages, file paths, suggested fixes, and impact assessment
- **Edge case identification**: Explicitly call out edge cases discovered during implementation

### Tool Usage Discipline

- **Git**: User handles commits unless explicitly delegated; ensure atomic commits with descriptive messages
- **Builds**: Verify backend changes with `dotnet build` and `dotnet test`; frontend changes with `npm run build`
- **Commands**: Prefer showing commands over executing them—let user maintain control of their environment
- **Documentation**: Update `planning_log.md` or equivalent at phase boundaries
- **Testing**: Run comprehensive test suites before claiming completion

### Coding Conventions

- **File Naming**: strictly use `kebab-case` for all files and directories (e.g., `user-profile.tsx`, `auth-service.ts`)
- **Component Naming**: use `PascalCase` for React components (e.g., `UserProfile`)
- **Documentation**: All exported components and functions must have JSDoc comments explaining their purpose, props, and return values.
- **Imports**: Use relative imports for internal files unless an alias is configured. Ensure import paths match the file system casing exactly.

---

## 3. Code Quality Expectations

### Architecture Boundaries

**Clean Architecture enforcement**:

- Domain entities contain business logic only (no infrastructure leakage)
- Application layer handles use cases and orchestration
- Infrastructure layer owns database, external APIs, cloud services
- API/Presentation layer manages HTTP contracts and DTO transformations

**Dependency Rules**:

- Domain → Zero external dependencies
- Application → Domain only
- Infrastructure → Application + Domain
- API/Presentation → All layers

### Security Posture

**All systems must**:

- Treat AI/LLM output as untrusted input requiring validation
- Enforce authorization at database layer (Firestore rules, row-level security)
- Use Secret Manager/Key Vault for credentials—never hardcode secrets
- Implement rate limiting and quota enforcement for expensive operations
- Validate input schemas strictly before processing
- Sanitize all user-generated content before storage or display
- Use parameterized queries exclusively—no string concatenation for SQL
- Implement CSRF protection for state-changing operations
- Set appropriate CORS policies (never `*` in production)
- Use secure session management with HttpOnly, Secure, SameSite cookies

### Testing Standards

- **Unit tests**: Cover domain logic with pure functions, no mocks unless necessary
- **Integration tests**: Verify API contracts, database interactions, external service integrations
- **Test isolation**: Use test doubles (`TestNarrativeService`, `TestRNGService`) to control non-deterministic behavior
- **Edge case coverage**: Explicit tests for boundary conditions, null/undefined, empty arrays, concurrent operations
- **Build hygiene**: Zero tolerance for broken builds—fix immediately before proceeding
- **Coverage thresholds**: Minimum 80% for domain logic, 60% for infrastructure
- **Performance tests**: Verify response times under load, identify bottlenecks

---

## 4. Documentation Philosophy

### Conciseness Over Verbosity

Documentation should be:

- **Technically precise**: Avoid hand-waving; specify concrete implementations
- **Assumption-explicit**: State what's assumed vs. what's verified
- **Tradeoff-aware**: Acknowledge costs, performance implications, scalability limits
- **Update-driven**: Reflect actual system state, not aspirational plans
- **Edge-case-documented**: Explicitly note known edge cases and their handling

### Artifact Structure

- **Architecture Decision Records (ADRs)**: For significant structural choices
- **Planning Logs**: Track phase progress, blockers, decisions, edge cases discovered
- **API Documentation**: OpenAPI specs, DTO contracts, authentication flows, rate limits
- **README files**: Setup instructions, local development guide, deployment steps, known issues
- **Runbooks**: Operational procedures for common scenarios (database migrations, cache invalidation, incident response)

---

## 5. Domain-Specific Patterns

### Real-Time Systems (Firebase/Firestore/Supabase Realtime)

- Prefer optimistic UI updates with server reconciliation
- Design offline-first where feasible
- Use Firestore/Postgres transactions for multi-document consistency
- Implement exponential backoff for retry logic
- Handle connection state changes gracefully
- Implement conflict resolution strategy (Last-Write-Wins, Operational Transform, or CRDT)
- Test offline→online sync thoroughly
- Set appropriate realtime subscription limits

### AI Integration (Gemini API, LLMs)

- **Locked prompts**: Version-controlled, schema-validated responses
- **Cost controls**: Caching, quota limits, free-tier optimization
- **Reliability**: Retry with backoff, graceful degradation, error budgets
- **Trust boundaries**: Parse JSON strictly, validate against schemas, surface errors to users
- **Rate limiting**: Implement per-user quotas to prevent abuse
- **Fallback strategies**: Provide sensible defaults when AI unavailable
- **Monitoring**: Track token usage, response times, error rates

### Cloud-Native (GCP/Azure/Supabase)

- Serverless-first architecture (Cloud Functions, Azure Functions, Edge Functions)
- Event-driven patterns (Pub/Sub, Service Bus, Postgres Triggers)
- Managed services over self-hosted (Firestore, App Services, Supabase)
- Infrastructure as code where appropriate
- Cost monitoring and alerting
- Multi-region considerations for latency
- Graceful degradation when services unavailable

---

## 6. Language-Specific Conventions

### TypeScript

```typescript
// Prefer explicit return types
function calculateFitScore(resume: Resume, job: JobPosting): FitScore {
  // Implementation
}

// Use type guards for runtime validation
function isValidResponse(data: unknown): data is GeminiResponse {
  return typeof data === "object" && data !== null && "analysis" in data;
}

// Exhaustive union handling
type Status = "pending" | "active" | "completed";
function handleStatus(status: Status): string {
  switch (status) {
    case "pending":
      return "Waiting";
    case "active":
      return "In Progress";
    case "completed":
      return "Done";
    default: {
      const _exhaustive: never = status;
      throw new Error(`Unhandled status: ${_exhaustive}`);
    }
  }
}

// Explicit error handling
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

// Always handle nullable types explicitly
function getUserEmail(userId: string): string | null {
  const user = findUser(userId);
  return user?.email ?? null; // Explicit null handling
}
```

### C#

```csharp
// Domain entities use private setters, factory methods
public class Game
{
    public Guid Id { get; private set; }
    private readonly List<Move> _moves = new();
    public IReadOnlyList<Move> Moves => _moves.AsReadOnly();

    public static Result<Game> Create(GameConfiguration config) { }
}

// Use Result<T> for operation outcomes, avoid exceptions for control flow
public Result<MoveResult> ExecuteMove(Move move) { }

// Explicit null handling
public string? GetUserEmail(Guid userId)
{
    var user = _userRepository.FindById(userId);
    return user?.Email;
}
```

### Python (Cloud Functions)

```python
# Structured logging with correlation IDs
import logging
from google.cloud import firestore

def handle_request(request):
    correlation_id = request.headers.get('X-Correlation-ID', str(uuid.uuid4()))
    logger = logging.getLogger(__name__)
    logger.info(f"Processing request", extra={"correlation_id": correlation_id})

    # Explicit error handling
    try:
        result = process_data(request.json)
        return {"success": True, "data": result}
    except ValidationError as e:
        logger.error(f"Validation failed", extra={"error": str(e)})
        return {"success": False, "error": "Invalid input"}, 400
    except Exception as e:
        logger.exception(f"Unexpected error")
        return {"success": False, "error": "Internal error"}, 500
```

---

## 7. Project Structure Templates

### Full-Stack .NET + React

```
/solution-root
├── src/
│   ├── ProjectName.Domain/         # Pure business logic
│   ├── ProjectName.Application/    # Use cases, CQRS handlers
│   ├── ProjectName.Infrastructure/ # EF Core, external APIs
│   └── ProjectName.Api/            # ASP.NET Core entry point
├── client/                         # React + Vite frontend
│   ├── src/
│   │   ├── features/               # Feature-based organization
│   │   ├── lib/                    # Shared utilities
│   │   └── types/                  # TypeScript definitions
├── tests/
│   ├── ProjectName.Domain.Tests/
│   └── ProjectName.Application.Tests/
├── docs/
│   ├── spec/                       # Requirements, specs
│   ├── architecture/               # ADRs, diagrams
│   └── api/                        # OpenAPI, contracts
└── agents.md                       # Project-specific agent rules
```

### Serverless GCP (TypeScript)

```
/project-root
├── functions/
│   ├── src/
│   │   ├── handlers/               # Cloud Function entry points
│   │   ├── services/               # Business logic
│   │   └── types/                  # Shared types
├── firestore/
│   ├── rules/                      # Security rules
│   └── indexes.json                # Composite indexes
├── client/                         # React frontend
├── docs/
└── agents.md
```

### Nx Monorepo (Modern Stack)

```
/monorepo-root
├── apps/
│   ├── web/                        # React + Vite
│   ├── mobile/                     # React Native + Expo
│   └── backend/                    # Node.js + Express
├── packages/
│   ├── shared-types/               # Zod schemas, types
│   ├── shared-ui/                  # Design system
│   └── shared-utils/               # Common utilities
├── docs/
│   ├── architecture/
│   ├── api/
│   └── runbooks/
└── agents.md
```

---

## 8. Troubleshooting Protocol

### Build Failures

1. **Stop immediately**: Do not proceed to implementation
2. **Isolate error**: Identify exact file, line, error message
3. **Check dependencies**: Verify package versions, restore packages
4. **Consult logs**: Review build output for root cause
5. **Fix atomically**: Address one error at a time, verify after each fix
6. **Update documentation**: Note resolution for future reference

### Test Failures

1. **Read failure message**: Understand expected vs. actual
2. **Check test isolation**: Verify no shared state between tests
3. **Inspect mocks**: Ensure test doubles match production behavior
4. **Analyze strict logic**: Domain logic should be deterministic
5. **Update tests if needed**: If requirements changed, update tests to match
6. **Verify edge cases**: Ensure edge cases are properly handled

### Runtime Errors

1. **Correlation ID**: Track error through logs using correlation IDs
2. **Stack trace**: Identify exact failure point
3. **Reproduce locally**: Set up minimal repro case
4. **Check boundaries**: Verify input validation, auth checks, quota enforcement
5. **Graceful degradation**: Ensure system handles failure without cascading
6. **Monitor impact**: Check if error is isolated or systemic

---

## 9. Production Readiness Framework

This section provides systematic guidance for the 28 critical areas that separate prototypes from production systems. **Consult this checklist before claiming any feature is complete.**

### 9.1 Edge Cases

**Before committing code, verify**:

- **Null/Undefined handling**: Every nullable field has explicit null checks
- **Empty collections**: Arrays/lists handle empty state gracefully
- **Boundary values**: Test min/max values (0, -1, MAX_INT, empty string)
- **Concurrent operations**: Multiple users modifying same resource simultaneously
- **Network interruptions**: Mid-request failures, timeout handling
- **Race conditions**: Async operations completing in unexpected order
- **Invalid state transitions**: User jumps between screens unexpectedly
- **Data type mismatches**: Backend sends unexpected format
- **Locale variations**: Date formats, number formats, currency
- **Screen size extremes**: Tiny phones, massive tablets, foldables

**Patterns to use**:

```typescript
// Defensive null checking
const userName = user?.profile?.name ?? "Anonymous";

// Array operations with empty state
const totalScore =
  scores.length > 0 ? scores.reduce((sum, s) => sum + s, 0) : 0;

// Concurrent operation protection
const updateTask = async (taskId: string, updates: Partial<Task>) => {
  const current = await db.tasks.findById(taskId);
  if (!current) throw new Error("Task not found");

  // Optimistic locking
  if (current.version !== updates.expectedVersion) {
    throw new ConflictError("Task was modified by another user");
  }

  return db.tasks.update(taskId, {
    ...updates,
    version: current.version + 1,
  });
};
```

**Escalate when**:

- Edge case requires architectural change
- Conflict resolution strategy needed
- Data migration required to fix inconsistent state

---

### 9.2 Authentication & Authorization

**Every protected endpoint must**:

- Verify JWT/session token validity
- Check token expiration explicitly
- Validate user still exists and is active
- Enforce role-based access control (RBAC)
- Log authentication attempts (success and failure)
- Implement rate limiting on auth endpoints
- Handle token refresh gracefully
- Support logout/session invalidation

**Database-level authorization**:

```sql
-- Supabase RLS Policy Example
CREATE POLICY "Users can only view own tasks"
ON tasks FOR SELECT
USING (auth.uid() = user_id);

-- Optimised pattern with subquery
CREATE POLICY "Users can update own projects"
ON projects FOR UPDATE
USING (user_id = (SELECT auth.uid()));
```

**Frontend auth patterns**:

```typescript
// Protected route wrapper
export function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { user, loading } = useAuth();

  if (loading) return <LoadingScreen />;
  if (!user) return <Navigate to="/login" />;

  return <>{children}</>;
}

// API client with automatic token refresh
class ApiClient {
  async request(url: string, options: RequestInit) {
    const token = await this.getValidToken();
    return fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        Authorization: `Bearer ${token}`,
      },
    });
  }

  private async getValidToken(): Promise<string> {
    const token = localStorage.getItem('token');
    if (!token) throw new AuthError('Not authenticated');

    const decoded = jwt.decode(token);
    if (Date.now() >= decoded.exp * 1000) {
      return this.refreshToken();
    }
    return token;
  }
}
```

**Security checklist**:

- [ ] Passwords hashed with bcrypt/argon2 (never plaintext)
- [ ] Session tokens rotated on privilege elevation
- [ ] Account lockout after N failed attempts
- [ ] Email verification before account activation
- [ ] Password reset tokens expire within 1 hour
- [ ] HTTPS enforced (redirect HTTP → HTTPS)
- [ ] Secure, HttpOnly, SameSite cookies
- [ ] CSRF protection on state-changing operations
- [ ] OAuth state parameter validated

**Escalate when**:

- Multi-tenancy isolation required
- Fine-grained permissions needed (resource-level ACLs)
- SSO/SAML integration requested

---

### 9.3 Database Setup & Migrations

**Migration strategy**:

- Use version-controlled migration files (never manual SQL)
- Test migrations on production snapshot before deployment
- Include rollback scripts for every migration
- Use transactions for multi-statement migrations
- Verify data integrity after migration

**Schema design principles**:

```sql
-- Always include audit fields
CREATE TABLE tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending',

  -- Audit fields
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ, -- Soft delete
  version INT NOT NULL DEFAULT 1, -- Optimistic locking

  -- Indexes
  CONSTRAINT tasks_status_check CHECK (status IN ('pending', 'active', 'completed'))
);

CREATE INDEX idx_tasks_user_id ON tasks(user_id);
CREATE INDEX idx_tasks_status ON tasks(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_tasks_created_at ON tasks(created_at DESC);
```

**Connection pooling**:

```typescript
// Proper connection management
const pool = new Pool({
  max: 20, // Maximum connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Always use transactions for multi-query operations
async function createProjectWithTasks(project: Project, tasks: Task[]) {
  const client = await pool.connect();
  try {
    await client.query("BEGIN");

    const projectResult = await client.query(
      "INSERT INTO projects (name, user_id) VALUES ($1, $2) RETURNING id",
      [project.name, project.userId],
    );

    const projectId = projectResult.rows[0].id;

    for (const task of tasks) {
      await client.query(
        "INSERT INTO tasks (project_id, title, user_id) VALUES ($1, $2, $3)",
        [projectId, task.title, project.userId],
      );
    }

    await client.query("COMMIT");
    return projectId;
  } catch (error) {
    await client.query("ROLLBACK");
    throw error;
  } finally {
    client.release();
  }
}
```

**Data integrity checklist**:

- [ ] Foreign keys defined for all relationships
- [ ] Appropriate indexes on frequently queried columns
- [ ] Unique constraints on business keys (email, username)
- [ ] Check constraints for enum-like fields
- [ ] NOT NULL constraints on required fields
- [ ] Default values for optional fields
- [ ] Timestamps for audit trail (created_at, updated_at)
- [ ] Soft delete pattern (deleted_at) rather than hard deletes

**Escalate when**:

- Schema change requires complex data transformation
- Migration will lock table for extended period
- Data inconsistencies discovered that need repair script

---

### 9.4 APIs and Rate Limits

**Rate limiting implementation**:

```typescript
// Per-user rate limit with Redis
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, "10 s"), // 10 requests per 10 seconds
});

export async function rateLimit(userId: string) {
  const { success, limit, remaining, reset } = await ratelimit.limit(userId);

  if (!success) {
    throw new RateLimitError({
      message: "Too many requests",
      limit,
      remaining: 0,
      reset,
    });
  }

  return { limit, remaining, reset };
}

// API route with rate limiting
app.post("/api/tasks", authenticate, async (req, res) => {
  try {
    const { limit, remaining, reset } = await rateLimit(req.user.id);

    res.set({
      "X-RateLimit-Limit": limit.toString(),
      "X-RateLimit-Remaining": remaining.toString(),
      "X-RateLimit-Reset": reset.toString(),
    });

    const task = await createTask(req.body);
    res.json(task);
  } catch (error) {
    if (error instanceof RateLimitError) {
      res.status(429).json({
        error: error.message,
        retryAfter: error.reset - Date.now(),
      });
    }
  }
});
```

**API design principles**:

- Use semantic HTTP status codes (200, 201, 400, 401, 403, 404, 429, 500)
- Include request IDs for debugging
- Implement idempotency for mutating operations
- Use pagination for list endpoints (cursor-based preferred)
- Version APIs explicitly (`/api/v1/...`)
- Document rate limits in API response headers
- Provide clear error messages with actionable guidance

**External API integration**:

```typescript
// Wrapper with retry logic and circuit breaker
class ExternalApiClient {
  private circuitOpen = false;
  private failureCount = 0;
  private readonly maxFailures = 5;

  async call<T>(endpoint: string, options: RequestInit): Promise<T> {
    if (this.circuitOpen) {
      throw new Error("Circuit breaker is open");
    }

    try {
      const response = await this.retryWithBackoff(
        () => fetch(endpoint, options),
        3, // max retries
      );

      this.failureCount = 0; // Reset on success
      return response.json();
    } catch (error) {
      this.failureCount++;
      if (this.failureCount >= this.maxFailures) {
        this.circuitOpen = true;
        setTimeout(() => {
          this.circuitOpen = false;
          this.failureCount = 0;
        }, 60000); // 1 minute timeout
      }
      throw error;
    }
  }

  private async retryWithBackoff<T>(
    fn: () => Promise<T>,
    maxRetries: number,
  ): Promise<T> {
    for (let i = 0; i < maxRetries; i++) {
      try {
        return await fn();
      } catch (error) {
        if (i === maxRetries - 1) throw error;
        await new Promise((resolve) =>
          setTimeout(resolve, Math.pow(2, i) * 1000),
        );
      }
    }
    throw new Error("Unreachable");
  }
}
```

**Rate limit tiers**:

- **Anonymous**: 10 requests/minute
- **Authenticated**: 100 requests/minute
- **Premium**: 1000 requests/minute
- **AI operations**: 5 requests/minute (expensive operations)

**Escalate when**:

- Rate limit strategy needs dynamic adjustment based on user tier
- Complex throttling logic required (burst allowance, gradual release)
- External API rate limits need coordinated management

---

### 9.5 Error Handling

**Comprehensive error handling strategy**:

```typescript
// Structured error types
class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number,
    public isOperational = true,
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(
    message: string,
    public fields: Record<string, string>,
  ) {
    super(message, "VALIDATION_ERROR", 400);
  }
}

class AuthenticationError extends AppError {
  constructor(message = "Authentication required") {
    super(message, "AUTH_ERROR", 401);
  }
}

class AuthorizationError extends AppError {
  constructor(message = "Insufficient permissions") {
    super(message, "AUTHZ_ERROR", 403);
  }
}

class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, "NOT_FOUND", 404);
  }
}

class ConflictError extends AppError {
  constructor(message: string) {
    super(message, "CONFLICT", 409);
  }
}

class RateLimitError extends AppError {
  constructor(public reset: number) {
    super("Rate limit exceeded", "RATE_LIMIT", 429);
  }
}

// Global error handler
app.use((error: Error, req: Request, res: Response, next: NextFunction) => {
  const correlationId = req.headers["x-correlation-id"] || uuid();

  // Log error with context
  logger.error("Request error", {
    correlationId,
    error: error.message,
    stack: error.stack,
    userId: req.user?.id,
    path: req.path,
    method: req.method,
  });

  // Send appropriate response
  if (error instanceof AppError && error.isOperational) {
    res.status(error.statusCode).json({
      error: {
        code: error.code,
        message: error.message,
        correlationId,
        ...(error instanceof ValidationError && { fields: error.fields }),
      },
    });
  } else {
    // Unknown error - don't leak details
    res.status(500).json({
      error: {
        code: "INTERNAL_ERROR",
        message: "An unexpected error occurred",
        correlationId,
      },
    });
  }
});
```

**Frontend error handling**:

```typescript
// Error boundary for React
class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { hasError: boolean; error: Error | null }
> {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    logger.error('React error boundary', {
      error: error.message,
      componentStack: errorInfo.componentStack,
    });
  }

  render() {
    if (this.state.hasError) {
      return (
        <ErrorScreen
          message="Something went wrong"
          onRetry={() => this.setState({ hasError: false, error: null })}
        />
      );
    }
    return this.props.children;
  }
}

// Query error handling with React Query
const { data, error, isError } = useQuery({
  queryKey: ['tasks'],
  queryFn: fetchTasks,
  retry: (failureCount, error) => {
    // Don't retry on auth errors
    if (error instanceof AuthenticationError) return false;
    return failureCount < 3;
  },
});

if (isError) {
  if (error instanceof AuthenticationError) {
    return <Navigate to="/login" />;
  }
  return <ErrorMessage error={error} />;
}
```

**Logging strategy**:

- **Debug**: Detailed execution flow (local only)
- **Info**: Business events (user registered, task completed)
- **Warn**: Recoverable errors (API retry succeeded, cache miss)
- **Error**: Operational errors (validation failed, resource not found)
- **Fatal**: Non-recoverable errors (database connection lost)

**Escalate when**:

- Error patterns indicate systemic issue
- New error type requires specialized handling
- Error recovery requires architectural change

---

### 9.6 Performance Optimization

**Frontend performance**:

```typescript
// Code splitting with React.lazy
const Dashboard = React.lazy(() => import('./pages/Dashboard'));
const Settings = React.lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<LoadingScreen />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}

// Memoization for expensive computations
const sortedTasks = useMemo(() => {
  return tasks
    .filter(t => !t.completed)
    .sort((a, b) => a.priority - b.priority);
}, [tasks]);

// Virtualization for large lists
import { FixedSizeList } from 'react-window';

function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={tasks.length}
      itemSize={50}
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>
          <TaskItem task={tasks[index]} />
        </div>
      )}
    </FixedSizeList>
  );
}

// Image optimization
<img
  src="/images/hero.jpg"
  srcSet="/images/hero-320w.jpg 320w,
          /images/hero-640w.jpg 640w,
          /images/hero-1280w.jpg 1280w"
  sizes="(max-width: 320px) 280px,
         (max-width: 640px) 600px,
         1200px"
  loading="lazy"
  alt="Hero image"
/>
```

**Backend performance**:

```typescript
// Database query optimization
// BAD: N+1 query problem
const users = await db.users.findMany();
for (const user of users) {
  user.tasks = await db.tasks.findMany({ where: { userId: user.id } });
}

// GOOD: Single query with join
const usersWithTasks = await db.users.findMany({
  include: { tasks: true },
});

// Pagination with cursor
async function getTasks(cursor?: string, limit = 20) {
  return db.tasks.findMany({
    take: limit + 1, // Fetch one extra to check for more
    ...(cursor && {
      cursor: { id: cursor },
      skip: 1, // Skip the cursor
    }),
    orderBy: { createdAt: "desc" },
  });
}

// Connection pooling (already covered in Database section)
// Use Redis for session storage instead of in-memory
const session = require("express-session");
const RedisStore = require("connect-redis").default;

app.use(
  session({
    store: new RedisStore({ client: redisClient }),
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
  }),
);
```

**Performance monitoring**:

- Set up Core Web Vitals tracking (LCP, FID, CLS)
- Monitor server response times (p50, p95, p99)
- Track database query performance
- Identify slow API endpoints
- Monitor bundle size over time

**Performance budget**:

- Initial load: < 3 seconds (3G connection)
- Time to Interactive: < 5 seconds
- Bundle size: < 300 KB (gzipped)
- API response time: < 200ms (p95)
- Database queries: < 100ms (p95)

**Escalate when**:

- Performance degrades below budget thresholds
- Optimization requires infrastructure changes (CDN, caching layer)
- Database scaling needed (read replicas, sharding)

---

### 9.7 State Management

**State architecture patterns**:

```typescript
// Zustand store with persistence and sync
import create from "zustand";
import { persist } from "zustand/middleware";

interface TaskStore {
  tasks: Task[];
  loading: boolean;
  error: Error | null;

  // Actions
  fetchTasks: () => Promise<void>;
  addTask: (task: Omit<Task, "id">) => Promise<void>;
  updateTask: (id: string, updates: Partial<Task>) => Promise<void>;
  deleteTask: (id: string) => Promise<void>;

  // Optimistic updates
  optimisticUpdate: (id: string, updates: Partial<Task>) => void;
  rollbackUpdate: (id: string, previousState: Task) => void;
}

const useTaskStore = create<TaskStore>()(
  persist(
    (set, get) => ({
      tasks: [],
      loading: false,
      error: null,

      fetchTasks: async () => {
        set({ loading: true, error: null });
        try {
          const tasks = await api.tasks.getAll();
          set({ tasks, loading: false });
        } catch (error) {
          set({ error: error as Error, loading: false });
        }
      },

      addTask: async (task) => {
        const tempId = `temp-${Date.now()}`;
        const tempTask = { ...task, id: tempId };

        // Optimistic update
        set((state) => ({ tasks: [...state.tasks, tempTask] }));

        try {
          const savedTask = await api.tasks.create(task);
          set((state) => ({
            tasks: state.tasks.map((t) => (t.id === tempId ? savedTask : t)),
          }));
        } catch (error) {
          // Rollback on error
          set((state) => ({
            tasks: state.tasks.filter((t) => t.id !== tempId),
            error: error as Error,
          }));
          throw error;
        }
      },

      optimisticUpdate: (id, updates) => {
        set((state) => ({
          tasks: state.tasks.map((t) =>
            t.id === id ? { ...t, ...updates } : t,
          ),
        }));
      },

      rollbackUpdate: (id, previousState) => {
        set((state) => ({
          tasks: state.tasks.map((t) => (t.id === id ? previousState : t)),
        }));
      },
    }),
    {
      name: "task-storage",
      // Only persist non-transient state
      partialize: (state) => ({
        tasks: state.tasks,
      }),
    },
  ),
);
```

**State normalization**:

```typescript
// Normalized state structure (avoids nested updates)
interface NormalizedState {
  tasks: {
    byId: Record<string, Task>;
    allIds: string[];
  };
  projects: {
    byId: Record<string, Project>;
    allIds: string[];
  };
  // Derived data
  ui: {
    selectedTaskId: string | null;
    filters: TaskFilters;
  };
}

// Selectors for derived data
const selectFilteredTasks = (state: NormalizedState) => {
  return state.tasks.allIds
    .map((id) => state.tasks.byId[id])
    .filter((task) => {
      if (state.ui.filters.status && task.status !== state.ui.filters.status) {
        return false;
      }
      if (
        state.ui.filters.projectId &&
        task.projectId !== state.ui.filters.projectId
      ) {
        return false;
      }
      return true;
    });
};
```

**State synchronization with backend**:

- Optimistic updates for immediate feedback
- Background sync with server reconciliation
- Conflict resolution strategy (Last-Write-Wins, Operational Transform)
- Offline queue for mutations
- Realtime subscriptions for shared state

**Escalate when**:

- Complex state requires advanced patterns (Redux, XState)
- Synchronization conflicts become frequent
- State shape change impacts multiple components

---

### 9.8 Caching

**Multi-layer caching strategy**:

```typescript
// 1. Browser cache (HTTP headers)
app.use((req, res, next) => {
  if (req.path.startsWith("/static")) {
    res.set("Cache-Control", "public, max-age=31536000, immutable");
  } else if (req.path.startsWith("/api")) {
    res.set("Cache-Control", "no-cache, no-store, must-revalidate");
  }
  next();
});

// 2. In-memory cache (for frequently accessed data)
import NodeCache from "node-cache";
const cache = new NodeCache({ stdTTL: 600 }); // 10 minutes

async function getUser(userId: string): Promise<User> {
  const cacheKey = `user:${userId}`;

  // Check cache first
  const cached = cache.get<User>(cacheKey);
  if (cached) return cached;

  // Fetch from database
  const user = await db.users.findById(userId);
  if (!user) throw new NotFoundError("User");

  // Store in cache
  cache.set(cacheKey, user);
  return user;
}

// 3. Redis cache (distributed)
import Redis from "ioredis";
const redis = new Redis(process.env.REDIS_URL);

async function getCachedOrFetch<T>(
  key: string,
  fetchFn: () => Promise<T>,
  ttl = 3600,
): Promise<T> {
  // Try cache first
  const cached = await redis.get(key);
  if (cached) {
    return JSON.parse(cached) as T;
  }

  // Fetch and cache
  const data = await fetchFn();
  await redis.setex(key, ttl, JSON.stringify(data));
  return data;
}

// 4. React Query cache (frontend)
const { data } = useQuery({
  queryKey: ["tasks", userId],
  queryFn: () => api.tasks.getAll(userId),
  staleTime: 5 * 60 * 1000, // Consider fresh for 5 minutes
  cacheTime: 10 * 60 * 1000, // Keep in cache for 10 minutes
});

// 5. CDN caching (for static assets)
// Use Cloudflare, CloudFront, or similar
// Configure Cache-Control headers appropriately
```

**Cache invalidation strategies**:

```typescript
// Time-based (TTL)
await redis.setex("key", 3600, JSON.stringify(data)); // 1 hour

// Event-based (invalidate on mutation)
async function updateTask(taskId: string, updates: Partial<Task>) {
  const task = await db.tasks.update(taskId, updates);

  // Invalidate caches
  await redis.del(`task:${taskId}`);
  await redis.del(`tasks:user:${task.userId}`);

  return task;
}

// Tag-based (invalidate related data)
await redis.set("task:123", JSON.stringify(task), "EX", 3600);
await redis.sadd("tag:user:456", "task:123");

// When invalidating user's data
const keys = await redis.smembers("tag:user:456");
await redis.del(...keys);
await redis.del("tag:user:456");
```

**Cache warming**:

- Pre-populate cache for frequently accessed data
- Use background jobs to refresh cache before expiry
- Implement cache-aside pattern for resilience

**Escalate when**:

- Cache hit rate below 80%
- Cache invalidation causing performance issues
- Distributed cache coordination needed

---

### 9.9 Offline Support

**Offline-first architecture**:

```typescript
// Service Worker for offline support
// sw.ts
const CACHE_NAME = "timid-v1";
const STATIC_ASSETS = ["/", "/index.html", "/styles.css", "/app.js"];

self.addEventListener("install", (event: ExtendableEvent) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(STATIC_ASSETS);
    }),
  );
});

self.addEventListener("fetch", (event: FetchEvent) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      // Return cached version or fetch from network
      return response || fetch(event.request);
    }),
  );
});

// Offline mutation queue
interface QueuedMutation {
  id: string;
  type: "create" | "update" | "delete";
  resource: string;
  data: unknown;
  timestamp: number;
}

class OfflineQueue {
  private queue: QueuedMutation[] = [];
  private processing = false;

  async add(mutation: Omit<QueuedMutation, "id" | "timestamp">) {
    const queuedMutation: QueuedMutation = {
      ...mutation,
      id: uuid(),
      timestamp: Date.now(),
    };

    this.queue.push(queuedMutation);
    await this.saveQueue();

    if (navigator.onLine) {
      this.process();
    }
  }

  private async process() {
    if (this.processing || this.queue.length === 0) return;

    this.processing = true;

    while (this.queue.length > 0) {
      const mutation = this.queue[0];

      try {
        await this.executeMutation(mutation);
        this.queue.shift();
        await this.saveQueue();
      } catch (error) {
        // If mutation fails, stop processing and wait for retry
        console.error("Mutation failed:", error);
        break;
      }
    }

    this.processing = false;
  }

  private async executeMutation(mutation: QueuedMutation) {
    switch (mutation.type) {
      case "create":
        return api[mutation.resource].create(mutation.data);
      case "update":
        return api[mutation.resource].update(mutation.data);
      case "delete":
        return api[mutation.resource].delete(mutation.data);
    }
  }

  private async saveQueue() {
    await localStorage.setItem("offline-queue", JSON.stringify(this.queue));
  }
}

// Network status detection
window.addEventListener("online", () => {
  console.log("Back online, syncing...");
  offlineQueue.process();
});

window.addEventListener("offline", () => {
  console.log("Gone offline, queueing mutations...");
});
```

**Offline data management**:

- Use IndexedDB for structured data (tasks, projects)
- Use localStorage for settings and small data
- Implement background sync when connection restored
- Show offline indicator in UI
- Queue mutations and sync when online
- Handle conflicts when data changed on server

**Escalate when**:

- Complex conflict resolution needed (CRDT, Operational Transform)
- Large datasets require selective sync
- Offline experience needs extensive testing

---

### 9.10 Responsive Design

**Mobile-first approach**:

```css
/* Base styles (mobile) */
.container {
  padding: var(--spacing-4);
  max-width: 100%;
}

/* Tablet and up */
@media (min-width: 768px) {
  .container {
    padding: var(--spacing-6);
    max-width: 768px;
    margin: 0 auto;
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .container {
    padding: var(--spacing-8);
    max-width: 1024px;
  }
}

/* Large screens */
@media (min-width: 1280px) {
  .container {
    max-width: 1280px;
  }
}
```

**Responsive patterns**:

- Use CSS Grid for layout flexibility
- Implement container queries for component-level responsiveness
- Use clamp() for fluid typography
- Provide touch-friendly tap targets (min 44x44px)
- Test on real devices, not just browser DevTools
- Support landscape and portrait orientations
- Handle safe areas (notches, rounded corners)

**Device-specific considerations**:

```typescript
// Detect device capabilities
const isTouchDevice = 'ontouchstart' in window;
const hasHover = window.matchMedia('(hover: hover)').matches;

// Adaptive UI based on capabilities
{isTouchDevice && (
  <TouchOptimizedMenu />
)}

{hasHover && (
  <HoverTooltip />
)}

// React Native: Platform-specific code
import { Platform } from 'react-native';

const styles = StyleSheet.create({
  container: {
    paddingTop: Platform.OS === 'ios' ? 44 : 0,
  },
});
```

**Breakpoint strategy**:

- **Mobile**: 320px - 767px (priority)
- **Tablet**: 768px - 1023px
- **Desktop**: 1024px+
- **Large Desktop**: 1280px+

**Escalate when**:

- Complex responsive behavior needed (conditional component loading)
- Layout issues on specific devices/browsers
- Performance impact on low-end devices

---

### 9.11 Testing

**Testing pyramid**:

```
        /\
       /E2E\      (Few, critical paths)
      /------\
     /  INT   \   (Moderate, API contracts)
    /----------\
   /   UNIT     \ (Many, business logic)
  /--------------\
```

**Unit testing patterns**:

```typescript
// Pure function testing (easy)
describe("calculateFitScore", () => {
  it("returns 0 for empty resume", () => {
    const resume = createEmptyResume();
    const job = createJob();
    expect(calculateFitScore(resume, job)).toBe(0);
  });

  it("returns 100 for perfect match", () => {
    const resume = createResume({ skills: ["React", "TypeScript"] });
    const job = createJob({ requiredSkills: ["React", "TypeScript"] });
    expect(calculateFitScore(resume, job)).toBe(100);
  });

  it("handles edge case: null skills array", () => {
    const resume = createResume({ skills: null });
    const job = createJob();
    expect(() => calculateFitScore(resume, job)).not.toThrow();
  });
});

// Store testing with mock API
describe("TaskStore", () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it("fetches tasks successfully", async () => {
    const mockTasks = [createTask(), createTask()];
    vi.mocked(api.tasks.getAll).mockResolvedValue(mockTasks);

    const store = useTaskStore.getState();
    await store.fetchTasks();

    expect(store.tasks).toEqual(mockTasks);
    expect(store.loading).toBe(false);
    expect(store.error).toBeNull();
  });

  it("handles fetch error gracefully", async () => {
    const error = new Error("Network error");
    vi.mocked(api.tasks.getAll).mockRejectedValue(error);

    const store = useTaskStore.getState();
    await store.fetchTasks();

    expect(store.tasks).toEqual([]);
    expect(store.error).toBe(error);
  });
});
```

**Integration testing**:

```typescript
// API endpoint testing with real database
describe("POST /api/tasks", () => {
  let testUser: User;
  let authToken: string;

  beforeAll(async () => {
    testUser = await createTestUser();
    authToken = await generateAuthToken(testUser);
  });

  afterAll(async () => {
    await deleteTestUser(testUser);
  });

  it("creates task successfully", async () => {
    const response = await request(app)
      .post("/api/tasks")
      .set("Authorization", `Bearer ${authToken}`)
      .send({ title: "Test task" });

    expect(response.status).toBe(201);
    expect(response.body).toMatchObject({
      id: expect.any(String),
      title: "Test task",
      userId: testUser.id,
    });
  });

  it("rejects unauthenticated requests", async () => {
    const response = await request(app)
      .post("/api/tasks")
      .send({ title: "Test task" });

    expect(response.status).toBe(401);
  });

  it("validates input", async () => {
    const response = await request(app)
      .post("/api/tasks")
      .set("Authorization", `Bearer ${authToken}`)
      .send({ title: "" });

    expect(response.status).toBe(400);
    expect(response.body.error.code).toBe("VALIDATION_ERROR");
  });
});
```

**E2E testing with Playwright**:

```typescript
// Critical user flows
test.describe("Task Management", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/login");
    await page.fill('[data-testid="email"]', "test@example.com");
    await page.fill('[data-testid="password"]', "password123");
    await page.click('[data-testid="login-button"]');
    await page.waitForURL("/dashboard");
  });

  test("creates and completes task", async ({ page }) => {
    // Create task
    await page.click('[data-testid="add-task-button"]');
    await page.fill('[data-testid="task-title"]', "Buy milk");
    await page.click('[data-testid="save-task"]');

    // Verify task appears
    await expect(page.locator('[data-testid="task-item"]')).toContainText(
      "Buy milk",
    );

    // Complete task
    await page.click('[data-testid="task-checkbox"]');

    // Verify completion
    await expect(page.locator('[data-testid="task-item"]')).toHaveClass(
      /completed/,
    );
  });

  test("handles offline mode", async ({ page, context }) => {
    // Go offline
    await context.setOffline(true);

    // Create task offline
    await page.click('[data-testid="add-task-button"]');
    await page.fill('[data-testid="task-title"]', "Offline task");
    await page.click('[data-testid="save-task"]');

    // Verify optimistic update
    await expect(page.locator('[data-testid="task-item"]')).toContainText(
      "Offline task",
    );

    // Go online
    await context.setOffline(false);

    // Verify sync
    await page.reload();
    await expect(page.locator('[data-testid="task-item"]')).toContainText(
      "Offline task",
    );
  });
});
```

**Testing checklist**:

- [ ] Unit tests for all business logic (80%+ coverage)
- [ ] Integration tests for API endpoints
- [ ] E2E tests for critical user flows
- [ ] Error case testing (network failures, invalid input)
- [ ] Edge case testing (empty state, max values, concurrent operations)
- [ ] Accessibility testing (screen readers, keyboard navigation)
- [ ] Performance testing (load times, large datasets)
- [ ] Security testing (auth bypass attempts, injection attacks)

**Escalate when**:

- Test suite takes too long to run (>5 minutes)
- Flaky tests appear frequently
- Complex mocking requirements indicate architectural issues

---

### 9.12-9.28 Additional Critical Areas

**9.12 Analytics**:

- Track key user actions (sign up, task created, goal completed)
- Monitor feature usage and engagement
- Set up conversion funnels
- Implement privacy-compliant tracking (respect DNT)
- Use tools: PostHog, Mixpanel, or Google Analytics

**9.13 Push Notifications**:

- Request permission at appropriate time (not on first launch)
- Implement notification preferences
- Handle notification click actions
- Test on physical devices (simulators unreliable)
- Rate limit frequency (respect user attention)

**9.14 CI/CD Pipeline**:

- Automated testing on every PR
- Linting and type checking in pipeline
- Automated deployments to staging
- Manual approval for production
- Rollback capability for failed deployments

**9.15 App Store Optimization**:

- Keyword research and optimization
- Compelling screenshots and preview video
- Clear, concise app description
- Respond to all reviews (positive and negative)
- Monitor ranking and adjust strategy

**9.16 Privacy and Compliance**:

- Privacy policy and terms of service (required)
- GDPR compliance for EU users
- CCPA compliance for California users
- Data deletion capability
- Audit logs for sensitive operations

**9.17 Security**:

- HTTPS everywhere (no mixed content)
- Content Security Policy headers
- Regular dependency updates
- Penetration testing before launch
- Bug bounty program for mature products

**9.18 Accessibility**:

- Semantic HTML elements
- ARIA labels where needed
- Keyboard navigation support
- Screen reader testing
- Color contrast compliance (WCAG AA)

**9.19 Localization**:

- Externalize all user-facing strings
- Use i18n library (i18next, react-intl)
- Handle RTL languages if needed
- Date/time formatting per locale
- Currency formatting per locale

**9.20 Version Control**:

- Meaningful commit messages
- Feature branches for new work
- Pull requests with code review
- Protected main branch
- Tag releases with semantic versioning

**9.21 Monitoring and Crash Reporting**:

- Sentry or similar for error tracking
- Performance monitoring (Datadog, New Relic)
- Uptime monitoring (UptimeRobot, Pingdom)
- Alert on critical errors
- Weekly review of error trends

**9.22 User Onboarding**:

- Progressive disclosure of features
- Interactive tutorials where appropriate
- Empty state guidance
- Success states to reinforce positive actions
- Skip options for experienced users

**9.23 Monetization**:

- Freemium model with clear upgrade path
- In-app purchases (if applicable)
- Subscription management
- Trial period handling
- Payment failure recovery flow

**9.24 Deep Linking**:

- Universal links (iOS) and App Links (Android)
- Handle deep link scenarios in app
- Test all deep link paths
- Analytics on deep link usage
- Fallback to web if app not installed

**9.25 App Updates and Backward Compatibility**:

- API versioning strategy
- Database migration plan
- Graceful degradation for old clients
- Force update mechanism for breaking changes
- Beta testing program

**9.26 Dependency Management**:

- Lock file committed to version control
- Regular dependency updates (weekly/monthly)
- Automated security vulnerability scanning
- Test thoroughly after updates
- Document breaking changes

**9.27 User Feedback Systems**:

- In-app feedback form
- Bug report with diagnostics
- Feature request tracking
- Public roadmap (optional)
- Close the feedback loop (respond to users)

**9.28 Scaling**:

- Horizontal scaling capability
- Database connection pooling
- Read replicas for read-heavy workloads
- Caching layer (Redis)
- CDN for static assets
- Load balancing
- Database sharding if needed

---

## 10. Anti-Patterns to Avoid

**Do Not**:

- Introduce `using` statements for infrastructure concerns in Domain layer
- Hardcode secrets, API keys, or environment-specific values
- Skip validation on AI-generated or user-submitted data
- Create circular dependencies between layers
- Use exceptions for control flow in business logic
- Implement optimistic concurrency without conflict resolution
- Deploy without testing authorization rules
- Assume free-tier limits are sufficient without monitoring
- Skip error handling "because it works in development"
- Ignore edge cases "because users won't do that"
- Deploy without rollback plan
- Skip documentation "to move faster"

**Instead**:

- Use dependency inversion (interfaces in Domain/Application)
- Externalize configuration (Secret Manager, App Settings)
- Validate strictly at trust boundaries
- Enforce unidirectional dependencies
- Return `Result<T>` or `Option<T>` types
- Implement Firestore transactions or optimistic locking
- Test security rules in isolation before deployment
- Monitor quota usage and set alerts
- Handle all error cases explicitly
- Test edge cases systematically
- Have automated rollback capability
- Document as you build

---

## 11. Escalation Criteria

**Request user input when**:

- Architectural decision impacts multiple layers or introduces new dependencies
- Tradeoff between cost, performance, and complexity is non-obvious
- Security implications are unclear or potentially breaking
- Test coverage would drop below acceptable threshold
- Change conflicts with existing `agents.md` rules
- External API integration requires credential setup
- Deployment configuration needs environment-specific values
- Edge case handling requires product decision
- Rate limit strategy needs business input
- Monetization or privacy policy implications

**Proceed autonomously when**:

- Fix is localized to single file/function
- Change maintains existing contracts and tests pass
- Refactoring improves readability without behavioral change
- Documentation update reflects current system state
- Test addition increases coverage without modifying source
- Error handling improvement follows established patterns
- Performance optimization within established patterns

---

## 12. Language & Communication Preferences

### UK English, Oxford Comma

- Use British spelling (optimise, colour, analyse)
- Apply Oxford comma consistently in lists
- Express ideas directly without comparative negation
- Avoid "This isn't X, it's Y" constructions

### Technical Precision

- Use software engineering terminology correctly (entity, aggregate, bounded context, saga)
- Distinguish domain logic from infrastructure concerns explicitly
- Specify concrete implementations over vague descriptions
- Acknowledge uncertainty when appropriate ("likely", "typically", "in most cases")

### Response Structure

1. **Core elements first**: Thesis, main points, key direction
2. **Specific areas to develop**: Offer targeted follow-up based on need
3. **Minimal meta-commentary**: Keep adjacent but separate from primary content
4. **Edge cases explicit**: Call out discovered edge cases clearly

---

## 13. Project-Specific Overrides

**This file establishes global defaults**. For project-specific rules:

1. Create `agents.md` in project root
2. Document tactical constraints (build commands, test patterns, domain rules)
3. Reference this file for general principles
4. Override only when project requirements genuinely differ

**Example `agents.md` structure**:

```markdown
# Project-Specific Agent Rules

Inherits from: `~/gemini.md` (global configuration)

## Critical Rules

- [Terminal Blindness Rule]
- [Project-Specific Build Commands]

## Tech Stack

- [Specific Technologies]

## Architecture Boundaries

- [Project-Specific Boundaries]

## Testing Strategy

- [Project-Specific Test Patterns]

## Deployment

- [Project-Specific Deployment Process]
```

---

## 14. Version Control

**This document is version-controlled**:

- Update `Last Updated` field when making changes
- Maintain changelog section for significant revisions
- Review quarterly to ensure alignment with evolving practices

### Changelog

- **2026-02-12**: Major update
  - Added comprehensive Production Readiness Framework (28 critical areas)
  - Expanded error handling patterns with structured error types
  - Added performance optimization guidelines
  - Included state management and caching strategies
  - Added offline support patterns
  - Expanded testing section with E2E examples
  - Added explicit edge case handling guidance
  - Included escalation criteria refinement
- **2026-01-29**: Initial version created
  - Established core engineering identity
  - Defined AI agent collaboration protocol
  - Documented architecture boundaries and security posture
  - Specified language conventions and project structure templates

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Z3DDIEZ) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
