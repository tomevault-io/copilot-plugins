## engine-rules-sp

> Golang Microservice Implementation Rule


# Golang Microservice Implementation Rule

## Overview

This rule provides comprehensive guidance for implementing enterprise-grade Golang microservices using hexagonal architecture, DDD principles, and complete testing strategies. Based on successful Rules Engine microservice implementations achieving production-ready quality.

## 1. Feedback Section

### Identified Issues in Generic Microservice Development:
- **Unclear Architecture**: Mixing business logic with infrastructure concerns
- **Incomplete Testing**: Missing behavioral tests and integration coverage
- **Poor API Design**: Inconsistent REST/gRPC interface definitions
- **Missing Observability**: Inadequate monitoring and tracing
- **Weak Error Handling**: Generic error responses without context
- **No Event Integration**: Missing domain event publishing/consuming

### Recommendations:
- Use hexagonal architecture for clear separation of concerns
- Implement comprehensive test pyramid with behavioral testing
- Define precise API contracts with OpenAPI and Protobuf
- Add distributed tracing and structured logging
- Create domain-specific error types with proper HTTP status mapping
- Integrate NATS for event-driven communication

## 2. Role and Context Definition

### Target Role: Backend Developer (Golang)
### Background Context:
- **Domain**: Business rules engine with high-performance requirements
- **Architecture**: Microservices with DDD bounded contexts
- **Technology Stack**: Go 1.21+, Gin/Echo, GORM, NATS, PostgreSQL, Redis
- **Performance**: <500ms response time, 1000+ TPS throughput
- **Deployment**: Kubernetes with Docker containers

## 3. Objective and Goals

### Primary Objective:
Create production-ready Golang microservice implementing specific business domain with hexagonal architecture, comprehensive testing, and enterprise-grade observability.

### Success Criteria:
- **Architecture**: Clean hexagonal architecture with clear boundaries
- **Testing**: >85% coverage with unit, integration, and behavioral tests
- **Performance**: Meet SLA requirements (response time, throughput)
- **APIs**: Complete OpenAPI 3.0 and Protobuf specifications
- **Observability**: Distributed tracing, structured logging, metrics
- **Security**: Input validation, authentication, authorization

## 4. Key Terms and Definitions

### Technical Terminology:
- **Hexagonal Architecture**: Ports and adapters pattern isolating business logic
- **Domain Entity**: Core business object with identity and behavior
- **Value Object**: Immutable object defined by attributes (Money, Email)
- **Repository**: Data access abstraction hiding persistence details
- **Use Case**: Application service orchestrating business operations
- **Domain Event**: Something meaningful that happened in the domain
- **Anti-Corruption Layer**: Translation layer for external system integration

## 5. Task Decomposition (Chain-of-Thought)

### Step 1: Project Structure Setup
- **Input**: Service name, domain requirements, API specifications
- **Process**: Create hexagonal architecture project structure with proper Go modules
- **Output**: Complete project skeleton with dependency injection setup
- **Human Validation Point**: Review project structure follows hexagonal architecture principles

### Step 2: Domain Layer Implementation
- **Input**: Domain requirements, entity definitions, business rules
- **Process**: Implement entities, value objects, domain services, and specifications
- **Output**: Pure domain layer with comprehensive unit tests
- **Human Validation Point**: Verify domain logic correctly implements business requirements

### Step 3: Application Layer Implementation
- **Input**: Use cases, domain layer, external service requirements
- **Process**: Create application services, command/query handlers, DTOs
- **Output**: Application services with input validation and orchestration
- **Human Validation Point**: Confirm use cases properly orchestrate domain operations

### Step 4: Infrastructure Layer Implementation
- **Input**: Database schemas, external API specifications, messaging requirements
- **Process**: Implement repositories, external adapters, event publishers/consumers
- **Output**: Infrastructure adapters with integration tests
- **Human Validation Point**: Validate infrastructure correctly implements port contracts

### Step 5: Interface Layer Implementation
- **Input**: API specifications (OpenAPI, Protobuf), authentication requirements
- **Process**: Create REST/gRPC controllers, middleware, request/response mapping
- **Output**: Complete API implementation with documentation
- **Human Validation Point**: Verify APIs match specifications and handle edge cases

