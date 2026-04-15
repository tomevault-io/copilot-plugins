## lightdom

> This is a blockchain-based DOM optimization platform that includes:

# LightDom Enterprise Coding Rules - Enhanced Edition

## Project Overview
This is a blockchain-based DOM optimization platform that includes:
- React/TypeScript frontend with Vite
- Solidity smart contracts for tokenization and optimization proof
- Express.js API server
- Web crawler and optimization engine
- PostgreSQL database integration

## 🚨 CRITICAL RULE #1: ALWAYS SEARCH EXISTING CODE FIRST

> **This is the PRIMARY rule that overrides all others. Never skip this step.**

**MANDATORY PROCESS - BEFORE writing ANY new code:**

### Step 1: Comprehensive Code Search (REQUIRED)

```bash
# 1. Search by functionality keywords
grep -r "relevant.*keyword" services/ src/ scripts/ api/

# 2. Find files by name pattern
find . -name "*relevant*" -type f -not -path "*/node_modules/*"

# 3. Search for class/function definitions
grep -r "class.*Feature\|function.*feature" . --include="*.ts" --include="*.js"

# 4. Check for imports and usage
grep -r "import.*Feature" . --include="*.ts" --include="*.js"

# 5. Use ripgrep for faster searches (if available)
rg "search term" --type ts --type js
```

### Step 2: Review Existing Services (REQUIRED)

Search these directories in order:
1. `services/` - 100+ service files, many may solve your problem
2. `api/` - API route handlers and middleware
3. `src/services/` - TypeScript services and business logic
4. `src/components/` - React components and UI elements
5. `scripts/` - Automation scripts and utilities
6. `utils/` - Utility functions and helpers
7. `src/core/` - Core business logic
8. `src/hooks/` - React hooks for state and logic

### Step 3: Check for Disconnected Code (REQUIRED)

Many features exist but aren't fully integrated. Look for:
- Unused imports
- Commented-out code that works
- Services in `services/` not referenced elsewhere
- Components not imported in any routes
- Utilities not exported in index files

### Step 4: Search Pattern Examples (REQUIRED)

**ALWAYS use these search patterns before creating new code:**

| Creating... | Must Search For... | Example Commands |
|-------------|-------------------|------------------|
| SEO extractor | `seo`, `attribute`, `extract`, `meta`, `og:` | `grep -r "seo" services/ src/` |
| Animation mining | `animation`, `styleguide`, `mine`, `css` | `find . -name "*animation*"` |
| TensorFlow model | `tensorflow`, `model`, `train`, `ml`, `ai` | `grep -r "tensorflow" .` |
| Database ops | `db`, `query`, `pool`, `connection`, `sql` | `grep -r "pool\\.query" .` |
| API endpoint | `router`, `endpoint`, `api`, `route`, `express` | `grep -r "router\\." api/` |
| UI component | `component`, `react`, `dashboard`, `ui` | `find src/components/ -name "*Dashboard*"` |
| Blockchain | `blockchain`, `contract`, `ethereum`, `web3` | `grep -r "ethers" contracts/` |
| Auth system | `auth`, `login`, `token`, `jwt`, `session` | `grep -r "authenticate" api/` |
| Crawler | `crawl`, `scrape`, `puppeteer`, `selenium` | `grep -r "puppeteer" services/` |
| Testing | `test`, `spec`, `mock`, `fixture` | `find test/ -name "*test*"` |

### Step 5: Decision Point (REQUIRED)

**Only proceed to write new code if ALL of these are true:**
- ✅ Completed comprehensive search (Steps 1-4)
- ✅ No existing solution found
- ✅ No existing code can be adapted/extended
- ✅ Confirmed with team/docs that feature doesn't exist
- ✅ Creating truly new functionality

**If ANY existing code is found:**
- ✅ Reuse it directly, or
- ✅ Extend it with new functionality, or
- ✅ Refactor it to be more generic, or
- ✅ Connect it to the rest of the system

### Rationale

We have **substantial existing code** (~100+ services, 200+ files, 50+ components) that may not be fully connected or documented. 

**Benefits of reusing existing code:**
1. 🚀 **Faster development** - No need to write from scratch
2. 🛡️ **Battle-tested** - Existing code is proven to work
3. 🔧 **Maintainable** - Fewer places to update
4. 📚 **Consistent** - Follows established patterns
5. 🐛 **Fewer bugs** - Existing code is already debugged
6. 📊 **Better architecture** - Improves code organization

**Costs of creating new code unnecessarily:**
1. ❌ **Duplication** - Multiple implementations of same feature
2. ❌ **Technical debt** - More code to maintain
3. ❌ **Inconsistency** - Different patterns for same problem
4. ❌ **Higher bug risk** - New code needs debugging
5. ❌ **Wasted effort** - Solving already-solved problems
6. ❌ **Confusion** - Which implementation to use?

### ⚠️ Violation Consequences

**If you create new code without searching first:**
- PR will be rejected with "search-first-violation" label
- Must demonstrate comprehensive search was performed
- Must justify why existing code cannot be used
- May be asked to refactor to use existing code

## Code Quality Standards

