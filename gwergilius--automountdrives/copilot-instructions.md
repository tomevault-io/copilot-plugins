## automountdrives

> We are using the `C:\Users\GergelyToth2\OneDrive - EPAM\Projects\Bayer\scripts\Startup\DriveMapper\` folder as a workspace (root folder). During the communication, the folder references starting with '~' character refers to this folder. For example, when I refers to the `~/docs` folder, that means the `C:\Users\GergelyToth2\OneDrive - EPAM\Projects\Bayer\scripts\Startup\DriveMapper\docs`folder

# Coding Standards

## Workspace
We are using the `C:\Users\GergelyToth2\OneDrive - EPAM\Projects\Bayer\scripts\Startup\DriveMapper\` folder as a workspace (root folder). During the communication, the folder references starting with '~' character refers to this folder. For example, when I refers to the `~/docs` folder, that means the `C:\Users\GergelyToth2\OneDrive - EPAM\Projects\Bayer\scripts\Startup\DriveMapper\docs`folder
  
## Naming conventions
- **Classes**: PascalCase (e.g. `AssetExtractionService`)
- **Methods**: PascalCase (e.g. `ExtractAssets`)
- **Private fields**: camelCase with underscore (e.g. `_extractionService`)
- **Properties**: PascalCase (e.g. `IsExtracting`)
- **Constants**: UPPER_CASE (e.g. `MAX_BUFFER_SIZE`)
- Class names are nouns (e.g. Person, NotificationService, HtmlClientFactory)
- Function names are verbs or verbs with subject (e.g. Initialize, CalculateResult, GetClient)
- Property names are nouns (e.g. Color, Instance, InvocationDepth)
- Boolean properties should have "Is", "Can", or "Has" prefix, but only where it adds value. Boolean fields should be prefixed similarly using "_is", "_can" or "_has" prefixes.

## Classes and Methods
- Extension methods in `Extensions` folder (in C# projects)
- All public methods with XML documentation
- Everey identifier should be named using English language

## C# Extension Methods
- `this` parameter is always the first parameter
- Null checks in all extension methods
- Throw ArgumentNullException for null parameters
- Detailed XML documentation with `<example>` blocks

## Dependency Injection (in dotnet, C# projects)
- Use **Microsoft.Extensions.DependencyInjection**
- Register services in **ConfigureServices** method
- Use **IServiceProvider** to retrieve services

## Error Handling (in dotnet, C# projects)
- Use FluentResults library for Result<T> types
- NotFoundError class for non-existent resources
- Detailed error messages with diagnostic information
- Throw Exception only if parameters are invalid
- Use **try-catch** blocks for I/O operations

## Blazor
- For Blazor pages/components, keep markup lean and place logic in separate `.razor.cs` code-behind partials

## Testing (in dotnet, C# projects)
- Write unit tests for all business logic
- Use xUnit testing framework
- Use Shouldly assertion library
- Test classes in logical grouping
- Theory tests for different input values
- Edge cases and error case testing
- Use **Mock** objects for external dependencies
- Create Mock objects using Mock.Of. Setup and Verify them using .AsMock() extension method from Test.Core component (e.g.: obj.AsMock().Setup(...))
- Integration and E2E tests use **ReqNRoll** (replaces SpecFlow)

## Documentation
- XML documentation for all public APIs
- `<summary>` for all classes and methods
- `<param>` for all parameters
- `<returns>` for all return values
- `<exception>` for all exceptions
- `<example>` for usage examples
- Each comment should be written in English

## Project Structure
- Extension classes in `Extensions/` folder
- Model classes in `Models/` folder
- Service classes in `Services/` folder
- Test classes in the corresponding `Tests/` folder

## Best Practices
- Use **interface-based design**
- Implement a **SOLID principles**
- Use **dependency injection** for each services
- Follow the **separation of concerns** principle
- Use **async/await** pattern for I/O operations

description:
globs:
alwaysApply: true
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gwergilius)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Gwergilius)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
