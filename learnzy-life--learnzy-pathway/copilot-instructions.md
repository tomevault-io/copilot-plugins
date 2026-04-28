## learnzy-pathway

> The application follows these data flow patterns:

# Data Flow Patterns

The application follows these data flow patterns:

1. **Services** - [src/services/](mdc:src/services)
   - Handle API calls and external data fetching
   - Manage business logic and async operations
   - Services should be stateless and focused on data operations
   - Each service should be organized by domain/feature
   - Use Supabase for database operations with project ID: duvqdpyxrkdmoyizepqv

2. **Hooks** - [src/hooks/](mdc:src/hooks)
   - Manage component state and lifecycle
   - Abstract complex stateful logic from components
   - Provide data and functionality to components
   - Should be reusable across multiple components
   - Example: [src/hooks/useResultsData.ts](mdc:src/hooks/useResultsData.ts)

3. **Context** - [src/context/](mdc:src/context)
   - Provide global or shared state
   - Use for data that needs to be accessed by many components
   - Should be limited to prevent prop drilling

4. **Data Flow**
   - Services fetch data from APIs/Supabase
   - Hooks consume services and manage local state
   - Components consume hooks and render UI
   - Context provides global state where needed

This separation of concerns ensures maintainable, testable code with clear responsibilities.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/learnzy-life) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
