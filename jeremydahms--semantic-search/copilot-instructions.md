## semantic-search

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Spring Boot 3 REST API that provides semantic search over industry codes (e.g., ICD-10 medical codes) using vector embeddings. It uses Ollama's `nomic-embed-text` model to generate 768-dimensional embeddings and PostgreSQL with the pgvector extension for vector similarity search.

**Core Technologies:**
- Java 21
- Spring Boot 3.5.6
- PostgreSQL 16 with pgvector extension
- Hibernate 6.6.4 with hibernate-vector
- Ollama (local LLM for embeddings)
- Maven
- Spring Retry (fault tolerance with exponential backoff)
- Apache Commons CSV (robust CSV parsing)
- Spring Boot Actuator (health monitoring)

## Development Commands

### Running the Application

**Local development (requires PostgreSQL + Ollama running locally):**
```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=local
```

**With Docker Compose (recommended):**
```bash
# Start services (PostgreSQL + Ollama)
docker-compose up -d

# Pull Ollama model (first time only)
./scripts/init-ollama.sh
# Or manually: docker exec semantic-search-ollama ollama pull nomic-embed-text

# Run Spring Boot app
./mvnw spring-boot:run
```

### Testing & Building

```bash
# Run tests
./mvnw test

# Build JAR
./mvnw clean package

# Run JAR
java -jar target/semantic-search-0.0.1-SNAPSHOT.jar
```

### Database Access

```bash
# Connect to PostgreSQL (Docker)
docker exec -it semantic-search-postgres psql -U postgres -d semantic_search

# Check if pgvector is installed
# Inside psql:
\dx vector

# View table schema
\d industry_codes

# Drop table to reset (WARNING: deletes all data)
DROP TABLE industry_codes;
```

## Architecture

### Request Flow

1. **Upload Flow** (POST /api/v1/codes or /api/v1/codes/upload-csv):
   - Controller receives code + description
   - OllamaService generates 768-dim embedding via HTTP to Ollama API (with retry on failure)
   - Embedding dimension validated (must be 768)
   - Entity saved to PostgreSQL with vector column and audit timestamps
   - CSV uploads use batched transactions (100 rows per batch, max 1000 rows total)
   - Each batch saved in separate transaction (REQUIRES_NEW) for partial success

2. **Search Flow** (GET /api/v1/codes/search):
   - Controller receives natural language query
   - OllamaService generates embedding for query (with retry on failure)
   - Repository uses pgvector's `<=>` operator (cosine distance)
   - Returns top N results sorted by similarity (1.0 = identical, 0.0 = unrelated)

### Key Components

**Entity Layer** (`entity/IndustryCode.java`):
- JPA entity with vector column using `@JdbcTypeCode(SqlTypes.VECTOR)`
- `@Array(length = 768)` specifies vector dimensions (must match Ollama model)
- Fluent setters return `this` for method chaining
- Audit fields with `@CreatedDate` and `@LastModifiedDate` (requires @EnableJpaAuditing)
- Unique constraint on `code` field (enforced at database level)
- Custom `equals()`, `hashCode()`, and `toString()` implementations

**Repository Layer** (`repository/CodeRepository.java`):
- Native SQL query using pgvector's `<=>` operator for cosine distance
- `findSimilarCodes()` returns interface projection `CodeSimilarity`
- Similarity calculated as `1 - (embedding <=> queryEmbedding)`

**Service Layer**:
- `OllamaService.java`: Wraps Ollama API calls, generates embeddings
  - `@Retryable` with exponential backoff (3 attempts: 1s, 2s, 4s delays)
  - Validates embedding dimensions after generation (must be 768)
  - Throws `OllamaApiException` on failures for proper error handling
- `CodeService.java`: Handles CSV parsing, CRUD operations, and batch processing
  - Uses Apache Commons CSV for robust parsing (handles quotes, commas, newlines)
  - Batched transactions with `@Transactional(propagation = REQUIRES_NEW)` for partial success
  - CSV row limit enforced (max 1000 rows, graceful termination if exceeded)
  - IOException handling for corrupted/invalid CSV files
  - Code uniqueness validation in updateCode() method

