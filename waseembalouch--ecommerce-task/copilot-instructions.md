## ecommerce-task

> This is a comprehensive learning project that implements a full-featured e-commerce platform across multiple backend frameworks (Node.js/Express, FastAPI, Go) while sharing a common React + Material UI frontend. The goal is to understand the strengths, weaknesses, and patterns of different backend technologies when building the same application.

# E-Commerce Multi-Stack Learning Project - Architecture Documentation

## Project Vision
This is a comprehensive learning project that implements a full-featured e-commerce platform across multiple backend frameworks (Node.js/Express, FastAPI, Go) while sharing a common React + Material UI frontend. The goal is to understand the strengths, weaknesses, and patterns of different backend technologies when building the same application.

## Architecture Overview

### High-Level Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Frontend Layer                            │
│              React + TypeScript + Material UI                │
│                   (Port: 5173)                               │
└────────────────────┬────────────────────────────────────────┘
                     │ HTTP/REST API
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                 Backend Layer (Choose One)                   │
├─────────────────────────────────────────────────────────────┤
│  Node.js/Express (Port: 3000) - Primary Implementation      │
│  FastAPI (Port: 3001) - Async Python Implementation         │
│  Go (Port: 3002) - Go Implementation                        │
└────────────┬─────────────────┬──────────────────────────────┘
             │                 │
             ▼                 ▼
┌────────────────────┐  ┌──────────────────┐
│    PostgreSQL      │  │      Redis       │
│   (Port: 5432)     │  │   (Port: 6379)   │
│                    │  │                  │
│  - Users           │  │  - Sessions      │
│  - Products        │  │  - Cart Data     │
│  - Orders          │  │  - Cache         │
│  - Reviews         │  │  - Rate Limiting │
└────────────────────┘  └──────────────────┘
```

### Design Principles

#### 1. **API-First Design**
- All backends implement the same REST API contract
- OpenAPI/Swagger specification serves as the single source of truth
- Frontend is completely decoupled from backend implementation

#### 2. **Microservices-Ready**
- Each backend runs as an independent service
- Can be run simultaneously on different ports
- Allows for easy A/B testing and performance comparison

#### 3. **Stateless Backend**
- Session data stored in Redis (not in-memory)
- Enables horizontal scaling
- JWT tokens for authentication (stateless)

#### 4. **Caching Strategy**
- Redis for session management
- Redis for shopping cart (fast access)
- Database query result caching for products
- Cache invalidation on updates

#### 5. **Security First**
- JWT-based authentication
- Role-based access control (RBAC)
- Password hashing (bcrypt/argon2)
- Input validation and sanitization
- SQL injection prevention (ORM/prepared statements)
- XSS protection
- CORS configuration
- Rate limiting
- Helmet.js security headers (Node.js)

## System Components

### 1. Frontend (React + Material UI)
**Purpose**: Provide a modern, responsive user interface for customers and administrators

**Key Responsibilities**:
- User authentication UI
- Product browsing and search
- Shopping cart management
- Checkout flow
- Order history
- Admin dashboard

**Technology Choices**:
- **React 18**: Latest version with concurrent rendering
- **TypeScript**: Type safety and better DX
- **Material UI v5**: Comprehensive component library
- **Vite**: Fast build tool and dev server
- **React Query**: Server state management and caching
- **Zustand**: Client state management (lightweight)
- **React Router v6**: Routing
- **Axios**: HTTP client with interceptors

See `frontend-react/CLAUDE.md` for detailed frontend architecture.

### 2. Backend Services

#### Node.js/Express (Primary Implementation)
**Purpose**: Full-featured REST API with TypeScript

**Technology Choices**:
- **Express**: Minimal, flexible web framework
- **TypeScript**: Type safety, better maintainability
- **Prisma**: Modern ORM with type safety
- **Zod**: Runtime validation
- **JWT**: Authentication
- **Multer**: File uploads
- **Jest**: Testing framework

See `backend-nodejs/CLAUDE.md` for detailed backend architecture.

#### FastAPI (Async Python Implementation)
**Purpose**: Modern async Python framework with auto-documentation

**Technology Choices**:
- **FastAPI**: High-performance async framework
- **SQLAlchemy 2.0**: Async ORM
- **Pydantic**: Data validation
- **OAuth2**: Authentication
- **Pytest**: Testing

See `backend-fastapi/CLAUDE.md` (to be created).

#### Go (Go Implementation)
**Purpose**: Demonstrate compiled language performance

**Technology Choices**:
- **Gin/Echo**: Fast Go web frameworks
- **GORM**: Go ORM
- **JWT-Go**: Authentication
- **Go validator**: Validation
- **Go testing**: Testing

See `backend-go/CLAUDE.md` (to be created).

### 3. Database Layer (PostgreSQL)
**Purpose**: Persistent data storage

**Schema Design**:
- Normalized database design (3NF)
- Foreign key constraints
- Indexes on frequently queried columns
- Full-text search indexes for product search
- Soft deletes for important data

**Key Tables**:
- users, addresses, categories, products, product_images
- orders, order_items, reviews, sessions (optional)

### 4. Cache Layer (Redis)
**Purpose**: Fast data access and session management

**Use Cases**:
- **Session Storage**: User sessions with TTL
- **Cart Storage**: Shopping cart data (ephemeral)
- **Cache**: Product details, category lists
- **Rate Limiting**: API rate limiting per user/IP
- **Pub/Sub**: Real-time features (future)

## Data Flow

### 1. User Authentication Flow
```
User → Frontend (Login Form)
  → Backend (POST /api/auth/login)
    → Database (Verify credentials)
    → Redis (Create session)
    → Generate JWT Token
  ← Backend (Return JWT + User data)
