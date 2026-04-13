## llm-topix

> This document serves as the authoritative development constitution for LLM TOPIX, establishing technical standards, architectural principles, and best practices derived from our successful RED → GREEN → REFACTOR development phases. Every team member must adhere to these guidelines to ensure code quality, maintainability, security, and performance consistency across the entire platform.

# LLM TOPIX Development Constitution

## Introduction

This document serves as the authoritative development constitution for LLM TOPIX, establishing technical standards, architectural principles, and best practices derived from our successful RED → GREEN → REFACTOR development phases. Every team member must adhere to these guidelines to ensure code quality, maintainability, security, and performance consistency across the entire platform.

The principles outlined here are not merely suggestions but mandatory requirements that have been proven through rigorous testing and real-world implementation in our TDD-driven development process.

## Coding Standards

### Naming Conventions

#### Variables and Functions
- **Python (Backend)**:
  - Local variables: `snake_case` (e.g., `latest_articles`, `execution_time_ms`)
  - Global variables: `snake_case` (e.g., `database_url`, `api_version`)
  - Constants: `SCREAMING_SNAKE_CASE` (e.g., `PERFORMANCE_THRESHOLD_MS`, `DATABASE_NAME_DEV`)
  - Functions/methods: `snake_case` (e.g., `get_latest_articles`, `create_error_response`)

- **TypeScript (Frontend)**:
  - Local/global variables: `camelCase` (e.g., `latestArticles`, `formattedDate`)
  - Constants: `SCREAMING_SNAKE_CASE` or `UPPER_CAMEL_CASE` (e.g., `API_BASE_URL`, `MAX_ARTICLES`)
  - Functions/methods: `camelCase` (e.g., `formatDate`, `validateArticle`)

#### Classes and Types
- **Python**: `PascalCase` (e.g., `ArticleService`, `DatabaseError`, `ApplicationError`)
- **TypeScript**: `PascalCase` (e.g., `ArticleCardProps`, `Article`, `ISO8601String`)

#### Files and Modules
- **Python**: `snake_case.py` (e.g., `article_service.py`, `response_helpers.py`, `constants.py`)
- **TypeScript**: 
  - Components: `PascalCase.tsx` (e.g., `ArticleCard.tsx`)
  - Utilities: `camelCase.ts` (e.g., `article.ts`, `responseHelpers.ts`)
  - Modules: `kebab-case.ts` (e.g., `api-client.ts`) when not components

### Formatting and Style

