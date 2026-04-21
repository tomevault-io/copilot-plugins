## op-patchstudio-companion

> - use Model-View-ViewModel pattern consistently

## MVVM Architecture
- use Model-View-ViewModel pattern consistently
- separate UI logic from business logic
- implement proper data binding and commands
- use dependency injection for service management

## ViewModels
### Structure & Organization
- inherit from `ViewModelBase` for common functionality
- implement `INotifyPropertyChanged` for property updates
- use `ObservableCollection<T>` for UI-bound collections
- implement proper disposal patterns for resources

### Property Implementation
- use backing fields with `SetProperty` for change notification
- implement property validation where appropriate
- use computed properties for derived values
- implement proper property change propagation

### Commands
- use `ICommand` interface for all user actions
- implement `RelayCommand` or `AsyncRelayCommand` for command execution
- provide proper command can-execute logic
- handle command execution errors gracefully

## Views
### XAML Structure
- use compiled bindings for better performance
- implement proper data context assignment
- use resource dictionaries for reusable styles
- follow Avalonia UI patterns and conventions

### Data Binding
- use `{Binding}` for simple property binding
- use `{x:Bind}` for compiled bindings where possible
- implement proper binding error handling
- use value converters for data transformation

## Models & Services
### Service Layer
- implement interfaces for all major services
- use dependency injection for service management
- implement proper async/await patterns
- provide proper error handling and logging

### Data Models
- use strongly-typed models for all data
- implement proper validation and constraints
- use immutable objects where appropriate
- provide proper serialization support

## Testing
### Unit Testing
- write unit tests for all ViewModels
- test command execution and property changes
- mock dependencies for isolated testing
- test error scenarios and edge cases

### Integration Testing
- test View-ViewModel integration
- test service integration and data flow
- test UI automation where appropriate
- validate end-to-end user workflows

## Best Practices
- keep ViewModels focused and single-purpose
- avoid business logic in Views
- use proper naming conventions
- implement proper error handling throughout
- maintain separation of concerns
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joseph-holland) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