### Step 6: Testing Implementation
- **Input**: All implemented layers, business scenarios, error conditions
- **Process**: Create comprehensive test suite following test pyramid
- **Output**: Complete test coverage with behavioral, integration, and unit tests
- **Human Validation Point**: Confirm tests cover all business scenarios and edge cases

### Step 7: Observability and Deployment
- **Input**: Service implementation, monitoring requirements, deployment specs
- **Process**: Add tracing, logging, metrics, health checks, Docker configuration
- **Output**: Production-ready service with observability and deployment artifacts
- **Human Validation Point**: Validate monitoring captures all critical service metrics

## 6. Context and Constraints

### Technical Context:
- **Go Version**: 1.21+ with generics support
- **Frameworks**: Gin/Echo (REST), gRPC-Go, GORM (ORM)
- **Database**: PostgreSQL with migrations, connection pooling
- **Messaging**: NATS JetStream for event-driven communication
- **Observability**: OpenTelemetry, Prometheus, structured logging

### Business Context:
- **Performance Requirements**: <500ms response time, 1000+ TPS throughput
- **Availability**: 99.9% uptime with graceful degradation
- **Scalability**: Horizontal scaling with stateless design
- **Security**: Authentication, authorization, input validation

### Negative Constraints:
- **Do NOT** mix business logic with infrastructure code
- **Do NOT** use global variables or singletons
- **Do NOT** skip integration tests for external dependencies
- **Do NOT** hardcode configuration values
- **Do NOT** ignore error handling and edge cases

## 7. Examples and Illustrations (Few-Shot)

### Example 1: Rules Management Service Implementation

#### Project Structure:
```
rules-management-service/
├── cmd/
│   └── main.go                 # Application entry point
├── internal/
│   ├── domain/                 # Domain layer (entities, value objects, interfaces)
│   │   ├── rule/
│   │   │   ├── rule.go
│   │   │   └── repository.go
│   │   └── shared/
│   │       └── errors.go
│   ├── application/            # Application layer (use cases, commands, queries)
│   │   ├── commands/
│   │   │   └── create_rule.go
│   │   └── queries/
│   │       └── get_rule.go
│   ├── infrastructure/         # Infrastructure layer (adapters, implementations)
│   │   ├── persistence/
│   │   │   └── postgres/
│   │   │       └── rule_repository.go
│   │   └── messaging/
│   │       └── nats_publisher.go
│   └── interfaces/             # Interface layer (controllers, DTOs)
│       └── rest/
│           ├── handlers/
│           │   └── rule_handler.go
│           └── dto/
│               └── dto.go
├── api/
│   ├── openapi.yaml
│   └── proto/
│       └── rule_service.proto
├── tests/
│   ├── unit/
│   ├── integration/
│   └── behavioral/
│       └── rule_management.feature
├── deployments/
│   ├── Dockerfile
│   └── k8s/
└── go.mod
```

