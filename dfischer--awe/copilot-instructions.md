## awe

> Okay, here's an AGENTS.md file outlining coding guidelines for the 'awe' project. This document is intended to ensure consistency, readability, and maintainability across the codebase for both Unreal Engine and Max for Live components.

Okay, here's an AGENTS.md file outlining coding guidelines for the 'awe' project. This document is intended to ensure consistency, readability, and maintainability across the codebase for both Unreal Engine and Max for Live components.
awe: Ableton Unreal Engine Integration - Coding Guidelines (AGENTS.md)
1. Introduction
This document outlines the coding guidelines and best practices for all contributors to the 'awe' project. Adhering to these guidelines will help maintain code quality, improve collaboration, and ensure the long-term stability and maintainability of the integration between Ableton Live and Unreal Engine. All agents (developers, contributors) are expected to follow these guidelines.
2. General Principles
 * Readability: Write code that is easy to read and understand. Prioritize clarity over overly clever or obscure solutions.
 * Consistency: Strive for consistency in naming, formatting, and architectural patterns throughout the project.
 * Simplicity (KISS): Keep It Simple, Stupid. Avoid unnecessary complexity.
 * Don't Repeat Yourself (DRY): Avoid code duplication. Utilize functions, classes, and reusable components.
 * Commenting:
   * Comment code that is complex, non-obvious, or critical.
   * Explain why something is done, not just what is being done (if the what is clear from the code).
   * Keep comments up-to-date with code changes.
 * Modularity: Design components to be as self-contained and reusable as possible.
 * Performance: Be mindful of performance implications, especially for real-time operations. Profile and optimize critical code paths.
 * Error Handling: Implement robust error handling and provide clear feedback to users or logs when errors occur.
3. Unreal Engine Specific Guidelines
Follow Epic Games' official coding standards and guidelines where applicable.
3.1. C++
 * Naming Conventions:
   * Adhere to Epic Games' C++ Naming Conventions (e.g., UClassName, FStructName, EMyEnum, bBooleanVariable, FunctionName(), VariableName).
   * Use clear, descriptive names for classes, functions, variables, and files.
 * Formatting:
   * Follow Epic's C++ code formatting style (indentation, bracing, spacing).
   * Use PascalCase for class, struct, enum, and function names.
   * Use PascalCase (with a b prefix for booleans) for member variables.
   * Use camelCase for local variables and function parameters.
 * Headers (.h files):
   * Use #pragma once for include guards.
   * Minimize include dependencies. Use forward declarations where possible.
   * Organize includes: Engine-specific, Plugin-specific, Standard library.
 * Comments:
   * Use // for single-line comments and /*... */ for multi-line comments.
   * Use Doxygen-style comments for documenting classes, functions, and significant variables for API documentation generation.
 * Performance:
   * Avoid unnecessary operations in performance-critical loops (e.g., Tick functions).
   * Use appropriate data structures for the task.
   * Profile code using Unreal Insights to identify bottlenecks.
 * Memory Management:
   * Understand and correctly use Unreal Engine's UObject memory management and garbage collection.
   * Use smart pointers (TSharedPtr, TUniquePtr, TWeakPtr) for non-UObject memory management where appropriate.
 * Logging:
   * Use UE_LOG for logging messages. Define appropriate log categories for the 'awe' plugin.
   * Use different verbosity levels (Error, Warning, Log, Verbose) appropriately.
3.2. Blueprints
 * Naming Conventions:
   * Prefix Blueprint assets appropriately (e.g., BP_MyActor, WBP_MyWidget, EUW_MyEditorTool, AS_MySound, SK_MySkeleton).
   * Variables: PascalCase (e.g., MyVariable, bIsActive).
   * Functions/Events: PascalCase (e.g., HandleInput, OnDataReceived).
 * Graph Organization:
   * Keep graphs clean, organized, and easy to follow.
   * Use comments (C key) to explain sections of logic.
   * Align nodes and wires neatly. Use reroute nodes to improve readability.
   * Break down complex logic into smaller functions or macros.
   * Group related nodes using comment boxes.
 * Variables:
   * Use appropriate data types.
   * Provide clear tooltips for public variables.
   * Organize variables into categories in the "My Blueprint" panel.
 * Performance:
   * Be mindful of operations performed on Event Tick. Minimize logic in Tick or use timers/event-driven approaches where possible.
   * Avoid unnecessary casting.
   * Understand the cost of Blueprint function calls, especially across different Blueprints.
 * Compilation:
   * Always compile Blueprints after making changes.
   * Address all compiler warnings and errors.
3.3. UMG (Unreal Motion Graphics) & Editor Utility Widgets (EUW) [1, 2, 3, 4, 5, 6, 7, 8]
 * Naming Conventions:
   * Widgets in the hierarchy should have clear, descriptive names (e.g., Button_Submit, Text_StatusMessage, ListView_ParameterMappings).
 * Hierarchy:
   * Maintain a clean and logical widget hierarchy in the UMG Designer.
 * Styling & Consistency:
   * Strive for visual consistency with the Unreal Editor's UI, especially for Editor Utility Widgets.
   * Use styles and themes where appropriate to maintain a consistent look and feel.
   * Follow UMG best practices for layout (e.g., using Fill vs. Auto sizing, Scale Boxes vs. Render Transforms for permanent scaling).[3]
 * Event Handling:
   * Use specific event dispatchers rather than relying heavily on Event Tick for UI updates.
 * EUW Specifics: [1, 7, 8]
   * EUWs are for editor tools. Ensure they are robust and do not cause editor instability.
   * Clearly indicate the purpose and usage of the EUW.
   * Parameter mapping UIs should be intuitive, allowing users to easily discover and connect sources and destinations.[7, 8] Consider dynamic lists and potentially node-graph interfaces for complex mappings.[2, 5]
