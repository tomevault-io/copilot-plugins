## spendcraft

> 1. General Principles


1. General Principles

Adhere to Clean Architecture: Separate concerns into layers (Presentation, Domain, Data). Use cases in the domain layer must be independent of UI or data implementations. Repositories abstract data sources.
MVVM Pattern: All UI-related logic must use ViewModels for state management, with Jetpack Compose for views. Avoid direct UI updates from repositories or use cases.
Kotlin-First Development: Write all new code in Kotlin 2.0.0. Leverage Kotlin features like coroutines, flows, sealed classes, and extension functions for concise, type-safe code.
Modularity: Add new features as separate modules under :feature if they introduce distinct functionality. Reuse shared components from :ui, :core, etc., without creating circular dependencies.
Error Handling: Use Kotlin's Result or sealed classes for errors. Handle exceptions gracefully, especially in authentication and database operations, with user-friendly messages.
Performance: Optimize for mobile constraints. Use coroutines for async tasks, avoid blocking calls on the main thread, and profile with Android Profiler if needed.
Security: Never store sensitive data (e.g., passwords) in plain text. Use hashing for passwords and secure session management via :core-session.

2. Code Style and Conventions

Naming: Follow Kotlin conventions—camelCase for variables/functions, PascalCase for classes. Use descriptive names (e.g., noteRepository instead of repo).
Indentation and Formatting: 4-space indentation. Use Kotlin's standard formatting; run ktlint or similar for checks.
Comments: Use KDoc for public APIs. Comment complex logic but avoid redundant comments (e.g., no "getter for note" on simple getters).
Imports: Organize imports alphabetically. Avoid wildcard imports except for Compose or testing utilities.
File Organization: Group related classes in packages (e.g., data.repository, domain.usecase). Keep files under 500 lines; split if necessary.

3. Dependency Management

Use Version Catalogs: Declare all dependencies in libs.versions.toml. Do not hardcode versions in build.gradle.kts files.
Injection: Use Hilt for all dependency injection. Annotate with @HiltViewModel for ViewModels, @Module for providers.
Allowed Libraries: Stick to the existing tech stack. For new needs:

UI: Jetpack Compose, Material 3, Coil.
Data: Room, DataStore, Coroutines/Flow.
Testing: JUnit, Mockito, Espresso.
Do not introduce new libraries without justification (e.g., no Retrofit if not needed for networking).


SDK Levels: Ensure compatibility with minSdk 26, targetSdk 34. Use BuildCompat for version-specific features.

4. Feature Development Rules

Notes Management:

Entities: Extend or relate to Note, Category, Tag, NoteTagCrossRef. Include timestamps and metadata (pinned, archived, favorite).
Operations: Use repositories for CRUD. Implement search/filtering via Room queries or Flows.
UI: Compose-based screens with state hoisting. Support rich text via custom Composables.


User Management:

Authentication: Use :feature:login patterns. Validate passwords securely.
Profiles: Store avatars as URIs or byte arrays in UserDetail.
Sessions: Manage via :core-session; invalidate on logout.


Organization Features:

Categories/Tags: Maintain many-to-many relationships. Support color-coding for tags.
Search: Implement fuzzy search if needed, but start with exact matches via Room.


New Features:

Create a new :feature:<name> module.
Define domain models/use cases in :core-domain.
Implement data sources in :core-data or :core-database.
Add navigation entries in :core-navigation.
Ensure backward compatibility with existing entities.
Follow the package structure like in the existing feature modules.



5. Testing Guidelines

Unit Tests: Cover 80%+ of domain and data layers. Use JUnit 4, Mockito for mocks, Coroutines Test for async.
UI Tests: Use Espresso for instrumentation tests on Compose screens.
Integration Tests: Test repository flows with in-memory Room databases.
Mocking: Mock external dependencies (e.g., databases) to isolate tests.
Run Tests: All new code must pass existing tests. Add tests for new functionality before implementation.

6. Build and Deployment

Gradle: Use Kotlin DSL. Enable ProGuard for release builds.
Variants: Support debug/release builds. Use buildConfigFields for environment-specific configs.
CI/CD: Assume standard Android CI; ensure code builds without errors.
Versioning: Follow Semantic Versioning for app versions.

7. AI-Specific Interaction Rules

Code Generation: When generating code, provide complete, compilable snippets. Include imports, Hilt injections, and tests.
Queries: If analyzing or modifying code, reference specific modules/files. Use tools like code execution for validation if available.
Limitations: Do not introduce breaking changes (e.g., altering entity schemas without migrations). If unsure, suggest human review.
Output Format: For code suggestions, use Markdown code blocks with file paths (e.g., app/src/main/kotlin/com/memcloud/NoteViewModel.kt).
Ethical Considerations: Ensure generated code respects user privacy and security features.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/theturker)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/theturker)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
