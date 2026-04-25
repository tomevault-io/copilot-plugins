## llm-buddy-rag

> You are working on LLM Buddy RAG, a sophisticated multimodal Retrieval-Augmented Generation system that handles both text and visual content. This is a performance-critical module designed to operate both as a library and as an API service.

# Claude Code Session Guidelines - LLM Buddy RAG

## Project Overview

You are working on LLM Buddy RAG, a sophisticated multimodal Retrieval-Augmented Generation system that handles both text and visual content. This is a performance-critical module designed to operate both as a library and as an API service.

## Critical Instructions

**MANDATORY**: At the start of every new conversation:
1. Always read PLANNING.md to understand the technical architecture and current implementation status
2. Check TASKS.md to see what work needs to be done
3. Before starting any task, mark it as "in progress" in TASKS.md
4. Upon completing any task, immediately mark it as "DONE" in TASKS.md
5. Add any newly discovered tasks or requirements to TASKS.md as you find them

## Coding Standards

### TypeScript Requirements
- Use TypeScript strict mode for all code
- Define explicit types for all function parameters and return values
- Use interfaces for data structures, types for unions and primitives
- Implement Zod schemas for runtime validation of external data
- Export types from a centralized `types` module

### Code Organization
```typescript
// Preferred structure for modules
export interface ServiceConfig {
  // Configuration interface
}

export class Service {
  private config: ServiceConfig;
  
  constructor(config: ServiceConfig) {
    this.config = config;
  }
  
  // Public methods with clear return types
  async process(input: Input): Promise<Output> {
    // Implementation
  }
}
```

### Performance Guidelines
- Use async/await for all I/O operations
- Implement connection pooling for database and API connections
- Use streaming for large data processing
- Implement proper error boundaries and recovery
- Add performance logging for operations >100ms

### Error Handling
```typescript
// Use custom error classes
export class RAGError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500
  ) {
    super(message);
    this.name = 'RAGError';
  }
}

// Wrap external calls
try {
  const result = await externalAPI.call();
  return result;
} catch (error) {
  throw new RAGError(
    'External API failed',
    'EXTERNAL_API_ERROR',
    503
  );
}
```

## Architecture Patterns

### Dual-Mode Operation
The system must support both library and API modes:

```typescript
// Library mode - direct usage
import { RAGEngine } from '@llm-buddy/rag';
const rag = new RAGEngine(config);
const results = await rag.search(query);

// API mode - HTTP interface
POST /api/search
{
  "query": "...",
  "options": {...}
}
```

### Dependency Injection
Use dependency injection for testability:

```typescript
export class SearchService {
  constructor(
    private vectorDB: VectorDatabase,
    private metadataDB: MetadataDatabase,
    private embedder: EmbeddingService
  ) {}
}
```

### Repository Pattern
Implement repositories for data access:

```typescript
export interface ContentRepository {
  save(content: Content): Promise<string>;
  findById(id: string): Promise<Content | null>;
  search(criteria: SearchCriteria): Promise<Content[]>;
  delete(id: string): Promise<boolean>;
}
```

## Testing Requirements

### Test Coverage
- Minimum 80% code coverage
- Unit tests for all business logic
- Integration tests for API endpoints
- Performance tests for critical paths

### Test Structure
```typescript
describe('SearchService', () => {
  let service: SearchService;
  let mockVectorDB: jest.Mocked<VectorDatabase>;
  
  beforeEach(() => {
    mockVectorDB = createMockVectorDB();
    service = new SearchService(mockVectorDB);
  });
  
  describe('search', () => {
    it('should return relevant results', async () => {
      // Test implementation
    });
  });
});
```

## API Design Principles

### RESTful Conventions
- Use proper HTTP methods (GET, POST, PUT, DELETE)
- Return appropriate status codes
- Include pagination for list endpoints
- Version the API from the start (/api/v1/)

### Request/Response Format
```typescript
// Consistent response structure
interface APIResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
  metadata?: {
    timestamp: string;
    requestId: string;
    version: string;
  };
}
```

### Validation
```typescript
// Use Zod for request validation
const SearchRequestSchema = z.object({
  query: z.string().min(1).max(1000),
  limit: z.number().int().min(1).max(100).default(10),
  offset: z.number().int().min(0).default(0),
  filters: z.object({
    type: z.enum(['text', 'image']).optional(),
    dateRange: z.object({
      from: z.string().datetime().optional(),
      to: z.string().datetime().optional()
    }).optional()
  }).optional()
});
```

## Database Patterns

