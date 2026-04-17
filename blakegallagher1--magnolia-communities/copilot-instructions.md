## magnolia-communities

> - **Purpose**: AI-assisted CRM + Deal Flow + Financial Screening + DD Suite for mobile home park acquisition

# GallagherMHP Command Platform - Cursor Rules

## Project Context
- **Purpose**: AI-assisted CRM + Deal Flow + Financial Screening + DD Suite for mobile home park acquisition
- **Target**: 20+ unit parks in East Baton Rouge and Louisiana
- **Data**: Live Socrata SODA v2 + ArcGIS REST integrations
- **Tech Stack**: Python/FastAPI backend, Next.js/React frontend, Postgres+PostGIS, Redis

## Code Style
- **Python**: Black formatting, type hints, comprehensive docstrings
- **TypeScript**: Strict mode, explicit types, functional components
- **Naming**: snake_case (Python), camelCase (TypeScript/JavaScript)
- **Imports**: Absolute imports preferred, organize by: stdlib → third-party → local

## Architecture Principles
1. **Always-fresh data**: Auto-check remote timestamps, re-ingest if stale
2. **Schema drift handling**: Detect column changes, generate migrations, emit alerts
3. **Observability**: Structured JSON logging, Prometheus metrics, health checks
4. **Security**: KMS secrets, least privilege, PII minimization, signed URLs
5. **Testing**: Unit tests for calculations, integration tests for services

## Financial Calculations (Must Be Exact)
- NOI = EGI - OpEx (where EGI = Gross - Vacancy)
- DSCR = NOI / Annual Debt Service
- Debt Yield = NOI / Loan Amount
- Cap Rate = NOI / Property Value
- IRR = Use numpy_financial.irr() with proper cash flow array

## External APIs
- **Socrata**: Use SODA v2 with $select, $where, $limit, $offset
- **ArcGIS**: Query with where, outFields, returnGeometry, f=json
- **Caching**: Redis with configurable TTL, cache keys include params
- **Retry**: Exponential backoff, max 5 retries, log failures

## Database
- **ORM**: SQLAlchemy async with declarative base
- **Geometry**: GeoAlchemy2 for PostGIS, always EPSG:4326
- **Migrations**: Alembic with auto-generate, review before applying
- **Indexes**: Add GIST indexes for geometry columns

## API Design
- **REST**: Semantic routes, consistent response shapes
- **Validation**: Pydantic models for request/response
- **Errors**: Use HTTPException with appropriate status codes
- **Pagination**: limit + offset, default limit=50, max=500

## Frontend
- **Components**: Functional, typed props, use hooks
- **State**: Zustand for global, React Query for server state
- **Forms**: react-hook-form with zod validation
- **Styling**: Tailwind utility classes, consistent color scheme

## Testing
- **Backend**: pytest with async fixtures, >80% coverage target
- **Financial**: Verify all formulas with known inputs/outputs
- **Integration**: Test database with rollback, mock external APIs
- **Frontend**: Component tests with testing-library

## Documentation
- **Code**: Docstrings with Args/Returns/Raises
- **API**: OpenAPI auto-generated from Pydantic models
- **README**: Keep updated with architecture changes
- **Comments**: Explain why, not what (code should be self-documenting)

## Performance
- **Queries**: Use select/join efficiently, avoid N+1
- **Caching**: Cache expensive spatial queries
- **Pagination**: Always paginate list endpoints
- **Indexes**: Add indexes for frequently filtered columns

## Security Checklist
- [ ] No secrets in code/logs
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention (use parameterized queries)
- [ ] Rate limiting on external API connectors
- [ ] CORS configured for production domains only
- [ ] Document access via signed URLs with expiration

## Commit Messages
- Format: `type(scope): description`
- Types: feat, fix, docs, refactor, test, chore
- Example: `feat(financial): add IRR calculation with stress scenarios`

## When Adding Features
1. Update models if schema changes
2. Create/update service with business logic
3. Add API endpoint with Pydantic validation
4. Write tests (unit + integration)
5. Update README/docs if user-facing
6. Consider observability (metrics, logs)

## Always Check Before Merging
- [ ] Tests pass
- [ ] Linter/formatter run
- [ ] Type checker passes
- [ ] API docs regenerated if endpoints changed
- [ ] Migration generated if schema changed
- [ ] README updated if user-facing changes
- [ ] No PII in logs
- [ ] Secrets encrypted/not committed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blakegallagher1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
