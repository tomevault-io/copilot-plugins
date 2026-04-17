## dubai-listings-api

> Global Rules for Production-Ready Backend Development


Global Rules for Production-Ready Backend Development

1. Code Quality & Standards
   Type Safety: Use TypeScript for all new projects
   Linting: Enforce strict ESLint rules with TypeScript support
   Code Style: Follow Airbnb JavaScript/TypeScript style guide
   Immutability: Prefer const over let, avoid var
   Pure Functions: Favor pure functions with clear input/output
   Error Handling: Implement comprehensive error handling at all layers
2. Security
   Dependencies: Regularly audit and update dependencies
   Secrets Management: Never hardcode secrets; use environment variables with validation
   Input Validation: Validate all incoming requests using Joi or class-validator
   Authentication: Implement JWT with refresh tokens
   Authorization: Use role-based access control (RBAC)
   CORS: Configure CORS to allow only trusted origins
   Security Headers: Implement security headers using Helmet
   Rate Limiting: Implement API rate limiting
3. Performance
   Database Indexing: Create indexes for frequently queried fields
   Query Optimization: Use select to fetch only required fields
   Pagination: Implement cursor-based pagination for large datasets
   Caching: Use Redis for caching frequent queries
   Connection Pooling: Configure database connection pooling
   Compression: Enable response compression
   Monitoring: Implement performance monitoring
4. Testing
   Test Coverage: Maintain ≥80% test coverage
   Test Types:
   Unit tests for business logic
   Integration tests for API endpoints
   E2E tests for critical user journeys
   Test Data: Use factories for test data generation
   Mocks: Mock external services and APIs
   CI/CD: Run tests on every push/PR
5. Logging & Monitoring
   Structured Logging: Use JSON format for logs
   Log Levels: Use appropriate log levels (error, warn, info, debug)
   Request ID: Include request IDs for tracing
   Error Tracking: Integrate with error tracking service (Sentry, etc.)
   Metrics: Collect and expose application metrics
   Health Checks: Implement readiness/liveness endpoints
6. API Design
   RESTful Principles: Follow REST conventions
   Versioning: Version all APIs (URL path or header)
   Documentation: Maintain OpenAPI/Swagger documentation
   Response Format: Standardize response structure
   Error Responses: Standardize error response format
   Pagination: Consistent pagination implementation
7. Database
   Migrations: Use database migration tools
   Indexes: Document all indexes and their purposes
   Transactions: Use transactions for multi-document operations
   Backups: Regular automated backups
   Schema Validation: Enforce schema validation at the database level
8. Containerization
   Multi-stage Builds: Use multi-stage Docker builds
   Minimal Images: Use minimal base images (e.g., Alpine)
   Non-root User: Run containers as non-root user
   Resource Limits: Set CPU/memory limits
   Health Checks: Implement container health checks
   Immutable Images: Don't modify containers at runtime
9. CI/CD
   Automated Testing: Run tests on every commit
   Automated Builds: Build Docker images on merge to main
   Automated Deployments: Deploy to staging on merge to main
   Manual Production Deployments: Require manual approval for production
   Rollback Plan: Have a rollback strategy
   Environment Parity: Keep environments as similar as possible
10. Documentation
    API Documentation: Keep OpenAPI/Swagger docs updated
    Architecture: Document system architecture
    Setup: Document local development setup
    Deployment: Document deployment procedures
    Troubleshooting: Document common issues and solutions
    Decision Records: Maintain ADRs (Architecture Decision Records)
11. Error Handling
    Custom Error Classes: Create specific error types
    Global Error Handler: Implement global error handling middleware
    Error Logging: Log errors with sufficient context
    User-friendly Messages: Return user-friendly error messages
    Error Codes: Use standard HTTP status codes
12. Dependencies
    Minimal Dependencies: Keep dependencies to a minimum
    Regular Updates: Regularly update dependencies
    Security Audits: Run security audits
    Peer Dependencies: Specify peer dependencies
    Lock Files: Use package-lock.json or yarn.lock
13. Environment Configuration
    Environment Variables: Use environment variables for configuration
    Validation: Validate environment variables at startup
    Environment-specific: Maintain separate configs for dev/staging/prod
    Secrets: Use secret management for sensitive data
14. Performance Optimization
    Profiling: Regularly profile application performance
    Bottlenecks: Identify and address performance bottlenecks
    Caching: Implement appropriate caching strategies
    Database Optimization: Optimize database queries and indexes
    Memory Management: Monitor and optimize memory usage
15. Scalability
    Stateless: Design to be stateless where possible
    Horizontal Scaling: Support horizontal scaling
    Background Jobs: Use queues for long-running tasks
    Database Sharding: Plan for database sharding if needed
16. Monitoring & Alerting
    Metrics Collection: Collect application metrics
    Log Aggregation: Centralized logging
    Alerting: Set up alerts for critical issues
    Dashboards: Create operational dashboards
    Incident Response: Document incident response procedures
17. Code Review
    PR Templates: Use pull request templates
    Code Owners: Define code owners
    Review Guidelines: Establish code review guidelines
    Automated Checks: Require passing tests and linting
    Small PRs: Encourage small, focused pull requests
18. Documentation as Code
    API First: Design API first, implement second
    Contract Testing: Test against API contracts
    Documentation Generation: Auto-generate documentation
    Keep Docs Updated: Documentation should evolve with code
19. Security Compliance
    OWASP Top 10: Protect against OWASP Top 10 vulnerabilities
    Regular Audits: Conduct security audits
    Dependency Scanning: Scan for vulnerable dependencies
    Secrets Scanning: Prevent secrets in code
20. Team Practices
    Pair Programming: Encourage pair programming
    Knowledge Sharing: Regular knowledge sharing sessions
    Mentorship: Establish mentorship for junior developers
    Retrospectives: Regular team retrospectives
    Continuous Learning

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Fathi123-max) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