**Controller Layer** (`controller/CodeController.java`):
- REST endpoints at `/api/v1/codes` (API versioning)
- Coordinates between services and repository
- Manual mapping from `CodeSimilarity` projection to `SearchResult` DTO
- Bean validation with `@Valid`, `@NotBlank`, `@Min`, `@Max` annotations
- Full CRUD operations: create, read, update, delete, search, list, upload CSV

### Vector Search Implementation

**Critical details:**
- pgvector cosine distance operator: `<=>` (0 = identical, 2 = opposite)
- Similarity score: `1 - distance` (converts to 0-1 scale, higher = more similar)
- Query embedding must be cast: `cast(:queryEmbedding as vector)`
- Embedding dimension is hardcoded to 768 (nomic-embed-text model)

**Repository query:**
```sql
SELECT id, code, description, embedding,
       (1 - (embedding <=> cast(:queryEmbedding as vector))) as similarity
FROM industry_codes
WHERE embedding IS NOT NULL
ORDER BY embedding <=> cast(:queryEmbedding as vector)
LIMIT :limit
```

## Configuration

### Application Profiles

**Default** (`application.properties`):
- Uses container hostnames: `postgres:5432`, `ollama:11434`
- For Docker Compose deployments

**Local** (`application-local.properties`):
- Uses localhost: `localhost:5432`, `http://localhost:11434`
- For running Spring Boot outside Docker

**To switch profiles:**
```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=local
```

### Important Properties

```properties
# Batch processing (CSV uploads)
spring.jpa.properties.hibernate.jdbc.batch_size=50
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true

# Schema management
spring.jpa.hibernate.ddl-auto=update  # Auto-creates tables

# Ollama Configuration (injectable via @Value)
ollama.api.url=${OLLAMA_API_URL:http://ollama:11434/api/embeddings}
ollama.model.name=${OLLAMA_MODEL_NAME:nomic-embed-text}

# CORS Configuration (externalized for deployment flexibility)
cors.allowed.origins=${CORS_ALLOWED_ORIGINS:http://localhost:3000,http://localhost:8080}

# Actuator endpoints (health monitoring)
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=when-authorized

# File upload limits
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
```

**Configuration Annotations Required:**
- `@EnableJpaAuditing` - Enables automatic `@CreatedDate` and `@LastModifiedDate` timestamps
- `@EnableRetry` - Enables `@Retryable` annotation support for fault tolerance

### Ollama Integration

**Model:** `nomic-embed-text` (768 dimensions)
**API endpoint:** `/api/embeddings`

**Request format:**
```json
{
  "model": "nomic-embed-text",
  "prompt": "Type 2 diabetes mellitus without complications"
}
```

**Response format:**
```json
{
  "embedding": [0.8643, 0.5875, ..., -0.1234]  // 768 floats
}
```

**If changing models:**
1. Update model name in `OllamaService.java:20`
2. Update vector dimension in `IndustryCode.java:29` (`@Array(length = ...)`)
3. Update dimension constant in `VectorConstants.EMBEDDING_DIMENSION`
4. Drop and recreate database table (dimension mismatch will cause errors)

**Health Monitoring:**
- Custom `OllamaHealthIndicator` checks Ollama service availability
- Accessible at `/actuator/health` endpoint
- Returns UP/DOWN status with service details
- Pings Ollama's `/api/tags` endpoint to verify reachability

## Common Development Tasks

### Adding a New Endpoint

1. Add method to `CodeController.java`
2. Inject required services via constructor
3. Use `repository.findSimilarCodes()` for vector search
4. Create DTO in `dto/` package if needed

### Modifying Vector Search

- Query is in `CodeRepository.java:15-23` (native SQL)
- pgvector operators: `<=>` (cosine), `<->` (L2), `<#>` (inner product)
- To change similarity metric, update operator and similarity calculation

### CSV Upload Format

Expected format (header required):
```csv
code,description
E11.9,Type 2 diabetes mellitus without complications
J44.0,"Chronic obstructive pulmonary disease with acute lower respiratory infection"
```

