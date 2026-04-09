## price-rules-ai-drools-service

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Building
```bash
# Clean and build
mvn clean install

# Skip tests
mvn clean install -DskipTests

# Run application locally
mvn spring-boot:run

# Run with specific profile
mvn spring-boot:run -Dspring.profiles.active=dev
```

### Testing
```bash
# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=TestClassName

# Run specific test method
mvn test -Dtest=TestClassName#testMethodName

# Run tests with coverage report (JaCoCo)
mvn clean verify

# View coverage report
open target/site/jacoco/index.html
```

### Docker
```bash
# Build Docker image
docker build -t price-dools-service -f docker/Dockerfile .

# Run Docker container
docker run -p 8080:8080 price-dools-service

# Run with environment variables
docker run -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e JDBC_DATABASE_URL=jdbc:postgresql://host:5432/dbname \
  -e JDBC_DATABASE_USERNAME=username \
  -e JDBC_DATABASE_PASSWORD=password \
  -e REDIS_HOST=redis-host \
  -e REDIS_PORT=6379 \
  -e REDIS_PASSWORD=redispassword \
  -e JWT_SECRET=yoursecret \
  price-dools-service
```

## Architecture Overview

The Price Rules AI Drools Service is a microservice that integrates the Drools rule engine with Spring Boot to provide dynamic pricing rule evaluation. It implements an AI-enhanced approach to optimize pricing rules before execution.

### Key Components

1. **Rule Engine Integration**
   - `DroolsIntegrationService`: Core interface for rule deployment, validation, and execution
   - Uses KIE (Knowledge Is Everything) APIs from Drools
   - Implements caching for rule execution performance

2. **Financial Metrics Processing**
   - `FinancialMetricsService`: Calculates and manages key financial metrics (ARR, TCV, ACV, CLV)
   - `PricingService`: Applies pricing strategies based on financial metrics

3. **AI-Enhanced Services**
   - `SequentialThinkingService`: Provides AI-driven analysis and decision making
   - `Context7Service`: Integrates with Context7 for documentation enhancement
   - `RuleCreationService`: AI-assisted rule generation and optimization

4. **Data Model**
   - `FinancialMetrics`: Core entity for financial calculations
   - `PricingRequest` / `PricingResult`: Data transfer objects for pricing operations

5. **Security Layer**
   - JWT-based authentication
   - Role-based access control

### Technology Stack

- Java 21
- Spring Boot 3.4.5
- Drools 8.47.0.Final
- Spring Cloud 2024.0.0
- Spring AI 0.1.0
- PostgreSQL (primary data store)
- Redis (caching)
- Liquibase (database migrations)

## Database Structure

The application uses PostgreSQL with Liquibase for schema management:

- **users**: Authentication and authorization
- **roles**: User role definitions
- **rule_sets**: Collections of related rules
- **rules**: Individual pricing rules
- **financial_metrics**: Financial data for customers
- **rule_execution_history**: Audit trail for rule executions

## API Structure

1. **Rule Management API** (`/api/rules`)
   - Create, read, update, delete rule sets and rules
   - Validate rules before deployment
   - Rule conflict detection and resolution

2. **Pricing Evaluation API** (`/api/pricing`)
   - Single and batch evaluation endpoints
   - Financial metrics calculation (ARR, TCV, ACV, CLV)

3. **AI-Enhanced Rule Creation** (`/api/ai/rules`)
   - AI-assisted rule generation and optimization
   - Sequential thinking for rule analysis

4. **Authentication API** (`/api/auth`)
   - JWT-based login/signup endpoints
   - User management

5. **Monitoring API** (`/api/actuator`)
   - Rule execution metrics
   - Performance statistics
   - Health checks and Prometheus metrics

## Configuration

The application uses Spring Boot's configuration system with profiles:

- **dev**: Development environment (default)
- **test**: Testing environment with H2 in-memory database
- **prod**: Production environment with environment variable configuration

Key configuration files:
- `application.yml`: Base configuration
- Profile-specific sections within application.yml

## Coding Conventions

1. **Service Layer**
   - Use interfaces for all services
   - Implement @Transactional on service methods as needed
   - Use constructor injection with @RequiredArgsConstructor

2. **Rule Files**
   - DRL files in src/main/resources/rules/
   - Follow Drools syntax and conventions
   - Include proper logging and rule documentation

3. **Controllers**
   - RESTful API design principles
   - Proper exception handling
   - Request validation

4. **Testing**
   - Unit tests for business logic
   - Integration tests for rule execution
   - Use TestContainers for database tests

## Task Master AI Instructions
**Import Task Master's development workflow commands and guidelines, treat as if import is in the main CLAUDE.md file.**
@./.taskmaster/CLAUDE.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drgaciw)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/drgaciw)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
