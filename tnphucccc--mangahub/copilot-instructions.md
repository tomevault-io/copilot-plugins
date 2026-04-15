## mangahub

> This document provides essential context for Gemini working on the MangaHub project. Read this first before making code suggestions or modifications.

# GEMINI.md

## Project Context for Gemini

This document provides essential context for Gemini working on the MangaHub project. Read this first before making code suggestions or modifications.

---

## Project Overview

**MangaHub** is a manga tracking system built for a university Network Programming course (IT096IU). It's an **academic project** demonstrating network protocols in Go, not a production application.

### Key Facts
- **Language**: Go 1.19+
- **Timeline**: 10-11 weeks (student team project)
- **Team Size**: 2 students
- **Primary Goal**: Demonstrate mastery of 5 network protocols
- **Grade Weight**: 30% of course grade (100 points total)

### Critical Requirements
This project **MUST** implement all 5 protocols:
1. ✅ HTTP REST API (25 pts) - User auth, manga CRUD, library management
2. ✅ TCP Server (20 pts) - Real-time progress synchronization
3. ✅ UDP Broadcaster (15 pts) - Chapter release notifications
4. ✅ WebSocket (15 pts) - Real-time chat
5. ✅ gRPC (10 pts) - Internal service communication

**DO NOT** suggest removing or combining protocols - each must be clearly demonstrated.

---

## Project Structure Philosophy

```
mangahub/
├── cmd/              # One binary per server (REQUIRED: separate processes)
│   ├── api-server/   # HTTP + WebSocket (ports 8080, 9093)
│   ├── tcp-server/   # TCP sync (port 9090)
│   ├── udp-server/   # UDP notifications (port 9091)
│   ├── grpc-server/  # gRPC internal (port 9092)
│   └── cli/          # Client CLI tool
├── internal/         # Private packages (business logic + protocol implementations)
├── pkg/              # Shared libraries (models, database, config)
├── proto/            # Protocol Buffer definitions
├── data/             # JSON manga data files
└── migrations/       # SQL schema migrations
```

### Why This Structure?
- **Academic Grading**: Instructors need to easily find each protocol implementation
- **Protocol Isolation**: Each server can be tested independently
- **Team Collaboration**: 2 students work on different servers without conflicts
- **Bonus Points**: Enables Docker Compose (10 pts) and other bonuses

---

## Design Principles

### 1. **Clarity Over Cleverness**
This is a learning project. Code should be:
- ✅ Easy to understand for junior developers
- ✅ Well-commented (explain *why*, not *what*)
- ✅ Following Go standard practices
- ❌ NOT overly optimized or using obscure patterns

**Example - GOOD:**
```go
// Broadcast progress update to all connected TCP clients
// Uses goroutines to prevent blocking on slow clients
func (s *Server) BroadcastProgress(update ProgressUpdate) {
    for clientID, conn := range s.clients {
        go func(c net.Conn, id string) {
            // ... send update
        }(conn, clientID)
    }
}
```

**Example - BAD:**
```go
// Too clever, hard to grade
func (s *Server) BroadcastProgress(u ProgressUpdate) {
    sync.Map.Range(func(k, v interface{}) bool {
        go func() { v.(net.Conn).Write(mustMarshal(u)) }()
        return true
    })
}
```

### 2. **Separate Protocol Implementations**
Each protocol lives in its own package:

```go
// ❌ WRONG - mixing protocols
package server
type Server struct {
    httpServer *http.Server
    tcpListener net.Listener
    udpConn *net.UDPConn
    // ... all in one struct
}

// ✅ CORRECT - separate servers
// internal/tcp/server.go
package tcp
type ProgressSyncServer struct { /* TCP-specific fields */ }

// internal/udp/server.go
package udp
type NotificationServer struct { /* UDP-specific fields */ }
```

**Reason**: Instructors grade each protocol separately.

### 3. **Shared Business Logic**
Don't duplicate code across servers:

```go
// ✅ CORRECT - shared service
// pkg/manga/service.go
type Service struct {
    repo Repository
}
func (s *Service) SearchManga(query string) ([]Manga, error) {
    // ... business logic
}

// HTTP handler calls it
// internal/manga/handler.go
func (h *Handler) Search(c *gin.Context) {
    results, err := h.service.SearchManga(c.Query("q"))
    // ...
}

// gRPC service calls it
// internal/grpc/service.go
func (s *Server) SearchManga(ctx context.Context, req *pb.SearchRequest) (*pb.SearchResponse, error) {
    results, err := s.mangaService.SearchManga(req.Query)
    // ...
}
```