### TypeScript/JavaScript
- Use TypeScript for all new code with strict type checking enabled
- Prefer `const` over `let`, avoid `var` completely
- Use meaningful variable and function names (descriptive, not abbreviated)
- Implement proper error handling with try-catch blocks and custom error types
- Use async/await over promises where possible, handle promise rejections properly
- Follow ESLint and Prettier configurations (create if missing)
- Write self-documenting code with JSDoc comments for complex functions
- Use proper TypeScript interfaces and types, avoid `any` type
- Implement proper null/undefined checking with optional chaining and nullish coalescing

### Functionality Testing Requirements
- ALL code must be tested for actual functionality, not just structure
- Test that services can actually start and connect
- Verify that APIs return real data, not mock responses
- Ensure database connections work in practice
- Test that Electron app can actually launch
- Verify blockchain integration is real, not mocked
- Check that all endpoints return actual data
- Test that authentication actually works
- Verify that mining functionality is real
- Ensure all features work end-to-end

### React/UI Development
- Use functional components with hooks exclusively
- Implement proper prop types and interfaces with strict typing
- Use CSS modules or styled-components for styling (avoid inline styles)
- Follow accessibility guidelines (WCAG 2.1 AA compliance)
- Implement responsive design principles (mobile-first approach)
- Use proper state management (Context API or Redux when needed)
- Avoid prop drilling - use composition patterns and custom hooks
- Implement proper error boundaries for component error handling
- Use React.memo and useMemo/useCallback for performance optimization
- Follow React 18+ patterns (concurrent features, Suspense, etc.)

### Solidity Smart Contracts
- Follow OpenZeppelin standards and best practices
- Implement proper access control with modifiers and role-based access
- Use events for important state changes (indexed parameters for efficiency)
- Implement reentrancy guards where necessary (nonReentrant modifier)
- Use Solidity 0.8+ built-in overflow protection (no SafeMath needed)
- Write comprehensive NatSpec documentation for all public functions
- Implement proper error handling with custom errors (Solidity 0.8.4+)
- Use proper gas optimization patterns (packed structs, efficient loops)
- Implement proper upgrade patterns (proxy contracts) for critical contracts
- Never use `tx.origin` for authorization, use `msg.sender`

### API Development
- Use RESTful API design principles with proper HTTP methods
- Implement proper HTTP status codes (200, 201, 400, 401, 403, 404, 500)
- Use middleware for authentication, validation, and error handling
- Implement rate limiting and CORS properly with configurable origins
- Use environment variables for all configuration (no hardcoded values)
- Implement proper logging and error handling with structured logging
- Follow OpenAPI/Swagger documentation standards
- Use proper request/response validation with schema validation
- Implement proper API versioning strategy
- Use proper authentication mechanisms (JWT, API keys, OAuth)

## Security Requirements

### General Security
- Never commit secrets, API keys, or private keys to version control
- Use environment variables for all sensitive configuration
- Implement proper input validation and sanitization (XSS, injection prevention)
- Use parameterized queries to prevent SQL injection
- Implement proper authentication and authorization (RBAC)
- Follow OWASP security guidelines and top 10 vulnerabilities
- Regular security audits and dependency updates
- Implement proper CORS configuration (not wildcard in production)
- Use HTTPS in production with proper SSL/TLS configuration
- Implement proper session management and CSRF protection

### Blockchain Security
- Use established libraries like OpenZeppelin for security patterns
- Implement proper access controls and multi-signature requirements
- Test all smart contracts thoroughly before deployment
- Use formal verification where possible for critical contracts
- Implement proper upgrade patterns (proxy contracts) for critical functionality
- Never use `tx.origin` for authorization, always use `msg.sender`
- Be aware of reentrancy vulnerabilities and use proper guards
- Implement proper gas optimization to prevent DoS attacks
- Use proper random number generation (Chainlink VRF for production)
- Implement proper event emission for off-chain monitoring

### Database Security
- Use connection pooling and prepared statements
- Implement proper database access controls and user permissions
- Encrypt sensitive data at rest and in transit
- Use proper backup and recovery procedures with encryption
- Implement database monitoring and logging
- Use proper database indexing for performance and security
- Implement proper data retention policies
- Use proper database migration strategies with rollback plans

### Database Migration Standards
**CRITICAL: Follow these rules for ALL database migrations**

See [DATABASE_MIGRATION_RULES.md](DATABASE_MIGRATION_RULES.md) for complete guidelines.

#### Migration File Requirements
- **Naming**: `database/migrations/YYYYMMDD_descriptive_name.sql`
- **Location**: Place all migrations in `database/migrations/` directory
- **Idempotency**: ALL scripts MUST be idempotent (safe to run multiple times)

#### SQL Script Standards
```sql
-- ✅ CORRECT: Use IF NOT EXISTS
CREATE TABLE IF NOT EXISTS my_table (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX IF NOT EXISTS idx_name ON my_table(name);

-- ✅ CORRECT: Foreign keys with actions
FOREIGN KEY (parent_id) REFERENCES parent(id) ON DELETE CASCADE;

-- ✅ CORRECT: PostgreSQL syntax for indexes
CREATE INDEX idx_url ON my_table(url);

-- ❌ INCORRECT: MySQL syntax
CREATE INDEX idx_url ON my_table(url(255));
```

