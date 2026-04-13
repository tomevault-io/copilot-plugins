## buyorbye

> > **FE location:** all front-end code, templates, and assets live under `./cmd/web`.

# BuyOrBye Project — Complete Architecture & Rules (CLAUDE.md)

> **FE location:** all front-end code, templates, and assets live under `./cmd/web`.

---

## Project Description
Go-based web application using layered architecture with strict separation of concerns. Purchase decision platform with a GORM persistence layer and Templ templating.

- See `@go.mod` for exact dependency versions.
- See `@README.md` for project overview.

---

## Tech Stack
- **Language:** Go `1.23.x` (latest stable)
- **Web Framework:** `gin-gonic/gin v1.10.1`
- **ORM:** `gorm.io/gorm v1.25.x`
- **Database:** `mysql v1.9.3` (prod) / `sqlite v1.6.0` (dev/test)
- **Templating:** `a-h/templ v0.2.x` (latest stable)
- **Front-End:** Tailwind CSS v3.4.x, HTMX v1.9.x, AlpineJS v3.x (advanced visual effects & animations), Templ-generated HTML
- **Validation:** `go-playground/validator v10.27.0`
- **Auth:** `golang-jwt/jwt/v5 v5.3.0`
- **Testing:** `testing` + `stretchr/testify v1.10.0` + `testcontainers-go v0.38.0`
- **Build:** Go modules (+ Tailwind CLI)

---

## Environment Setup
- Go `1.23.x+`
- MySQL for production, SQLite for development/testing
- `go mod download` for dependencies
- Env vars via `.env` with `godotenv v1.5.1`
- Docker required for `testcontainers` integration tests
- `go install github.com/a-h/templ/cmd/templ@latest` for Templ generation
- Tailwind CLI: standalone binary **or** `npx tailwindcss` (Node optional; standalone preferred)

---

## Commands

### App
- `go run cmd/app/main.go` — Start dev server
- `go build -o bin/app cmd/app/main.go` — Build binary
- `go vet ./...` — Static analysis

### Templ
- `templ generate` — Generate Go from `.templ` files (must run before build)
- `templ generate --watch` — Watch mode for development

### Tests
- `go test ./...` — All tests
- `go test -race ./...` — With race detector
- `go test -cover ./...` — Coverage
- `go test -v ./internal/services/...` — Services only
- `go test -v ./internal/repositories/...` — Repos only
- `go test -v ./internal/handlers/...` — Handlers only

### Front-End
- **Dev (watch):**  
  `tailwindcss -c cmd/web/tailwind.config.js -i cmd/web/src/css/input.css -o cmd/web/static/css/output.css --watch`
- **Prod (minify):**  
  `tailwindcss -c cmd/web/tailwind.config.js -i cmd/web/src/css/input.css -o cmd/web/static/css/output.css --minify`

### Build Sequence (MANDATORY ORDER)
```bash
# Development
templ generate
tailwindcss -c cmd/web/tailwind.config.js -i cmd/web/src/css/input.css -o cmd/web/static/css/output.css
go run cmd/app/main.go

# Production
templ generate
tailwindcss -c cmd/web/tailwind.config.js -i cmd/web/src/css/input.css -o cmd/web/static/css/output.css --minify
go build -o bin/app cmd/app/main.go
```

---

## Architecture Rules — **STRICT ENFORCEMENT**

### Layer Responsibilities
- **Handlers:** HTTP transport only, use **DTOs** exclusively, render Templ templates or JSON
- **Services:** Business logic only, use **domain** structs exclusively
- **Repositories:** Persistence with **GORM** only, use **model** structs exclusively
- **Templates (Templ):** Presentation only; render **DTOs**; never see domain/models
- **Front-End (HTMX/Alpine/Tailwind):** UI behavior, interactivity, progressive enhancement only
- **No cross-layer violations allowed**

### Data Flow (**MANDATORY**)
```
Middleware → Validation → Handler → Service → Repository → Database
              DTO      →  Domain  →  Model
              ↓
         Templ Templates ← Handler (DTOs only)
              ↓
         HTMX requests → Handlers (partial DTO render) → Templ partials
```

### Layer Import Rules (**STRICT**)
- **Handlers:** Can import `internal/types` (DTOs) only, never `internal/domain` or `internal/models`
- **Services:** Can import `internal/domain` only, never `internal/models` or `internal/types`
- **Repositories:** Can import `internal/models` only, never `internal/domain` or `internal/types`
- **Templates:** Can only receive `internal/types` (DTOs), never domain/model structs