#### Domain Entity Example:
```go
// internal/domain/rule/rule.go
package rule

import (
    "time"
    "github.com/google/uuid"
    "rules-engine/internal/domain/shared"
)

// Rule represents a business rule aggregate
type Rule struct {
    id          RuleID
    name        string
    description string
    dslContent  string
    status      Status
    priority    Priority
    version     int
    createdAt   time.Time
    updatedAt   time.Time
    events      []shared.DomainEvent
}

// RuleID is a typed identifier for rules
type RuleID struct {
    value uuid.UUID
}

func NewRuleID() RuleID {
    return RuleID{value: uuid.New()}
}

func (id RuleID) String() string {
    return id.value.String()
}

// Status represents rule lifecycle status
type Status int

const (
    StatusDraft Status = iota
    StatusUnderReview
    StatusApproved
    StatusActive
    StatusInactive
    StatusDeprecated
)

// Priority represents rule execution priority
type Priority int

const (
    PriorityLow Priority = iota
    PriorityMedium
    PriorityHigh
    PriorityCritical
)

// NewRule creates a new rule with validation
func NewRule(name, description, dslContent string, priority Priority) (*Rule, error) {
    if err := validateRuleName(name); err != nil {
        return nil, shared.NewDomainError("invalid rule name", err)
    }
    
    if err := validateDSLContent(dslContent); err != nil {
        return nil, shared.NewDomainError("invalid DSL content", err)
    }
    
    rule := &Rule{
        id:          NewRuleID(),
        name:        name,
        description: description,
        dslContent:  dslContent,
        status:      StatusDraft,
        priority:    priority,
        version:     1,
        createdAt:   time.Now(),
        updatedAt:   time.Now(),
        events:      make([]shared.DomainEvent, 0),
    }
    
    // Raise domain event
    rule.addEvent(NewRuleCreatedEvent(rule.id, rule.name, rule.priority))
    
    return rule, nil
}

// SubmitForApproval changes rule status to under review
func (r *Rule) SubmitForApproval() error {
    if r.status != StatusDraft {
        return shared.NewDomainError("cannot submit rule for approval", 
            fmt.Errorf("rule status is %v, expected %v", r.status, StatusDraft))
    }
    
    r.status = StatusUnderReview
    r.updatedAt = time.Now()
    r.addEvent(NewRuleSubmittedForApprovalEvent(r.id, r.name))
    
    return nil
}

// Approve marks rule as approved
func (r *Rule) Approve(approvedBy string) error {
    if r.status != StatusUnderReview {
        return shared.NewDomainError("cannot approve rule", 
            fmt.Errorf("rule status is %v, expected %v", r.status, StatusUnderReview))
    }
    
    r.status = StatusApproved
    r.updatedAt = time.Now()
    r.addEvent(NewRuleApprovedEvent(r.id, r.name, approvedBy))
    
    return nil
}

// GetEvents returns domain events and clears them
func (r *Rule) GetEvents() []shared.DomainEvent {
    events := r.events
    r.events = make([]shared.DomainEvent, 0)
    return events
}

func (r *Rule) addEvent(event shared.DomainEvent) {
    r.events = append(r.events, event)
}

// Private validation functions
func validateRuleName(name string) error {
    if strings.TrimSpace(name) == "" {
        return errors.New("rule name cannot be empty")
    }
    if len(name) > 100 {
        return errors.New("rule name cannot exceed 100 characters")
    }
    return nil
}

func validateDSLContent(content string) error {
    if strings.TrimSpace(content) == "" {
        return errors.New("DSL content cannot be empty")
    }
    // Add DSL syntax validation here
    return nil
}
```

#### Repository Interface:
```go
// internal/domain/rule/repository.go
package rule

import (
    "context"
)

// Repository defines the contract for rule persistence
type Repository interface {
    Save(ctx context.Context, rule *Rule) error
    FindByID(ctx context.Context, id RuleID) (*Rule, error)
    FindByName(ctx context.Context, name string) (*Rule, error)
    List(ctx context.Context, criteria ListCriteria) ([]*Rule, error)
    Delete(ctx context.Context, id RuleID) error
    ExistsByName(ctx context.Context, name string) (bool, error)
}

// ListCriteria defines filtering and pagination options
type ListCriteria struct {
    Status     *Status
    Priority   *Priority
    CreatedBy  string
    PageSize   int
    PageOffset int
    SortBy     string
    SortOrder  string
}
```

