## art-decor-ai

> - Use functional components only

# Cursor Rules for Art.Decor.AI

## Code Style

### TypeScript/React (Frontend)
- Use functional components only
- Strict TypeScript - no `any` types
- Follow atomic design: atoms, molecules, organisms
- Prefix event handlers with `handle` (e.g., handleClick)
- Extract complex logic to custom hooks
- Use early returns for readability

### Python (Backend/AI)
- Follow PEP 8 style guide
- Use type hints for all functions
- Async/await for I/O operations
- Comprehensive docstrings
- Error handling with try/except and logging

## Project Structure

### Frontend (`frontend/src/`)
```
app/          → Next.js pages
components/   → UI components (atomic design)
  atoms/      → Basic components (Button, Input)
  molecules/  → Composite components (Card, Form)
  organisms/  → Complex sections (Header, Grid)
hooks/        → Custom React hooks
services/     → API service layer
types/        → TypeScript interfaces
utils/        → Helper functions
```

### Backend (`backend/app/`)
```
controllers/  → Request handlers
services/     → Business logic
models/       → Pydantic models
routes/       → API route definitions
utils/        → Utility functions
```

### AI Models (`ai-model/`)
```
agents/       → LangChain agents
chains/       → LangChain chains
graphs/       → LangGraph workflows
models/       → Model wrappers (YOLO, CLIP, etc.)
prompts/      → LLM prompt templates
utils/        → ML utilities
```

## Naming Conventions

- **Files**: `snake_case.py` (Python), `PascalCase.tsx` (React components)
- **Variables**: `camelCase` (TS), `snake_case` (Python)
- **Components**: `PascalCase`
- **Functions**: `camelCase` (TS), `snake_case` (Python)
- **Constants**: `UPPER_SNAKE_CASE`

## Import Order

### TypeScript
1. React/Next.js imports
2. Third-party libraries
3. Internal imports (@/components, @/hooks, etc.)
4. Relative imports
5. Types

### Python
1. Standard library
2. Third-party packages
3. Local application imports

## Best Practices

- **DRY**: Don't repeat yourself
- **SOLID**: Follow SOLID principles
- **Testing**: Write tests for critical paths
- **Logging**: Use proper logging levels
- **Error Handling**: Always handle errors gracefully
- **Documentation**: Document complex logic
- **Performance**: Optimize where it matters
- **Security**: Never commit secrets or API keys

## AI/ML Specific

- Use LangChain for LLM interactions
- Use LangGraph for multi-step workflows
- Cache embeddings to avoid recomputation
- Batch process when possible
- Monitor API costs
- Log all AI interactions for debugging

## Git Workflow

- Commit messages: Use conventional commits
  - `feat:` new feature
  - `fix:` bug fix
  - `docs:` documentation
  - `refactor:` code refactoring
  - `test:` adding tests
- Branch naming: `feature/description` or `fix/description`
- Keep commits atomic and focused

## Testing

- Unit tests for utilities and services
- Integration tests for API endpoints
- E2E tests for critical user flows
- Mock external API calls in tests

## Performance

- Lazy load images and components
- Use React.memo for expensive components
- Debounce search inputs
- Paginate large lists
- Cache API responses appropriately

## Accessibility

- Use semantic HTML
- Add ARIA labels where needed
- Ensure keyboard navigation works
- Test with screen readers
- Maintain good color contrast

---
> Source: [zeeshanarif513/art-decor-ai](https://github.com/zeeshanarif513/art-decor-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