### Transaction Boundaries (**STRICT**)
- **Services:** Own transaction boundaries, never repositories
- **Single Entity:** Repository methods can run without explicit transactions
- **Multi-Entity:** Service methods must wrap in transactions
- **Read-Only:** No transactions needed for queries
- **Error Handling:** Always rollback on service-layer errors

### Error Handling Layer Rules (**MANDATORY**)
- **Repository Errors:** Return domain errors, never GORM errors
- **Service Errors:** Transform and add business context, return domain errors
- **Handler Errors:** Transform domain errors to DTOs with appropriate HTTP status
- **Template Errors:** Always render error states, never crash
- **Error Propagation:** Repository → Service → Handler → Template (with transformations)

### GORM Usage Rules
- GORM code **ONLY** in repository layer
- Never use GORM in services or handlers
- All DB operations via repository interfaces
- Use GORM v1.25.x features appropriately (transactions, preloading, etc.)
- AutoMigrate for schema management (not SQL files)
- No raw SQL with user input (SQL injection prevention)

### Templ Usage Rules
- Templates render **DTOs only**
- Generate with `templ generate` before building
- **Partial contract:** HTMX endpoints return **fragment components** (no full layout)
- Import DTOs explicitly from `internal/types`
- Use proper Templ syntax with @ for composition
- Never use `templ.Raw()` with user input (XSS prevention)

---

## Claude Code CLI Specific Rules

### File Creation Order (CRITICAL)
Claude must ALWAYS create files in this sequence:
1. Domain models first (`internal/domain/`)
2. DTOs second (`internal/types/`)
3. Repository interfaces in services
4. Repository implementations
5. Services
6. Handlers
7. Templ templates LAST
8. Run `templ generate` after creating .templ files
9. Run Tailwind build after CSS changes

NEVER create templates before DTOs exist
NEVER create handlers before services exist

### Template Development Rules
- Write `.templ` files, not `.html`
- Run `templ generate` after EVERY .templ change
- Create golden test files FIRST (`testdata/*.golden.html`)
- Never mix partial and page responses
- Always include proper package declaration

Example Claude must follow:
```templ
package pages

import "internal/types"

templ PurchasePage(dto types.PurchasePageDTO) {
    @components.Layout(dto.Title) {
        // content
    }
}
```

### Build Dependencies Check
Claude must verify before running:
```bash
which templ        # Must be installed (v0.2.x)
which tailwindcss  # Must be v3.4.x+
tailwindcss --version  # Verify version
ls .env           # Must exist with required vars
```

---

## Project Structure

```
cmd/
  app/
    main.go               # Gin router, middleware, static mount
  web/                    # FRONT-END ROOT
    templates/            # .templ files (pages, partials, components)
      layouts/            # Base layouts, shells
      pages/              # Full pages: *_page.templ
      partials/           # HTMX fragments: *_partial.templ  
      components/         # Reusable UI bits (Card, Toast, Table...)
      shared/             # Meta tags, scripts includes
    src/                  # Source files
      css/
        input.css         # Tailwind source (@tailwind base/components/utilities)
      js/
        alpine.boot.js    # Alpine init (stores, transitions)
        htmx.boot.js      # HTMX config (headers, swapping)
    static/               # Compiled/served assets
      css/
        output.css        # Compiled Tailwind CSS
      js/
        htmx.min.js       # HTMX library
      images/             # Static images
      icons/              # SVG icons
      favicon.svg
      manifest.json
      robots.txt
    tailwind.config.js
    postcss.config.js     # optional
    package.json          # For Tailwind/Node deps (optional)
    
internal/
  database/               # GORM setup/migrations
    migration.go          # AutoMigrate all models
  models/                 # DB models (repository layer only)
  domain/                 # Business entities & rules (pure)
  repositories/           # GORM implementations
  services/               # Business logic
  handlers/               # HTTP endpoints (DTOs ↔ services)
  types/                  # DTOs organized by feature
    auth_dto.go          # All auth-related DTOs
    purchase_dto.go      # All purchase-related DTOs
    finance_dto.go       # All finance-related DTOs
    health_dto.go        # All health-related DTOs
    decision_dto.go      # All decision-related DTOs
    user_dto.go          # All user-related DTOs
    common_dto.go        # Shared DTOs (pagination, error, etc.)
  middleware/             # CORS, JWT, validation, CSRF
  
tests/
  integration/
  testutils/
  testdata/
    *.golden.html        # Golden test files for templates
```

> **Note:** New FE work must live under `./cmd/web`. No legacy template locations allowed.

---

## Front-End Architecture

### Principles
- **Progressive enhancement:** App works without JS; HTMX/Alpine enhance progressively.
- **Separation of concerns:**  
  - **HTMX** — server-driven interactions (requests, swaps, partial updates)  
  - **Alpine** — view-layer state only, micro-interactions, animations (no business logic)  
  - **Tailwind** — styling, tokens, responsive behavior
