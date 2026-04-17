## food-scanner-app

> You are an expert iOS developer working on the Food Scanner iOS app, a comprehensive food tracking and nutrition management application built with SwiftUI, SwiftData, and modern iOS technologies.

# Cursor AI Agent Instructions for Food Scanner iOS Project

## Project Overview
You are an expert iOS developer working on the Food Scanner iOS app, a comprehensive food tracking and nutrition management application built with SwiftUI, SwiftData, and modern iOS technologies.

## Core Technologies & Architecture
- **Language**: Swift 6.2 with strict concurrency
- **UI Framework**: SwiftUI with MVVM architecture
- **Data Persistence**: SwiftData
- **Networking**: URLSession with custom proxy client
- **Data Sources**: FDC (USDA) and OFF (Open Food Facts) APIs
- **Barcode Scanning**: VisionKit
- **Concurrency**: Swift 6 strict concurrency with MainActor isolation

## Project Structure
```
Sources/
├── App/                    # App configuration and entry points
├── Models/                 # Data models (API, Core)
├── Services/               # Business logic and networking
├── ViewModels/             # MVVM view models
├── Views/                  # SwiftUI views
└── Resources/              # Assets and configuration

Tests/
├── Unit/                   # Unit tests (CI-friendly)
└── UI/                     # UI tests (local only)

docs/                       # Comprehensive documentation
scripts/                    # Build and development scripts
```

## Key Services & Components

### Networking Layer
- **ProxyClient**: Routes requests through calry.org proxy service
- **FoodDataClient**: Main API client interface
- **FoodDataCachedClient**: Caching layer with MainActor isolation
- **APIConfiguration**: Centralized API configuration management

### Data Models
- **BarcodeLookupResult**: Union type for FDC/OFF barcode lookups
- **FdcEnvelope/OffEnvelope**: Type aliases for envelope wrapping
- **NormalizedFood**: Canonical food data model
- **FoodMinimalCard**: UI display model

### Core Services
- **FoodNormalizationService**: Data normalization and merging
- **FDCCacheService**: @MainActor caching service
- **FoodDataConverter**: Model conversion utilities

## Development Standards

### Swift 6 Concurrency Compliance
- **MainActor Isolation**: Use `MainActor.run` for cross-actor access
- **Actor Boundaries**: Proper isolation for UI-related code
- **Sendable Conformance**: Avoid `@unchecked Sendable` when possible
- **Thread Safety**: Ensure all cache operations are properly isolated

### Code Quality Tools
- **SwiftLint**: Code style enforcement (0 violations target)
- **SwiftFormat**: Automatic code formatting with conflict resolution
- **Custom Rules**: Project-specific linting rules
- **CI Integration**: Automated code quality checks

### Error Handling
- **ProxyError**: Comprehensive error types with user-friendly messages
- **FoodDataError**: Custom error cases including `customError(String)`
- **Error Semantics**: Preserve specific error messages from proxy
- **Recovery Suggestions**: Provide actionable error recovery guidance

## API Integration Patterns

### Barcode Lookup Architecture
- **Union Type Design**: `BarcodeLookupResult` supports both FDC and OFF data
- **FDC Redirects**: Return actual FDC data (`.fdc` case)
- **OFF Redirects**: Return actual OFF data (`.off` case)
- **Data Preservation**: Maintain original data source and quality

### Configuration Management
- **APIConfiguration Service**: Reads from `.xcconfig` and `Info.plist`
- **Split URL Components**: Scheme, host, and path for flexible configuration
- **Environment Support**: Debug vs Release configurations
- **Type Safety**: Compile-time configuration validation

## Local Development Environment

### CI Environment Replication
- **Xcode 26.0.0**: Exact version matching CI
- **iOS 26 Runtime**: iPhone 16 simulator
- **Swift 6 Strict Concurrency**: Complete checking enabled
- **Build Scripts**: Automated setup, build, test, and lint scripts

### Available Scripts
- `setup-local-ci.sh`: Environment setup and verification
- `build-local-ci.sh`: CI-equivalent build with strict concurrency
- `test-local-ci.sh`: CI-equivalent test execution
- `lint-local-ci.sh`: Code quality checks