#### Python Standards
- **Formatter**: Black (mandatory, no exceptions)
- **Linter**: Flake8 with line length 88 characters (Black's default)
- **Type Checker**: MyPy with strict mode enabled
- **Import Order**: isort with Black compatibility
- **Docstrings**: Google-style format for all public methods

```python
# Example of proper Python formatting
def get_latest_articles(self, limit: int = 5) -> List[Dict[str, Any]]:
    """Retrieve the latest articles from the database.
    
    Args:
        limit: Maximum number of articles to retrieve.
        
    Returns:
        List of article dictionaries with required fields.
        
    Raises:
        DatabaseError: When database connection fails.
        ValidationError: When limit parameter is invalid.
    """
    start_time = time.time()
    try:
        # Implementation here
        pass
    finally:
        self._check_performance(start_time)
```

#### TypeScript Standards
- **Formatter**: Prettier with 2-space indentation
- **Linter**: ESLint with strict TypeScript configuration
- **Line Length**: 100 characters maximum
- **Semicolons**: Always required
- **Quotes**: Single quotes for strings, double quotes for JSX attributes

```typescript
// Example of proper TypeScript formatting
export const ArticleCard: React.FC<ArticleCardProps> = React.memo(({
  article,
  className,
  onClick,
}) => {
  const formattedDate = useMemo(
    () => formatDate(article.published_at),
    [article.published_at]
  );
  
  return (
    <article
      className={`${styles.articleCard} ${className || ''}`}
      role="article"
      aria-label={`Article: ${article.title}`}
    >
      {/* Component content */}
    </article>
  );
});
```

## Type System and Language Features

### Python Type Hints

#### Mandatory Type Annotations
All functions, methods, and class attributes must have comprehensive type hints:

```python
from typing import Dict, List, Any, Optional, Union, Final
from abc import ABC, abstractmethod

# Constants with Final type
DATABASE_HOST: Final[str] = os.getenv('DB_HOST', 'localhost')
PERFORMANCE_THRESHOLD_MS: Final[float] = 50.0

# Class with typed attributes
class ArticleService:
    def __init__(self, db_session: Any) -> None:
        self._db_session: Any = db_session
        self._cache: Dict[str, Any] = {}
    
    def get_latest_articles(self, limit: int = 5) -> List[Dict[str, Any]]:
        # Method implementation
        pass
```

#### Custom Type Definitions
Use `NewType` and `TypedDict` for enhanced type safety:

```python
from typing import NewType, TypedDict

# Branded types for domain concepts
ArticleId = NewType('ArticleId', int)
ISO8601String = NewType('ISO8601String', str)

# Structured dictionaries
class ArticleDict(TypedDict):
    id: ArticleId
    title: str
    summary_truncated: str
    published_at: ISO8601String
    source_url: str
```

### TypeScript Type System

#### Strict Mode Requirements
TypeScript must be configured with the strictest possible settings:

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true
  }
}
```

#### Branded Types Implementation
Implement branded types for enhanced type safety:

```typescript
// Branded type definitions
export type ISO8601String = string & { readonly brand: unique symbol };
export type URLString = string & { readonly brand: unique symbol };
export type ArticleId = number & { readonly brand: unique symbol };

// Type guards for runtime validation
export const isISO8601String = (value: string): value is ISO8601String => {
  return !isNaN(Date.parse(value));
};

export const isURLString = (value: string): value is URLString => {
  try {
    new URL(value);
    return true;
  } catch {
    return false;
  }
};
```

#### Interface vs Type Usage
- **Interfaces**: For object shapes that may be extended or implemented
- **Types**: For unions, intersections, and computed types

```typescript
// Use interfaces for extensible object shapes
export interface Article {
  readonly id: ArticleId;
  readonly title: string;
  readonly summary_truncated: string;
  readonly published_at: ISO8601String;
  readonly source_url: URLString;
}

