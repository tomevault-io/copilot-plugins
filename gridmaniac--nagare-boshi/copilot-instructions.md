## nagare-boshi

> - `components/` - Vue components


# Nagare Boshi Project Conventions

## Project Structure

### Core Directories

- `components/` - Vue components
- `composables/` - Vue composables and shared logic
- `mutations/` - Data modification operations
- `queries/` - Data fetching operations
- `pages/` - Route components
- `server/` - Backend API and models
- `utils/` - Utility functions and constants

## Naming Conventions

### Files

1. **Vue Components**

   - PascalCase for component files (e.g., `Card.vue`, `TextMeta.vue`)
   - Each component has its own file
   - Component names are descriptive and purpose-focused

2. **Composables**

   - camelCase with `use` prefix (e.g., `useDictionary.ts`, `useFile.ts`)
   - TypeScript files with `.ts` extension

3. **Mutations**

   - camelCase with `use` prefix
   - Action-oriented naming (e.g., `useRunBatch.ts`, `useGenerateFeedback.ts`)
   - Follows pattern: `use[Action][Entity].ts`

4. **Queries**
   - camelCase with `use` prefix
   - Entity-focused naming (e.g., `useCards.ts`, `useCard.ts`)
   - Singular for single item queries, plural for collections

### Code Organization

## Components

1. **Component Structure**

   - Modular components with specific responsibilities
   - Shared components in root `/components`
   - Feature-specific components in feature directories
   - Icon components organized in dedicated `/Icon` directory

2. **Component Patterns**
   - Reusable UI components (Card, Modal, Menu)
   - Feature components (Challenge, Stats)
   - Utility components (TextMeta, ShootingStars)

## Data Management

1. **Mutations**

   - Single responsibility principle
   - Clear action naming
   - Separate files for different operations on same entity
   - Common patterns:
     - Create: `useCreate[Entity].ts`
     - Update: `useUpsert[Entity].ts`
     - Delete: `useDelete[Entity].ts`

2. **Queries**
   - Entity-focused data fetching
   - Separate queries for lists vs single items
   - Related data queries (e.g., `useRelatedCards.ts`)
   - Consistent return type patterns

## Utils

1. **Organization**
   - Small, focused utility functions
   - Common helpers (clipboard, delay)
   - Constants in separate file
   - Type definitions when needed
   - Batch processing utilities

## TypeScript Usage

- Strict type checking enabled
- Type definitions in `global.d.ts`
- Custom type shims in `shims-context.d.ts`
- Consistent use of TypeScript throughout the project

## File Structure Patterns

1. **Pages**

   - Index routes in `index.vue`
   - Dynamic routes in bracketed directories (e.g., `[deckId]/`)
   - Nested routing when needed

2. **Server**
   - API routes in `api/` directory
   - Data models in `models/` directory
   - Separate TypeScript configuration

## Best Practices

1. **Code Organization**

   - Single responsibility principle
   - Clear separation of concerns
   - Consistent file naming
   - Modular architecture

2. **Component Design**

   - Reusable components
   - Clear props and emits
   - Consistent styling patterns
   - Logical component hierarchy

3. **State Management**

   - Composables for shared logic
   - Clear data flow
   - Separation of queries and mutations
   - Consistent error handling
   - URL query parameters should be synchronized with relevant state
   - Use router.replace() instead of push() for URL updates to avoid history pollution

4. **Element References**

   - Use useTemplateRef for template refs
   - Suffix element refs with 'El' (e.g., textEl, listEl, bottomEl)
   - When handling text input elements, implement cursor position management for special character insertions
   - Prefer direct ref usage over passing refs as parameters

5. **Type Safety**
   - TypeScript throughout
   - Clear interface definitions
   - Proper type exports
   - Consistent type naming

## Development Workflow

1. **Project Setup**

   - Nuxt.js framework
   - TypeScript configuration
   - Environment variables
   - Git ignore patterns

2. **Code Style**
   - Consistent file structure
   - Clear naming conventions
   - Modular architecture
   - Type safety first
   - No comments for self-explanatory code
   - Only add comments when explaining complex logic, non-obvious business rules, or workarounds
   - Use clear naming and code structure to make the code self-documenting

