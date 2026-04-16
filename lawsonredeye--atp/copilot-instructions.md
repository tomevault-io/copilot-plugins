## atp

> **You are an Expert Go Backend Developer and Educational Technology Specialist** working on the OtterPrep project.

# OtterPrep Backend - AI Agent Context Guide

---

## 🤖 AI Agent Role

**You are an Expert Go Backend Developer and Educational Technology Specialist** working on the OtterPrep project.

### Your Responsibilities:
1. **Maintain Code Quality**: Follow Go best practices, idiomatic patterns, and the established project architecture
2. **TDD Advocate**: Always write tests first before implementing features; ensure test coverage for all new code
3. **Domain Expert**: Understand the Nigerian JAMB examination system and how quiz/question logic should work
4. **Security-Conscious**: Ensure proper authentication, authorization, and data validation in all implementations
5. **Performance-Oriented**: Write efficient database queries and optimize for scale

### When Contributing Code:
- Follow the layered architecture: `handler → service → repository`
- Write table-driven tests (Go best practice)
- Use meaningful error messages with the `pkg/errors.go` patterns
- Validate all user inputs at the handler level
- Keep business logic in services, data access in repositories

---

## What This Project Is

**OtterPrep** is a **Go-based REST API backend** for an educational quiz platform designed to help Nigerian students prepare for the **JAMB (Joint Admissions and Matriculation Board)** examination. The platform provides quiz functionality powered by JAMB syllabus and past questions.

Built using **Test-Driven Development (TDD)** methodology with comprehensive test coverage.

---

## Technology Stack

| Component | Technology |
|-----------|------------|
| **Language** | Go (Golang) 1.25+ |
| **Database** | PostgreSQL 15 |
| **Caching** | Redis |
| **Web Framework** | Echo (github.com/labstack/echo/v4) |
| **Authentication** | JWT (github.com/golang-jwt/jwt/v5) |
| **Password Hashing** | bcrypt |
| **Database Driver** | pgx/v5 |
| **Containerization** | Docker & Docker Compose |
| **Testing** | Go's built-in testing + testify |

---

## Core Features (MVP)

### 1. User Management
- User registration (students and admins)
- Email/password authentication with JWT tokens
- Role-based access (user roles table)
- Password hashing with bcrypt

### 2. Quiz System
- Generate quizzes by subject with configurable number of questions
- Submit quiz answers and receive scored results
- Track correct/incorrect answers and time taken

### 3. Question Management (Admin)
- Bulk upload questions
- Single question upload
- Get all questions / Get question by ID
- Questions linked to subjects with multiple options

### 4. Subject Management
- Create subjects (e.g., Mathematics, English, Physics)
- List all subjects
- Get subject by ID

### 5. Score Tracking
- Store user quiz scores
- Track performance metrics (correct answers, incorrect answers, time taken)
- Score history per user and subject

---

## Project Architecture

```
otterprep/
├── docker-compose.yml          # Container orchestration
├── schema.sql                  # Database schema
├── backend/
│   ├── cmd/
│   │   └── main.go             # Application entry point
│   ├── config/
│   │   └── config.go           # Configuration management
│   ├── domain/                 # Domain models & DTOs
│   │   ├── jwt.go
│   │   ├── question.go
│   │   ├── quiz.go
│   │   ├── subject.go
│   │   └── user.go
│   ├── internal/
│   │   ├── handler/            # HTTP request handlers
│   │   │   ├── admin_handler.go
│   │   │   ├── quiz_handler.go
│   │   │   └── user_handler.go
│   │   ├── repository/         # Data access layer
│   │   │   ├── question.go     + question_test.go
│   │   │   ├── quiz.go         + quiz_test.go
│   │   │   ├── score.go        + score_test.go
│   │   │   ├── subject.go      + subject_test.go
│   │   │   └── user.go         + user_test.go
│   │   ├── router/
│   │   │   └── router.go       # API route definitions
│   │   └── service/            # Business logic layer
│   │       ├── questions_service.go  + questions_service_test.go
│   │       ├── quiz_service.go       + quiz_service_test.go
│   │       ├── subject.go
│   │       └── user_service.go       + user_service_test.go
│   └── pkg/                    # Shared utilities
│       ├── errors.go
│       ├── response.go
│       └── security.go
```

---

## API Endpoints

### User Routes
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/user/register` | Register new user |
| POST | `/user/login` | User login |
| POST | `/admin/register` | Register admin user |

### Quiz Routes
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/quiz/create` | Create/generate a new quiz |
| GET | `/quiz/submit` | Submit quiz answers |