#### Migration Checklist
Before committing a migration, verify:
- [ ] File follows naming convention `YYYYMMDD_descriptive_name.sql`
- [ ] Uses `CREATE TABLE IF NOT EXISTS` (or justified DROP TABLE CASCADE)
- [ ] Uses `CREATE INDEX IF NOT EXISTS`
- [ ] Foreign keys have ON DELETE/UPDATE clauses
- [ ] Timestamps have DEFAULT CURRENT_TIMESTAMP
- [ ] Tables have updated_at triggers
- [ ] Tables/columns have COMMENT documentation
- [ ] Tested on clean database
- [ ] Tested for idempotency (run twice, no errors)
- [ ] No MySQL-specific syntax
- [ ] Proper indexes on foreign keys and query columns

#### Documentation Requirements
See [DATABASE_TABLES_ANALYSIS.md](DATABASE_TABLES_ANALYSIS.md) for:
- List of all existing tables (400+ tables documented)
- Missing/recommended tables for crawler system
- Migration status and priorities
- Table relationship diagrams

## Performance Standards

### Frontend Performance
- Implement code splitting and lazy loading for routes and components
- Optimize bundle size and use tree shaking effectively
- Use proper caching strategies (browser cache, CDN, service workers)
- Optimize images and assets (WebP, lazy loading, responsive images)
- Implement proper loading states and error boundaries
- Monitor Core Web Vitals (LCP, FID, CLS) and maintain good scores
- Use proper state management to avoid unnecessary re-renders
- Implement proper memory management and cleanup in useEffect
- Use proper virtualization for large lists and data sets

### Backend Performance
- Implement proper caching (Redis/Memcached) for frequently accessed data
- Use database indexing appropriately and monitor query performance
- Implement connection pooling for database connections
- Use async processing for heavy operations (queues, workers)
- Monitor and profile application performance regularly
- Implement proper rate limiting to prevent abuse
- Use proper compression (gzip, brotli) for responses
- Implement proper database query optimization and monitoring
- Use proper load balancing and horizontal scaling

### Blockchain Performance
- Optimize gas usage in smart contracts (efficient data structures)
- Use events instead of storage for non-critical data
- Implement proper batch operations to reduce transaction costs
- Consider layer 2 solutions for high-frequency operations
- Use proper gas estimation and limit handling
- Implement proper transaction batching and queuing
- Monitor blockchain performance and gas costs
- Use proper contract optimization techniques

## Testing Requirements

### Unit Testing
- Maintain minimum 80% code coverage (branches, functions, lines, statements)
- Write tests for all business logic and utility functions
- Use proper mocking and stubbing for external dependencies
- Test edge cases and error conditions thoroughly
- Use Jest for JavaScript/TypeScript testing with proper configuration
- Use Hardhat for Solidity testing with comprehensive test coverage
- Implement proper test data management and cleanup
- Use proper test naming conventions and organization
- Implement proper test isolation and parallel execution

### Integration Testing
- Test API endpoints thoroughly with various scenarios
- Test database interactions and data integrity
- Test smart contract interactions and state changes
- Use proper test data management and cleanup
- Implement end-to-end testing for critical user flows
- Test error handling and edge cases in integration scenarios
- Use proper test environment setup and teardown
- Implement proper test data seeding and cleanup

### Smart Contract Testing
- Test all functions and edge cases with comprehensive scenarios
- Test access control and permissions thoroughly
- Test reentrancy and other security vulnerabilities
- Use fuzzing and property-based testing for critical functions
- Test upgrade scenarios and migration paths
- Test gas optimization and cost scenarios
- Test error conditions and exception handling
- Implement proper test coverage for all contract functions

## Documentation Standards

### Code Documentation
- Write comprehensive README files with setup and usage instructions
- Document all public APIs and interfaces with examples
- Use JSDoc for JavaScript/TypeScript functions with proper annotations
- Use NatSpec for Solidity contracts with comprehensive documentation
- Document deployment procedures and configuration requirements
- Maintain changelog and release notes with proper versioning
- Document environment variables and configuration options
- Document API endpoints with request/response examples

### Architecture Documentation
- Document system architecture and design decisions
- Create sequence diagrams for complex flows and interactions
- Document database schema and relationships
- Document smart contract architecture and interactions
- Maintain API documentation with OpenAPI/Swagger
- Document deployment architecture and infrastructure
- Document monitoring and observability setup
- Document security architecture and measures

## Enterprise Workflow Rules

### Development Workflow
- Follow GitFlow branching strategy with main, develop, feature, release, and hotfix branches
- All development must happen on feature branches, never directly on main/develop
- Use feature flags for gradual rollouts and A/B testing
- Implement trunk-based development for high-velocity teams
- Use short-lived branches (max 2-3 days) to reduce merge conflicts
- Implement automated branch protection rules
- Require linear history for main branch (no merge commits)
- Use proper commit message conventions (conventional commits)

### Code Review Workflow
- All code must be reviewed by at least 2 senior developers
- Use automated code review tools (SonarQube, CodeClimate, etc.)
- Implement review assignment rotation to prevent bottlenecks
- Require approval from code owners for critical files
- Use review templates with mandatory checklists
- Implement review time limits (max 24 hours for normal changes)
- Escalate reviews that exceed time limits
- Track review metrics and team performance