- **Fragments first:** Every interactive region has a matching Templ **partial**.
- **Business logic remains server-side** (services). Browser holds **presentation** only.

### HTMX Standards

#### Route Patterns (COMPLETE & STRICT)

**Page Routes (Full HTML with Layout):**
- `GET /[resource]` → `[Resource]IndexPage` 
- `GET /[resource]/:id` → `[Resource]DetailPage`
- `GET /[resource]/new` → `[Resource]NewPage`
- `GET /[resource]/:id/edit` → `[Resource]EditPage`
- `GET /auth/login` → `LoginPage`
- `GET /auth/register` → `RegisterPage`

**Partial Routes (Fragments Only - No Layout):**
- `GET /ui/partials/[resource]/list` → `[Resource]ListPartial`
- `GET /ui/partials/[resource]/:id` → `[Resource]DetailPartial`
- `POST /ui/partials/[resource]` → `[Resource]CreatePartial`
- `PUT /ui/partials/[resource]/:id` → `[Resource]UpdatePartial`
- `DELETE /ui/partials/[resource]/:id` → `EmptyResponse` or `RedirectHeader`
- `GET /ui/partials/notifications` → `NotificationListPartial`

**Form Action Routes (State Changes with Redirects):**
- `POST /[resource]` → `CreateAction` → `Redirect`
- `PUT /[resource]/:id` → `UpdateAction` → `Redirect`  
- `DELETE /[resource]/:id` → `DeleteAction` → `Redirect`
- `POST /auth/login` → `LoginAction` → `Redirect`
- `POST /auth/logout` → `LogoutAction` → `Redirect`

**Error Routes:**
- `GET /error/404` → `NotFoundPage`
- `GET /error/500` → `ServerErrorPage`
- `GET /ui/partials/error` → `ErrorBannerPartial`

Claude must NEVER mix these patterns.

#### HTMX Configuration
```javascript
// cmd/web/src/js/htmx.boot.js
document.body.addEventListener('htmx:configRequest', (e) => {
    // CSRF Token injection
    const token = document.querySelector('meta[name="csrf-token"]')?.content
    if (token) e.detail.headers['X-CSRF-Token'] = token
    
    // Add request ID for tracing
    e.detail.headers['X-Request-ID'] = crypto.randomUUID()
})

// Global error handling
document.body.addEventListener('htmx:responseError', (e) => {
    const target = e.detail.target
    target.innerHTML = '<div class="alert alert-error">Something went wrong. Please try again.</div>'
})

// Loading states
document.body.addEventListener('htmx:beforeRequest', (e) => {
    const indicator = document.querySelector('#global-loading')
    if (indicator) indicator.classList.remove('hidden')
})

document.body.addEventListener('htmx:afterRequest', (e) => {
    const indicator = document.querySelector('#global-loading')
    if (indicator) indicator.classList.add('hidden')
})
```

Use `hx-boost="true"` for PJAX-style nav where safe.
Use `hx-trigger` deliberately (`change`, `keyup changed delay:300ms`, `revealed`, etc.).
Optional SSE: `hx-sse="connect:/events"`.

#### Middleware Application Rules (STRICT)
- **CSRF Protection:** All state-changing routes (POST, PUT, DELETE, PATCH)
- **Auth Middleware:** All routes except `/auth/login`, `/auth/register`, public pages
- **Rate Limiting:** `/auth/*` endpoints, form submissions
- **Middleware Order:** Logging → CORS → Rate Limit → CSRF → Auth → Handler

### Alpine.js Standards

#### State Management Rules
Claude must NEVER:
- Store business data in Alpine (only UI state)
- Make API calls from Alpine (use HTMX)
- Calculate business logic in Alpine

Claude must ALWAYS:
- Use Alpine only for: open/closed, active tab, animation state, form validation hints
- Keep Alpine data ephemeral and UI-focused
- Sync with server via HTMX events

#### Alpine Configuration
```javascript
// cmd/web/src/js/alpine.boot.js
document.addEventListener('alpine:init', () => {
    // Global stores for UI state only
    Alpine.store('ui', {
        sidebarOpen: false,
        theme: localStorage.getItem('theme') || 'light',
        notifications: [],
        toggleSidebar() {
            this.sidebarOpen = !this.sidebarOpen
        },
        setTheme(theme) {
            this.theme = theme
            localStorage.setItem('theme', theme)
        }
    })
    
    // Magic helpers
    Alpine.magic('formatCurrency', () => (amount) => {
        return new Intl.NumberFormat('en-US', {
            style: 'currency',
            currency: 'USD'
        }).format(amount)
    })
    
    Alpine.magic('formatDate', () => (date) => {
        return new Intl.DateTimeFormat('en-US').format(new Date(date))
    })
})
```