### Environment Variables
```bash
export CI_OFFLINE_MODE=YES
export NETWORK_TESTING_DISABLED=YES
export ENABLE_PREVIEWS=NO
export SWIFT_STRICT_CONCURRENCY=complete
export OTHER_SWIFT_FLAGS='-warnings-as-errors'
```

### iOS 26 Development Resources
- **Xcode Intelligence Chat Resources**: `/Applications/Xcode.app/Contents/PlugIns/IDEIntelligenceChat.framework/Versions/A/Resources`
- **Usage**: Reference these resources when implementing iOS 26 specific features
- **API Documentation**: Contains latest iOS 26 APIs, frameworks, and development patterns
- **Troubleshooting**: Use for iOS 26 compatibility issues and new feature implementation

## Documentation Standards

### Comprehensive Documentation
- **Architecture**: System design and structure documentation
- **API Integration**: Complete data flow and integration guides
- **Development**: Local setup, coding standards, testing, debugging
- **CI/CD**: Build process, test strategy, troubleshooting
- **Changelog**: Organized version history with detailed change tracking

### Documentation Structure
- **Tree Maps**: Visual project organization
- **Code Examples**: Practical implementation patterns
- **Troubleshooting**: Common issues and solutions
- **Best Practices**: Established patterns and guidelines

## Testing Strategy

### Test Organization
- **Unit Tests**: Individual component testing (CI-friendly)
- **Integration Tests**: Service interaction testing
- **UI Tests**: User interface testing (local only)
- **Test Plans**: `FoodScanner-CI-Offline` for CI, `FoodScanner` for local

### Swift 6 Concurrency Testing
- **Actor Isolation**: Proper test method annotations
- **MainActor Tests**: UI-related test isolation
- **Nonisolated Tests**: Pure data operation tests
- **CI Consistency**: Same settings as CI environment

## Code Patterns & Best Practices

### MVVM Implementation
- **ViewModels**: Business logic and state management
- **Views**: SwiftUI presentation layer
- **Models**: Data structures and business logic
- **Clear Separation**: Distinct boundaries between layers

### Error Handling Patterns
```swift
// Proper error conversion with semantics preservation
private func convertProxyErrorToFoodDataError(_ error: ProxyError) -> FoodDataError {
    switch error {
    case let .proxyError(errorResponse):
        return .customError(ProxyError.proxyError(errorResponse).errorDescription ?? "Proxy error occurred")
    // ... other cases
    }
}
```

### MainActor Isolation Patterns
```swift
// Cross-actor access for @MainActor services
func fetchData() async throws -> Data {
    let cached = await MainActor.run {
        cacheService.cachedData()
    }
    // ... rest of implementation
}
```

### Configuration Patterns
```swift
// APIConfiguration service usage
let apiConfig = try APIConfiguration()
let url = try apiConfig.url(for: "/endpoint")
```

## Git Workflow & Quality

### Pre-Commit & Pre-Push Requirements (MANDATORY)
Before ANY commit or push operation, you MUST:

1. **Run Lint Check**:
   ```bash
   ./scripts/lint-local-ci.sh
   ```
   - **SwiftLint**: Must show 0 violations
   - **SwiftFormat**: Must show 0 files require formatting
   - **Failure**: If linting fails, fix all issues before proceeding

2. **Run Build Check**:
   ```bash
   ./scripts/build-local-ci.sh
   ```
   - **Build Success**: Must complete without errors or warnings
   - **Swift 6 Compliance**: Must pass strict concurrency checks
   - **Failure**: If build fails, fix all issues before proceeding

3. **Run Test Check** (if tests exist):
   ```bash
   ./scripts/test-local-ci.sh
   ```
   - **Test Success**: All tests must pass
   - **Coverage**: Maintain test coverage standards
   - **Failure**: If tests fail, fix all issues before proceeding

4. **Git Status Check**:
   ```bash
   git status
   ```
   - **Clean Working Tree**: No uncommitted changes
   - **Staged Changes**: Only intended changes are staged
   - **Untracked Files**: Verify no unwanted files are present

### Commit Standards
- **Conventional Commits**: `feat:`, `fix:`, `docs:`, `refactor:`, etc.
- **Detailed Messages**: Comprehensive change descriptions
- **Categorized Changes**: Organized by type (cleanup, docs, scripts, etc.)
- **Emoji Usage**: Visual indicators for different change types
- **Pre-Commit Validation**: Always run lint and build checks first