// Use types for unions and computed types
export type ArticleStatus = 'draft' | 'published' | 'archived';
export type ArticleCardVariant = 'default' | 'compact' | 'featured';
```

## API Design and Response Standards

### RESTful API Principles

#### Endpoint Naming Conventions
- **Resources**: Plural nouns (`/articles`, `/users`)
- **Actions**: HTTP verbs (`GET`, `POST`, `PUT`, `DELETE`)
- **Hierarchical**: Use nested paths for relationships (`/articles/{id}/comments`)
- **Versioning**: Include version in URL or headers (`/api/v1/articles`)

```python
# Example endpoint definitions
@articles_bp.route('/api/articles/latest', methods=['GET'])
@articles_bp.route('/api/articles/<int:article_id>', methods=['GET'])
@articles_bp.route('/api/articles', methods=['POST'])
```

#### HTTP Methods Usage
- **GET**: Retrieve data (idempotent, cacheable)
- **POST**: Create new resources
- **PUT**: Update entire resource (idempotent)
- **PATCH**: Partial resource updates
- **DELETE**: Remove resources (idempotent)

### Standardized Response Formats

#### Success Response Structure
```json
{
  "status": "success",
  "data": {
    "articles": [...],
    "count": 5,
    "pagination": {
      "page": 1,
      "limit": 5,
      "total": 100
    }
  },
  "message": "Articles retrieved successfully"
}
```

#### Error Response Structure
All API errors must follow this standardized format:

```json
{
  "status": "error",
  "error": {
    "code": "DATABASE_UNAVAILABLE",
    "message": "Database service is temporarily unavailable. Please try again later.",
    "details": {
      "timestamp": "2024-01-15T10:30:00Z",
      "request_id": "req_abc123",
      "retry_after": 30
    }
  }
}
```

#### Error Code Taxonomy
- **VALIDATION_ERROR**: Input validation failures (400)
- **AUTHENTICATION_REQUIRED**: Missing or invalid authentication (401)
- **AUTHORIZATION_DENIED**: Insufficient permissions (403)
- **RESOURCE_NOT_FOUND**: Requested resource doesn't exist (404)
- **RATE_LIMIT_EXCEEDED**: Too many requests (429)
- **DATABASE_UNAVAILABLE**: Database connection issues (503)
- **INTERNAL_SERVER_ERROR**: Unexpected server errors (500)

#### HTTP Status Code Guidelines
- **200**: Successful GET requests
- **201**: Successful POST (resource created)
- **204**: Successful DELETE (no content)
- **400**: Client errors (validation, syntax)
- **401**: Authentication required
- **403**: Authorization denied
- **404**: Resource not found
- **429**: Rate limit exceeded
- **500**: Internal server error
- **503**: Service unavailable

## Architecture and Design Principles

### Backend Architecture (Flask)

#### Directory Structure Philosophy
```
backend/app/
├── config/           # Configuration and constants
│   ├── constants.py  # Application constants
│   ├── database.py   # Database configuration
│   └── __init__.py
├── models/           # SQLAlchemy data models
│   ├── article.py    # Article model definition
│   └── __init__.py
├── services/         # Business logic layer
│   ├── article_service.py
│   ├── formatters.py
│   └── __init__.py
├── routes/           # API route definitions
│   ├── articles.py   # Article-related endpoints
│   └── __init__.py
├── utils/            # Utility functions
│   ├── response_helpers.py
│   └── __init__.py
├── exceptions.py     # Custom exception definitions
└── app.py           # Application factory
```

#### Service Layer Pattern
All business logic must be encapsulated in service classes:

```python
class ArticleService:
    """Handles article-related business logic with performance monitoring."""
    
    def __init__(self, db_session: Any) -> None:
        self._db_session = db_session
        
    def get_latest_articles(self, limit: int = 5) -> List[Dict[str, Any]]:
        """Retrieve latest articles with performance monitoring."""
        start_time = time.time()
        try:
            # Business logic implementation
            return articles
        finally:
            self._check_performance(start_time)
    
    def _check_performance(self, start_time: float) -> None:
        """Monitor and log performance metrics."""
        execution_time_ms = (time.time() - start_time) * 1000
        if execution_time_ms > PERFORMANCE_THRESHOLD_MS:
            logger.warning(f"Operation exceeded performance threshold: {execution_time_ms:.2f}ms")
```

#### SQLAlchemy Best Practices
- **Session Management**: Use dependency injection for database sessions
- **Query Optimization**: Include performance monitoring in all queries
- **Relationship Loading**: Use explicit loading strategies (`lazy`, `eager`)
- **Indexing**: Define indexes for frequently queried columns

```python
class Article(db.Model):
    """Article model with optimized querying and validation."""
    
    __tablename__ = 'articles'
    
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    title = db.Column(db.String(255), nullable=False, index=True)
    published_at = db.Column(db.DateTime, nullable=False, index=True)
    source_url = db.Column(db.Text, nullable=False, unique=True)
    
    __table_args__ = (
        db.Index('idx_articles_published_at', 'published_at', postgresql_using='btree'),
        db.Index('idx_articles_title_search', 'title', postgresql_using='gin'),
    )