#### Application Service Example:
```go
// internal/application/commands/create_rule.go
package commands

import (
    "context"
    "rules-engine/internal/domain/rule"
    "rules-engine/internal/domain/shared"
)

// CreateRuleCommand represents the command to create a new rule
type CreateRuleCommand struct {
    Name        string `json:"name" validate:"required,min=3,max=100"`
    Description string `json:"description" validate:"max=500"`
    DSLContent  string `json:"dslContent" validate:"required"`
    Priority    string `json:"priority" validate:"required,oneof=LOW MEDIUM HIGH CRITICAL"`
    CreatedBy   string `json:"createdBy" validate:"required"`
}

// CreateRuleResult represents the result of creating a rule
type CreateRuleResult struct {
    RuleID   string `json:"ruleId"`
    Name     string `json:"name"`
    Status   string `json:"status"`
    Version  int    `json:"version"`
}

// CreateRuleHandler handles rule creation commands
type CreateRuleHandler struct {
    ruleRepo     rule.Repository
    eventBus     shared.EventBus
    validator    shared.Validator
}

func NewCreateRuleHandler(
    ruleRepo rule.Repository, 
    eventBus shared.EventBus,
    validator shared.Validator,
) *CreateRuleHandler {
    return &CreateRuleHandler{
        ruleRepo:  ruleRepo,
        eventBus:  eventBus,
        validator: validator,
    }
}

// Handle processes the create rule command
func (h *CreateRuleHandler) Handle(ctx context.Context, cmd CreateRuleCommand) (*CreateRuleResult, error) {
    // Validate command input
    if err := h.validator.Validate(cmd); err != nil {
        return nil, shared.NewValidationError("invalid create rule command", err)
    }
    
    // Check if rule name already exists
    exists, err := h.ruleRepo.ExistsByName(ctx, cmd.Name)
    if err != nil {
        return nil, shared.NewInfrastructureError("failed to check rule existence", err)
    }
    if exists {
        return nil, shared.NewBusinessError("rule name already exists", cmd.Name)
    }
    
    // Parse priority
    priority, err := rule.ParsePriority(cmd.Priority)
    if err != nil {
        return nil, shared.NewValidationError("invalid priority", err)
    }
    
    // Create new rule
    newRule, err := rule.NewRule(cmd.Name, cmd.Description, cmd.DSLContent, priority)
    if err != nil {
        return nil, err // Domain error already wrapped
    }
    
    // Save rule
    if err := h.ruleRepo.Save(ctx, newRule); err != nil {
        return nil, shared.NewInfrastructureError("failed to save rule", err)
    }
    
    // Publish domain events
    events := newRule.GetEvents()
    for _, event := range events {
        if err := h.eventBus.Publish(ctx, event); err != nil {
            // Log error but don't fail the command
            // In production, consider implementing outbox pattern
            shared.LogError("failed to publish event", err, map[string]any{
                "eventType": event.Type(),
                "ruleID":    newRule.ID().String(),
            })
        }
    }
    
    return &CreateRuleResult{
        RuleID:  newRule.ID().String(),
        Name:    newRule.Name(),
        Status:  newRule.Status().String(),
        Version: newRule.Version(),
    }, nil
}
```

#### REST Handler Example:
```go
// internal/interfaces/rest/handlers/rule_handler.go
package handlers

import (
    "net/http"
    "github.com/gin-gonic/gin"
    "rules-engine/internal/application/commands"
    "rules-engine/internal/application/queries"
    "rules-engine/internal/interfaces/rest/dto"
    "rules-engine/internal/domain/shared"
)

// RuleHandler handles HTTP requests for rule operations
type RuleHandler struct {
    createRuleHandler *commands.CreateRuleHandler
    getRuleHandler    *queries.GetRuleHandler
    listRulesHandler  *queries.ListRulesHandler
}

func NewRuleHandler(
    createRuleHandler *commands.CreateRuleHandler,
    getRuleHandler *queries.GetRuleHandler,
    listRulesHandler *queries.ListRulesHandler,
) *RuleHandler {
    return &RuleHandler{
        createRuleHandler: createRuleHandler,
        getRuleHandler:    getRuleHandler,
        listRulesHandler:  listRulesHandler,
    }
}

// CreateRule handles POST /api/v1/rules
func (h *RuleHandler) CreateRule(c *gin.Context) {
    var req dto.CreateRuleRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, dto.ErrorResponse{
            Error:   "invalid request body",
            Message: err.Error(),
            Code:    "INVALID_REQUEST",
        })
        return
    }
    
    // Map DTO to command
    cmd := commands.CreateRuleCommand{
        Name:        req.Name,
        Description: req.Description,
        DSLContent:  req.DSLContent,
        Priority:    req.Priority,
        CreatedBy:   getUserFromContext(c),
    }
    
    // Execute command
    result, err := h.createRuleHandler.Handle(c.Request.Context(), cmd)
    if err != nil {
        handleError(c, err)
        return
    }
    
    // Map result to response DTO
    response := dto.CreateRuleResponse{
        RuleID:  result.RuleID,
        Name:    result.Name,
        Status:  result.Status,
        Version: result.Version,
    }
    
    c.JSON(http.StatusCreated, response)
}

// GetRule handles GET /api/v1/rules/:id
func (h *RuleHandler) GetRule(c *gin.Context) {
    ruleID := c.Param("id")
    if ruleID == "" {
        c.JSON(http.StatusBadRequest, dto.ErrorResponse{
            Error:   "missing rule ID",
            Code:    "MISSING_PARAMETER",
        })
        return
    }
    
    query := queries.GetRuleQuery{RuleID: ruleID}
    result, err := h.getRuleHandler.Handle(c.Request.Context(), query)
    if err != nil {
        handleError(c, err)
        return
    }
    
    response := dto.MapRuleToResponse(result)
    c.JSON(http.StatusOK, response)
}

// handleError maps domain errors to appropriate HTTP responses
func handleError(c *gin.Context, err error) {
    switch e := err.(type) {
    case *shared.ValidationError:
        c.JSON(http.StatusBadRequest, dto.ErrorResponse{
            Error:   "validation failed",
            Message: e.Error(),
            Code:    "VALIDATION_ERROR",
        })
    case *shared.BusinessError:
        c.JSON(http.StatusConflict, dto.ErrorResponse{
            Error:   "business rule violation",
            Message: e.Error(),
            Code:    "BUSINESS_ERROR",
        })
    case *shared.NotFoundError:
        c.JSON(http.StatusNotFound, dto.ErrorResponse{
            Error:   "resource not found",
            Message: e.Error(),
            Code:    "NOT_FOUND",
        })
    case *shared.InfrastructureError:
        c.JSON(http.StatusInternalServerError, dto.ErrorResponse{
            Error:   "internal server error",
            Code:    "INTERNAL_ERROR",
        })
    default:
        c.JSON(http.StatusInternalServerError, dto.ErrorResponse{
            Error:   "unexpected error",
            Code:    "UNKNOWN_ERROR",
        })
    }
}

func getUserFromContext(c *gin.Context) string {
    if user, exists := c.Get("user"); exists {
        return user.(string)
    }
    return "system"
}
```