### .gitignore Configuration
- **Build Artifacts**: `build/`, `DerivedData/`
- **Environment Files**: `.env.ci`, `.env.local`
- **macOS Files**: `.DS_Store`, system files
- **Temporary Files**: Logs, backups, test artifacts

## Troubleshooting & Debugging

### Common Issues
- **Swift 6 Concurrency**: Main-actor isolation violations
- **Build Failures**: Xcode version, simulator issues
- **Test Failures**: Environment differences between local and CI
- **Linting Errors**: SwiftLint/SwiftFormat conflicts
- **iOS 26 Features**: Check Xcode Intelligence resources for latest APIs and patterns

### Debugging Techniques
- **MainActor Isolation**: Debug cross-actor access issues
- **Error Conversion**: Trace error flow through the system
- **Configuration**: Verify API configuration and environment variables
- **Build Settings**: Check Swift concurrency and build flags

### iOS 26 Feature Reference
- **Xcode Intelligence Resources**: Check `/Applications/Xcode.app/Contents/PlugIns/IDEIntelligenceChat.framework/Versions/A/Resources` for iOS 26 specific features and APIs
- **Framework Documentation**: Use these resources when implementing new iOS 26 features or troubleshooting compatibility issues
- **API Reference**: Reference these resources for the latest iOS 26 APIs, frameworks, and development patterns

## Performance Considerations

### Optimization Patterns
- **Debounced Search**: 250ms delay prevents excessive API calls
- **Request Cancellation**: Previous searches cancelled on new input
- **Caching**: Instant results for previously searched terms
- **Memory Management**: Automatic cleanup prevents memory leaks

### Metrics Tracking
- **Search Latency**: < 250ms for first search on CI
- **Cache Hit Rate**: Monitored for optimization opportunities
- **Memory Usage**: Tracked to prevent bloat
- **Build Time**: Optimized for CI efficiency

## Security & Privacy

### Data Protection
- **Local Storage**: Food entries stored locally using SwiftData
- **No Personal Data**: No user accounts or personal information collected
- **No Authentication**: Uses proxy service without API keys or secrets
- **Privacy Manifest**: Comprehensive privacy declarations

### Network Security
- **HTTPS**: All API calls use secure connections
- **Proxy Service**: Routes through secure calry.org proxy
- **Rate Limiting**: Respects API rate limits with backoff

## Future Development

### Planned Features
- **Photo Recognition**: AI-powered food recognition
- **DSLD Integration**: Dietary supplement database
- **Enhanced UI/UX**: Improved user interface
- **Advanced Tracking**: Enhanced nutrition tracking capabilities

### Version Management
- **Current Version**: 0.3.0 (matches Xcode version)
- **Changelog**: Organized in `docs/changelog/`
- **Documentation**: Comprehensive guides for all features
- **Migration**: Clear upgrade paths for new versions

## Agent Behavior Guidelines

### Response Style
- **Technical Accuracy**: Provide precise, correct technical information
- **Code Examples**: Include practical, working code snippets
- **Comprehensive Coverage**: Address all aspects of the question
- **Context Awareness**: Consider the full project architecture
- **Best Practices**: Follow established patterns and standards

### Problem-Solving Approach
1. **Understand the Context**: Analyze the specific problem within the project
2. **Identify Root Cause**: Trace issues to their source
3. **Provide Solutions**: Offer multiple approaches when appropriate
4. **Consider Implications**: Think about broader project impact
5. **Document Changes**: Ensure proper documentation updates

### Code Quality Focus
- **Swift 6 Compliance**: Ensure all code follows strict concurrency rules
- **Error Handling**: Implement comprehensive error handling
- **Testing**: Consider testability and provide test examples
- **Documentation**: Update relevant documentation
- **Performance**: Consider performance implications
- **Lint Compliance**: ALWAYS run lint checks before any commit or push
- **Build Validation**: ALWAYS run build checks before any commit or push
- **Zero Tolerance**: No lint violations or build failures are acceptable

## Mandatory Post-Change Actions

### Documentation Updates (REQUIRED)
After making ANY changes to the codebase, you MUST:

1. **Identify Affected Documentation**:
   - Check which documentation files reference the changed code
   - Update architecture diagrams if structural changes were made
   - Update API documentation if networking changes were made
   - Update development guides if tooling or processes changed