← Frontend (Store token, redirect)
```

### 2. Product Search Flow
```
User → Frontend (Search input)
  → Backend (GET /api/products?search=xyz)
    → Redis (Check cache)
      → Cache Hit: Return cached results
      → Cache Miss:
        → Database (Full-text search query)
        → Redis (Cache results with TTL)
    ← Backend (Return products)
← Frontend (Display results)
```

### 3. Add to Cart Flow
```
User → Frontend (Click "Add to Cart")
  → Backend (POST /api/cart/items)
    → Verify authentication (JWT)
    → Database (Check product stock)
    → Redis (Update cart hash)
      → cart:{userId}:product:{productId} = quantity
  ← Backend (Return updated cart)
← Frontend (Update cart UI)
```

### 4. Checkout Flow
```
User → Frontend (Checkout form)
  → Backend (POST /api/orders)
    → Verify authentication
    → Redis (Get cart items)
    → Database Transaction:
      → Create order record
      → Create order_items records
      → Update product stock
      → Clear cart in Redis
      → Commit transaction
  ← Backend (Return order confirmation)
← Frontend (Show success, redirect)
```

## API Design Patterns

### 1. RESTful Conventions
- **GET**: Retrieve resources (idempotent)
- **POST**: Create new resources
- **PUT**: Update entire resource
- **PATCH**: Partial update
- **DELETE**: Remove resource

### 2. Response Format
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation successful",
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100
  }
}
```

### 3. Error Format
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

### 4. Pagination
```
GET /api/products?page=1&limit=20
```

### 5. Filtering
```
GET /api/products?category=electronics&minPrice=100&maxPrice=500
```

### 6. Sorting
```
GET /api/products?sort=-price (descending)
GET /api/products?sort=+createdAt (ascending)
```

## Security Architecture

### 1. Authentication
- JWT tokens with expiration
- Refresh token mechanism
- Password requirements (min length, complexity)
- Bcrypt/Argon2 for password hashing
- Account lockout after failed attempts

### 2. Authorization
- Role-based access control (Customer, Admin)
- Middleware checks for protected routes
- Resource ownership validation

### 3. Input Validation
- Schema validation (Zod, Pydantic)
- SQL injection prevention (ORM)
- XSS prevention (sanitization)
- File upload validation (type, size)

### 4. Rate Limiting
- Per-user rate limits
- Per-IP rate limits for public endpoints
- Redis-based implementation

### 5. CORS Configuration
- Allowed origins whitelist
- Credentials support
- Preflight caching

## Performance Optimization

### 1. Database
- Indexes on foreign keys and search columns
- Connection pooling
- Query optimization (N+1 prevention)
- Pagination for large datasets

### 2. Caching
- Redis for frequently accessed data
- Cache invalidation strategy
- TTL-based expiration

### 3. API
- Response compression (gzip)
- Field filtering (partial responses)
- Lazy loading for images
- CDN for static assets (future)

### 4. Frontend
- Code splitting
- Lazy loading routes
- Image optimization
- React Query caching

## Development Workflow

### 1. Local Development
```bash
# Start all services
docker-compose up -d

# Access services
Frontend: http://localhost:5173
Node.js API: http://localhost:3000
PostgreSQL: localhost:5432
Redis: localhost:6379
```

