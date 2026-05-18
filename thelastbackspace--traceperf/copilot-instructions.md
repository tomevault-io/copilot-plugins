## custom-rule

> ├── src/                      # Source code

# TracePerf Development Guidelines

## 📂 Project Structure

```
traceperf/
├── src/                      # Source code
│   ├── index.ts              # Main entry point
│   ├── core/                 # Core functionality
│   │   ├── logger.ts         # Base logger implementation
│   │   ├── config.ts         # Configuration management
│   │   └── constants.ts      # Shared constants
│   ├── trackers/             # Performance tracking modules
│   │   ├── execution.ts      # Execution flow tracking
│   │   ├── performance.ts    # Performance monitoring
│   │   └── memory.ts         # Memory usage tracking
│   ├── formatters/           # Output formatting
│   │   ├── cli.ts            # CLI output formatting
│   │   ├── json.ts           # JSON output formatting
│   │   └── ascii.ts          # ASCII art generation
│   ├── utils/                # Utility functions
│   │   ├── timing.ts         # Timing utilities
│   │   ├── stack.ts          # Call stack utilities
│   │   └── colors.ts         # Terminal color utilities
│   └── types/                # TypeScript type definitions
│       ├── index.ts          # Type exports
│       ├── logger.ts         # Logger types
│       └── config.ts         # Configuration types
├── test/                     # Test files
│   ├── unit/                 # Unit tests
│   ├── integration/          # Integration tests
│   └── fixtures/             # Test fixtures
├── examples/                 # Example usage
│   ├── basic-logging.js      # Basic logging examples
│   ├── execution-flow.js     # Execution flow examples
│   └── performance.js        # Performance tracking examples
├── docs/                     # Documentation
│   ├── api/                  # API documentation
│   ├── guides/               # Usage guides
│   └── examples/             # Example documentation
├── scripts/                  # Build and utility scripts
├── .github/                  # GitHub configuration
│   └── workflows/            # GitHub Actions workflows
├── .eslintrc.js              # ESLint configuration
├── .prettierrc               # Prettier configuration
├── jest.config.js            # Jest configuration
├── tsconfig.json             # TypeScript configuration
├── package.json              # Package manifest
└── README.md                 # Project README
```

## 🧩 Code Architecture

### Core Principles

1. **Modularity**: Each component should have a single responsibility
2. **Extensibility**: Design for extension with plugins and custom formatters
3. **Performance**: Minimize overhead, especially in production environments
4. **Type Safety**: Use TypeScript for all code to ensure type safety
5. **Testing**: Maintain high test coverage for all components

### Design Patterns

1. **Singleton**: Use for the main logger instance
2. **Factory**: For creating different types of loggers and formatters
3. **Observer**: For event-based logging and notifications
4. **Decorator**: For adding functionality to loggers
5. **Strategy**: For different logging strategies based on environment

## 💻 Coding Standards

### General Guidelines

1. Use TypeScript for all code
2. Follow functional programming principles where appropriate
3. Minimize side effects
4. Use immutable data structures when possible
5. Avoid global state except for the main logger instance
6. Use async/await for asynchronous operations
7. Implement proper error handling throughout

### Naming Conventions

1. **Files**: Use kebab-case for filenames (e.g., `execution-tracker.ts`)
2. **Classes**: Use PascalCase for class names (e.g., `ExecutionTracker`)
3. **Functions/Methods**: Use camelCase for functions (e.g., `trackExecution()`)
4. **Constants**: Use UPPER_SNAKE_CASE for constants (e.g., `DEFAULT_TIMEOUT`)
5. **Interfaces/Types**: Use PascalCase with prefix I for interfaces (e.g., `ILoggerConfig`)
6. **Private Properties**: Use underscore prefix for private properties (e.g., `_config`)

### Code Style

1. Use 2 spaces for indentation
2. Maximum line length of 100 characters
3. Use semicolons at the end of statements
4. Use single quotes for strings
5. Always use curly braces for control structures
6. Add trailing commas in multi-line object/array literals
7. Use explicit type annotations for function parameters and return types

## 🚀 Performance Optimization

### General Optimizations