---

## Technical Stack & Constraints

### Required Libraries
```go
// Core framework (specified in project doc)
"github.com/gin-gonic/gin"              // HTTP web framework
"github.com/golang-jwt/jwt/v4"          // JWT authentication
"github.com/gorilla/websocket"          // WebSocket support
"github.com/mattn/go-sqlite3"           // SQLite driver
"google.golang.org/grpc"                // gRPC framework
"google.golang.org/protobuf"            // Protocol Buffers
```

### Database Constraints
- ✅ **Use SQLite3** (specified in requirements)
- ✅ **3 core tables**: users, manga, user_progress
- ❌ **Do NOT suggest PostgreSQL/MySQL** unless specifically asked for bonus features
- ❌ **No ORMs** (use database/sql directly - demonstrates SQL knowledge)

### Data Constraints
- Must have **200 manga total** (100 manual + 100 from MangaDx API)
- Must store manga as **JSON files** in `data/manga/` directory
- Minimum **30-40 different series** across genres

---

## Coding Standards

### Go Style Guide
Follow [Effective Go](https://go.dev/doc/effective_go) and [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md):

**Naming Conventions:**
```go
// ✅ Exported types (PascalCase)
type MangaService struct {}

// ✅ Unexported types (camelCase)
type progressUpdate struct {}

// ✅ Interface names end with -er or describe behavior
type Broadcaster interface {}

// ✅ Package names are lowercase, no underscores
package manga  // not manga_service
```

**Error Handling:**
```go
// ✅ Always check errors explicitly
result, err := doSomething()
if err != nil {
    return fmt.Errorf("failed to do something: %w", err)
}

// ❌ Never ignore errors
result, _ := doSomething()  // FORBIDDEN
```

**Concurrency:**
```go
// ✅ Use goroutines for I/O-bound tasks
go func() {
    // ... handle connection
}()

// ✅ Always use context for cancellation
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

// ✅ Use channels for communication
updates := make(chan ProgressUpdate, 100)
```

### Project-Specific Conventions

**1. Handler → Service → Repository Pattern**
```go
// HTTP Layer (internal/manga/handler.go)
func (h *Handler) GetManga(c *gin.Context) {
    manga, err := h.service.GetByID(c.Param("id"))
    // ... return JSON
}

// Business Logic (internal/manga/service.go)
func (s *Service) GetByID(id string) (*models.Manga, error) {
    return s.repo.FindByID(id)
}

// Data Access (internal/manga/repository.go)
func (r *Repository) FindByID(id string) (*models.Manga, error) {
    // ... SQL query
}
```

**2. Models in pkg/models/**
```go
// ✅ All data structures in pkg/models/
package models

type Manga struct {
    ID          string    `json:"id" db:"id"`
    Title       string    `json:"title" db:"title"`
    Author      string    `json:"author" db:"author"`
    Genres      []string  `json:"genres" db:"genres"`  // JSON array as string in DB
    Status      string    `json:"status" db:"status"`
    TotalChaps  int       `json:"total_chapters" db:"total_chapters"`
    Description string    `json:"description" db:"description"`
}
```

**3. Configuration via YAML**
```yaml
# configs/dev.yaml
server:
  host: "localhost"
  http_port: 8080
  tcp_port: 9090
  udp_port: 9091
  grpc_port: 9092
  websocket_port: 9093

database:
  path: "./data/mangahub.db"
```

---

## Protocol Implementation Guidelines

### HTTP REST API (internal/manga/, internal/user/, etc.)

**Required Endpoints:**
```go
// Authentication
POST   /auth/register
POST   /auth/login

// Manga
GET    /manga              // Search with query params
GET    /manga/:id          // Get details
POST   /users/library      // Add to library
GET    /users/library      // Get user's library
PUT    /users/progress     // Update reading progress
```

**Patterns to Use:**
```go
// ✅ Gin framework with middleware
router := gin.Default()
router.Use(middleware.Logger())
router.Use(middleware.Auth())  // JWT validation

// ✅ Group related routes
api := router.Group("/api/v1")
{
    manga := api.Group("/manga")
    {
        manga.GET("", handler.Search)
        manga.GET("/:id", handler.GetByID)
    }
}
```

### TCP Progress Sync (internal/tcp/)

**Key Requirements:**
- Accept multiple concurrent connections
- Broadcast progress updates to all connected clients
- Use JSON for messages over TCP
- Handle graceful disconnections

**Pattern to Use:**
```go
type ProgressSyncServer struct {
    Port        string
    Connections map[string]net.Conn  // userID -> connection
    Broadcast   chan ProgressUpdate
    mu          sync.RWMutex
}

// ✅ Use goroutine per connection
func (s *ProgressSyncServer) handleConnection(conn net.Conn) {
    defer conn.Close()
    // ... read/write loop
}

// ✅ Broadcast to all without blocking
func (s *ProgressSyncServer) broadcastLoop() {
    for update := range s.Broadcast {
        s.mu.RLock()
        for _, conn := range s.Connections {
            go s.sendUpdate(conn, update)  // Non-blocking
        }
        s.mu.RUnlock()
    }
}
```

### UDP Notifications (internal/udp/)

**Key Requirements:**
- Listen for client registrations
- Store client UDP addresses
- Broadcast notifications to all registered clients

**Pattern to Use:**
```go
type NotificationServer struct {
    Port    string
    Clients []net.UDPAddr
    mu      sync.RWMutex
}

// ✅ UDP is connectionless
func (s *NotificationServer) Broadcast(notif Notification) {
    data, _ := json.Marshal(notif)
    s.mu.RLock()
    defer s.mu.RUnlock()

    for _, addr := range s.Clients {
        s.conn.WriteToUDP(data, &addr)  // Fire and forget
    }
}
```

### WebSocket Chat (internal/websocket/)

**Key Requirements:**
- Hub pattern for message broadcasting
- Handle user join/leave
- Support multiple chat rooms (bonus)

**Pattern to Use (Hub-Based):**
```go
type Hub struct {
    Clients    map[*Client]bool
    Broadcast  chan Message
    Register   chan *Client
    Unregister chan *Client
}

// ✅ Central hub goroutine
func (h *Hub) Run() {
    for {
        select {
        case client := <-h.Register:
            h.Clients[client] = true
        case client := <-h.Unregister:
            delete(h.Clients, client)
        case message := <-h.Broadcast:
            for client := range h.Clients {
                client.Send <- message
            }
        }
    }
}
```

### gRPC Service (internal/grpc/ + proto/)

**Key Requirements:**
- Define 2-3 services in .proto files
- Implement unary RPC calls
- Use for internal service-to-service communication

**Pattern to Use:**
```protobuf
// proto/manga.proto
syntax = "proto3";
package manga;

service MangaService {
    rpc GetManga(GetMangaRequest) returns (MangaResponse);
    rpc SearchManga(SearchRequest) returns (SearchResponse);
}

message GetMangaRequest {
    string manga_id = 1;
}

message MangaResponse {
    string id = 1;
    string title = 2;
    // ... other fields
}
```

```go
// internal/grpc/service.go
type Server struct {
    pb.UnimplementedMangaServiceServer
    mangaService *manga.Service
}

func (s *Server) GetManga(ctx context.Context, req *pb.GetMangaRequest) (*pb.MangaResponse, error) {
    manga, err := s.mangaService.GetByID(req.MangaId)
    if err != nil {
        return nil, status.Errorf(codes.NotFound, "manga not found")
    }
    return &pb.MangaResponse{
        Id:    manga.ID,
        Title: manga.Title,
        // ...
    }, nil
}
```

---

## Database Schema

**Required Tables (from project spec):**

```sql
-- migrations/001_create_users.sql
CREATE TABLE users (
    id TEXT PRIMARY KEY,
    username TEXT UNIQUE NOT NULL,
    email TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- migrations/002_create_manga.sql
CREATE TABLE manga (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    author TEXT,
    genres TEXT,  -- JSON array stored as TEXT
    status TEXT CHECK(status IN ('ongoing', 'completed', 'hiatus')),
    total_chapters INTEGER DEFAULT 0,
    description TEXT,
    cover_url TEXT
);

-- migrations/003_create_user_progress.sql
CREATE TABLE user_progress (
    user_id TEXT NOT NULL,
    manga_id TEXT NOT NULL,
    current_chapter INTEGER DEFAULT 0,
    status TEXT CHECK(status IN ('reading', 'completed', 'plan_to_read', 'on_hold', 'dropped')),
    rating INTEGER CHECK(rating BETWEEN 1 AND 10),
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, manga_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (manga_id) REFERENCES manga(id) ON DELETE CASCADE
);

CREATE INDEX idx_user_progress_user ON user_progress(user_id);
CREATE INDEX idx_user_progress_manga ON user_progress(manga_id);
CREATE INDEX idx_manga_status ON manga(status);
```

**DO NOT** modify this schema without good reason - it matches project requirements.

---

## Testing Requirements

### What Must Be Tested

**1. Unit Tests (in same package):**
```go
// internal/manga/service_test.go
func TestMangaService_Search(t *testing.T) {
    // ... test business logic
}
```

**2. Integration Tests (test/ directory):**
```go
// test/integration/http_test.go
func TestHTTPServer_MangaEndpoints(t *testing.T) {
    // Start test server
    // Make HTTP requests
    // Verify responses
}
```

**3. Protocol Tests (demonstrate each protocol works):**
```go
// test/integration/tcp_test.go
func TestTCPServer_ProgressSync(t *testing.T) {
    // Connect as client
    // Send progress update
    // Verify broadcast received
}
```

### Testing Commands
```bash
# Run all tests
make test

# Run with coverage
make test-coverage

# Run integration tests only
make test-integration

# Test specific protocol
go test ./internal/tcp/...
```

---

## Common Development Tasks

### Starting All Servers
```bash
# Development mode (separate terminals)
go run cmd/api-server/main.go
go run cmd/tcp-server/main.go
go run cmd/udp-server/main.go
go run cmd/grpc-server/main.go

# Production mode (Docker Compose)
docker-compose up
```

### Database Operations
```bash
# Run migrations
make migrate-up

# Rollback migrations
make migrate-down

# Seed database with manga data
make seed-db

# Reset database (drop + migrate + seed)
make reset-db
```

### Generating Code
```bash
# Generate gRPC code from .proto files
make proto

# Generate mocks for testing
make mocks
```

### Data Import
```bash
# Fetch 100 manga from MangaDx API
go run scripts/import_mangadex.go

# Manual entries should be added to data/manga/manual_entries.json
```

---

## DO NOT Suggestions

### ❌ **Do NOT suggest these changes:**

1. **Combining servers into one binary**
   - Reason: Must demonstrate separate protocol implementations

2. **Using PostgreSQL instead of SQLite**
   - Reason: Spec requires SQLite (unless bonus feature requested)

3. **Adding ORMs (GORM, SQLBoiler, etc.)**
   - Reason: Project should demonstrate raw SQL knowledge

4. **Removing any of the 5 protocols**
   - Reason: All are grading requirements

5. **Complex design patterns (Factory, Strategy, etc.)**
   - Reason: Academic project for junior developers

6. **Frontend frameworks (React, Vue)**
   - Reason: CLI-only project (unless bonus feature requested)

7. **Microservices frameworks (Kubernetes, Istio)**
   - Reason: Over-engineering for 11-week project

8. **NoSQL databases (MongoDB, Redis)**
   - Reason: Out of scope (Redis OK as bonus for caching)

---

## When to Suggest Bonus Features

The project allows bonus features for extra credit. Suggest these **ONLY IF**:
- ✅ Core 5 protocols are fully implemented
- ✅ Student explicitly asks for bonus ideas
- ✅ Time allows (Week 8+)

**Approved Bonus Features (from spec):**
- Docker Compose setup (10 pts) - **HIGH PRIORITY**
- Advanced search with filters (5 pts)
- User reviews & ratings (8 pts)
- Friend system (5 pts)
- Reading statistics dashboard (8 pts)
- Redis caching (10 pts)
- CI/CD pipeline (10 pts)

---

## File Creation Order (For New Features)

When adding a new feature (e.g., "User Reviews"), follow this order:

1. **Model** (`pkg/models/review.go`)
   ```go
   type Review struct {
       ID        string
       UserID    string
       MangaID   string
       Rating    int
       Text      string
       CreatedAt time.Time
   }
   ```

2. **Migration** (`migrations/004_create_reviews.sql`)
   ```sql
   CREATE TABLE reviews (...);
   ```

3. **Repository** (`internal/review/repository.go`)
   ```go
   func (r *Repository) Create(review *models.Review) error
   func (r *Repository) FindByMangaID(mangaID string) ([]models.Review, error)
   ```

4. **Service** (`internal/review/service.go`)
   ```go
   func (s *Service) SubmitReview(review *models.Review) error
   func (s *Service) GetMangaReviews(mangaID string) ([]models.Review, error)
   ```

5. **Handler** (`internal/review/handler.go`)
   ```go
   func (h *Handler) Submit(c *gin.Context)
   func (h *Handler) GetByManga(c *gin.Context)
   ```

6. **Routes** (add to `cmd/api-server/main.go`)
   ```go
   api.POST("/manga/:id/reviews", reviewHandler.Submit)
   api.GET("/manga/:id/reviews", reviewHandler.GetByManga)
   ```

7. **Tests** (`internal/review/service_test.go`)

---

## Debugging Tips

### Common Issues & Solutions

**Problem: TCP connections not closing properly**
```go
// ✅ Always use defer
func handleConnection(conn net.Conn) {
    defer conn.Close()  // MUST have this
    // ... handle connection
}
```

**Problem: Database locked errors (SQLite)**
```go
// ✅ Use connection pooling
db.SetMaxOpenConns(1)  // SQLite doesn't support concurrent writes
db.SetMaxIdleConns(1)
```

**Problem: Goroutine leaks**
```go
// ✅ Always provide exit mechanism
func (s *Server) Start() error {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    go s.worker(ctx)  // Worker checks ctx.Done()
}
```

**Problem: JSON unmarshaling fails**
```go
// ✅ Check for pointer vs value
var manga models.Manga
json.Unmarshal(data, &manga)  // Note the &
```

---

## Project Milestones (For Progress Tracking)

### Week 1-2: Foundation ✅
- [ ] Project structure created
- [ ] Database schema + migrations
- [ ] User authentication (register, login, JWT)
- [ ] Basic manga CRUD (HTTP)

### Week 3-4: TCP Protocol ✅
- [ ] TCP server accepts connections
- [ ] Progress update broadcasting
- [ ] Client connection handling
- [ ] Integration with HTTP API

### Week 5: UDP Protocol ✅
- [ ] UDP server listening
- [ ] Client registration
- [ ] Notification broadcasting

### Week 6: WebSocket Protocol ✅
- [ ] WebSocket hub implemented
- [ ] Real-time chat working
- [ ] User join/leave notifications

### Week 7: gRPC Protocol ✅
- [ ] .proto files defined
- [ ] gRPC server running
- [ ] 2-3 services implemented
- [ ] Integration tests passing

### Week 8: Integration ✅
- [ ] All 5 protocols working together
- [ ] Data collection complete (200 manga)
- [ ] End-to-end testing

### Week 9: Polish ✅
- [ ] CLI tool completed
- [ ] Docker Compose working
- [ ] Documentation finished

### Week 10: Demo Preparation ✅
- [ ] Demo script prepared
- [ ] All protocols demonstrated
- [ ] Bonus features (if time)

---

## Questions to Ask Before Suggesting Code

1. **"Which protocol is this for?"**
   - Ensure code goes in correct server package

2. **"Is this a core requirement or bonus feature?"**
   - Don't over-engineer core features

3. **"Will this be easy to grade?"**
   - Code should be obvious, not clever

4. **"Does this follow Go idioms?"**
   - Check against Effective Go

5. **"Is this testable?"**
   - Avoid tight coupling

---

## Final Reminders

### For Gemini:
- ✅ This is a **learning project**, not production code
- ✅ **Clarity > Optimization** (students must understand their code)
- ✅ **All 5 protocols** must be clearly separated and demonstrated
- ✅ Follow **Go best practices** (instructors will check)
- ✅ Suggest **bonus features** only when asked
- ❌ Don't suggest **over-engineering** (Kubernetes, complex patterns)
- ❌ Don't suggest **removing protocols** or requirements

### For Developers:
- Read the project specification PDFs in the repo root
- Check `/docs/architecture.md` for design decisions
- Run `make help` to see all available commands
- Ask on the team Discord before making major changes

---

## Useful Resources

- [Effective Go](https://go.dev/doc/effective_go)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- [Go Project Layout](https://github.com/golang-standards/project-layout)
- [gRPC Go Quick Start](https://grpc.io/docs/languages/go/quickstart/)
- [Gorilla WebSocket Examples](https://github.com/gorilla/websocket/tree/master/examples)

---

**Last Updated**: 2025-12-14
**Project Version**: v1.0.0-dev
**Target Go Version**: 1.19+

---

*This file should be read by Gemini before providing code suggestions or making modifications to the MangaHub project.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tnphucccc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tnphucccc)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
