## opencode-bootcamp

> This document provides guidelines for AI agents working on this Spring Boot project using Gradle.

# Agent Guidelines for Spring Boot Gradle Project

This document provides guidelines for AI agents working on this Spring Boot project using Gradle.

## Project Overview
- **Framework**: Spring Boot 4.0.5
- **Build Tool**: Gradle (Groovy DSL)
- **Java Version**: 21
- **Package Structure**: `com.example.demo`

## Build Commands

### Basic Commands
```bash
# Build the project
./gradlew build

# Clean build
./gradlew clean build

# Run the application
./gradlew bootRun

# Build without tests
./gradlew build -x test
```

### Testing Commands
```bash
# Run all tests
./gradlew test

# Run a single test class
./gradlew test --tests "com.example.demo.YourTestClass"

# Run a single test method
./gradlew test --tests "com.example.demo.YourTestClass.yourTestMethod"

# Run tests with coverage
./gradlew test jacocoTestReport

# Run specific test task
./gradlew test --rerun-tasks
```

### Development Commands
```bash
# Check dependencies
./gradlew dependencies

# Generate IDEA project files
./gradlew idea

# Check for dependency updates
./gradlew dependencyUpdates

# Run application with dev tools
SPRING_PROFILES_ACTIVE=dev ./gradlew bootRun
```

## Code Style Guidelines

### Naming Conventions
- **Classes**: PascalCase (e.g., `UserController`, `OrderService`)
- **Methods**: camelCase (e.g., `getUserById`, `processOrder`)
- **Variables**: camelCase (e.g., `userRepository`, `orderNumber`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`)
- **Packages**: lowercase with dots (e.g., `com.example.demo.controller`)

### Import Organization
```java
// 1. Java and Jakarta imports
import java.util.List;
import jakarta.persistence.*;

// 2. Spring Framework imports
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.*;

// 3. Third-party imports
import lombok.Data;
import io.swagger.v3.oas.annotations.*;

// 4. Project imports
import com.example.demo.model.User;
import com.example.demo.repository.UserRepository;
```

### Class Structure
```java
package com.example.demo.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class UserService {
    // Field declarations
    
    // Constructor (if not using Lombok)
    
    // Public methods
    
    // Private helper methods
}
```

### Error Handling
- Use `@RestControllerAdvice` for global exception handling
- Create custom exception classes extending `RuntimeException`
- Always log exceptions with appropriate levels
- Use meaningful error messages and HTTP status codes

### Spring-Specific Guidelines
- Use constructor injection (prefer `@RequiredArgsConstructor` from Lombok)
- Annotate REST controllers with `@RestController`
- Use `@Service`, `@Repository`, `@Component` appropriately
- Prefer `@ConfigurationProperties` for configuration
- Use `@Transactional` for database operations

## Testing Guidelines

### Test Structure
```java
package com.example.demo.service;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @BeforeEach
    void setUp() {
        // Setup common test data
    }
    
    @Test
    void shouldReturnUserWhenFound() {
        // Given
        User expected = new User(1L, "John");
        when(userRepository.findById(1L)).thenReturn(Optional.of(expected));
        
        // When
        User result = userService.getUserById(1L);
        
        // Then
        assertThat(result).isEqualTo(expected);
        verify(userRepository).findById(1L);
    }
}
```

### Test Naming
- Use descriptive test names that explain the behavior
- Follow pattern: `should[ExpectedBehavior]When[Condition]`
- Use `@DisplayName` for better test reports

### Mocking Guidelines
- Mock external dependencies (repositories, services, clients)
- Use `@Spy` for partial mocking when needed
- Verify interactions with mocks using `verify()`
- Use argument captors for complex verifications

## Database Guidelines
- Use JPA annotations for entity mapping
- Create repository interfaces extending `JpaRepository`
- Use `@Entity` and `@Table` annotations
- Define proper indexes and constraints
- Use `@Version` for optimistic locking

## REST API Guidelines
- Use proper HTTP methods (GET, POST, PUT, DELETE, PATCH)
- Return appropriate HTTP status codes
- Use DTOs for request/response objects
- Document APIs with OpenAPI annotations
- Validate request data with `@Valid`

## Security Guidelines
- Use Spring Security for authentication/authorization
- Configure CORS properly
- Validate input to prevent injection attacks
- Use HTTPS in production
- Implement proper error handling for security exceptions

## Common Patterns to Follow
1. **Repository-Service-Controller pattern** for separation of concerns
2. **DTO pattern** for API boundaries
3. **Builder pattern** for complex object creation
4. **Strategy pattern** for interchangeable algorithms
5. **Factory pattern** for object creation

## Quality Checks
```bash
# Check code format (if spotless is configured)
./gradlew spotlessApply

# Run static analysis
./gradlew checkstyleMain checkstyleTest

# Check for vulnerabilities
./gradlew dependencyCheckAnalyze

# Generate documentation
./gradlew javadoc
```

## Development Workflow
1. Write failing tests first (TDD)
2. Implement the feature
3. Run tests and ensure they pass
4. Run linting and formatting tools
5. Build and verify the application
6. Create/update documentation

## Notes for Agents
- Always run tests after making changes
- Follow existing patterns in the codebase
- Maintain backward compatibility when possible
- Document significant changes
- Use meaningful commit messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/metaphor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/metaphor)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
