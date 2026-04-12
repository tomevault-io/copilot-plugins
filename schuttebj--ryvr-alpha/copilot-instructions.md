## ryvr-alpha

> This rule defines the development workflow and processes for the Ryvr AI Platform. Use this rule when onboarding new developers, planning development tasks, or implementing the project's development pipeline.

# Development Workflow

> [!AGENT_REQUEST]
> This rule defines the development workflow and processes for the Ryvr AI Platform. Use this rule when onboarding new developers, planning development tasks, or implementing the project's development pipeline.

## Development Environments

### Local Development
- Use Docker-based local development environment
- Configure environment to match production server settings
- Create `.env` files for environment-specific configurations
- Implement hot reloading for frontend development
- Set up local database with sample data

### Staging Environment
- Mirror production environment settings
- Deploy from staging branch
- Implement automated testing before deployment
- Require manual QA approval before promotion to production
- Reset staging database periodically with sanitized production data

### Production Environment
- Deploy from main/master branch only
- Implement deployment window protocols
- Require code review approval before merging to main
- Configure backup before any deployment
- Monitor closely after deployment

## Version Control

### Branch Strategy
- `main` - Production-ready code
- `staging` - Pre-production testing
- `develop` - Integration branch for features
- `feature/*` - Individual feature development
- `bugfix/*` - Bug fixes
- `hotfix/*` - Emergency production fixes

### Commit Guidelines
- Use conventional commit messages (feat, fix, docs, style, refactor, test, chore)
- Include ticket number in commit message (e.g., `feat(seo): implement keyword research #123`)
- Keep commits focused on single logical changes
- Do not commit sensitive data or credentials
- Maintain a clean commit history (rebase when appropriate)

## Code Review Process

### Pull Request Requirements
- Include description of changes
- Reference related tickets
- Include testing instructions
- Pass all automated checks
- Address all code quality issues

### Review Checklist
- Code follows architectural guidelines
- Adheres to coding standards
- Proper test coverage
- Documentation updated
- Security considerations addressed
- Performance impact assessed

## Testing Approach

### Test Levels
- Unit tests for individual components
- Integration tests for API interactions
- End-to-end tests for critical workflows
- Performance tests for high-load operations
- Security tests for sensitive functions

### Continuous Integration
- Run tests on each pull request
- Generate code coverage reports
- Perform static code analysis
- Check for security vulnerabilities
- Validate database migrations

## Documentation Requirements

### Code Documentation
- Inline comments for complex logic
- PHPDoc/JSDoc for all functions and classes
- README files for each major component
- Architecture diagrams for system interactions
- API documentation for all endpoints

### User Documentation
- Admin user guides
- Client user guides
- API documentation for external consumers
- Troubleshooting guides
- Video tutorials for key features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/schuttebj)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/schuttebj)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
