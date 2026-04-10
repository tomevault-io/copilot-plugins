## readers-app

> A personal project designed to combat information overload and create a focused, personal knowledge base from online content.

# Readers App - Agent Documentation

A personal project designed to combat information overload and create a focused, personal knowledge base from online content.

## Project Overview

The app allows users to save web content locally, creating a searchable personal library of articles, blog posts, and tutorials. Users provide a URL, the app scrapes and stores the content, making it available for later reading and reference.

## Architecture

```
readers-app/
├── backend-service/      # Main API server (Go + Fiber)
│   ├── cmd/              # Application entrypoint
│   ├── components/       # Templ UI components
│   ├── internal/         # Internal packages (messaging)
│   ├── migrations/      # SQL migrations
│   ├── web/             # Static assets
│   └── .air.toml       # Hot reload configuration
├── resource-service/    # Web scraping service (Go)
│   ├── cmd/            # Application entrypoint
│   ├── internal/       # Internal packages (scraping, messaging)
│   └── files/          # Scraped content storage
├── api-tests/          # Bruno API tests
└── docker-compose.yaml # PostgreSQL & RabbitMQ
```

## How It Works

### Core Workflow: Saving a Resource

When a user saves a URL, the following happens:

1. **API Request**: User sends `POST /api/resource` with `{ "url": "https://example.com/article" }`
2. **Validation**: Backend validates the URL, cleans it (removes www, normalizes path)
3. **Database Creation**: Creates a resource record with status `PENDING`
4. **User Association**: Links the resource to the user in `user_resources` table
5. **Queue Message**: Publishes scraping task to RabbitMQ
6. **Async Processing**: Resource-service picks up the task, scrapes the URL
7. **Response**: Resource-service publishes result back to backend
8. **Update**: Backend updates the resource with title, excerpt, language, and final status

### Message Flow

```
┌─────────────┐     ┌─────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   Client    │────▶│   Backend   │────▶│    RabbitMQ      │────▶│ Resource Service│
└─────────────┘     └─────────────┘     │ scraping.exchange│     └──────────────────┘
                                        │                  │              │
                                        │ scraping.request │              │
                                        │    (queue)       │              ▼
                                        │                  │     ┌──────────────────┐
                                        │ scraping.response│◀────│ Scrapes URL      │
                                        │    (queue)       │     │ Saves to files/  │
                                        └──────────────────┘     └──────────────────┘
                                               │
                                               ▼
                                        ┌──────────────────┐
                                        │ Updates Database │
                                        │ with Results     │
                                        └──────────────────┘
```

### Database Schema

| Table | Description |
|-------|-------------|
| `users` | Authenticated users (Firebase UID as primary key) |
| `resources` | Saved URLs with metadata (url, title, excerpt, language, status_id) |
| `resources_status` | Lookup table: PENDING, OK, FAILED |
| `user_resources` | Many-to-many relationship between users and resources |

### RabbitMQ Configuration

| Exchange | Type | Queue | Routing Key |
|----------|------|-------|-------------|
| `scraping.exchange` | topic | `scraping.requests` | `scraping.request` |
| `scraping.exchange` | topic | `scraping.responses` | `scraping.response` |

### Message Formats

**Scraping Request** (backend → resource-service):
```json
{
  "user_id": "firebase-uid",
  "resource_id": "uuid",
  "url": "example.com/article"
}
```

**Scraping Response** (resource-service → backend):

Success:
```json
{
  "resource_id": "uuid",
  "status": "OK",
  "title": "Article Title",
  "excerpt": "Brief description...",
  "language": "en"
}
```

Failure:
```json
{
  "resource_id": "uuid",
  "status": "FAILED"
}
```

## Tech Stack

- **Backend**: Go 1.25.5, Fiber framework, GORM
- **Database**: PostgreSQL 16
- **Messaging**: RabbitMQ 3-management
- **Auth**: Firebase
- **Frontend**: Templ, Tailwind CSS, DaisyUI
- **Scraping**: Colly, go-readability

## Getting Started

### Prerequisites

- Go 1.25.5+
- Docker & Docker Compose
- Node.js (for Tailwind CSS)

### Setup

1. Start infrastructure services:
   ```bash
   docker-compose up -d
   ```

2. Backend service runs on port **8080**
3. RabbitMQ management UI: http://localhost:15672 (admin/admin)

### Environment Variables

Required variables are defined in `.env` files in each service directory:
- `backend-service/.env`
- `resource-service/.env`

### Running the Services

**Backend Service:**
```bash
cd backend-service
air
```

**Resource Service:**
```bash
cd resource-service
go run cmd/main.go
```

**Frontend CSS:**
```bash
cd backend-service
npm run build:css
```

### Development Commands

- `go run cmd/main.go` - Run directly (no hot reload)
- `templ generate` - Generate Templ components
- `air` - Hot reload (backend-service)

## Code Conventions

- Use standard Go project layout
- No linter configured
- No tests currently configured
- Templ components use `.templ` extension with corresponding `.go` files

## API Testing

API tests are located in `api-tests/` and use Bruno. Import the collection to test endpoints.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomascaceres14)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tomascaceres14)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