```

### Frontend Architecture (Next.js/React)

#### Component Design Philosophy
Follow atomic design principles with performance optimization:

```typescript
// Atomic component with performance optimization
export const ArticleCard: React.FC<ArticleCardProps> = React.memo(({
  article,
  className,
  onClick,
}) => {
  // Memoize expensive computations
  const formattedDate = useMemo(
    () => formatDate(article.published_at),
    [article.published_at]
  );
  
  // Extract event handlers to prevent re-renders
  const handleClick = useCallback(() => {
    onClick?.(article.id);
  }, [onClick, article.id]);
  
  const handleKeyDown = useCallback((event: React.KeyboardEvent) => {
    if (event.key === 'Enter' || event.key === ' ') {
      event.preventDefault();
      handleClick();
    }
  }, [handleClick]);
  
  return (
    <article
      className={`${styles.articleCard} ${className || ''}`}
      role="article"
      aria-label={`Article: ${article.title}`}
      tabIndex={onClick ? 0 : -1}
      onClick={handleClick}
      onKeyDown={handleKeyDown}
    >
      {/* Component JSX */}
    </article>
  );
});
```

#### State Management Strategy
- **Local State**: React hooks for component-specific state
- **Shared State**: React Context for application-wide state
- **Server State**: Custom hooks with caching and invalidation
- **Form State**: Controlled components with validation

#### CSS Modules Implementation
Replace CSS-in-JS with CSS Modules for better performance:

```css
/* ArticleCard.module.css */
.articleCard {
  /* Performance-optimized styles */
  transform: translateZ(0); /* GPU acceleration */
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  will-change: transform; /* Hint browser for optimization */
}

.articleCard:hover {
  transform: translateY(-2px);
}

/* Accessibility-first design */
@media (prefers-reduced-motion: reduce) {
  .articleCard {
    transition: none;
    will-change: auto;
  }
}

@media (prefers-contrast: high) {
  .articleCard {
    border: 2px solid #000;
  }
}
```

### SOLID Principles Application

#### Single Responsibility Principle
Each class and function should have one reason to change:

```python
# ✅ Good: Single responsibility
class ArticleFormatter:
    """Handles article data formatting only."""
    
    def format_articles_list(self, articles: List[Any]) -> List[Dict[str, Any]]:
        return [self._format_single_article(article) for article in articles]

# ❌ Bad: Multiple responsibilities
class ArticleManager:
    """Handles database access, formatting, AND caching."""
    pass
```

#### Open/Closed Principle
Classes should be open for extension, closed for modification:

```python
from abc import ABC, abstractmethod

class ArticleFormatter(ABC):
    """Abstract base for article formatting strategies."""
    
    @abstractmethod
    def format(self, article: Any) -> Dict[str, Any]:
        pass

class ListViewFormatter(ArticleFormatter):
    """Formats articles for list display."""
    
    def format(self, article: Any) -> Dict[str, Any]:
        # List-specific formatting
        pass

class DetailViewFormatter(ArticleFormatter):
    """Formats articles for detailed display."""
    
    def format(self, article: Any) -> Dict[str, Any]:
        # Detail-specific formatting
        pass
```

## Error Handling and Logging

### Custom Exception Hierarchy

```python
class ApplicationError(Exception):
    """Base exception for all application errors."""
    
    def __init__(self, message: str, error_code: str = None) -> None:
        super().__init__(message)
        self.message = message
        self.error_code = error_code
        self.timestamp = datetime.utcnow()

class ValidationError(ApplicationError):
    """Raised when input validation fails (HTTP 400)."""
    pass

class DatabaseError(ApplicationError):
    """Raised when database operations fail (HTTP 503)."""
    pass

class NotFoundError(ApplicationError):
    """Raised when requested resource is not found (HTTP 404)."""
    pass

class AuthenticationError(ApplicationError):
    """Raised when authentication is required or fails (HTTP 401)."""
    pass
```

### Logging Standards

#### Log Levels and Usage
- **DEBUG**: Detailed diagnostic information
- **INFO**: General application flow information
- **WARNING**: Performance issues, recoverable errors
- **ERROR**: Error conditions that need attention
- **CRITICAL**: Serious errors that may abort the program

```python
import logging
import structlog

# Structured logging configuration
logger = structlog.get_logger(__name__)

