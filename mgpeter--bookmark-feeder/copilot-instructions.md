## bookmark-feeder

> Solution Structure Rules

# Solution Structure Rules

Rules for maintaining the solution and project structure.

<rule>
name: solution_structure
description: Enforces the solution and project organization standards
filters:
  - type: file_extension
    pattern: "\\.(csproj|sln)$"
  - type: event
    pattern: "file_create|file_modify"

actions:
  - type: reject
    conditions:
      - pattern: "^(?!{ProjectPrefix}\\.[A-Za-z]+/{ProjectPrefix}\\.[A-Za-z]+\\.csproj$)"
        message: "Project files must follow the {ProjectPrefix}.{ProjectName} naming convention and be in their own directory"

  - type: suggest
    message: |
      Solution Structure Standards:

      1. Root Structure:
         ```
         /
         ├── {SolutionName}.sln
         ├── {SolutionName}.AppHost/       # Aspire host application
         ├── {SolutionName}.ServiceDefaults/ # Shared Aspire configuration
         ├── {SolutionName}.Web/        # Web Frontend
         ├── {SolutionName}.ApiService/        # Backend API
         ├── {SolutionName}.Core/           # Core business logic
         ├── {SolutionName}.Data/           # Data access layer
         ├── {SolutionName}.Infrastructure/ # Infrastructure and integration services
         ├── {SolutionName}.Dto/           # Data transfer objects
         └── {SolutionName}.{ProjectName}/ # Additional projects following convention
         └── {SolutionName}.{ProjectName}.Tests/ # Test projects
         ```

      2. Project Naming Convention:
         - Must start with "{ProjectPrefix}."
         - Use PascalCase for project names
         - Project directory must match project name
         - Each project in its own directory

      3. Common Project Types:
         - *.AppHost - Aspire orchestration
         - *.ServiceDefaults - Shared Aspire configuration
         - *.Web - Frontend
         - *.ApiService - Backend API
         - *.Core/Data/Dto - Business/Data layers
         - *.Services - Shared services
         - *.Tests - Test projects
         - *.Contracts - Service contracts/interfaces

      4. Testing Standards:
         - Use xUnit for unit testing
         - Use Shouldly for assertions
         - Use NSubstitute for mocking
         - Test project naming: {ProjectName}.Tests
         - Test class naming: {ClassName}Tests
         - Test method naming: {MethodName}_Scenario_ExpectedResult
         - Place tests in matching projects and folder structure


      5. Project Dependencies:
         - AppHost → All projects (for orchestration)
         - Web → Dto, Core, Contracts
         - ApiService → Dto, Core, Data, Contracts
         - Core → Dto
         - Data → Core, Dto
         - Services → Core, Dto, Contracts
         - *.Tests → {ProjectName} (test project references source project)

examples:
  - input: |
      # Bad: Incorrect project structure/naming
      src/MyProject.csproj
      ProjectWebApi/Project.WebApi.csproj
      Project.WebApi.csproj
      MyTests/MyTests.csproj

      # Good: Correct project structure/naming
      {SolutionName}.WebApi/{SolutionName}.WebApi.csproj
      {SolutionName}.Services/{SolutionName}.Services.csproj
      {SolutionName}.AppHost/{SolutionName}.AppHost.csproj
      {SolutionName}.Core.Tests/{SolutionName}.Core.Tests.csproj
    output: "Correctly organized solution structure"

metadata:
  priority: high
  version: 1.0
</rule> 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mgpeter)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mgpeter)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