#### Behavioral Test Example:
```gherkin
# tests/behavioral/rule_management.feature
Feature: Rule Management
  As a business user
  I want to manage business rules
  So that I can control system behavior

  Background:
    Given I am authenticated as a business user
    And the rules management service is running

  Scenario: Successfully create a new rule
    Given I have a valid rule definition
    When I submit a create rule request with:
      | name        | Customer Discount Rule |
      | description | Apply discount for VIP customers |
      | dslContent  | customer.tier == "VIP" |
      | priority    | HIGH |
    Then the rule should be created successfully
    And the rule status should be "DRAFT"
    And a "RuleCreated" event should be published

  Scenario: Prevent duplicate rule names
    Given a rule with name "Existing Rule" already exists
    When I try to create a rule with name "Existing Rule"
    Then I should receive a "BUSINESS_ERROR" response
    And the error message should contain "rule name already exists"

  Scenario: Submit rule for approval
    Given I have a rule in "DRAFT" status
    When I submit the rule for approval
    Then the rule status should change to "UNDER_REVIEW"
    And a "RuleSubmittedForApproval" event should be published

  Scenario: Validate DSL content on rule creation
    When I try to create a rule with empty DSL content
    Then I should receive a "VALIDATION_ERROR" response
    And the error message should contain "DSL content cannot be empty"
```

### Example 2: Integration Test Implementation