### Pull Request Workflow
- Use descriptive PR titles following conventional commit format
- Include comprehensive PR descriptions with:
  - What changes were made and why
  - Testing instructions and test coverage
  - Screenshots/videos for UI changes
  - Breaking changes documentation
  - Performance impact analysis
  - Security considerations
- Link PRs to relevant issues/tickets
- Use PR templates for consistency
- Implement automated PR validation
- Require all CI checks to pass before merge
- Use draft PRs for work-in-progress features

### Testing Workflow
- Implement Test-Driven Development (TDD) for critical business logic
- Use Behavior-Driven Development (BDD) for user-facing features
- Run tests in parallel across multiple environments
- Implement test data management and cleanup
- Use contract testing for microservices
- Implement visual regression testing for UI changes
- Use mutation testing to verify test quality
- Implement test result reporting and analytics

### Deployment Workflow
- Use blue-green or canary deployment strategies
- Implement automated rollback capabilities
- Use feature toggles for safe deployments
- Implement database migration strategies
- Use infrastructure as code for environment consistency
- Implement deployment approval workflows for production
- Use deployment pipelines with multiple stages
- Implement deployment monitoring and alerting

### Release Management
- Use semantic versioning (SemVer) for all releases
- Implement automated changelog generation
- Use release branches for version management
- Implement release notes automation
- Use release gates and quality checkpoints
- Implement rollback procedures for failed releases
- Use release calendars and communication plans
- Implement post-release monitoring and validation

### Quality Assurance Workflow
- Implement continuous integration with automated builds
- Use automated code quality gates
- Implement security scanning in CI/CD pipeline
- Use performance testing in staging environments
- Implement accessibility testing automation
- Use cross-browser and cross-device testing
- Implement load testing for critical paths
- Use chaos engineering for resilience testing

### Monitoring and Observability Workflow
- Implement comprehensive logging strategy with structured logging
- Use proper log levels and log aggregation
- Implement distributed tracing across services
- Use metrics collection and dashboards
- Implement alerting with proper escalation
- Use error tracking and crash reporting
- Implement user analytics and behavior tracking
- Use performance monitoring and optimization

### Security Workflow
- Implement security scanning in CI/CD pipeline
- Use dependency vulnerability scanning
- Implement secret scanning and detection
- Use static application security testing (SAST)
- Implement dynamic application security testing (DAST)
- Use interactive application security testing (IAST)
- Implement security code review checklists
- Use threat modeling for new features

### Documentation Workflow
- Use documentation as code approach
- Implement automated documentation generation
- Use version-controlled documentation
- Implement documentation review process
- Use living documentation that stays current
- Implement API documentation automation
- Use architectural decision records (ADRs)
- Implement knowledge sharing sessions

### Incident Response Workflow
- Implement incident response playbooks
- Use incident management tools (PagerDuty, etc.)
- Implement escalation procedures
- Use post-incident review process
- Implement incident communication plans
- Use incident metrics and reporting
- Implement preventive measures tracking
- Use incident simulation and training

### Change Management Workflow
- Implement change request process
- Use change advisory board (CAB) for major changes
- Implement change impact analysis
- Use change approval workflows
- Implement change communication plans
- Use change rollback procedures
- Implement change metrics and reporting
- Use change risk assessment

## Git and Version Control

### Commit Standards
- Use conventional commit format: `type(scope): description`
- Write clear and descriptive commit messages
- Keep commits atomic and focused on single changes
- Use proper branching strategy (feature/develop/main)
- Require pull request reviews for all changes
- Use commit message templates
- Implement commit message validation
- Use commit signing for security

### Branch Naming
- `feature/description` for new features
- `bugfix/description` for bug fixes
- `hotfix/description` for critical fixes
- `refactor/description` for code refactoring
- `chore/description` for maintenance tasks
- `docs/description` for documentation changes
- `test/description` for test-related changes
- `perf/description` for performance improvements

### Branch Protection Rules
- Require pull request reviews before merging
- Require status checks to pass before merging
- Require branches to be up to date before merging
- Restrict pushes to main branch
- Require linear history
- Require signed commits
- Require conversation resolution before merging
- Implement branch naming conventions enforcement

## Deployment and DevOps

### Environment Management
- Use separate environments (dev/staging/prod)
- Implement proper environment variable management
- Use infrastructure as code (Docker, Kubernetes)
- Implement proper CI/CD pipelines
- Use proper secret management (AWS Secrets Manager, etc.)

### Monitoring and Logging
- Implement comprehensive logging with structured format
- Use proper log levels (DEBUG, INFO, WARN, ERROR, FATAL)
- Implement proper error tracking and alerting
- Monitor application performance and resource usage
- Set up alerts for critical issues and thresholds
- Use proper log aggregation and analysis tools

## Blockchain-Specific Guidelines

### Smart Contract Development
- Use established patterns and libraries (OpenZeppelin)
- Implement proper upgrade mechanisms (proxy patterns)
- Use events for off-chain integration and monitoring
- Implement proper gas optimization techniques
- Follow security best practices and audit requirements
- Use proper testing frameworks (Hardhat, Foundry)
- Implement proper deployment scripts and verification