Use declarative bindings: `:class`, `:aria-expanded`, `@click`, `@keydown.escape.window`.

### Tailwind Standards
- Centralize tokens in `tailwind.config.js`
- Utility-first; extract patterns with `@apply` in `cmd/web/src/css/input.css` under `@layer components`
- Name UI regions with stable IDs for HTMX targets
- Performance constraints:
  - Bundle size: `output.css < 50KB`
  - Use content purging in production
  - Critical CSS inline in production layout

---

## Lazy Loading Implementation

### Component-Level Lazy Loading
Goal: Load each component/region independently when visible.

#### Lazy Slot Component (Templ)
```templ
// cmd/web/templates/components/lazy_slot.templ
package components

templ LazySlot(id, url string) {
    <div id={ id }
         hx-get={ url }
         hx-trigger="revealed"
         hx-swap="outerHTML"
         hx-indicator="#global-loading">
        @SkeletonCard()
    </div>
}

templ SkeletonCard() {
    <div class="animate-pulse">
        <div class="h-4 bg-gray-200 rounded w-3/4 mb-2"></div>
        <div class="h-4 bg-gray-200 rounded w-1/2"></div>
    </div>
}
```

#### Handler for Partial
```go
// internal/handlers/purchase_ui.go
func (h *PurchaseUI) ListPartial(c *gin.Context) {
    items, err := h.service.ListRecent(c.Request.Context())
    if err != nil {
        c.Status(http.StatusInternalServerError)
        // Return error partial, not full page
        templates.RenderErrorBannerPartial(c, types.ErrorDTO{
            Message: "Failed to load purchases",
        })
        return
    }
    
    dto := types.PurchaseListDTO{
        Items: types.FromDomainList(items),
    }
    templates.RenderPurchaseListPartial(c, dto) // fragment only
}
```

Claude must NOT lazy load critical above-fold content.

---

## Template Error Handling Pattern

Claude must implement error states in every template:

```templ
templ PurchaseListPartial(dto types.PurchaseListDTO) {
    if dto.Error != nil {
        @components.ErrorBanner(dto.Error.Message)
    } else if len(dto.Items) == 0 {
        @components.EmptyState("No purchases found", "Start by adding your first purchase")
    } else {
        <ul class="divide-y">
            for _, item := range dto.Items {
                @PurchaseListItem(item)
            }
        </ul>
    }
}
```

---

## CSRF Implementation (Complete)

**All 4 steps must be implemented:**

1. **Middleware sets token:**
```go
// internal/middleware/csrf.go
func CSRFMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        token := generateCSRFToken()
        c.Set("csrf_token", token)
        c.SetCookie("csrf", token, 3600, "/", "", true, true)
        c.Next()
    }
}
```

2. **Handler passes to layout:**
```go
dto := types.LayoutDTO{
    Title:     "Dashboard",
    CSRFToken: c.GetString("csrf_token"),
}
```

3. **Layout renders meta tag:**
```templ
templ Layout(dto types.LayoutDTO) {
    <meta name="csrf-token" content={ dto.CSRFToken }/>
}
```

4. **HTMX reads from meta** (see htmx.boot.js above)

Claude must verify all 4 steps are implemented.

---

## DTO Organization Rules (STRICT)

### File Structure (MANDATORY)
Claude must organize DTOs by feature domain, not by request/response type:

```go
// internal/types/auth_dto.go
type LoginRequestDTO struct {
    Email    string `json:"email" binding:"required,email"`
    Password string `json:"password" binding:"required,min=8"`
}
type LoginResponseDTO struct {
    Token string `json:"token"`
    User  UserDTO `json:"user"`
}
type RegisterRequestDTO struct {
    Email    string `json:"email" binding:"required,email"`
    Password string `json:"password" binding:"required,min=8"`
    Name     string `json:"name" binding:"required,min=2"`
}
type TokenResponseDTO struct {
    AccessToken  string `json:"access_token"`
    RefreshToken string `json:"refresh_token"`
    ExpiresIn    int64  `json:"expires_in"`
}

// internal/types/purchase_dto.go  
type PurchaseCreateDTO struct {
    Name     string  `json:"name" binding:"required,min=2,max=100"`
    Amount   float64 `json:"amount" binding:"required,gt=0"`
    Category string  `json:"category" binding:"required,oneof=food clothing electronics"`
}
type PurchaseListDTO struct {
    Items []PurchaseItemDTO `json:"items"`
    Error *ErrorDTO         `json:"error,omitempty"`
}
type PurchaseDetailDTO struct {
    ID          string    `json:"id"`
    Name        string    `json:"name"`
    Amount      float64   `json:"amount"`
    Category    string    `json:"category"`
    CreatedAt   time.Time `json:"created_at"`
    UpdatedAt   time.Time `json:"updated_at"`
}

// internal/types/common_dto.go
type ErrorDTO struct {
    Message string `json:"message"`
    Code    string `json:"code,omitempty"`
}
type PaginationDTO struct {
    Page    int `json:"page"`
    PerPage int `json:"per_page"`
    Total   int `json:"total"`
}
type LayoutDTO struct {
    Title     string `json:"title"`
    CSRFToken string `json:"csrf_token"`
    User      *UserDTO `json:"user,omitempty"`
}

// NEVER create generic dto.go with mixed concerns
```

