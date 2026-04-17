## viewservier

> This is a performance POC for a horizontally scalable view server handling real-time grid updates via WebSocket connections, with Kafka event streaming and React frontend.

# View Server Performance POC - Cursor Rules

## Project Overview
This is a performance POC for a horizontally scalable view server handling real-time grid updates via WebSocket connections, with Kafka event streaming and React frontend.

## General Guidelines

### Code Quality
- Write clean, readable, and maintainable code
- Use meaningful variable and function names
- Add comprehensive error handling and logging
- Include performance-focused implementations
- Add comments for complex business logic
- Follow established patterns within each technology stack

### Performance Focus
- Optimize for low latency (<50ms end-to-end)
- Handle high throughput (10,000+ messages/second)
- Minimize memory allocations in hot paths
- Use efficient data structures (ConcurrentHashMap, etc.)
- Implement proper connection pooling and resource management

### Architecture Principles
- Keep Kafka messages small (<100KB, target 10-50KB)
- Use chunked loading for large datasets
- Implement proper error recovery and reconnection logic
- Design for horizontal scaling
- Separate concerns clearly between layers

### Logging and Monitoring
- Use structured logging with appropriate levels
- Include performance metrics and timing
- Log message sizes and throughput statistics
- Add health check endpoints
- Include connection state tracking

### Security (Basic for POC)
- Validate input sizes and formats
- Implement basic rate limiting
- Use secure WebSocket connections where possible
- Sanitize user inputs

### Technology-Specific Rules
See component-specific .cursorrules files:
- `view-server/.cursorrules` - Spring Boot backend rules
- `frontend/.cursorrules` - React frontend rules  
- `mock-data-generator/.cursorrules` - Kafka producer rules

## File Naming Conventions
- Java: PascalCase for classes, camelCase for methods/variables
- JavaScript: camelCase for functions/variables, PascalCase for components
- Configuration: kebab-case for file names
- Docker: lowercase with hyphens

## Error Handling Patterns
- Always handle WebSocket connection failures
- Implement retry logic with exponential backoff
- Validate message sizes before processing
- Gracefully degrade on resource exhaustion
- Log errors with sufficient context for debugging

## Testing Approach
- Focus on performance and load testing
- Test WebSocket connection scenarios
- Validate message size constraints
- Test chunked loading behavior
- Include integration tests for Kafka flow 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ksh29-coder) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