### Token Standards
- Follow ERC-20/ERC-721/ERC-1155 standards strictly
- Implement proper transfer restrictions and compliance
- Use established token libraries and patterns
- Implement proper metadata standards
- Follow regulatory requirements for token implementations
- Implement proper token economics and distribution

### Testing and Deployment
- Use testnets for development and testing
- Implement proper deployment scripts with verification
- Use multi-signature wallets for production deployments
- Implement proper verification processes for contracts
- Use proper gas estimation and optimization
- Implement proper monitoring and alerting for contracts

## Code Review Guidelines

### Review Checklist
- Security vulnerabilities and best practices
- Performance implications and optimizations
- Code quality and maintainability
- Test coverage and quality
- Documentation completeness and accuracy
- Architecture compliance and patterns
- Error handling and edge cases
- Accessibility and usability considerations

### Review Process
- All code must be reviewed before merging
- Use automated tools for code quality checks
- Focus on security and performance implications
- Provide constructive feedback and suggestions
- Document review decisions and rationale
- Follow consistent review processes and standards

## Error Handling

### General Error Handling
- Use proper error types and messages
- Implement proper error logging and tracking
- Provide meaningful error messages to users
- Implement proper error recovery mechanisms
- Use proper HTTP status codes
- Implement proper error monitoring and alerting
- Use proper error boundaries and fallbacks

### Blockchain Error Handling
- Handle transaction failures gracefully
- Implement proper retry mechanisms with exponential backoff
- Use proper error events and logging
- Handle network issues and timeouts
- Implement proper gas estimation and handling
- Use proper error recovery and rollback mechanisms

## Performance Monitoring

### Key Metrics
- Response times and throughput
- Error rates and availability
- Database performance and query times
- Smart contract gas usage and costs
- Frontend performance metrics (Core Web Vitals)
- Resource usage and scaling metrics
- User experience metrics

### Monitoring Tools
- Application performance monitoring (APM)
- Database monitoring and query analysis
- Blockchain monitoring tools and gas tracking
- Frontend performance monitoring
- Error tracking and logging
- Infrastructure monitoring and alerting

## Compliance and Governance

### Data Protection
- Follow GDPR and CCPA requirements
- Implement proper data retention policies
- Use proper data encryption and protection
- Implement proper access controls and audit trails
- Regular compliance audits and assessments
- Implement proper data privacy and consent management

### Audit Requirements
- Maintain comprehensive audit trails
- Implement proper logging and monitoring
- Regular security assessments and penetration testing
- Code quality audits and reviews
- Performance audits and optimization
- Compliance audits and regulatory requirements

## Emergency Procedures

### Incident Response
- Follow proper incident response procedures
- Implement proper escalation paths and communication
- Maintain incident documentation and post-mortems
- Conduct post-incident reviews and improvements
- Implement proper rollback procedures
- Maintain emergency contact lists and procedures

### Security Incidents
- Follow security incident response procedures
- Implement proper containment measures
- Notify appropriate stakeholders and authorities
- Document security incidents and lessons learned
- Conduct security post-mortems and improvements
- Implement preventive measures and monitoring

## Continuous Improvement

### Regular Reviews
- Conduct regular code reviews and quality assessments
- Implement continuous integration and delivery
- Regular performance reviews and optimization
- Security assessment updates and improvements
- Technology stack updates and modernization
- Process improvement and optimization

### Learning and Development
- Stay updated with latest technologies and best practices
- Participate in code reviews and knowledge sharing
- Share knowledge and best practices across teams
- Attend relevant conferences and training
- Contribute to open source projects and communities
- Implement continuous learning and development programs

## Project-Specific Guidelines

### LightDom Platform
- Focus on DOM optimization algorithms and efficiency
- Implement efficient web crawling and data extraction
- Use proper data structures for optimization results
- Implement proper caching for optimization results
- Monitor optimization performance metrics
- Implement proper DOM analysis and optimization techniques

### Token Economics
- Implement proper token distribution mechanisms
- Use proper economic models and incentives
- Implement proper governance mechanisms
- Monitor token metrics and usage patterns
- Implement proper token utility and value propositions
- Use proper token economics modeling and analysis

### Integration Points
- Ensure proper API integration and error handling
- Implement proper data synchronization and consistency
- Use proper event handling and messaging
- Implement proper error handling across systems
- Use proper monitoring and observability across integrations
- Implement proper testing and validation for integrations

## Tools and Technologies

### Required Tools
- TypeScript/JavaScript (ES6+) with strict configuration
- React 18+ with hooks and modern patterns
- Vite for build tooling and development
- Solidity 0.8+ with OpenZeppelin libraries
- Hardhat for smart contract development and testing
- Express.js for API development
- PostgreSQL for database operations
- Docker for containerization and deployment

### Recommended Tools
- ESLint and Prettier for code formatting and linting
- Jest and Vitest for testing
- Storybook for component development
- Sentry for error tracking and monitoring
- Grafana for monitoring and visualization
- Redis for caching and session management
- Nginx for load balancing and reverse proxy
- Kubernetes for orchestration and scaling

## Quality Gates

### Pre-commit
- Code formatting and linting (ESLint, Prettier)
- Unit test execution with coverage requirements
- Security scanning and vulnerability checks
- Type checking and validation
- Pre-commit hooks validation
- Dependency vulnerability checks
- Code complexity analysis
- **FUNCTIONALITY TESTING** - Run `npm run compliance:check` to test actual functionality