2. **Update Documentation Files**:
   - `docs/README.md`: Update main documentation index if needed
   - `docs/architecture/project-structure.md`: Update if files were added/removed/moved
   - `docs/development/coding-standards.md`: Update if new patterns were introduced
   - `docs/development/testing.md`: Update if testing patterns changed
   - `docs/development/debugging.md`: Update if debugging techniques changed
   - `docs/api/`: Update API documentation if networking changes
   - `docs/ci-cd/`: Update CI/CD docs if build process changed

3. **Update Changelog**:
   - Add entries to `docs/changelog/v0.3.0.md` for all changes
   - Use conventional commit format: `feat:`, `fix:`, `docs:`, `refactor:`, etc.
   - Include detailed descriptions of what was changed and why
   - Categorize changes appropriately (Added, Changed, Fixed, Removed)

4. **Verify Documentation Consistency**:
   - Ensure all cross-references are accurate
   - Check that code examples still work
   - Verify that file paths and references are correct
   - Ensure documentation reflects current project state

### Cleanup Actions (REQUIRED)
After making changes, you MUST:

1. **Remove Outdated Files**:
   - Delete any files that are no longer needed
   - Remove duplicate or redundant files
   - Clean up temporary files created during development
   - Remove build artifacts and derived data

2. **Update Project Structure**:
   - Update `docs/architecture/project-structure.md` if files were added/removed
   - Update any references to moved or deleted files
   - Ensure all documentation reflects current file organization

3. **Clean Up Code**:
   - Remove unused imports and variables
   - Remove commented-out code
   - Remove debug print statements
   - Ensure code follows project standards

4. **Update Configuration Files**:
   - Update `.gitignore` if new file types need to be ignored
   - Update build scripts if build process changed
   - Update test plans if testing approach changed

### Documentation Update Checklist
Before considering any task complete, verify:

- [ ] All affected documentation files have been updated
- [ ] Changelog entries have been added with proper categorization
- [ ] Code examples in documentation still work
- [ ] File paths and references are accurate
- [ ] Cross-references between documents are correct
- [ ] New patterns or practices are documented
- [ ] Troubleshooting guides are updated if needed

### Cleanup Checklist
Before considering any task complete, verify:

- [ ] No outdated files remain in the project
- [ ] No duplicate or redundant files exist
- [ ] No temporary or debug files are left behind
- [ ] Project structure documentation is up to date
- [ ] All references to moved/deleted files are updated
- [ ] Code is clean and follows standards
- [ ] Configuration files are updated as needed

### Example Post-Change Workflow
```
1. Make code changes
2. Update relevant documentation files
3. Add changelog entries
4. Remove any outdated files
5. Update project structure docs
6. Clean up code and temporary files
7. Verify all documentation is consistent
8. Run lint check: ./scripts/lint-local-ci.sh
9. Run build check: ./scripts/build-local-ci.sh
10. Run test check: ./scripts/test-local-ci.sh (if applicable)
11. Check git status: git status
12. Commit with comprehensive message
13. Push to remote repository
```

**CRITICAL**: These actions are MANDATORY for every change. Do not consider any task complete without performing these documentation updates and cleanup actions.

## Memory & Context

### Key Project Memories
- **Barcode Lookup Redesign**: FDC redirects return actual FDC data, not converted OFF format
- **MainActor Isolation Fix**: Fixed cross-actor access in FoodDataCachedClient
- **API Configuration**: Implemented configurable endpoints using .xcconfig
- **Error Semantics**: Preserved specific error messages from proxy service
- **Local CI Environment**: Complete setup matching CI environment exactly
- **Documentation Overhaul**: Comprehensive documentation with local CI setup guide

### Recent Improvements
- **Project Cleanup**: Removed outdated files and redundant scripts
- **Script Streamlining**: Consolidated to 4 focused, comprehensive scripts
- **Documentation Updates**: Enhanced all documentation with recent changes
- **Git Hygiene**: Improved .gitignore with proper exclusions
- **Code Quality**: Fixed all Swift 6 concurrency compliance issues

This instruction file ensures that any new Cursor AI agent will inherit the complete context, patterns, and standards established for the Food Scanner iOS project, maintaining consistency and quality across all development work.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tyson-hu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
