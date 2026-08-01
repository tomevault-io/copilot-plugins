## cline-tips

> - main.go should be minimal, serving only as entrypoint

## Project Intelligence

### Go Code Styling Rules
- main.go should be minimal, serving only as entrypoint
- All packages under ./pkg/ directory
- Keep files under 300 lines by splitting into multiple files
- Group small utility functions in util.go within each package
- Each exported function requires detailed documentation comments
- Run go fmt ./... before committing code
- Make frequent, small git commits while coding to create recovery points
- Use git checkout [commit-hash] -- path/to/file.go to recover accidentally deleted code
- Group imports in three blocks: standard library, external deps, internal packages
- Break up large packages into multiple files with clear responsibilities

### Project Structure
- Define key components in the README.md
- Place all packages under pkg/
- Place executables under cmd/
- Documentation in docs/
- Configuration in config/
- Unit tests alongside the code they test
- Integration tests in separate test/ directory

### Logging Best Practices
- Use structured logging (e.g., logrus, zap)
- Initialize package-level loggers
- Use appropriate log levels (debug, info, warn, error)
- Include context with log entries
- When viewing logs:
  - Always use --tail flag to limit log output
  - Never use -f/--follow in scripts as these commands never return
  - Recommended: kubectl logs pod-name --tail=100 (for K8s projects)

### Documentation Standards
- Document all exported functions, types, and packages
- Use consistent formatting for godoc comments
- Include examples for complex functions
- Keep README.md up to date with usage instructions
- Document configuration options and their effects
- Create CONTRIBUTING.md for collaboration guidelines

### Testing Requirements
- Write tests for all packages
- Maintain high test coverage for core logic
- Use table-driven tests for multiple test cases
- Mock external dependencies
- Name tests using the TestFunctionName_Scenario convention
- Add benchmarks for performance-critical code
- Include integration tests for end-to-end verification

### Error Handling
- Always check error returns
- Provide context when wrapping errors (e.g., fmt.Errorf("context: %w", err))
- Use custom error types for domain-specific errors
- Include appropriate logging with error context
- Return early on errors to avoid deep nesting
- Consider implementing the error interface for custom error types

### Performance Considerations
- Use profiling tools for optimization
- Benchmark before and after optimizations
- Consider memory usage for large data structures
- Use sync.Pool for frequently allocated objects
- Implement worker pools for concurrent operations
- Be aware of garbage collection impact
- Consider using sync.Mutex for shared resources

---
> Source: [SupportTools/cline-tips](https://github.com/SupportTools/cline-tips) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
