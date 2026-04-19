## rental-investment-analyzer-chrome

> 1. `docs/architecture/tech-architecture.md`

# Cursor Rules for Rental Investment Analyzer

## Project Documentation Structure

### Core Documentation Files
1. `docs/architecture/tech-architecture.md`
   - Defines the technical architecture of the extension
   - Contains component structure, data models, and data flow
   - Reference for developers to understand system design
   - Must be kept in sync with code changes

2. `docs/specs/product-specs.md`
   - Product requirements and specifications
   - UI/UX design guidelines
   - Feature definitions and user experience flows
   - Reference for product decisions and user-facing features

3. `docs/tasks/task-breakdown.md`
   - Current project status and progress tracking
   - Task organization and prioritization
   - Implementation status of features
   - Future roadmap and considerations

### Documentation Rules
1. **Architecture Documentation**
   - Keep `tech-architecture.md` updated when:
     - Adding new components
     - Modifying data models
     - Changing data flow
     - Updating state management
   - Include TypeScript interfaces for new data types
   - Document component responsibilities
   - Update data flow diagrams

2. **Product Specifications**
   - Update `product-specs.md` when:
     - Adding new features
     - Modifying UI/UX
     - Changing user flows
     - Updating requirements
   - Include visual descriptions
   - Document user interactions
   - Specify performance requirements

3. **Task Management**
   - Keep `task-breakdown.md` current by:
     - Marking completed tasks
     - Adding new tasks
     - Updating priorities
     - Tracking blockers
   - Use consistent formatting:
     - ✅ for completed tasks
     - [ ] for pending tasks
     - Clear categorization
   - Update progress tracking section

## Code Organization Rules

### Directory Structure
```
rental-investment-analyzer/
├── src/
│   ├── components/          # React components
│   ├── services/           # Business logic
│   ├── types/             # TypeScript definitions
│   ├── constants/         # Configuration
│   └── utils/            # Helper functions
├── docs/
│   ├── architecture/      # Technical docs
│   ├── specs/            # Product specs
│   └── tasks/            # Task tracking
└── tests/                # Test files
```

### Component Rules
1. **Naming Conventions**
   - Use PascalCase for components
   - Use camelCase for utilities
   - Prefix interfaces with 'I'
   - Use descriptive, purpose-indicating names

2. **File Organization**
   - One component per file
   - Co-locate related components
   - Group by feature/domain
   - Keep files focused and small

3. **Type Safety**
   - Use TypeScript for all new code
   - Define interfaces for props
   - Avoid 'any' type
   - Document complex types

### State Management Rules
1. **ConfigManager**
   - Singleton pattern
   - Type-safe operations
   - Persistent storage
   - Real-time updates

2. **Component State**
   - Local state for UI
   - Props for data flow
   - Context for global state
   - Avoid prop drilling

### Testing Rules
1. **Unit Tests**
   - Test business logic
   - Mock external dependencies
   - Use meaningful descriptions
   - Follow AAA pattern

2. **Integration Tests**
   - Test component interactions
   - Verify data flow
   - Test user interactions
   - Mock API calls

3. **End-to-End Tests**
   - Test Zillow integration
   - Verify extension loading
   - Test user flows
   - Validate calculations

## Development Workflow

### Code Review Process
1. **Documentation Updates**
   - Update relevant .md files
   - Document architectural changes
   - Update task status
   - Add implementation notes

2. **Testing Requirements**
   - Add tests for new features
   - Update existing tests
   - Verify test coverage
   - Document test scenarios

3. **Performance Considerations**
   - Monitor bundle size
   - Optimize calculations
   - Minimize re-renders
   - Profile performance

### Version Control
1. **Commit Messages**
   - Use conventional commits
   - Reference task numbers
   - Describe changes clearly
   - Link to documentation

2. **Branch Strategy**
   - feature/ for new features
   - bugfix/ for fixes
   - docs/ for documentation
   - test/ for test updates

## Maintenance Rules

### Documentation Maintenance
1. **Regular Updates**
   - Review docs weekly
   - Update task status
   - Verify architecture docs
   - Check product specs

2. **Version Tracking**
   - Document breaking changes
   - Update changelog
   - Track deprecations
   - Note migration steps

### Code Quality
1. **Linting**
   - Follow ESLint rules
   - Use Prettier formatting
   - Fix warnings promptly
   - Maintain consistency

2. **Performance**
   - Monitor metrics
   - Optimize bottlenecks
   - Profile regularly
   - Document improvements

## Project Context

### Key Concepts
1. **Data Model**
   - PropertyData: Listing information
   - UserCalculationInputs: User preferences
   - CalculatedMetrics: Investment analysis

2. **Core Features**
   - Real-time analysis
   - Property data extraction
   - Investment calculations
   - User preference management

3. **Technical Stack**
   - React + TypeScript
   - Chrome Extension
   - Tailwind CSS
   - Chrome Storage API

### Development Priorities
1. **Current Focus**
   - Testing implementation
   - Documentation completion
   - Performance optimization
   - User experience improvements

2. **Future Goals**
   - Advanced metrics
   - Market analysis
   - Export functionality
   - Platform expansion

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/srrmlwn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