def get_latest_articles(self) -> List[Dict[str, Any]]:
    logger.info("Fetching latest articles", limit=5)
    
    try:
        start_time = time.time()
        articles = self._fetch_from_database()
        
        execution_time = (time.time() - start_time) * 1000
        logger.info(
            "Articles fetched successfully",
            count=len(articles),
            execution_time_ms=execution_time
        )
        
        if execution_time > PERFORMANCE_THRESHOLD_MS:
            logger.warning(
                "Performance threshold exceeded",
                execution_time_ms=execution_time,
                threshold_ms=PERFORMANCE_THRESHOLD_MS
            )
        
        return articles
        
    except DatabaseError as e:
        logger.error(
            "Database error during article fetch",
            error=str(e),
            error_code=e.error_code
        )
        raise
```

### Frontend Error Boundaries

```typescript
interface ErrorBoundaryState {
  hasError: boolean;
  error?: Error;
}

export class ArticleErrorBoundary extends React.Component<
  React.PropsWithChildren<{}>,
  ErrorBoundaryState
> {
  constructor(props: React.PropsWithChildren<{}>) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log error to monitoring service
    console.error('Article component error:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div role="alert" aria-live="assertive">
          <h2>Something went wrong loading articles</h2>
          <p>Please refresh the page or try again later.</p>
        </div>
      );
    }
    
    return this.props.children;
  }
}
```

## Security Guidelines

### Input Validation and Sanitization

#### Backend Validation
All inputs must be validated at multiple layers:

```python
from marshmallow import Schema, fields, validate

class ArticleRequestSchema(Schema):
    """Validates article creation requests."""
    
    title = fields.Str(
        required=True,
        validate=validate.Length(min=1, max=255),
        error_messages={'required': 'Title is required'}
    )
    
    summary = fields.Str(
        required=True,
        validate=validate.Length(min=10, max=1000),
        error_messages={'required': 'Summary is required'}
    )
    
    source_url = fields.Url(
        required=True,
        error_messages={'required': 'Source URL is required', 'invalid': 'Invalid URL format'}
    )

def create_article(data: Dict[str, Any]) -> Dict[str, Any]:
    """Create article with comprehensive validation."""
    schema = ArticleRequestSchema()
    try:
        validated_data = schema.load(data)
    except ValidationError as e:
        raise ValidationError(f"Invalid input: {e.messages}")
    
    # Additional business logic validation
    if Article.query.filter_by(source_url=validated_data['source_url']).first():
        raise ValidationError("Article with this URL already exists")
    
    return validated_data
```

#### Frontend Validation
Client-side validation for user experience, but never trust client data:

```typescript
export const validateArticleForm = (data: ArticleFormData): ValidationResult => {
  const errors: Record<string, string> = {};
  
  if (!data.title?.trim()) {
    errors.title = 'Title is required';
  } else if (data.title.length > 255) {
    errors.title = 'Title must be less than 255 characters';
  }
  
  if (!data.summary?.trim()) {
    errors.summary = 'Summary is required';
  } else if (data.summary.length < 10) {
    errors.summary = 'Summary must be at least 10 characters';
  }
  
  if (!data.sourceUrl?.trim()) {
    errors.sourceUrl = 'Source URL is required';
  } else if (!isValidURL(data.sourceUrl)) {
    errors.sourceUrl = 'Please enter a valid URL';
  }
  
  return {
    isValid: Object.keys(errors).length === 0,
    errors
  };
};
```

### SQL Injection Prevention
Always use parameterized queries with SQLAlchemy ORM:

```python
# ✅ Safe: Using ORM with parameterized queries
def get_articles_by_title(self, title_pattern: str) -> List[Article]:
    return Article.query.filter(
        Article.title.ilike(f'%{title_pattern}%')
    ).limit(10).all()

# ❌ Dangerous: Raw SQL with string interpolation
def get_articles_unsafe(self, title: str) -> List[Article]:
    query = f"SELECT * FROM articles WHERE title LIKE '%{title}%'"
    return db.session.execute(query).fetchall()
```

### XSS Protection
Prevent cross-site scripting with proper output encoding:

```typescript
// ✅ Safe: Using React's built-in XSS protection
export const ArticleTitle: React.FC<{ title: string }> = ({ title }) => {
  return <h2>{title}</h2>; // React automatically escapes this
};