### DTO Validation
```go
// Always use validation tags
type PurchaseCreateDTO struct {
    Name     string  `json:"name" binding:"required,min=2,max=100"`
    Amount   float64 `json:"amount" binding:"required,gt=0"`
    Category string  `json:"category" binding:"required,oneof=food clothing electronics home"`
}

// Initialize validator once
var validatorInstance = validator.New()
```

### DTO Domain Coverage (COMPLETE LIST)
- `auth_dto.go` — Authentication, authorization, tokens
- `user_dto.go` — User profiles, preferences, settings
- `purchase_dto.go` — Purchase decisions, items, categories
- `finance_dto.go` — Financial data, budgets, expenses, income
- `health_dto.go` — Health profiles, conditions, medical data
- `decision_dto.go` — Decision processes, outcomes, history
- `common_dto.go` — Shared DTOs (error, pagination, layout)

---

## Security Rules (COMPLETE)

### Input Validation & Sanitization
- **Validate all input** at handler layer using `go-playground/validator`
- **Use struct tags** for automatic validation
- **Sanitize user input** before storage (trim spaces, validate formats)
- **Never trust client data** — always re-validate server-side

### Template Security (XSS Prevention)
- **Templ auto-escapes** by default (XSS safe)
- **Never use `templ.Raw()`** with user input
- **CSP headers** to prevent inline script injection
- **Validate HTML attributes** when using dynamic values

### SQL Injection Prevention
- **GORM only** — no raw SQL with user input
- **Parameterized queries** when raw SQL is necessary
- **Input validation** before database operations
- **Escape special characters** in search queries

### Authentication & Authorization
- **JWT tokens** with 15-minute expiry
- **Refresh tokens** with 7-day expiry
- **Bcrypt cost 14** for passwords
- **Session management** secure cookies only
- **Role-based access control** where needed

### CSRF Protection
- **Required on all state-changing operations** (POST, PUT, DELETE, PATCH)
- **Token validation** on server side
- **SameSite cookies** for additional protection
- **Referrer header validation** for sensitive operations

### Rate Limiting
- **Auth endpoints:** 5 attempts per minute per IP
- **Form submissions:** 10 requests per minute per user
- **API endpoints:** 100 requests per minute per user
- **Global limit:** 1000 requests per minute per IP

### Security Headers (MANDATORY)
```go
// Security headers middleware
func SecurityHeaders() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Header("X-Content-Type-Options", "nosniff")
        c.Header("X-Frame-Options", "DENY")
        c.Header("X-XSS-Protection", "1; mode=block")
        c.Header("Strict-Transport-Security", "max-age=31536000; includeSubDomains")
        c.Header("Content-Security-Policy", "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'")
        c.Next()
    }
}
```

---

## Testing Patterns

### Front-End Testing Sequence
1. Create golden test files first (`testdata/*.golden.html`)
2. Write handler tests with expected HTML fragments
3. Test HTMX interactions with httptest
4. Verify partial vs full page responses

### Golden Tests for Templates
```go
func TestPurchaseListPartial_Renders(t *testing.T) {
    dto := types.PurchaseListDTO{
        Items: []types.PurchaseItemDTO{
            {ID: "1", Name: "Coffee", Amount: 4.50},
        },
    }
    
    var buf bytes.Buffer
    err := templates.PurchaseListPartial(dto).Render(context.Background(), &buf)
    require.NoError(t, err)
    
    golden := filepath.Join("testdata", "purchase_list.golden.html")
    if *update {
        os.WriteFile(golden, buf.Bytes(), 0644)
    }
    
    expected, _ := os.ReadFile(golden)
    assert.Equal(t, string(expected), buf.String())
}
```

