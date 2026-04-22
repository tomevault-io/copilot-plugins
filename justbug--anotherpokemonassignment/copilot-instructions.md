## anotherpokemonassignment

> iOS MVVM architecture and Swift coding guidelines


# iOS MVVM Architecture Guidelines

## MVVM Pattern Structure
This iOS app follows the MVVM (Model-View-ViewModel) pattern with clear separation of concerns.

### Architecture Components

#### Models
- Data models should be Codable for JSON serialization
- Use structs for value types
- Include proper property names matching API responses

#### Views
- ViewControllers handle UI presentation only
- Use storyboards or programmatic UI
- Implement UITableViewDataSource and UITableViewDelegate for lists
- Handle user interactions and delegate to ViewModels

#### ViewModels
- ViewModels contain business logic and state management
- Use `@Published` properties for reactive UI updates
- Handle data transformation and validation
- Coordinate between Services and UseCases

### Service Layer
- **APIClient**: [APIClient.swift](mdc:AnotherPokemonAssignment/AnotherPokemonAssignment/Networking/APIClient.swift) handles HTTP requests
- **Services**: Business logic services in [Service/](mdc:AnotherPokemonAssignment/AnotherPokemonAssignment/Service/) directory
- **UseCases**: Coordinate services and handle business rules

### Swift Coding Standards

#### Naming Conventions
- Use camelCase for variables and functions
- Use PascalCase for types and protocols
- Use descriptive names that explain intent
- Prefix private properties with underscore when needed

#### Error Handling
- Use Result types for operations that can fail
- Implement proper error propagation
- Create custom error types for domain-specific errors
- Handle network errors gracefully

#### Memory Management
- Use weak references to avoid retain cycles
- Implement proper cleanup in deinit
- Use lazy properties for expensive initialization

### Data Flow
1. View triggers action in ViewModel
2. ViewModel calls UseCase
3. UseCase coordinates Services
4. Service makes API call or accesses storage
5. Data flows back through the chain
6. ViewModel updates published properties
7. View reacts to property changes

### Testing Strategy
- Write unit tests for ViewModels and UseCases
- Mock Services for isolated testing
- Test error handling paths
- Verify UI state changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/justbug) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