### Vector Database (FAISS)
- Initialize with appropriate index type (IVF, HNSW)
- Implement periodic index optimization
- Handle index persistence and recovery
- Monitor memory usage

### Metadata Database (SQLite)
```sql
-- Use proper schema design
CREATE TABLE content (
  id TEXT PRIMARY KEY,
  type TEXT NOT NULL CHECK(type IN ('text', 'image')),
  content TEXT,
  embedding_id TEXT,
  metadata JSON,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_content_type ON content(type);
CREATE INDEX idx_content_created_at ON content(created_at);
```

## Performance Optimization

### Caching Strategy
```typescript
// Implement multi-level caching
class CacheManager {
  private memoryCache: LRUCache<string, any>;
  private diskCache?: DiskCache;
  
  async get(key: string): Promise<any> {
    // Check memory first, then disk
    return this.memoryCache.get(key) || 
           await this.diskCache?.get(key);
  }
}
```

### Batch Processing
```typescript
// Process in batches for efficiency
async function processBatch<T, R>(
  items: T[],
  processor: (batch: T[]) => Promise<R[]>,
  batchSize: number = 100
): Promise<R[]> {
  const results: R[] = [];
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await processor(batch);
    results.push(...batchResults);
  }
  return results;
}
```

## Security Guidelines

### Input Validation
- Sanitize all user inputs
- Implement rate limiting
- Use parameterized queries for SQL
- Validate file uploads (type, size, content)

### API Security
```typescript
// Implement API key authentication
export async function authenticateRequest(
  req: FastifyRequest
): Promise<User> {
  const apiKey = req.headers['x-api-key'];
  if (!apiKey) {
    throw new RAGError('Missing API key', 'AUTH_REQUIRED', 401);
  }
  
  const user = await validateAPIKey(apiKey);
  if (!user) {
    throw new RAGError('Invalid API key', 'AUTH_FAILED', 401);
  }
  
  return user;
}
```

## Monitoring and Logging

### Structured Logging
```typescript
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
  },
});

// Use structured logging
logger.info({
  action: 'search',
  query: query.substring(0, 100),
  resultCount: results.length,
  duration: endTime - startTime,
}, 'Search completed');
```

### Metrics Collection
```typescript
// Track key metrics
interface Metrics {
  searchLatency: Histogram;
  documentCount: Gauge;
  errorRate: Counter;
  apiRequests: Counter;
}
```

## Development Workflow

### Branch Strategy
- Main branch for stable releases
- Develop branch for integration
- Feature branches for new features
- Hotfix branches for urgent fixes

### Commit Messages
```
feat: Add image embedding generation
fix: Resolve memory leak in vector search
perf: Optimize batch processing pipeline
docs: Update API documentation
test: Add integration tests for search
```

### Code Review Checklist
- TypeScript types are explicit and correct
- Error handling is comprehensive
- Tests are included and passing
- Performance impact is considered
- Documentation is updated
- Security implications reviewed

## Package Structure

### Monorepo Organization
```
packages/
  rag-core/          # Core engine
    src/
      services/      # Business logic
      repositories/  # Data access
      utils/        # Utilities
    tests/
  rag-api/          # API server
    src/
      routes/       # API endpoints
      middleware/   # Express/Fastify middleware
      validators/   # Request validation
  rag-client/       # Smart client
    src/
      client.ts     # Main client class
      modes/        # Library/API modes
  rag-types/        # Shared types
    src/
      index.ts      # Type definitions
```

## Common Pitfalls to Avoid

1. **Memory Leaks**: Always clean up resources (connections, file handles, timers)
2. **Blocking Operations**: Never use synchronous I/O in production code
3. **Unbounded Growth**: Implement limits on all collections and queues
4. **SQL Injection**: Always use parameterized queries
5. **Race Conditions**: Use proper locking for shared resources
6. **API Versioning**: Plan for backwards compatibility from day one
7. **Error Swallowing**: Always log errors before handling them
8. **Hard-coded Values**: Use configuration for all environment-specific values

## Key Technical Decisions

1. **Fastify over Express**: Better performance for high-throughput scenarios
2. **FAISS over Pinecone**: Local processing for data privacy and cost
3. **SQLite over PostgreSQL**: Simpler deployment for embedded scenarios
4. **Zod over Joi**: Better TypeScript integration and performance
5. **Pino over Winston**: Faster JSON logging with better structure
6. **Vitest over Jest**: Faster test execution with better ESM support

## Remember

- Performance is critical - measure everything
- The system must work in both library and API modes
- Multimodal support (text + images) is a core differentiator
- Developer experience is as important as end-user experience
- Always consider the MCP integration use case for design decisions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gongiskhan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