### 2. Database Migrations
```bash
# Node.js (Prisma)
npm run migrate:dev

# Flask (Alembic)
flask db migrate
flask db upgrade

# FastAPI (Alembic)
alembic revision --autogenerate
alembic upgrade head
```

### 3. Testing
```bash
# Backend tests
npm test (Node.js)
pytest (Python)
go test (Go)

# Frontend tests
npm test
```

### 4. Code Quality
- ESLint + Prettier (JavaScript/TypeScript)
- Black + Flake8 (Python)
- gofmt + golangci-lint (Go)
- Pre-commit hooks

## Deployment Strategy

### 1. Containerization
- Each service in its own Docker container
- Docker Compose for local development
- Kubernetes-ready (future)

### 2. Environment Management
- `.env` files for configuration
- Different configs for dev/staging/prod
- Secrets management (future)

### 3. CI/CD Pipeline
- GitHub Actions (or similar)
- Automated testing
- Docker image building
- Deployment automation

## Monitoring & Logging (Future)

### 1. Application Logging
- Structured logging (JSON)
- Log levels (DEBUG, INFO, WARN, ERROR)
- Centralized logging (ELK stack)

### 2. Monitoring
- Health check endpoints
- Performance metrics
- Error tracking (Sentry)
- Uptime monitoring

### 3. Analytics
- API usage statistics
- User behavior tracking
- Performance bottlenecks

## Comparison Framework

### Metrics to Compare
1. **Performance**: Response times, throughput
2. **Development Speed**: Time to implement features
3. **Code Complexity**: Lines of code, maintainability
4. **Type Safety**: Compile-time vs runtime errors
5. **Ecosystem**: Available libraries, community
6. **Learning Curve**: Ease of learning
7. **Resource Usage**: Memory, CPU usage

### Testing Methodology
- Same database with same data
- Same Redis instance
- Load testing with Apache Bench / k6
- Measure response times for each endpoint
- Document findings

## Future Enhancements
- [ ] Payment integration (Stripe/PayPal)
- [ ] Email notifications (SendGrid)
- [ ] Real-time notifications (WebSockets)
- [ ] Product recommendations
- [ ] Search autocomplete
- [ ] Wishlist feature
- [ ] Product reviews with images
- [ ] Multi-currency support
- [ ] Multi-language support (i18n)
- [ ] GraphQL API (alternative to REST)
- [ ] Mobile app (React Native)
- [ ] Advanced analytics dashboard
- [ ] Inventory management system
- [ ] Shipping integration
- [ ] Tax calculation

## Project Files Structure
```
.
├── CLAUDE.md (this file)
├── PLAN.md
├── README.md
├── docker-compose.yml
├── docker-compose.prod.yml
├── .env.example
├── .gitignore
│
├── backend-nodejs/
│   └── CLAUDE.md
│
├── backend-fastapi/
│   └── CLAUDE.md
│
├── backend-go/
│   └── CLAUDE.md
│
├── frontend-react/
│   └── CLAUDE.md
│
└── shared/
    ├── docs/
    │   └── api-specification.yml (OpenAPI)
    ├── postman/
    │   └── collection.json
    └── scripts/
        ├── seed-data.sql
        └── test-data.sql
```

## Getting Started
1. Read this architecture document
2. Review `PLAN.md` for implementation roadmap
3. Check individual `CLAUDE.md` files for component-specific details
4. Start with Node.js backend implementation
5. Build React frontend
6. Port to other backend frameworks
7. Compare and document findings

## Questions & Decisions Log

### Decision 1: Why TypeScript for Node.js?
**Decision**: Use TypeScript for both backend and frontend
**Rationale**:
- Type safety reduces runtime errors
- Better IDE support and autocomplete
- Easier refactoring
- Industry standard for modern Node.js projects

### Decision 2: Why Prisma over other ORMs?
**Decision**: Use Prisma for Node.js backend
**Rationale**:
- Type-safe database client
- Excellent TypeScript integration
- Modern migration system
- Good documentation and community

### Decision 3: Why Material UI?
**Decision**: Use Material UI for frontend
**Rationale**:
- Comprehensive component library
- Good documentation
- Customizable theming
- Accessibility built-in
- Production-ready components

### Decision 4: Why Redis for cart?
**Decision**: Store cart in Redis instead of database
**Rationale**:
- Fast read/write operations
- Automatic TTL for abandoned carts
- Reduces database load
- Industry standard for session/cart data

---

**Document Version**: 1.0
**Last Updated**: 2025-11-14
**Maintained By**: Development Team

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/waseembalouch)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/waseembalouch)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