// ✅ Safe: Sanitizing HTML content
import DOMPurify from 'dompurify';

export const ArticleContent: React.FC<{ content: string }> = ({ content }) => {
  const sanitizedContent = DOMPurify.sanitize(content);
  return <div dangerouslySetInnerHTML={{ __html: sanitizedContent }} />;
};
```

### Configuration Security
Externalize all sensitive configuration:

```python
# ✅ Secure: Environment variables with defaults
DATABASE_HOST: Final[str] = os.getenv('DB_HOST', 'localhost')
DATABASE_PASSWORD: Final[str] = os.getenv('DB_PASSWORD', '')
SECRET_KEY: Final[str] = os.getenv('SECRET_KEY', '')

# Validate required environment variables
def validate_environment() -> None:
    """Ensure all required environment variables are set."""
    required_vars = ['SECRET_KEY', 'DB_PASSWORD']
    missing_vars = [var for var in required_vars if not os.getenv(var)]
    
    if missing_vars:
        raise EnvironmentError(f"Missing required environment variables: {missing_vars}")
```

### CORS Configuration
Configure CORS securely for production:

```python
from flask_cors import CORS

def configure_cors(app: Flask) -> None:
    """Configure CORS with secure defaults."""
    cors_origins = os.getenv('CORS_ORIGINS', 'http://localhost:3000').split(',')
    
    CORS(app, 
         origins=cors_origins,
         methods=['GET', 'POST', 'PUT', 'DELETE'],
         allow_headers=['Content-Type', 'Authorization'],
         supports_credentials=True,
         max_age=3600)
```

## Testing Strategy

### Test-Driven Development (TDD)

#### TDD Cycle Implementation
1. **RED**: Write a failing test that describes the desired functionality
2. **GREEN**: Write the minimal code to make the test pass
3. **REFACTOR**: Improve the code while keeping tests green

```python
# Example TDD cycle for article service
class TestArticleService:
    """Test cases following TDD principles."""
    
    def test_get_latest_articles_returns_five_articles(self):
        """RED: Test that fails initially."""
        service = ArticleService(mock_db_session)
        articles = service.get_latest_articles()
        
        assert len(articles) == 5
        assert all('title' in article for article in articles)
    
    def test_get_latest_articles_performance_requirement(self):
        """Test performance requirements are met."""
        service = ArticleService(mock_db_session)
        
        start_time = time.time()
        articles = service.get_latest_articles()
        execution_time = (time.time() - start_time) * 1000
        
        assert execution_time < 50.0  # Must complete within 50ms
    
    def test_get_latest_articles_handles_database_error(self):
        """Test error handling for database failures."""
        mock_db_session.query.side_effect = SQLAlchemyError("Connection failed")
        service = ArticleService(mock_db_session)
        
        with pytest.raises(DatabaseError) as exc_info:
            service.get_latest_articles()
        
        assert "Database service unavailable" in str(exc_info.value)
```

### Testing Categories and Coverage

#### Unit Tests (90%+ Coverage Required)
Test individual functions and methods in isolation:

```python
@pytest.fixture
def article_service():
    """Provide isolated article service for testing."""
    mock_session = Mock()
    return ArticleService(mock_session)

def test_format_single_article(article_service):
    """Test article formatting logic."""
    raw_article = Mock()
    raw_article.title = "Test Article"
    raw_article.published_at = datetime(2024, 1, 15, 10, 30)
    
    formatted = article_service._format_single_article(raw_article)
    
    assert formatted['title'] == "Test Article"
    assert formatted['published_at'] == "2024-01-15T10:30:00Z"
```

#### Integration Tests
Test API endpoints and database interactions:

```python
def test_get_latest_articles_endpoint(client, db_session):
    """Test complete API endpoint with database."""
    # Setup test data
    articles = [create_test_article() for _ in range(5)]
    db_session.add_all(articles)
    db_session.commit()
    
    response = client.get('/api/articles/latest')
    
    assert response.status_code == 200
    data = response.get_json()
    assert len(data) == 5
    assert all('title' in article for article in data)