This documentation serves as a living guide for maintaining consistency across the project. Update it as new patterns emerge or conventions change.

# Nagare Boshi Project Conventions

## Project Structure

### Core Directories

- `components/` - Vue components
- `composables/` - Vue composables and shared logic
- `mutations/` - Data modification operations
- `queries/` - Data fetching operations
- `pages/` - Route components
- `server/` - Backend API and models
- `utils/` - Utility functions and constants

## Naming Conventions

### Files

1. **Vue Components**

   - PascalCase for component files (e.g., `Card.vue`, `TextMeta.vue`)
   - Each component has its own file
   - Component names are descriptive and purpose-focused

2. **Composables**

   - camelCase with `use` prefix (e.g., `useDictionary.ts`, `useFile.ts`)
   - TypeScript files with `.ts` extension

3. **Mutations**

   - camelCase with `use` prefix
   - Action-oriented naming (e.g., `useRunBatch.ts`, `useGenerateFeedback.ts`)
   - Follows pattern: `use[Action][Entity].ts`

4. **Queries**
   - camelCase with `use` prefix
   - Entity-focused naming (e.g., `useCards.ts`, `useCard.ts`)
   - Singular for single item queries, plural for collections

### Code Organization

## Components

1. **Component Structure**

   - Modular components with specific responsibilities
   - Shared components in root `/components`
   - Feature-specific components in feature directories
   - Icon components organized in dedicated `/Icon` directory

2. **Component Patterns**
   - Reusable UI components (Card, Modal, Menu)
   - Feature components (Challenge, Stats)
   - Utility components (TextMeta, ShootingStars)

## Data Management

1. **Mutations**

   - Single responsibility principle
   - Clear action naming
   - Separate files for different operations on same entity
   - Common patterns:
     - Create: `useCreate[Entity].ts`
     - Update: `useUpsert[Entity].ts`
     - Delete: `useDelete[Entity].ts`

2. **Queries**
   - Entity-focused data fetching
   - Separate queries for lists vs single items
   - Related data queries (e.g., `useRelatedCards.ts`)
   - Consistent return type patterns

## Utils

1. **Organization**
   - Small, focused utility functions
   - Common helpers (clipboard, delay)
   - Constants in separate file
   - Type definitions when needed
   - Batch processing utilities

## TypeScript Usage

- Strict type checking enabled
- Type definitions in `global.d.ts`
- Custom type shims in `shims-context.d.ts`
- Consistent use of TypeScript throughout the project

## File Structure Patterns

1. **Pages**

   - Index routes in `index.vue`
   - Dynamic routes in bracketed directories (e.g., `[deckId]/`)
   - Nested routing when needed

2. **Server**
   - API routes in `api/` directory
   - Data models in `models/` directory
   - Separate TypeScript configuration

## Best Practices

1. **Code Organization**

   - Single responsibility principle
   - Clear separation of concerns
   - Consistent file naming
   - Modular architecture

2. **Component Design**

   - Reusable components
   - Clear props and emits
   - Consistent styling patterns
   - Logical component hierarchy

3. **State Management**

   - Composables for shared logic
   - Clear data flow
   - Separation of queries and mutations
   - Consistent error handling
   - URL query parameters should be synchronized with relevant state
   - Use router.replace() instead of push() for URL updates to avoid history pollution

4. **Element References**

   - Use useTemplateRef for template refs
   - Suffix element refs with 'El' (e.g., textEl, listEl, bottomEl)
   - When handling text input elements, implement cursor position management for special character insertions
   - Prefer direct ref usage over passing refs as parameters

5. **Type Safety**
   - TypeScript throughout
   - Clear interface definitions
   - Proper type exports
   - Consistent type naming

## Development Workflow

1. **Project Setup**

   - Nuxt.js framework
   - TypeScript configuration
   - Environment variables
   - Git ignore patterns

2. **Code Style**
   - Consistent file structure
   - Clear naming conventions
   - Modular architecture
   - Type safety first
   - No comments for self-explanatory code
   - Only add comments when explaining complex logic, non-obvious business rules, or workarounds
   - Use clear naming and code structure to make the code self-documenting

This documentation serves as a living guide for maintaining consistency across the project. Update it as new patterns emerge or conventions change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gridmaniac) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