```go
// tests/integration/rule_repository_test.go
package integration

import (
    "context"
    "testing"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "rules-engine/internal/domain/rule"
    "rules-engine/internal/infrastructure/persistence/postgres"
    "rules-engine/tests/testutil"
)

func TestRuleRepository_Integration(t *testing.T) {
    // Setup test database
    db := testutil.SetupTestDB(t)
    defer testutil.CleanupTestDB(t, db)
    
    repo := postgres.NewRuleRepository(db)
    ctx := context.Background()

    t.Run("Save and FindByID", func(t *testing.T) {
        // Arrange
        rule, err := rule.NewRule("Test Rule", "Test Description", "test.dsl", rule.PriorityMedium)
        require.NoError(t, err)
        
        // Act
        err = repo.Save(ctx, rule)
        require.NoError(t, err)
        
        // Assert
        foundRule, err := repo.FindByID(ctx, rule.ID())
        require.NoError(t, err)
        assert.Equal(t, rule.Name(), foundRule.Name())
        assert.Equal(t, rule.Status(), foundRule.Status())
    })

    t.Run("ExistsByName", func(t *testing.T) {
        // Arrange
        rule, err := rule.NewRule("Unique Rule", "Description", "unique.dsl", rule.PriorityLow)
        require.NoError(t, err)
        err = repo.Save(ctx, rule)
        require.NoError(t, err)
        
        // Act & Assert
        exists, err := repo.ExistsByName(ctx, "Unique Rule")
        require.NoError(t, err)
        assert.True(t, exists)
        
        exists, err = repo.ExistsByName(ctx, "Non-existent Rule")
        require.NoError(t, err)
        assert.False(t, exists)
    })

    t.Run("List with criteria", func(t *testing.T) {
        // Arrange
        testRules := createTestRules(t, repo)
        
        criteria := rule.ListCriteria{
            Status:   &rule.StatusDraft,
            PageSize: 10,
        }
        
        // Act
        rules, err := repo.List(ctx, criteria)
        require.NoError(t, err)
        
        // Assert
        assert.True(t, len(rules) > 0)
        for _, r := range rules {
            assert.Equal(t, rule.StatusDraft, r.Status())
        }
    })
}

func createTestRules(t *testing.T, repo rule.Repository) []*rule.Rule {
    ctx := context.Background()
    var rules []*rule.Rule
    
    for i := 0; i < 3; i++ {
        rule, err := rule.NewRule(
            fmt.Sprintf("Test Rule %d", i),
            fmt.Sprintf("Description %d", i),
            fmt.Sprintf("test%d.dsl", i),
            rule.PriorityMedium,
        )
        require.NoError(t, err)
        
        err = repo.Save(ctx, rule)
        require.NoError(t, err)
        
        rules = append(rules, rule)
    }
    
    return rules
}
```

### Example 3: Application Wiring in `main.go`

A crucial part of a microservice is the composition root, where all dependencies are wired together.

```go
// cmd/main.go
package main

import (
    "log"
    // ... other imports
)

func main() {
    // 1. Load configuration
    cfg := config.DefaultConfig()

    // 2. Initialize database connection
    db, err := gorm.Open(postgres.Open(cfg.Database.DSN), &gorm.Config{})
    // ... error handling ...

    // 3. Initialize adapters (repositories, event bus, etc.)
    ruleRepo := persistence.NewRuleRepository(db)
    eventPublisher, _ := nats.NewEventPublisher(cfg.NATS)
    
    // 4. Initialize domain services and application handlers
    validationService := dsl.NewSimpleValidator()
    createRuleHandler := commands.NewCreateRuleHandler(ruleRepo, eventPublisher, validationService)
    // ... other handlers ...

    // 5. Initialize API handlers (e.g., REST)
    ruleHandler := handlers.NewRuleHandler(createRuleHandler, /*...*/)

    // 6. Setup and run the server (e.g., Gin)
    router := gin.Default()
    // ... setup routes ...
    router.Run(":8080")
}
```

### Example 4: Strategy Pattern for Varying Business Logic

When a domain requires different algorithms for the same conceptual operation, the Strategy Pattern is an effective solution. This was used in the `rules-evaluation-service` to handle different rule categories.

#### a. Define the Strategy Interface
```go
// internal/domain/evaluation/evaluation_strategy.go
package evaluation

// EvaluationStrategy defines the interface for different rule evaluation algorithms.
type EvaluationStrategy interface {
    Evaluate(dslContent string, context Context) (Result, error)
}
```

#### b. Create Concrete Strategy Implementations
```go
// internal/infrastructure/strategies/promotions_strategy.go
package strategies

// PromotionsStrategy implements the evaluation logic for promotional rules.
type PromotionsStrategy struct{}

func (s *PromotionsStrategy) Evaluate(...) (evaluation.Result, error) {
    // Promotions-specific logic here...
}
```

#### c. Use a Factory or Service to Select the Strategy
```go
// internal/domain/evaluation/evaluation_service.go
package evaluation

// Service is the main application service for rule evaluation.
type Service struct {
    strategies map[string]EvaluationStrategy
}

func (s *Service) GetStrategyForCategory(category string) (EvaluationStrategy, error) {
    strategy, ok := s.strategies[category]
    if !ok {
        return nil, fmt.Errorf("no strategy for category: %s", category)
    }
    return strategy, nil
}
```

## 8. Output Specifications

