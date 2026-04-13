## db-tinyos

> 1. **Modular Architecture**


## Project Management Rules for DB_tinyOS

### Code Structure and Organization

1. **Modular Architecture**
   - Maintain clear separation between the web API, data processing scripts, and web interface
   - Keep related functionality grouped in appropriate directories
   - Create dedicated modules for shared utilities

2. **Configuration Management**
   - Store all configuration in environment variables via `.env` files
   - Never hardcode credentials in source code
   - Use a consistent pattern for accessing configuration

### Logging and Observability

3. **Structured Logging Implementation**
   - Implement JSON-formatted structured logging for all services
   - Include timestamps, severity levels, service names, and correlation IDs in all logs
   - Add contextual information to logs (user IDs, request details, etc.)
   - Set up centralized log collection for easier monitoring and troubleshooting

4. **Error Handling**
   - Implement comprehensive exception handling with appropriate logging
   - Create standardized error responses for the API
   - Log stack traces for debugging but return sanitized errors to users

### Database Interaction

5. **Database Access Patterns**
   - Use connection pooling for database access
   - Ensure all database connections are properly closed
   - Implement timeout handling for database operations
   - Use parameterized queries to prevent SQL injection

6. **Data Migration and Schema Management**
   - Document all schema changes in version-controlled migration files
   - Test migrations in development environments before deployment
   - Implement rollback procedures for failed migrations

### Security

7. **Authentication and Authorization**
   - Implement proper authentication for API endpoints
   - Use HTTPS for all communications
   - Validate and sanitize all user inputs
   - Implement proper access controls based on user roles

### Testing and Quality Assurance

8. **Automated Testing**
   - Write unit tests for critical functions and API endpoints
   - Implement integration tests for database operations
   - Set up end-to-end tests for critical user journeys
   - Ensure all tests run before merging code changes

### Deployment and Operations

9. **Deployment Procedures**
   - Document deployment steps in version-controlled documentation
   - Implement automated deployment pipelines where possible
   - Use environment-specific configuration for development, staging, and production
   - Include database backup procedures before deployments

10. **Monitoring and Alerting**
    - Set up monitoring for API availability and performance
    - Create alerts for critical errors and performance degradation
    - Implement health check endpoints for all services
    - Monitor database performance and resource utilization

### Documentation

11. **Code Documentation**
    - Document all functions with docstrings explaining purpose, parameters, and return values
    - Keep README files updated with setup and usage instructions
    - Document API endpoints with example requests and responses
    - Create and maintain technical architecture documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mstrar76) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