### Handler Test Patterns
```go
func TestPurchaseHandler_ListPartial_ReturnsFragment(t *testing.T) {
    // Setup
    mockService := &mocks.PurchaseService{}
    handler := NewPurchaseHandler(mockService)
    
    // Mock expectations
    mockService.On("ListRecent", mock.Anything).Return(purchases, nil)
    
    // Request
    w := httptest.NewRecorder()
    req := httptest.NewRequest("GET", "/ui/partials/purchases/list", nil)
    
    router := gin.New()
    router.GET("/ui/partials/purchases/list", handler.ListPartial)
    router.ServeHTTP(w, req)
    
    // Assertions
    assert.Equal(t, 200, w.Code)
    assert.Contains(t, w.Body.String(), `<ul class="divide-y">`)
    assert.NotContains(t, w.Body.String(), `<!DOCTYPE`)  // Ensure partial
}
```

### Integration Testing Rules
- **Use testcontainers** for database integration tests
- **Test full request-response cycle** including middleware
- **Verify CSRF protection** on state-changing operations
- **Test error handling** at all layers
- **Performance testing** for critical paths

---

## Environment-Specific Configuration

### Development
```env
ENV=development
DB_DRIVER=sqlite
DB_DSN=file:dev.db?cache=shared&mode=rwc
LOG_LEVEL=debug
TEMPL_DEV=true
TAILWIND_DEV=true
CSRF_SECURE=false
JWT_SECRET=dev-secret-key
```

### Production  
```env
ENV=production
DB_DRIVER=mysql
DB_DSN=user:pass@tcp(localhost:3306)/buyorbye?parseTime=true
LOG_LEVEL=info
TEMPL_DEV=false
TAILWIND_DEV=false
CSRF_SECURE=true
JWT_SECRET=prod-secret-key-32-chars-min
CACHE_STATIC=31536000  # 1 year for fingerprinted assets
```

### Test
```env
ENV=test
DB_DRIVER=sqlite
DB_DSN=:memory:
LOG_LEVEL=error
TEMPL_DEV=true
TAILWIND_DEV=false
CSRF_SECURE=false
JWT_SECRET=test-secret-key
```

---

## Performance Guidelines

### Front-End Performance
- **Bundle size:** `output.css < 50KB`
- **JavaScript size:** No framework > 10KB individually
- **Lazy loading:** Below-fold components only
- **Image optimization:** `loading="lazy"` for images
- **Critical CSS:** Inline critical CSS in production
- **Asset fingerprinting:** Cache static assets with versioning

### Database Performance
- **GORM preloading:** Use for related data: `db.Preload("Purchases").Find(&users)`
- **Pagination:** Always implement: `db.Limit(20).Offset(page * 20)`
- **Indexes:** Add on frequently queried fields
- **N+1 prevention:** Monitor and fix with preloading
- **Batch operations:** Use for bulk inserts/updates

### Application Performance
- **Connection pooling:** Configure for production
- **Memory management:** Monitor goroutine leaks
- **Caching:** Implement for frequently accessed data
- **Monitoring:** Add metrics and tracing

---

## Development Workflow

1. **Write failing tests** (service/handler/template golden)
2. **Implement minimal code** to pass tests
3. **Refactor with tests green**
4. **Run `templ generate`** if templates changed
5. **Build Tailwind** (watch in dev; minify for prod)
6. **Run full test suite** with race detector
7. **Verify strict layer separation**
8. **Confirm coverage thresholds**
9. **Validate config** across environments
10. **Ensure structured logging** (remove `fmt.Print`)
11. **Manually verify** HTMX lazy loading & partial contracts
12. **Security review** for new endpoints

### Parallel Development Setup
```bash
# Terminal 1: Tailwind watch
tailwindcss -c cmd/web/tailwind.config.js -i cmd/web/src/css/input.css -o cmd/web/static/css/output.css --watch

# Terminal 2: Templ watch
templ generate --watch

# Terminal 3: Go with auto-reload (using air or similar)
air
```

---

## Do Not

