## cacheflow

> A Spring Boot starter implementing Russian Doll caching patterns with multi-level cache hierarchy (Local → Redis → Edge).

# CacheFlow Spring Boot Starter

A Spring Boot starter implementing Russian Doll caching patterns with multi-level cache hierarchy (Local → Redis → Edge).

## Project Structure

```
src/main/kotlin/io/cacheflow/spring/
├── annotation/          # Cache annotations (@CacheFlow, @CacheFlowEvict)
├── aspect/             # AOP aspects for caching interception
├── autoconfigure/      # Spring Boot auto-configuration
├── dependency/         # Dependency tracking and resolution
├── fragment/          # Fragment caching implementation
├── versioning/        # Cache versioning system
└── service/           # Core cache services
```

## Quick Commands

### Build and Test
```bash
# Full build with tests and quality checks
./gradlew build

# Run tests only
./gradlew test

# Run with coverage report
./gradlew test jacocoTestReport

# Code quality analysis
./gradlew detekt

# Security scan
./gradlew dependencyCheckAnalyze
```

### Development Workflow
```bash
# Quality gate (run before commits)
./gradlew detekt test jacocoTestReport

# Clean build
./gradlew clean build

# Generate documentation
./gradlew dokka
```

## Key Features

- **Russian Doll Caching**: Nested fragment composition with dependency tracking
- **Multi-level Cache**: Local → Redis → Edge cache hierarchy
- **Automatic Invalidation**: Dependency-based cache invalidation
- **Spring Boot Integration**: Auto-configuration and starter patterns
- **Performance Monitoring**: Metrics and observability built-in

## Current Focus

Working on `feature/caching-improvement` branch with:
- Comprehensive testing framework
- Enhanced dependency tracking
- Fragment composition features
- Performance optimizations

## Code Standards

- **Test Coverage**: Maintain 90%+ coverage
- **Code Quality**: Zero Detekt violations
- **Documentation**: KDoc for all public APIs
- **Security**: Input validation and secure patterns
- **Performance**: Sub-millisecond cache operations

## Architecture Patterns

### Fragment Caching
```kotlin
@CacheFlowFragment(
    key = "user-profile:#{id}",
    dependencies = ["user:#{id}", "settings:#{id}"],
    ttl = 1800L
)
fun renderUserProfile(@PathVariable id: Long): String
```

### Dependency Tracking
```kotlin
@CacheFlowEvict(patterns = ["user:#{id}"])
fun updateUser(id: Long, user: User)
```

### Fragment Composition
```kotlin
@CacheFlowComposition(
    fragments = ["header:#{userId}", "content:#{userId}", "footer:global"]
)
fun renderUserPage(@PathVariable userId: Long): String
```

## Testing Strategy

- **Unit Tests**: 60-70% of test suite
- **Integration Tests**: 20-30% with Spring context
- **Performance Tests**: 5-10% for benchmarking
- **Coverage Target**: 90%+ for all components

## Common Tasks

### Adding New Features
1. Follow Russian Doll caching patterns
2. Implement comprehensive tests first
3. Add proper dependency tracking
4. Update documentation
5. Verify performance impact

### Bug Fixes
1. Write failing test first
2. Implement minimal fix
3. Verify no regression
4. Update docs if needed
5. Check performance impact

### Refactoring
1. Ensure backward compatibility
2. Maintain test coverage
3. Preserve performance
4. Update documentation
5. Follow existing patterns

## Important Files

- `AI_MAINTENANCE_RULES.md` - Comprehensive AI guidelines
- `.ai-context.md` - Project context for AI assistants
- `.ai-patterns.md` - Code patterns and examples
- `docs/RUSSIAN_DOLL_CACHING_GUIDE.md` - Implementation guide

## Quality Gates

All changes must pass:
- ✅ Detekt analysis (zero violations)
- ✅ Test suite (90%+ coverage)
- ✅ Security scan (no high severity)
- ✅ Performance benchmarks
- ✅ Documentation updates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mmorrison) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