### Functionality Validation (CRITICAL)
- Run enhanced validation: `npm run compliance:check`
- Test Electron app can actually start
- Verify API server returns real data (not mock)
- Check database connectivity works
- Test blockchain integration is real
- Verify all services can start and connect
- Ensure frontend is accessible
- Test that all features work end-to-end

### Pre-merge
- Full test suite execution with coverage requirements
- Code coverage requirements (minimum 80%)
- Security vulnerability scanning and assessment
- Performance testing and optimization
- Code review approval (minimum 2 reviewers)
- Architecture compliance check
- Documentation completeness check

### Pre-deployment
- Integration testing and validation
- End-to-end testing for critical flows
- Security audit and penetration testing
- Performance validation and optimization
- Documentation review and updates
- Load testing and capacity planning
- Accessibility testing and compliance
- Cross-browser compatibility testing

### Post-deployment
- Health check validation and monitoring
- Performance monitoring and optimization
- Error rate monitoring and alerting
- User acceptance testing and feedback
- Rollback capability verification
- Monitoring and alerting validation
- Post-deployment validation and testing

## Enterprise Metrics and KPIs

### Development Metrics
- Code coverage percentage and quality
- Test execution time and success rate
- Build success rate and deployment frequency
- Lead time for changes and feature delivery
- Mean time to recovery (MTTR) and incident response
- Change failure rate and rollback frequency
- Code review coverage and quality
- Technical debt ratio and management

### Quality Metrics
- Defect density and bug escape rate
- Test automation percentage and coverage
- Code complexity metrics and maintainability
- Security vulnerability count and severity
- Performance benchmark compliance
- Accessibility compliance score
- User satisfaction and experience metrics
- Documentation completeness and accuracy

### Process Metrics
- Cycle time and throughput
- Work in progress (WIP) and queue time
- Process efficiency and optimization
- Resource utilization and capacity
- Team velocity and productivity
- Sprint completion rate and quality
- Release frequency and stability
- Customer satisfaction and feedback

### Business Metrics
- Feature delivery time and quality
- Customer satisfaction and adoption
- User engagement and retention
- Performance impact on business metrics
- Cost per deployment and development
- ROI on development efforts and investments
- Time to market and competitive advantage
- Innovation index and technology adoption

## Enterprise Governance

### Decision Making Framework
- Use RACI matrix for decision roles and responsibilities
- Implement decision logs and documentation
- Use consensus-based decisions for architecture
- Implement escalation procedures and approval workflows
- Use data-driven decision making and analysis
- Document decision rationale and context
- Implement decision review process and validation
- Use decision templates and checklists

### Risk Management
- Implement risk assessment procedures and analysis
- Use risk registers and tracking systems
- Implement mitigation strategies and controls
- Use risk monitoring and reporting
- Implement contingency planning and response
- Use risk communication protocols and procedures
- Implement risk review processes and updates
- Use risk metrics and dashboards

### Compliance Management
- Implement compliance monitoring and reporting
- Use compliance dashboards and metrics
- Implement audit procedures and assessments
- Use compliance reporting and documentation
- Implement training programs and certification
- Use compliance metrics and KPIs
- Implement corrective actions and improvements
- Use compliance reviews and assessments

### Knowledge Management
- Implement knowledge sharing sessions and programs
- Use documentation repositories and systems
- Implement code review learning and best practices
- Use post-mortem analysis and lessons learned
- Implement best practice sharing and adoption
- Use knowledge metrics and tracking
- Implement training programs and development
- Use knowledge retention strategies and systems

## Rule Validation and Compliance

### Automated Validation
- Implement automated rule compliance checking
- Use linting rules that enforce coding standards
- Implement automated security scanning and checks
- Use automated testing to validate rule compliance
- Implement automated documentation validation
- Use automated performance monitoring and alerts
- Implement automated accessibility testing
- Use automated code quality gates and checks

### Manual Validation
- Implement regular code review processes
- Use manual security assessments and audits
- Implement manual performance reviews and optimization
- Use manual accessibility testing and compliance
- Implement manual documentation reviews
- Use manual architecture reviews and assessments
- Implement manual compliance audits and checks
- Use manual quality assurance and testing

### Continuous Improvement
- Implement regular rule review and updates
- Use feedback from development teams and processes
- Implement rule effectiveness monitoring and metrics
- Use industry best practices and standards updates
- Implement rule automation and tooling improvements
- Use rule compliance reporting and analysis
- Implement rule training and education programs
- Use rule evolution and adaptation processes

## DeepSeek-OCR & Optical Compression Best Practices

### Overview
DeepSeek-OCR implements "Contexts Optical Compression" - converting text to 2D visual representations for 10-20× compression while maintaining 97%+ accuracy. This section provides rules for optimal usage in LightDom.

**Reference Documentation**: See `DEEPSEEK_OCR_INTEGRATION.md` and `DEEPSEEK_OCR_BEST_PRACTICES.md` for complete details.

### Rule 1: Compression Ratio Selection
**ALWAYS choose compression ratio based on use case:**