### Format Requirements:
- **Project Structure**: Hexagonal architecture with clear layer separation
- **Code Quality**: Go fmt, golint, go vet compliance with 85%+ test coverage
- **API Documentation**: Complete OpenAPI 3.0 and Protobuf specifications
- **Database Schema**: Proper migrations with versioning and rollback capability

### Quality Criteria:
- **Performance**: Meet SLA requirements (<500ms response, 1000+ TPS)
- **Reliability**: Graceful error handling with proper HTTP status codes
- **Observability**: Distributed tracing, structured logging, health checks
- **Security**: Input validation, authentication middleware, secure configuration

## 9. Validation Checkpoints

### Pre-execution Validation:
- [ ] Domain requirements clearly understood and documented
- [ ] API contracts defined with OpenAPI 3.0 and Protobuf
- [ ] Database schema designed with proper relationships
- [ ] Integration points identified (NATS, external services)

### Mid-execution Validation:
- [ ] Domain layer implemented with comprehensive unit tests
- [ ] Application services properly orchestrate business operations
- [ ] Infrastructure adapters correctly implement port contracts
- [ ] REST/gRPC APIs handle all specified scenarios

### Post-execution Validation:
- [ ] All tests pass with >85% coverage
- [ ] Performance requirements met under load testing
- [ ] Security scanning shows no critical vulnerabilities
- [ ] Documentation complete and accurate
- [ ] Service deployable to Kubernetes with proper configuration

## Implementation Tasks Breakdown

### Phase 1: Architecture Setup (Days 1-2)
1. **Project Structure Creation**
   - Initialize Go module with proper naming
   - Create hexagonal architecture folder structure
   - Setup dependency injection container
   - Configure logging and error handling

2. **Domain Layer Foundation**
   - Define core entities and value objects
   - Implement domain services and specifications
   - Create repository interfaces
   - Add domain event structures

### Phase 2: Business Logic Implementation (Days 3-5)
1. **Domain Entities and Business Rules**
   - Implement aggregate roots with business logic
   - Add entity methods with proper validation
   - Create domain events for state changes
   - Write comprehensive unit tests

2. **Application Services**
   - Create command and query handlers
   - Implement input validation and mapping
   - Add transaction management
   - Handle domain events publishing

### Phase 3: Infrastructure Integration (Days 6-8)
1. **Database Integration**
   - Implement repository pattern with GORM
   - Create database migrations
   - Add connection pooling and health checks
   - Write integration tests

2. **External Service Integration**
   - Implement NATS event publishing/consuming
   - Add Redis caching layer
   - Create anti-corruption layers for external APIs
   - Mock external dependencies for testing

### Phase 4: API Implementation (Days 9-10)
1. **REST API Development**
   - Create Gin/Echo handlers with proper routing
   - Implement request/response DTOs
   - Add authentication and authorization middleware
   - Generate OpenAPI documentation

2. **gRPC API Development**
   - Define Protobuf service contracts
   - Implement gRPC server handlers
   - Add interceptors for logging and metrics
   - Create client libraries for internal communication

### Phase 5: Testing and Quality Assurance (Days 11-12)
1. **Comprehensive Testing**
   - Write behavioral tests using Gherkin
   - Create integration tests for all layers
   - Add contract tests for external dependencies
   - Implement performance and load tests

2. **Observability and Production Readiness**
   - Add distributed tracing with OpenTelemetry
   - Implement structured logging
   - Create Prometheus metrics
   - Add health checks and readiness probes

## Best Practices

### Code Organization:
- Use dependency injection for loose coupling
- Implement interfaces for all external dependencies
- Keep domain logic pure without infrastructure concerns
- Use typed errors for better error handling

### Testing Strategy:
- Follow test pyramid: unit (70%), integration (20%), E2E (10%)
- Use table-driven tests for multiple scenarios
- Mock external dependencies appropriately
- Test error conditions and edge cases

### Performance Optimization:
- Use connection pooling for database connections
- Implement caching for frequently accessed data
- Add request timeouts and circuit breakers
- Monitor and profile performance regularly

### Security Considerations:
- Validate all inputs at API boundaries
- Use parameterized queries to prevent SQL injection
- Implement proper authentication and authorization
- Scan dependencies for known vulnerabilities

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jplazaroalonso) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
