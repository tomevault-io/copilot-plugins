## terrapersonmanagerapp

> This is a simple C# console application that manages a list of people. It reads person data from a `data.csv` file, including their ID, name, and age. The application then processes this list to distinguish between adults (age 18 and over) and children. Finally, it prints both lists to the console.

# GEMINI.md

## Project Overview

This is a simple C# console application that manages a list of people. It reads person data from a `data.csv` file, including their ID, name, and age. The application then processes this list to distinguish between adults (age 18 and over) and children. Finally, it prints both lists to the console.

The project is built using .NET 10.0 and serves as a basic example of file I/O, data manipulation, and object-oriented programming in C#.

## Building and Running

To build and run this project, you will need the .NET SDK installed.

### Building the Project

Open a terminal in the `PersonManagerApp` directory and run the following command to build the application:

```bash
dotnet build
```

### Running the Application

To run the application, use the following command from the `PersonManagerApp` directory:

```bash
dotnet run
```

The application will then output the lists of adults and children to the console.

## Development Conventions

*   **Code Style:** The project follows standard C# coding conventions.
*   **Data Handling:** The `data.csv` file is the single source of truth for person data. It is read at runtime.
*   **Project Structure:**
    *   `PersonManagerApp.csproj`: The main project file.
    *   `Program.cs`: Contains the main application logic and the `Person` class definition.
    *   `data.csv`: CSV file containing the person data.

### Naming Patterns
- The code is seperated into mangement classes and dataclasses
 - DataClasses are all classes representing domain object, have no methods and are placed into a namespace containing ".DataClasses"
 - MangementClasses contain application logic but no data
 - Neither of both are classes like
  - Program.cs is the application startup class
- EVERY management class must follow a so called "naming pattern"
- A naming pattern is a suffix in the class name, indicating the purpose of the class
- Every naming pattern has allowed methods
 - Full methods: The Name of the method must exactly match the method name of the pattern, optionally with a "Async" suffix
 - Pattern Methods: Pattern Methods define the allowed prefix of method names (e.g. GetPayed() or (GetMarried() if the naming pattern allows the Get* method pattern
- A class can only follow one naming pattern
- A class following a naming pattern can only consist of allowed full methods oder pattern methods
- No other methods are allowed
- Allowed naming patterns are ({x} are Domain names like Person, Car, File, etc)
 - {x}Manager (Full: Add, Remove, Update, Pattern: Get, Is, Are)
 - {x}Repository (Full: Insert, Update, Delete, Query)
 - {x}DisplayCommands (Pattern: Display)
 - {x}Parser (Pattern: Parsefrom)
 - {x}Reader (Pattern: Read)
- if i ask you to do a "naming pattern review" or "npr", please check if the production code (not the unit test code) in this project alligns with the naming patterns
    - Just check if each class name of a management class corresponds to a naming pattern
    - dont change code, just output the results
    - Dont remove any files, if i ask u to fix the found issues during a npr!

### Unit Test guidelines
- For each production class (e.g. Foo), create a Testclass (e.g. FooTests)
- Each Teast class must be placed inside of project UnitTests
- Use [Fact], [Theory], [InlineData], [MemberData] and [ClassData] for Unit Tests
- Use classes named with the suffix TestSetup for code executed before the first and after the last test
- Use classes named with the suffix TestData, if the [ClassData] attribute is used
- Name Each Test flollowing the pattern: sut_precondition_postcondition
	- sut = method to test
- Structure each test method following the AAA-Pattern (wihtout comments)
- Name of the variable contining the result is "result"
- the tested class must be stored inside a readonly class field with the name _sut
- ALL assertions must be done using FluentAssertions
- if i ask you to do a "test code review" or "tcr", please check if the test code in this project alligns with this Test Guidelines
    - Just check if the tests align to this rules
    - Dont execute the tests, just rework the structure
    - NEVER change the production code
    - Dont remove any files, if i ask u to fix the found issues during a tcr!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DavidTielke)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DavidTielke)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