```javascript
// ✅ GOOD: Use case-specific compression
const legalDoc = await ocrService.processDocument(input, {
  compressionRatio: 0.15, // 6-7× for legal docs (99% precision)
});

const trainingData = await ocrService.processDocument(input, {
  compressionRatio: 0.05, // 20× for training data (60-90% precision)
});

// ❌ BAD: Generic compression for all cases
const result = await ocrService.processDocument(input, {
  compressionRatio: 0.1 // Not optimized for use case
});
```

**Compression Guidelines:**
- Legal/Medical: 0.15-0.20 (5-7×, 99%+ precision)
- Business Docs: 0.10-0.15 (7-10×, 95%+ precision)
- Archives: 0.08-0.12 (8-13×, 90%+ precision)
- Training Data: 0.05-0.08 (13-20×, 60-90% precision)

### Rule 2: Vision Token Optimization
**ALWAYS calculate target vision tokens based on LLM context window:**

```javascript
// ✅ GOOD: Calculate compression for context window
function calculateOptimalCompression(documentSize, contextWindow) {
  const estimatedTokens = Math.ceil(documentSize / 4);
  const targetTokens = Math.floor(contextWindow * 0.8); // Use 80% of context
  return Math.max(0.05, Math.min(0.2, targetTokens / estimatedTokens));
}

// ❌ BAD: Hardcoded compression without context consideration
const compression = 0.1; // May exceed context window
```

### Rule 3: Batch Processing
**ALWAYS use batch processing for multiple documents:**

```javascript
// ✅ GOOD: Batch processing with proper sizing
const results = await ocrService.processBatch(documents, {
  batchSize: 20, // Optimal for GPU (15-25)
  outputFormat: 'json'
});

ocrService.on('batch-completed', ({ batchNumber, documentsProcessed }) => {
  console.log(`Batch ${batchNumber}: ${documentsProcessed} docs processed`);
});

// ❌ BAD: Processing one by one
for (const doc of documents) {
  await ocrService.processDocument(doc); // Inefficient
}
```

**Batch Size Guidelines:**
- GPU (A100): 15-25 documents
- GPU (other): 10-15 documents
- CPU: 5-10 documents

### Rule 4: Output Format Selection
**ALWAYS choose format based on downstream processing:**

```javascript
// ✅ GOOD: JSON for programmatic processing
const structured = await ocrService.extractStructuredData(input, {
  outputFormat: 'json',
  extractTables: true,
  extractFigures: true
});

// ✅ GOOD: Markdown for CMS
const markdown = await ocrService.processDocument(input, {
  outputFormat: 'markdown',
  preserveLayout: true
});

// ❌ BAD: Using text and losing structure
const result = await ocrService.processDocument(input, {
  outputFormat: 'text' // Loses tables, layout, figures
});
```

### Rule 5: Error Handling & Retries
**ALWAYS implement retry logic with exponential backoff:**

```javascript
// ✅ GOOD: Robust retry with backoff
async function processWithRetry(input, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await ocrService.processDocument(input);
    } catch (error) {
      if (attempt < maxRetries) {
        const delay = Math.pow(2, attempt - 1) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}

// ❌ BAD: No retry logic
const result = await ocrService.processDocument(input); // May fail
```

### Rule 6: Performance Monitoring
**ALWAYS track and analyze performance metrics:**

```javascript
// ✅ GOOD: Comprehensive monitoring
ocrService.on('document-processed', (event) => {
  metrics.record('ocr.compression_ratio', event.compressionRatio);
  metrics.record('ocr.confidence', event.confidence);
  metrics.record('ocr.processing_time_ms', event.processingTimeMs);
  
  if (event.confidence < 0.9) {
    alerts.warn(`Low OCR confidence: ${event.confidence}`);
  }
});

// Periodic stats review
setInterval(() => {
  const stats = ocrService.getStats();
  console.log('OCR Stats:', {
    avgCompression: stats.averageCompressionRatio.toFixed(2),
    avgConfidence: stats.averageConfidence.toFixed(3)
  });
}, 60000);

// ❌ BAD: No monitoring
const result = await ocrService.processDocument(input); // No visibility
```

### Rule 7: Language Hints
**ALWAYS provide language hints when known:**

```javascript
// ✅ GOOD: Providing language hints
const result = await ocrService.processDocument(input, {
  language: documentMetadata.language || 'en',
  outputFormat: 'markdown'
});

// ✅ GOOD: Auto-detect for unknown
const result = await ocrService.processDocument(input, {
  language: undefined // Auto-detect
});
console.log('Detected:', result.metadata.language);

// ❌ BAD: Never providing hints
const result = await ocrService.processDocument(input); // Less accurate
```

### Rule 8: Large Document Handling
**ALWAYS chunk large documents to avoid timeouts:**

```javascript
// ✅ GOOD: Chunking large documents
async function processLargeDocument(url, chunkSizeMB = 10) {
  const chunks = await splitDocumentIntoChunks(url, chunkSizeMB);
  const results = await ocrService.processBatch(chunks, {
    batchSize: 10,
    compressionRatio: 0.1
  });
  return combineChunkedResults(results);
}

// ❌ BAD: Processing huge document at once
const result = await ocrService.processDocument({
  url: 'huge-100mb-document.pdf' // May timeout
});
```