4. Max for Live Specific Guidelines [9]
Refer to Ableton's official "Max for Live Production Guidelines" where applicable for comprehensive details.[9]
4.1. Patching
 * Clarity & Organization: [9]
   * Patches should be clean, legible, and well-organized.
   * Use comments within the Max patch to explain complex sections or non-obvious logic.
   * Align objects and patch cords neatly.
   * Group related objects using panel or comment objects.
 * Naming Conventions:
   * Local Naming: For named objects intended to be local to a device (e.g., send, receive, coll, buffer~), prefix the name with three dashes (---) (e.g., [s ---oscOutput], [coll ---mappings]).[9] Max will replace --- with a unique identifier for that device instance.
   * Use clear and descriptive names for send/receive objects, colls, buffer~s, etc.
 * Performance:
   * Be mindful of CPU usage, especially for devices intended for real-time use.
   * Avoid unnecessary calculations or message handling in high-frequency signal paths.
   * Use efficient Max objects and techniques.
 * Error Handling:
   * Implement basic error checking (e.g., for network connections, expected data types).
   * Provide feedback to the user via the Max console or UI elements if errors occur.
4.2. UI/UX Design [10, 9]
 * Consistency with Ableton Live: [9]
   * Design device UIs to feel familiar and consistent with Ableton Live's native devices.
   * Use standard Max UI objects that integrate well with Live's look and feel.
 * Parameter Presentation: [9]
   * Use live.* UI objects for parameters that need to be automated, mapped, and saved with the Live Set.
   * Short Name vs. Long Name: Use the 'Short Name' for concise labels in the UI and the 'Long Name' for more descriptive tooltips in the Max Inspector.[9]
   * Parameter Types: Use appropriate parameter types (Int, Float, Enum) as defined in the Max Inspector.[9]
 * Device State & Presets: [9]
   * Ensure all user-configurable parameters are correctly stored and recalled with the Ableton Live Set or device presets.
   * Test saving and loading Live Sets containing the device with modified parameters.
   * Parameter Visibility should be set to "Automated and Stored" or "Stored Only" for parameters that need to be saved.[9]
 * Device Window:
   * Consider using "Fixed Initial Window Location" or "Set Device Width" for a consistent device presentation.[9]
4.3. OSC Communication
 * Address Naming:
   * Define a clear, consistent, and hierarchical OSC address scheme for communication between Max for Live and Unreal Engine.
   * Example: /awe/track/[track_index]/param/[param_name]
   * Document the OSC address scheme thoroughly.
 * Data Types:
   * Ensure data types are handled correctly on both the sending (Max for Live) and receiving (Unreal Engine) ends.
   * Normalize data where appropriate (e.g., 0.0-1.0 for float parameters).
 * Configuration:
   * Provide a clear and simple way for users to configure network settings (IP address, port) in the Max for Live device.
5. Version Control (Git)
 * Commit Messages:
   * Write clear, concise, and descriptive commit messages.
   * Follow a consistent format (e.g., conventional commits: feat: Add new OSC mapping feature).
   * Reference issue numbers if applicable.
 * Branching:
   * Develop features and bug fixes in separate branches (e.g., feature/my-new-feature, fix/bug-in-osc-parser).
   * Keep branches focused and short-lived.
   * Merge branches into develop (or a similar integration branch) via Pull Requests.
   * Ensure main (or master) branch always reflects a stable, releasable state.
 * .gitignore:
   * Ensure appropriate Unreal Engine and Max/MSP files/folders are ignored (e.g., DerivedDataCache, Intermediate, Saved, Max build folders, user-specific settings).
6. Code Review
 * All code changes should be reviewed by at least one other contributor before being merged into the main development branch.
 * Reviewers should check for adherence to these guidelines, correctness, performance, and potential issues.
 * Provide constructive feedback during reviews.
 * Authors should address review comments and ensure their changes are well-tested.
7. Documentation
 * In-Code Documentation: As described in the language-specific sections (C++, Blueprints, Max).
 * User Documentation: Create and maintain clear documentation for end-users on how to install, configure, and use 'awe'. This includes setup for both Ableton Live/Max for Live and Unreal Engine.
 * API Documentation: If 'awe' exposes APIs for other developers, ensure these are well-documented.
8. Testing
 * Unit Tests: Where feasible, write unit tests for critical C++ functions and potentially complex Blueprint logic.
 * Integration Tests: Test the end-to-end communication between Ableton Live and Unreal Engine thoroughly.
   * Verify correct data transmission and parameter mapping for various data types (triggers, continuous data).
   * Test both online (real-time) and offline (baked automation) workflows.
 * User Acceptance Testing (UAT): [11]
   * Define acceptance criteria for key features based on user stories.
   * Test features against these criteria from the perspective of the defined user personas.
 * Performance Testing: Benchmark latency and resource usage under various conditions.
This AGENTS.md provides a comprehensive starting point. It should be considered a living document and may be updated as the 'awe' project evolves and new best practices emerge.

---
> Source: [dfischer/awe](https://github.com/dfischer/awe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
