## bella-openai4j

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

bella-openai4j is a Java library for interacting with OpenAI's GPT models and API services. It's originally forked from TheoKanning/openai-java and continues development to support the latest OpenAI API features. The library uses Java 1.8 and Maven as the build system.

## Multi-Module Maven Structure

This is a Maven multi-module project with the following structure:

- **Root** (`/`): Parent POM managing the overall project
- **API** (`api/`): Core POJOs and data models for OpenAI API requests/responses
- **Client** (`client/`): Retrofit-based HTTP client implementation 
- **Service** (`service/`): High-level service layer with convenience methods
- **Example** (`example/`): Usage examples and demos

## Common Development Commands

### Building the Project
```bash
# Clean and compile all modules
mvn clean compile

# Package all modules (creates JARs)
mvn clean package

# Install to local Maven repository
mvn clean install

# Skip tests during build
mvn clean package -DskipTests
```

### Running Tests
```bash
# Run all tests
mvn test

# Run tests for specific module
mvn test -pl service
mvn test -pl api
mvn test -pl client

# Run specific test class
mvn test -Dtest=ChatCompletionTest
mvn test -Dtest=AssistantTest -pl service
```

### Maven Profiles
```bash
# Build with release profile (includes sources, javadocs, GPG signing)
mvn clean package -Prelease
```

## Code Architecture

### API Module (`api/`)
Contains all POJOs representing OpenAI API entities:
- Request/response objects for each API endpoint
- Uses Jackson for JSON serialization/deserialization
- Snake case JSON properties with `@JsonProperty` annotations
- Lombok annotations: `@Data`, `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor`

### Client Module (`client/`)
- `OpenAiApi`: Retrofit interface defining HTTP endpoints
- `AuthenticationInterceptor`: Handles API key authentication
- `OrganizationAndProjectHeaderInterceptor`: Manages organization/project headers

### Service Module (`service/`)
- `OpenAiService`: Main service class providing high-level API methods
- Streaming support with RxJava2 Flowables
- Assistant streaming with `AssistantStreamManager` and `AssistantEventHandler`
- Function execution framework with `FunctionExecutor` and `FunctionDefinition`

## Key Features and Components

### Environment Variables
- `OPENAI_API_KEY`: API key (required)
- `OPENAI_API_BASE_URL`: Custom base URL (optional, defaults to https://api.openai.com/v1/)

### OpenAiService Configuration
The service supports multiple initialization patterns:
```java
// Default (uses environment variables)
OpenAiService service = new OpenAiService();

// With API key
OpenAiService service = new OpenAiService(apiKey);

// With API key and base URL  
OpenAiService service = new OpenAiService(apiKey, baseUrl);

// With timeout
OpenAiService service = new OpenAiService(apiKey, Duration.ofSeconds(30));
```

### Function/Tool System
- `FunctionDefinition`: Defines callable functions with parameters and executors
- `FunctionExecutorManager`: Manages multiple function executions
- `ChatTool`: Wrapper for tools in chat completions
- Support for both legacy functions and current tool-based approach

### Assistant API
- Complete Assistants v2 API support
- Streaming capabilities with event handlers
- Thread and message management
- Vector store operations
- Run and run step handling

### Streaming Support
- Chat completion streaming with `streamChatCompletion()`
- Assistant streaming with `AssistantStreamManager`
- RxJava2 Flowables for reactive programming

## Testing Structure

### API Tests (`api/src/test/`)
- JSON serialization/deserialization tests
- Fixture files in `src/test/resources/fixtures/`
- `JsonTest.java` validates all POJO mappings

### Service Tests (`service/src/test/`)
- Integration tests for each API endpoint
- Assistant-specific tests in `assistants/` package
- Mock tests and real API call tests
- Test resources include sample files and data

## Development Guidelines

### Adding New API Endpoints
1. Create POJOs in API module with proper Jackson annotations
2. Add endpoint to `OpenAiApi` interface in client module  
3. Implement service method in `OpenAiService`
4. Add integration tests
5. Create JSON fixtures for testing

### Code Conventions
- Use camelCase for Java fields, snake_case for JSON with `@JsonProperty`
- Include comprehensive Javadoc comments
- Follow existing patterns for request builders and response handling
- Use Lombok annotations consistently

### Dependencies
- **Jackson**: JSON processing (2.14.2)
- **Retrofit**: HTTP client (2.9.0) 
- **RxJava2**: Reactive programming
- **Lombok**: Code generation (1.18.30)
- **JUnit Jupiter**: Testing (5.10.1)

## Examples Package
The `example/` module contains comprehensive usage examples:
- `ChatExample`: Basic chat completions
- `FunctionsExample`: Function calling
- `AssistantExample`: Assistant API usage
- `ServiceCreateExample`: Service configuration options
- `ToolUtil`: Utility functions for examples

## Version Information
- Current version: 0.23.44
- Group ID: top.bella  
- Java compatibility: 1.8+
- Published to Maven Central under io.github.lambdua group

---
> Source: [LianjiaTech/bella-openai4j](https://github.com/LianjiaTech/bella-openai4j) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