```

#### Frontend Testing Strategy
Test React components thoroughly:

```typescript
// Component testing with React Testing Library
describe('ArticleCard Component', () => {
  const mockArticle: Article = {
    id: 1 as ArticleId,
    title: 'Test Article',
    summary_truncated: 'Test summary...',
    published_at: '2024-01-15T10:30:00Z' as ISO8601String,
    source_url: 'https://example.com' as URLString,
  };
  
  describe('UI Display', () => {
    test('renders article information correctly', () => {
      render(<ArticleCard article={mockArticle} />);
      
      expect(screen.getByText('Test Article')).toBeInTheDocument();
      expect(screen.getByText('Test summary...')).toBeInTheDocument();
    });
  });
  
  describe('Accessibility', () => {
    test('has proper ARIA attributes', () => {
      render(<ArticleCard article={mockArticle} />);
      
      const article = screen.getByRole('article');
      expect(article).toHaveAttribute('aria-label', 'Article: Test Article');
    });
    
    test('supports keyboard navigation', () => {
      const mockClick = jest.fn();
      render(<ArticleCard article={mockArticle} onClick={mockClick} />);
      
      const article = screen.getByRole('article');
      fireEvent.keyDown(article, { key: 'Enter' });
      
      expect(mockClick).toHaveBeenCalledWith(1);
    });
  });
  
  describe('Performance', () => {
    test('renders within performance budget (16ms for 60fps)', () => {
      const startTime = performance.now();
      render(<ArticleCard article={mockArticle} />);
      const renderTime = performance.now() - startTime;
      
      expect(renderTime).toBeLessThan(16);
    });
    
    test('does not re-render when props are unchanged', () => {
      const { rerender } = render(<ArticleCard article={mockArticle} />);
      const firstRender = screen.getByRole('article');
      
      rerender(<ArticleCard article={mockArticle} />);
      const secondRender = screen.getByRole('article');
      
      expect(firstRender).toBe(secondRender); // Same DOM node (memoized)
    });
  });
});
```

### Test Naming Conventions
- **Test classes**: `Test{ComponentName}` (e.g., `TestArticleService`)
- **Test methods**: `test_{what_it_tests}_{expected_outcome}` (e.g., `test_get_latest_articles_returns_five_articles`)
- **Test files**: `test_{module_name}.py` or `{Component}.test.tsx`

### Mock and Stub Guidelines
- **Mock external dependencies**: Database, HTTP requests, file system
- **Stub return values**: For predictable test scenarios
- **Never mock the system under test**: Only mock its dependencies

```python
@patch('app.services.article_service.time')
def test_performance_monitoring(mock_time, article_service):
    """Test that performance monitoring works correctly."""
    mock_time.time.side_effect = [0.0, 0.060]  # 60ms execution
    
    with patch.object(article_service, 'logger') as mock_logger:
        article_service.get_latest_articles()
        
        mock_logger.warning.assert_called_once()
        assert "exceeded performance threshold" in mock_logger.warning.call_args[0][0]
```

## Git and Branch Strategy

### Branch Naming Conventions
- **Feature branches**: `feat/description-of-feature`
- **Bug fixes**: `fix/description-of-bug`
- **Refactoring**: `refactor/description-of-refactor`
- **Documentation**: `docs/description-of-docs`
- **Chores**: `chore/description-of-chore`

### Commit Message Format (Conventional Commits)
```
type(scope): description

[optional body]

[optional footer(s)]
```

#### Commit Types
- **feat**: New feature for the user
- **fix**: Bug fix for the user
- **docs**: Documentation only changes
- **style**: Formatting, missing semicolons, etc.
- **refactor**: Code change that neither fixes a bug nor adds a feature
- **perf**: Code change that improves performance
- **test**: Adding missing tests or correcting existing tests
- **chore**: Changes to build process or auxiliary tools

#### Examples
```
feat(articles): add pagination support to latest articles API