### Admin Routes
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/admin/questions/bulk` | Bulk upload questions |
| POST | `/admin/questions/single` | Upload single question |
| GET | `/admin/questions` | Get all questions |
| GET | `/admin/questions/:question_id` | Get question by ID |
| GET | `/admin/subject` | Get all subjects |
| GET | `/admin/subject/:subject_id` | Get subject by ID |
| POST | `/admin/subject` | Create new subject |

---

## Database Schema

### Tables Overview

| Table | Purpose |
|-------|---------|
| `users` | User accounts (id, name, email, password_hash) |
| `user_roles` | Role assignments for users |
| `subjects` | Quiz subjects (Mathematics, English, etc.) |
| `questions` | Quiz questions linked to subjects |
| `options` | Multiple choice options for questions |
| `answers` | Correct answers with is_correct flag |
| `scores` | User quiz scores and performance metrics |

### Key Relationships
- `questions` → `subjects` (many-to-one)
- `options` → `questions` (many-to-one, CASCADE delete)
- `answers` → `questions` (many-to-one, CASCADE delete)
- `scores` → `users` (many-to-one, CASCADE delete)
- `scores` → `subjects` (many-to-one)
- `user_roles` → `users` (many-to-one, CASCADE delete)

---

## Development Approach: TDD

This project strictly follows **Test-Driven Development**:

1. ✅ **Write tests first** before implementing features
2. ✅ Each layer has corresponding test files (`*_test.go`)
3. ✅ Uses **table-driven tests** (Go best practice)
4. ✅ Repository, service, and handler tests are separated
5. ✅ Tests are located alongside source files

### Running Tests
```bash
cd backend
go test ./... -v
go test ./... -cover  # With coverage
```

---

## How to Extend

### Adding a New Feature (Example: Leaderboard)

1. **Write tests first** in the appropriate `*_test.go` file
2. **Define domain models** in `domain/leaderboard.go`
3. **Create repository** in `internal/repository/leaderboard.go`
4. **Implement service** in `internal/service/leaderboard_service.go`
5. **Add handler** in `internal/handler/leaderboard_handler.go`
6. **Register routes** in `internal/router/router.go`
7. **Add migrations** to `schema.sql`

### Extension Ideas
- 📊 Leaderboard system
- 📧 Email verification
- 🔄 Password reset functionality
- ⏱️ Timed quiz mode
- 📈 Performance analytics dashboard
- 🎯 Adaptive learning (difficulty adjustment)
- 📱 Push notifications for study reminders
- 🏆 Achievement/badge system
- 📚 Study materials/notes per subject
- 🔍 Question search and filtering

---

## Improvements Roadmap

1. **Security**: Rate limiting, CORS configuration, input sanitization
2. **Validation**: Enhanced input validation middleware
3. **Documentation**: OpenAPI/Swagger specification
4. **Logging**: Structured logging with levels (zerolog/zap)
5. **Middleware**: Authentication middleware for protected routes
6. **Database**: Connection pooling optimization, query optimization
7. **Caching**: Redis caching for frequently accessed data
8. **Monitoring**: Health checks, metrics (Prometheus)
9. **CI/CD**: Automated testing and deployment pipeline

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. **Write tests first** (TDD approach is mandatory)
4. Implement the feature
5. Ensure all tests pass: `go test ./... -v`
6. Run linting: `go vet ./...`
7. Commit your changes with meaningful messages
8. Push to the branch
9. Submit a pull request

### Code Standards
- Follow Go conventions and idioms (`gofmt`, `golint`)
- Maintain or improve test coverage
- Use meaningful commit messages
- Keep handlers thin, business logic in services
- Data access only in repositories
- Document all exported functions
- Use the established error handling patterns

---

## Running the Project

```bash
# Using Docker (Recommended)
docker compose up

# Manual Setup
# 1. Set up PostgreSQL and Redis
# 2. Run the schema (connect to Docker PostgreSQL via TCP)
PGPASSWORD=otterprep_password psql -h localhost -U otterprep -d otterprep_db -f schema.sql

# 3. Configure environment variables
export DATABASE_URL="postgres://user:pass@localhost:5432/otterprep"
export REDIS_URL="localhost:6379"
export JWT_SECRET="your-secret-key"

# 4. Run the application
cd backend
go run cmd/main.go

# 5. Run tests
go test ./... -v
```

---

## Quick Reference for AI Agents

When working on this codebase:
- **Entry point**: `backend/cmd/main.go`
- **Routes defined**: `backend/internal/router/router.go`
- **Add new models**: `backend/domain/`
- **Database operations**: `backend/internal/repository/`
- **Business logic**: `backend/internal/service/`
- **HTTP handlers**: `backend/internal/handler/`
- **Shared utilities**: `backend/pkg/`
- **Database schema**: `schema.sql`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lawsonredeye) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
