## plaschema

> - Follow PSR-12 coding standards for PHP

# Plaschema Project Rules and Patterns

## Code Style

- Follow PSR-12 coding standards for PHP
- Use camelCase for variables and methods
- Use PascalCase for classes
- Use snake_case for database columns
- Use ; and not && for combining powershell commands

## Project Structure

- Keep controllers thin, move business logic to services
- Use repositories for complex database queries
- Store global helpers in dedicated helper files
- Organize CSS/JS by component when possible
- Use CacheService for all caching operations, not direct Cache facade calls

## Implementation Patterns

- Use Laravel's built-in features instead of reinventing solutions
- Prefer dependency injection over facade usage in complex classes
- Use form requests for validation logic
- Use resource controllers for CRUD operations
- Use model-specific cache keys for better cache organization
- Apply automatic cache invalidation through model observers
- Use cache tags for targeted cache clearing

## Documentation Requirements

- Add DocBlocks to classes and methods
- Document all non-trivial code blocks with comments
- Keep README updated with installation and configuration instructions
- Document API endpoints if created
- Document cache key patterns and cache invalidation strategies

## Development Workflow

- Create migrations for all database changes
- Use feature branches for new features
- Keep commits focused and descriptive
- Run Redis locally when developing features that use caching

## Caching Strategy

- Use the CacheService for all caching operations
- Generate model-specific cache keys using modelKey() and collectionKey() methods
- Skip caching for filtered or personalized content
- Apply appropriate cache durations based on content volatility
- Clear related caches when models are updated
- Tag caches for more targeted cache clearing

## Search Implementation

- Use dedicated SearchService for search operations
- Implement multi-criteria filtering through query building
- Cache search results with appropriate keys
- Skip caching for personalized search results
- Use progressive loading for search result displays
- Implement context-specific empty states for no results

## API Implementation

- Use Laravel Sanctum for API authentication
- Implement OpenAPI/Swagger annotations for API documentation
- Use API resources for consistent data transformation
- Apply cache tags for optimized API response caching
- Use proper HTTP status codes for different response types
- Validate all API inputs with appropriate validation rules
- Include proper error messages in API responses
- Implement pagination for list endpoints
- Add filtering options for collections

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benzino98) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