- Import GORM in services/handlers
- Use model structs outside repository layer
- Use DTOs in services (domain only)
- Pass domain/models to Templ templates
- Call repositories from handlers (go through services)
- Put business logic in handlers, repositories, or front-end code
- Commit without full tests (incl. race detector)
- Skip error handling at layer boundaries
- Use direct SQL where GORM suffices
- Expose internal model structures in API responses
- Forget `templ generate` before building
- Import concrete repo implementations in service constructors (use interfaces)
- Mix MySQL/SQLite specifics without compatibility handling
- Create DTOs outside `internal/types/`
- Mix different domain DTOs in the same file
- Omit validation struct tags on DTOs
- Return full pages from partial endpoints
- Add inline scripts/styles that break CSP
- Store business data in Alpine.js
- Make API calls from Alpine (use HTMX)
- Lazy load above-fold content
- Create templates before DTOs exist
- Create handlers before services exist
- Use `templ.Raw()` with user input
- Skip CSRF protection on state-changing operations
- Store sensitive data in localStorage
- Trust client-side validation only
- Mix page and partial route patterns
- Use Go version 1.24.x (doesn't exist)
- Create files in wrong order (domain → DTO → repo → service → handler → template)
- Skip transaction boundaries in multi-entity operations
- Ignore security headers in production
- Use localStorage for sensitive data
- Create DTOs by request/response type instead of feature domain

---

## Claude Code CLI Instructions

### Architecture Enforcement
- Use "think hard" for new layer interactions or complex UX/server contracts
- Use "ultrathink" for architectural refactors across FE/BE boundaries
- Enforce strict separation (DTOs ↔ Templates; Services ↔ Repos)
- Start with tests (incl. template golden tests) for new UI flows
- Consider MySQL/SQLite compatibility in all GORM code
- Always follow the file creation order (Domain → DTO → Repo → Service → Handler → Template)
- Run `templ generate` after every .templ file change
- Verify all 4 CSRF implementation steps
- Create golden test files before implementing templates
- Never mix page and partial endpoints
- Use Alpine only for UI state, never business logic
- Organize DTOs by feature domain, not by request/response type
- Check for `templ` and `tailwindcss` before running builds

### Development Process Validation
Before any significant changes, Claude must verify:
1. **Layer boundaries** are respected in all imports
2. **DTO organization** follows feature-based structure
3. **Route patterns** follow page vs partial conventions
4. **Build sequence** is executable and correct
5. **Security measures** are implemented (CSRF, headers, validation)
6. **Error handling** is present at all layer boundaries
7. **Transaction boundaries** are correctly placed
8. **Template contracts** match DTO structures

### Code Review Checklist
For every code change, verify:
- [ ] **No cross-layer violations** (imports, data passing)
- [ ] **DTOs organized by feature** in `internal/types/`
- [ ] **Templates receive DTOs only**, never domain/model structs
- [ ] **Route patterns consistent** (page vs partial)
- [ ] **CSRF protection** on state-changing operations
- [ ] **Error states handled** in templates
- [ ] **Validation tags present** on all DTO fields
- [ ] **Tests written** before implementation
- [ ] **Golden tests updated** for template changes
- [ ] **Build sequence** still works after changes

---

## Common Pitfalls & Solutions

| Issue | Root Cause | Solution |
|-------|------------|----------|
| Template not rendering | Missing `templ generate` | Run `templ generate` after any .templ change |
| CSS not updating | Tailwind watch not running | Check Tailwind watch process, verify paths |
| HTMX not sending CSRF | Incomplete 4-step implementation | Verify middleware → handler → template → HTMX chain |
| Partial returns full page | Wrong handler endpoint | Check route pattern: `/ui/partials/` vs page routes |
| Alpine storing business data | Misunderstanding of Alpine role | Move to server, use HTMX for data operations |
| N+1 queries | Missing GORM Preload | Use `db.Preload("Relation").Find()` |
| Tests failing on CI | MySQL/SQLite differences | Use compatible GORM features, test both DBs |
| Golden tests failing | Template structure changed | Update with `--update` flag, review changes |
| DTOs mixed up | Wrong organization pattern | Organize by feature domain, not request type |
| Build fails | Incorrect file creation order | Follow: Domain → DTO → Repo → Service → Handler → Template |
| Cross-layer imports | Architecture violation | Check import statements, use interfaces |
| Performance issues | Missing indexes/pagination | Add database indexes, implement pagination |
| Security vulnerabilities | Missing validation/headers | Implement all security rules, use middleware |
| Error handling gaps | Missing layer transformations | Transform errors at each boundary |
| Transaction deadlocks | Wrong transaction boundaries | Use service-level transactions only |

---

## Migration Guide (For Existing Projects)

If migrating from a non-compliant structure to this architecture:

### Phase 1: Structure Cleanup (Days 1-2)
1. **Move frontend code** to `cmd/web/` structure
2. **Reorganize DTOs** by feature in `internal/types/`
3. **Fix import paths** after DTO reorganization
4. **Update build scripts** to use correct paths
5. **Verify `templ generate` works** after moves

### Phase 2: Layer Separation (Days 3-5)
1. **Remove cross-layer imports** (GORM in handlers, DTOs in services)
2. **Implement missing interfaces** for repositories
3. **Add DTO transformations** at all boundaries
4. **Fix data flow violations** (ensure proper layer data types)
5. **Add missing validation** on DTOs

### Phase 3: Route & Template Fixes (Days 6-8)
1. **Separate page and partial routes** following patterns
2. **Update HTMX endpoints** to `/ui/partials/` pattern
3. **Fix template data contracts** (DTOs only)
4. **Implement error states** in all templates
5. **Add lazy loading** for appropriate components

### Phase 4: Security & Testing (Days 9-10)
1. **Implement complete CSRF flow** (all 4 steps)
2. **Add security headers** middleware
3. **Create golden tests** for all templates
4. **Add missing handler/service tests**
5. **Implement integration tests** with testcontainers

### Phase 5: Performance & Production (Days 11-12)
1. **Add database indexes** and pagination
2. **Implement caching** where appropriate
3. **Optimize bundle sizes** (CSS/JS)
4. **Configure production settings** (security, logging)
5. **Add monitoring and metrics**

---

## Monitoring & Observability

### Required Metrics
- **Request latency** by endpoint
- **Error rates** by handler
- **Database query performance**
- **Template rendering time**
- **Asset load times**
- **Memory usage** and garbage collection
- **Active connections** and goroutines

### Logging Standards
```go
// Use structured logging with context
logger.Info("Purchase created",
    zap.String("user_id", userID),
    zap.String("purchase_id", purchaseID),
    zap.Float64("amount", amount),
    zap.Duration("duration", time.Since(start)),
)

// Error logging with stack traces
logger.Error("Database operation failed",
    zap.Error(err),
    zap.String("operation", "CreatePurchase"),
    zap.String("user_id", userID),
    zap.Stack("stack"),
)
```

### Health Checks
```go
// Implement health check endpoints
GET /health/live   → 200 if app is running
GET /health/ready  → 200 if app can serve traffic (DB connected, etc.)
GET /health/deps   → Status of all dependencies
```

---

## Documentation Standards

### Code Documentation
- **Package comments** for all packages
- **Function comments** for exported functions
- **Type comments** for all DTOs and domain objects
- **Example usage** in comments for complex functions

### API Documentation
- **OpenAPI specs** for all endpoints
- **Request/response examples** for each DTO
- **Error response documentation**
- **Authentication requirements** clearly marked

### Architecture Decision Records
Document all major decisions:
- **Why this layer separation?**
- **Why HTMX over React/Vue?**
- **Why Templ over html/template?**
- **Why feature-based DTO organization?**

---

## Deployment Guidelines

### Container Configuration
```dockerfile
# Multi-stage build
FROM golang:1.23-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

# Build assets first
COPY cmd/web/ ./cmd/web/
RUN templ generate
RUN tailwindcss -i cmd/web/src/css/input.css -o cmd/web/static/css/output.css --minify

# Build application
COPY . .
RUN go build -o bin/app cmd/app/main.go

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/bin/app .
COPY --from=builder /app/cmd/web/static ./static
CMD ["./app"]
```

### Environment Variables (Production)
```bash
ENV=production
PORT=8080
DB_DRIVER=mysql
DB_DSN=${DATABASE_URL}
JWT_SECRET=${JWT_SECRET}
CSRF_SECRET=${CSRF_SECRET}
LOG_LEVEL=info
CACHE_STATIC=31536000
SECURE_COOKIES=true
TRUSTED_PROXIES=10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
```

### Load Balancer Configuration
- **Health checks** on `/health/ready`
- **Session affinity** if using server-side sessions
- **Static asset caching** with long TTL
- **GZIP compression** for text responses

---

## Appendix: Verification Commands

### Pre-Development Checklist
```bash
# Verify tools are installed
go version                    # Should be 1.23.x
templ version                # Should be v0.2.x
tailwindcss --version        # Should be v3.4.x

# Verify project structure
ls cmd/web/templates/        # Should exist
ls internal/types/           # Should exist  
ls internal/domain/          # Should exist

# Verify build works
templ generate
tailwindcss -i cmd/web/src/css/input.css -o cmd/web/static/css/output.css
go build -o bin/app cmd/app/main.go
```

### Post-Development Validation
```bash
# Run all tests
go test ./... -race -cover

# Verify no cross-layer imports
grep -r "gorm" internal/handlers/  # Should be empty
grep -r "gorm" internal/services/  # Should be empty
grep -r "internal/models" internal/services/  # Should be empty
grep -r "internal/domain" internal/handlers/  # Should be empty

# Verify DTO organization
ls internal/types/ | grep -E "(auth|purchase|finance|health|decision|user|common)_dto.go"

# Verify template generation
find cmd/web/templates/ -name "*.templ" -exec sh -c 'test -f "${1%.*}_templ.go"' _ {} \; -print

# Security audit
gosec ./...
```

This completes the comprehensive, corrected CLAUDE.md rule file that addresses all the inconsistencies and gaps found in the original version.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DuckDHD)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DuckDHD)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