- Implement limit and offset parameters
- Add pagination metadata to response
- Update tests to cover new functionality

Closes #123
```

```
fix(frontend): resolve memory leak in ArticleCard component

- Add cleanup in useEffect hook
- Remove event listeners on unmount
- Add performance test to prevent regression

Performance impact: Reduces memory usage by 15%
```

### Pull Request Guidelines

#### PR Template
```markdown
## Description
Brief description of the changes and why they were made.

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update

## Testing
- [ ] All existing tests pass
- [ ] New tests added for new functionality
- [ ] Code coverage maintains 90%+ threshold
- [ ] Performance tests pass (< 50ms backend, < 16ms frontend)

## Checklist
- [ ] Code follows the style guidelines in CLAUDE.md
- [ ] Self-review completed
- [ ] Code is properly commented
- [ ] Documentation updated if needed
- [ ] No new security vulnerabilities introduced
```

#### Review Process
1. **Automated checks**: All CI/CD checks must pass
2. **Code review**: At least one team member must review
3. **Testing**: Manual testing for UI changes
4. **Performance**: Verify performance requirements are met
5. **Security**: Review for security implications

## Dependency Management

### Python (Poetry)

#### pyproject.toml Structure
```toml
[tool.poetry]
name = "llm-topix-backend"
version = "0.1.0"
description = "Backend API for LLM TOPIX platform"

[tool.poetry.dependencies]
python = "^3.11"
flask = "^2.3.0"
sqlalchemy = "^2.0.0"
marshmallow = "^3.20.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.4.0"
black = "^23.7.0"
flake8 = "^6.0.0"
mypy = "^1.5.0"

[tool.poetry.group.test.dependencies]
pytest-cov = "^4.1.0"
factory-boy = "^3.3.0"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

#### Dependency Update Process
1. **Regular updates**: Monthly dependency review
2. **Security updates**: Immediate response to security advisories
3. **Testing**: Full test suite must pass with updates
4. **Documentation**: Update if API changes affect our code

### Node.js (npm)

#### package.json Structure
```json
{
  "name": "llm-topix-frontend",
  "version": "0.1.0",
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.2.0",
    "typescript": "^5.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "eslint": "^8.48.0",
    "prettier": "^3.0.0",
    "jest": "^29.6.0"
  },
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "test": "jest",
    "lint": "eslint . --ext .ts,.tsx",
    "type-check": "tsc --noEmit"
  }
}
```

#### Library Addition Process
1. **Evaluation**: Assess necessity, alternatives, and maintenance status
2. **Security**: Check for known vulnerabilities
3. **Size impact**: Consider bundle size implications
4. **Testing**: Ensure compatibility with existing code
5. **Documentation**: Update relevant documentation

### Development Environment Requirements

#### Required Tools
- **Python 3.11+**: Backend development
- **Node.js 18+**: Frontend development
- **Docker & Docker Compose**: Local development environment
- **Poetry**: Python dependency management
- **npm/yarn**: Node.js dependency management

#### IDE Configuration
Recommended VS Code extensions:
- Python (Microsoft)
- TypeScript and JavaScript Language Features
- ESLint
- Prettier
- SQLAlchemy Extension Pack

#### Pre-commit Hooks
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.7.0
    hooks:
      - id: black
  
  - repo: https://github.com/pycqa/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
  
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.5.0
    hooks:
      - id: mypy
```

---

## Conclusion

This development constitution represents the culmination of our successful TDD implementation and refactoring phases. Every principle, pattern, and practice documented here has been tested in real-world scenarios and proven to enhance code quality, maintainability, security, and performance.

Adherence to these guidelines is not optional—it is a fundamental requirement for maintaining the high standards that define the LLM TOPIX platform. Through consistent application of these principles, we ensure that our codebase remains robust, scalable, and ready for future enhancements.

Remember: These standards evolve with our understanding and the technology landscape. Regular reviews and updates to this constitution ensure we continue to follow industry best practices while maintaining the stability and reliability our users depend on.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Be114)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Be114)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