1. **Lazy Initialization**: Initialize components only when needed
2. **Caching**: Cache expensive operations and computed values
3. **Batching**: Batch log operations when possible
4. **Sampling**: Implement sampling for high-volume logs
5. **Async Logging**: Use non-blocking operations for I/O

### Production Optimizations

1. **Tree Shaking**: Ensure the library is tree-shakeable
2. **Dead Code Elimination**: Remove unused code in production builds
3. **Conditional Compilation**: Use environment flags to remove debug code
4. **Minimal Dependencies**: Minimize external dependencies
5. **Bundle Size**: Keep the bundle size as small as possible

## 📝 Documentation Standards

### Code Documentation

1. Use JSDoc comments for all public APIs
2. Include examples in documentation
3. Document parameters, return values, and exceptions
4. Add inline comments for complex logic
5. Keep comments up-to-date with code changes

### Example JSDoc Format

```typescript
/**
 * Tracks the execution of a function and logs performance metrics
 * 
 * @param {Function} fn - The function to track
 * @param {TrackOptions} [options] - Optional tracking configuration
 * @returns {any} The return value of the tracked function
 * @throws {Error} If the function throws an error
 * 
 * @example
 * // Basic usage
 * tracePerf.track(() => {
 *   // Code to track
 *   return result;
 * });
 * 
 * // With options
 * tracePerf.track(() => {
 *   // Code to track
 *   return result;
 * }, { threshold: 100, label: 'Custom Label' });
 */
function track<T>(fn: () => T, options?: TrackOptions): T {
  // Implementation
}
```

## 🧪 Testing Guidelines

### Testing Strategy

1. **Unit Tests**: Test individual components in isolation
2. **Integration Tests**: Test interactions between components
3. **Performance Tests**: Benchmark performance and memory usage
4. **Edge Cases**: Test boundary conditions and error handling
5. **Mocking**: Use mocks for external dependencies

### Test Structure

1. Use descriptive test names
2. Follow the Arrange-Act-Assert pattern
3. Group related tests together
4. Use beforeEach/afterEach for setup and teardown
5. Aim for high test coverage (>90%)

## 🔒 Security Considerations

1. **Input Validation**: Validate all user inputs
2. **Sensitive Data**: Avoid logging sensitive information
3. **Sanitization**: Sanitize log output to prevent injection attacks
4. **Rate Limiting**: Implement rate limiting for high-volume logging
5. **Permissions**: Use appropriate file permissions for log files

## 🚢 Deployment & Release Process

1. **Semantic Versioning**: Follow semver for versioning
2. **Changelogs**: Maintain detailed changelogs
3. **Release Notes**: Provide comprehensive release notes
4. **Continuous Integration**: Use CI/CD for automated testing and deployment
5. **Release Candidates**: Use release candidates for major versions

## 🔄 Continuous Improvement

1. **Code Reviews**: Require code reviews for all changes
2. **Static Analysis**: Use static analysis tools to catch issues
3. **Performance Monitoring**: Continuously monitor performance
4. **User Feedback**: Actively seek and incorporate user feedback
5. **Benchmarking**: Regularly benchmark against alternatives

## 🌟 Best Practices for Specific Features

### Execution Flow Tracking

1. Use proxies or function wrappers to track function calls
2. Maintain a call stack to track nested calls
3. Use high-resolution timers for accurate timing
4. Extract function names using reflection
5. Implement efficient ASCII art generation

### Performance Monitoring

1. Use `process.hrtime()` for high-precision timing
2. Implement configurable thresholds for bottleneck detection
3. Collect statistical data for performance analysis
4. Use sampling for high-frequency function calls
5. Implement memory usage tracking with minimal overhead

### Conditional Logging

1. Use environment variables for mode detection
2. Implement efficient log filtering
3. Use compile-time optimizations for production builds
4. Provide runtime configuration options
5. Implement log level hierarchies

### Nested Logging

1. Use a stack-based approach for tracking nesting
2. Implement efficient indentation management
3. Provide automatic group closing
4. Use visual indicators for hierarchy
5. Implement collapsible groups for complex logs 

---
> Source: [thelastbackspace/traceperf](https://github.com/thelastbackspace/traceperf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