Parser: `CodeService.uploadCodesFromCsv()` (lines 135-249)
- Uses Apache Commons CSV for RFC 4180 compliant parsing
- Handles quoted fields, embedded commas, and newlines correctly
- Skips empty lines and malformed rows (logs warnings)
- Processes in batches of 100 rows (configurable via `VectorConstants.CSV_BATCH_SIZE`)
- Each batch saved in separate transaction (`REQUIRES_NEW`) for partial success
- Max 1000 rows per file (configurable via `VectorConstants.MAX_CSV_ROWS`)
- Gracefully stops at row limit with warning (doesn't fail entire upload)
- IOException handling for corrupted/invalid files (returns 400 Bad Request)
- Rate limiting: 100ms delay between batches to avoid overwhelming Ollama

### Testing Vector Search

```bash
# Upload sample data (CSV)
curl -X POST http://localhost:8080/api/v1/codes/upload-csv \
  -F "file=@data/sample_codes.csv"

# Create single code
curl -X POST http://localhost:8080/api/v1/codes \
  -H "Content-Type: application/json" \
  -d '{"code":"E11.9","description":"Type 2 diabetes mellitus without complications"}'

# Search (URL-encode query)
curl "http://localhost:8080/api/v1/codes/search?query=high%20blood%20sugar&limit=5"

# Get all codes (paginated)
curl "http://localhost:8080/api/v1/codes?page=0&size=20"

# Update code
curl -X PUT http://localhost:8080/api/v1/codes/1 \
  -H "Content-Type: application/json" \
  -d '{"code":"E11.9","description":"Type 2 diabetes mellitus"}'

# Check health
curl http://localhost:8080/actuator/health
```

## Troubleshooting

### "Connection refused" to Ollama
- Verify Ollama is running: `curl http://localhost:11434/api/tags`
- Check correct profile is active (`local` vs default)
- Docker: `docker-compose logs ollama`

### "Model not found" error
- Pull model: `docker exec semantic-search-ollama ollama pull nomic-embed-text`
- Or: `ollama pull nomic-embed-text` (if running locally)

### Vector dimension mismatch
- Error: "expected 768 dimensions, got X"
- Solution: Drop table and let Hibernate recreate it
- Root cause: Changed model without updating `@Array(length = 768)`

### Slow CSV uploads
- Normal behavior: Each row requires an Ollama API call (~1-2 sec each)
- 100 codes = ~2-3 minutes
- Progress logged to console: "Processed: E11.9 (5 total)"

### Tables not created
- Check `spring.jpa.hibernate.ddl-auto=update` is set
- Check database connection (credentials in application.properties)
- View SQL: `spring.jpa.show-sql=true`

### Health check shows Ollama as DOWN
- Check Actuator endpoint: `curl http://localhost:8080/actuator/health`
- Verify Ollama is running: `curl http://localhost:11434/api/tags`
- Check `ollama.api.url` property matches your Ollama instance
- Review logs for connection errors: `docker-compose logs ollama`

### Retry exhausted errors (Ollama)
- Indicates Ollama failed after 3 retry attempts (1s, 2s, 4s delays)
- Check Ollama service health and logs
- Verify model is pulled: `docker exec semantic-search-ollama ollama list`
- Check network connectivity between Spring Boot and Ollama
- Adjust retry settings in `OllamaService.java` if needed

### CSV upload returns 400 Bad Request
- **"Unable to read CSV file"**: File is corrupted or not valid UTF-8
- **"File must be a CSV"**: Filename doesn't end with .csv (case-insensitive)
- **"File is empty"**: Uploaded file has no content
- Check CSV format: header row required with "code,description" columns
- Verify file encoding is UTF-8
- Max file size: 10MB (configurable in application.properties)

### Duplicate code errors
- Error: "Code 'XXX' already exists with ID N"
- The `code` field has a unique constraint at database level
- Either update the existing code or use a different code value
- Check existing codes: `curl http://localhost:8080/api/v1/codes/{code}`

### CSV uploads process only 1000 rows
- This is expected behavior (max enforced via `VectorConstants.MAX_CSV_ROWS`)
- First 1000 rows are processed successfully
- Warning logged: "CSV file contains more than 1000 rows..."
- Successfully processed rows are saved (not rolled back)
- Split large files into multiple CSVs if needed

---
> Source: [JeremyDahms/semantic-search](https://github.com/JeremyDahms/semantic-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
