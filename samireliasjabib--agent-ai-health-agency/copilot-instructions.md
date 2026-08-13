## sofware-arquitecture-rule

> - Use folder-based routing: `routes/dashboard/route.tsx` instead of `routes/dashboard.tsx`

## 🏗️ Clean Architecture Rules

### **File Organization Rules**

#### **Routes Structure**
- Use folder-based routing: `routes/dashboard/route.tsx` instead of `routes/dashboard.tsx`
- Each route folder contains only `route.tsx`
- Keep routes thin - only data loading and basic logic
- Move all business logic to services

#### **Components Structure**
- `components/ui/` → Only shadcn/ui components (no custom logic)
- NO custom dashboard components in components folder
- Build dashboard UI directly in route files using shadcn/ui components
- Compose complex UIs using multiple shadcn components

#### **Lib Structure - Layered Architecture**
```
lib/
├── services/       # Business logic (pure functions)
├── repositories/   # Data access (Prisma queries)
├── hooks/          # React hooks (state + effects)
├── helpers/        # Pure utility functions
├── constants/      # Static data and config
├── types/          # TypeScript interfaces
└── utils.ts        # shadcn utilities only
```

### **Layer Responsibilities**

#### **Services Layer (`lib/services/`)**
- Pure business logic functions
- No React hooks or components
- No direct database calls (use repositories)
- Input validation and data transformation
- Example: `dashboard.service.ts`, `auth.service.ts`

#### **Repositories Layer (`lib/repositories/`)**
- All Prisma database queries
- CRUD operations only
- No business logic
- Return raw data or Prisma models
- Example: `user.repository.ts`, `client.repository.ts`

#### **Hooks Layer (`lib/hooks/`)**
- React hooks that combine services + state
- Handle loading states and errors
- Connect services to React components
- Example: `use-dashboard.ts`, `use-clients.ts`

#### **Helpers Layer (`lib/helpers/`)**
- Pure utility functions
- No side effects
- Reusable across app
- Example: `date.helpers.ts`, `format.helpers.ts`

#### **Constants Layer (`lib/constants/`)**
- Static configuration
- Mock data for MVP
- Route definitions
- App configuration
- Example: `routes.ts`, `config.ts`, `mock-data.ts`

### **Code Style Rules**

#### **Functional Programming**
- Prefer pure functions over classes
- Immutable data transformations
- Function composition
- No side effects in business logic

#### **Import Rules**
```typescript
// Route files import order:
1. React/Remix imports
2. UI components (shadcn/ui)
3. Hooks (from lib/hooks)
4. Types (from lib/types)
5. Constants (from lib/constants)

// Service files import order:
1. External libraries
2. Types
3. Helpers
4. Repositories
```

#### **Naming Conventions**
- Services: `[domain].service.ts` 
- Repositories: `[domain].repository.ts`
- Hooks: `use-[feature].ts`
- Helpers: `[category].helpers.ts`
- Types: `[domain].types.ts`
- Constants: `[category].ts`

### **Data Flow Rules**

#### **Route → Hook → Service → Repository → Database**
```
route.tsx
  ├─ uses hook (useClients)
  │   ├─ calls service (clientService.getAll)
  │   │   ├─ calls repository (clientRepository.findAll)
  │   │   │   └─ executes Prisma query
  │   │   └─ transforms data
  │   └─ manages React state
  └─ renders UI with shadcn components
```

#### **No Direct Database Calls**
- Routes never call Prisma directly
- Services never call Prisma directly
- Only repositories call Prisma
- Pass data up through layers

#### **State Management**
- Use React hooks for local state
- No global state management for MVP
- Keep state close to where it's used
- Use custom hooks to share stateful logic

### **TypeScript Rules**

#### **Strict Typing**
- No `any` types
- Define interfaces for all data shapes
- Use Prisma generated types when possible
- Export types from dedicated type files

#### **Type Organization**
- Domain-specific types in separate files
- Share common types across domains
- Use `as const` for constants
- Prefer interfaces over types for objects

### **Testing Rules**

#### **Testable Architecture**
- Pure functions are easily testable
- Mock repositories in service tests
- Mock services in hook tests
- Keep side effects in outermost layers

### **Performance Rules**

#### **Efficient Data Loading**
- Use Remix loaders for initial data
- Implement optimistic updates
- Cache expensive computations
- Minimize database queries

### **Error Handling Rules**

#### **Layered Error Handling**
- Repositories: Handle database errors
- Services: Handle business logic errors
- Hooks: Handle async errors and loading states
- Routes: Handle user-facing errors

### **MVP Simplification Rules**

#### **Start Simple**
- Use mock data in constants
- Implement real services gradually
- Keep repository methods basic
- Add complexity incrementally

#### **No Premature Optimization**
- Don't over-engineer for MVP
- Focus on clean separation of concerns
- Build features as needed
- Refactor when patterns emerge

### **File Naming Examples**
```
routes/
├── dashboard/route.tsx
├── login/route.tsx
└── auth.google.callback/route.tsx

lib/
├── services/
│   ├── dashboard.service.ts
│   └── auth.service.ts
├── repositories/
│   ├── user.repository.ts
│   └── client.repository.ts
├── hooks/
│   ├── use-dashboard.ts
│   └── use-auth.ts
├── helpers/
│   ├── date.helpers.ts
│   └── format.helpers.ts
├── constants/
│   ├── routes.ts
│   └── mock-data.ts
└── types/
    ├── dashboard.types.ts
    └── client.types.ts
```

## 🎯 Key Principles

1. **Separation of Concerns**: Each layer has one responsibility
2. **Dependency Direction**: Always import down the stack
3. **Testability**: Pure functions are easy to test
4. **Maintainability**: Clear structure makes changes easy
5. **Scalability**: Add features without refactoring structure

Follow these rules to maintain clean, scalable, and maintainable code architecture.

---
> Source: [samireliasjabib/agent-ai-health-agency](https://github.com/samireliasjabib/agent-ai-health-agency) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
