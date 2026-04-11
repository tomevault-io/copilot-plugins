## lineschedulenotifier

> This document defines the coding standards and project-specific constraints for this Ktor project. All AI-generated code and documentation must strictly adhere to these rules.

# Project Coding Guidelines and Instructions

This document defines the coding standards and project-specific constraints for this Ktor project. All AI-generated code and documentation must strictly adhere to these rules.

## 1. General Constraints
- **Language:** Use English (ASCII characters only) for all content, including code, comments, KDoc, and documentation (Markdown).
- **Character Sets:** Do not use Japanese characters, emojis, or any non-ASCII symbols.
- **Line Endings:** Use `LF` (Line Feed) for all files.

## 2. Formatting and Style
- **Code Style:** Refer to the `.editorconfig` file located in the project root for detailed formatting rules (indentation, spacing, etc.).
- **Imports:** - Include only necessary import statements.
    - **No Wildcard Imports:** Do not use asterisks (`*`) for imports; specify each class or function explicitly.
- **Null Safety:** Avoid using the not-null assertion operator (`!!`). Instead, prefer safe calls (`?.`), the Elvis operator (`?:`), or explicit checks that throw meaningful exceptions (e.g., `requireNotNull`, `checkNotNull`).

## 3. File Header and Copyright
- **Copyright Notice:** Every new class or file must include the following Apache License 2.0 header.
- **Dynamic Date:** Replace `$current_year` with the actual year of creation (e.g., 2026).

```kotlin
/*
 * Copyright (c) $current_year SallyLueNoa
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
```

## 4. Documentation
- **KDoc:** Provide clear and concise KDoc for all public classes, interfaces, and functions.
- **Override Methods:** Do not provide KDoc descriptions for overridden methods (methods with the `override` keyword), as they inherit documentation from their supertype.
- **Test Code:** Do not provide KDoc for test classes or test methods.
- **Test Method Naming:** Name test functions descriptively to convey their purpose. Follow the pattern `test<TargetMethod>_<Condition>`, where `<TargetMethod>` is the method under test and `<Condition>` describes the test case (e.g., `testHandleWebhook_success`, `testExecutePush_invalidData`).
- **Tone:** Keep all technical communication professional and objective.

## 5. Development Workflow
- When generating new files, always ensure they are placed in the appropriate package structure.
- Adhere to the existing architectural patterns (e.g., Routing, Dependency Injection) found within the project.

## 6. Architectural Reference
- **Design Diagrams:** The project's design specifications are located in `docs/designs/images` as PlantUML (`.puml`) files.
- **Verification:** Before generating or modifying code, refer to the System Architecture, Class Diagrams, and Sequence Diagrams in that directory to ensure structural and behavioral consistency.
- **Consistency:** All generated classes, methods, and interactions must align with the definitions provided in the UML diagrams.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sallyluenoa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sallyluenoa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
