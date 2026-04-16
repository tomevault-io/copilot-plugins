## genexus18platformsdk

> This project is an extension plugin for the **GeneXus 18 IDE**, designed to provide additional development and debugging tools. It integrates directly into the GeneXus environment to assist with variable management, logging, object analysis, and history export.

# GeneXus 18 Platform SDK - Menu Plugin (MenuV3)

## Project Overview
This project is an extension plugin for the **GeneXus 18 IDE**, designed to provide additional development and debugging tools. It integrates directly into the GeneXus environment to assist with variable management, logging, object analysis, and history export.

The codebase has been modernized to follow **Clean Architecture** and **SOLID principles**, moving away from legacy monolithic structures to a modular, testable design.

## Architecture
The project follows a strict Clean Architecture pattern:

*   **Core:** Contains the domain logic, entities, value objects, and interfaces. It has no dependencies on external libraries or infrastructure.
*   **Infrastructure:** Implements the interfaces defined in Core (e.g., GeneXus API interactions, Logging).
*   **Services:** specialized business logic services (Export, Analysis, Variables).
*   **Presentation:** UI components and Facades to expose functionality.
*   **Commands:** The entry points for GeneXus IDE commands.

## Technology Stack
*   **Language:** C# 7.3
*   **Framework:** .NET Framework 4.7.2
*   **SDK:** GeneXus 18 Platform SDK
*   **IDE:** Visual Studio 2017 or higher

## Building and Installation

### Prerrequisites
*   GeneXus 18 Platform SDK installed.
*   `GX_SDK_DIR`, `GX_PROGRAM_DIR_18_U10` environment variables (referenced in `.csproj` post-build events).

### Build Commands
1.  **Restore Dependencies:**
    ```bash
    nuget restore Menu.sln
    ```
2.  **Compile:**
    ```bash
    msbuild Menu.sln /p:Configuration=Release
    ```

### Post-Build Process
The project includes a post-build event that:
1.  Runs the GeneXus `Updater` tool to update the catalog.
2.  Installs the package into the GeneXus installation using `/install`.

## Key Features & Commands

| Command | Shortcut | Description |
| :--- | :--- | :--- |
| **Generate Log Debug Form** | `Ctrl+Shift+H` | Generates a debug logging form. |
| **Show Object History** | `Ctrl+Shift+G` | Export the history of objects in the KB. |
| **Export Table Structure** | `Ctrl+Shift+T` | Exports database table structures. |
| **Export Procedure Source** | `Ctrl+Shift+P` | Exports procedure source code. |
| **Extract Variables** | Context Menu | Context-aware variable extraction tools. |

## Development Conventions

*   **Clean Code:** Strictly adhere to SOLID principles.
*   **Modularity:** Keep functions small (< 20 lines) and classes focused (SRP).
*   **Naming:** Use descriptive, self-documenting names.
*   **Legacy Code:** New features should use the new architecture. Legacy files (e.g., `VariableHelper.cs`) are deprecated in favor of refactored counterparts (e.g., `VariableService`).

## Directory Structure
*   `Commands/`: GeneXus command definitions and handlers.
*   `Core/`: Domain logic and interfaces (Clean Architecture).
*   `Infrastructure/`: Implementations of Core interfaces.
*   `Services/`: Business logic implementations.
*   `UI/`: Windows Forms and user controls.
*   `Utilities/`: Helper classes (some legacy).

## 🌟 SDK Best Practices

1.  **Selection & Context**: Use `CommandData.Context` and check for `ISelectionContainer` for tree-based selection. Use `LSI.Packages.Extensiones.Utilidades.Entorno.CurrentEditingPart` for editor context.
2.  **Object Persistence**: Use `obj.Timestamp` for last modification and `obj.Save()` to commit changes.
3.  **Transaction Exploration**: Use `level.Structure.GetAttributes()` to iterate over attributes in any level. Use `trnAttr.Attribute` to access technical metadata (Type, Length).
4.  **Editor Interaction**: Avoid `dynamic` to prevent runtime errors. Use **Reflection** to call IDE methods like `Select(line, column)`.
5.  **Clipboard Robustness**: When simulating Copy/Paste, use retry loops with brief sleeps (e.g., 50ms) to ensure the Windows clipboard has time to update.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/williamgarciadev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