### Rule 9: Result Caching
**ALWAYS cache OCR results to avoid reprocessing:**

```javascript
// ✅ GOOD: Caching results
async function processWithCache(input, options = {}) {
  const cacheKey = generateCacheKey(input, options);
  const cached = await cache.get(cacheKey);
  if (cached) return cached;
  
  const result = await ocrService.processDocument(input, options);
  await cache.set(cacheKey, result, { ttl: 86400 }); // 24 hours
  return result;
}

// ❌ BAD: Always reprocessing
const result = await ocrService.processDocument(input); // Wastes resources
```

### Rule 10: Confidence Validation
**ALWAYS validate confidence and handle low results:**

```javascript
// ✅ GOOD: Validating confidence
async function processReliably(input, options = {}) {
  const result = await ocrService.processDocument(input, options);
  
  if (result.metadata.confidence < 0.9) {
    console.warn('Low confidence:', result.metadata.confidence);
    
    // Retry with lower compression
    if (options.compressionRatio > 0.1) {
      return processReliably(input, {
        ...options,
        compressionRatio: Math.min(options.compressionRatio * 1.5, 0.2)
      });
    }
    
    // Flag for manual review
    await flagForManualReview(result);
  }
  
  return result;
}

// ❌ BAD: Ignoring confidence
const result = await ocrService.processDocument(input);
processText(result.text); // May be inaccurate
```

### Integration Patterns

#### DOM Content Compression
```javascript
// ✅ Use optical compression for large DOM structures
async function optimizeLargeDOM(domContent) {
  const sizeKB = Buffer.from(domContent.html).length / 1024;
  if (sizeKB > 100) {
    return await ocrService.compressDOMOptically(domContent, {
      targetTokens: 200,
      preserveLayout: true
    });
  }
  return domContent.html;
}
```

#### Training Data Generation
```javascript
// ✅ Generate compressed training data
async function generateTrainingData(urls) {
  const trainingData = [];
  for (const url of urls) {
    const compressed = await ocrService.processDocument(
      { url: await crawler.screenshot(url) },
      {
        compressionRatio: 0.05, // High compression
        outputFormat: 'json',
        extractTables: true
      }
    );
    trainingData.push(compressed);
  }
  return trainingData;
}
```

#### Document Archive
```javascript
// ✅ Archive with searchable content
async function archiveDocument(document) {
  const ocr = await ocrService.processDocument(
    { url: document.url },
    {
      compressionRatio: 0.1,
      outputFormat: 'markdown',
      preserveLayout: true
    }
  );
  
  await db.documents.create({
    id: document.id,
    content: ocr.text,
    metadata: ocr.metadata,
    searchVector: createSearchVector(ocr.text)
  });
}
```

### Configuration Best Practices

**Environment Variables:**
```bash
OCR_WORKER_ENDPOINT=http://localhost:8000
DEEPSEEK_OCR_ENDPOINT=https://api.deepseek.com/ocr
DEEPSEEK_API_KEY=your_api_key_here
OCR_DEFAULT_COMPRESSION_RATIO=0.1
OCR_MAX_IMAGE_SIZE_MB=25
OCR_TIMEOUT_MS=30000
```

**Service Configuration:**
```javascript
const ocrService = new DeepSeekOCRService({
  defaultCompressionRatio: 0.1,
  maxCompressionRatio: 0.05,
  targetVisionTokens: 100,
  minOcrPrecision: 0.97,
  preserveLayout: true,
  extractTables: true,
  extractFigures: true
});
```

### Testing Best Practices

```javascript
describe('DeepSeek-OCR', () => {
  it('should maintain 97%+ precision at 10× compression', async () => {
    const result = await ocrService.processDocument(testDoc, {
      compressionRatio: 0.1
    });
    expect(result.metadata.confidence).toBeGreaterThan(0.97);
    expect(result.metadata.compressionRatio).toBeGreaterThan(9);
  });
  
  it('should achieve 20× compression for training', async () => {
    const result = await ocrService.processDocument(testDoc, {
      compressionRatio: 0.05
    });
    expect(result.metadata.compressionRatio).toBeGreaterThan(15);
  });
});
```

### Performance Targets

Based on DeepSeek-OCR paper benchmarks:
- A100 GPU: 200K+ pages/day at 10× compression (97% precision)
- A100 GPU: 150K+ pages/day at 15× compression (85% precision)
- A100 GPU: 100K+ pages/day at 20× compression (60% precision)
- CPU: 10-20K pages/day at 10× compression (97% precision)

### Key Resources
- Integration Guide: `DEEPSEEK_OCR_INTEGRATION.md`
- Best Practices: `DEEPSEEK_OCR_BEST_PRACTICES.md`
- Service Implementation: `services/deepseek-ocr-service.js`
- OCR Worker: `services/ocr-worker/app/main.py`
- ArXiv Paper: https://arxiv.org/abs/2510.18234v1
- GitHub: https://github.com/deepseek-ai/DeepSeek-OCR

Remember: These rules are living documents that should be updated as the project evolves and new best practices emerge. All team members should be familiar with these rules and follow them consistently. Regular review and updates of these workflows ensure they remain relevant and effective for enterprise-level development.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DashZeroAlionSystems) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
