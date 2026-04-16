## core-ledger-ui

> You are an expert in TypeScript, Angular, and scalable web application development. You write functional, maintainable, performant, and accessible code following Angular and TypeScript best practices.


You are an expert in TypeScript, Angular, and scalable web application development. You write functional, maintainable, performant, and accessible code following Angular and TypeScript best practices.

## TypeScript Best Practices

- Use strict type checking
- Prefer type inference when the type is obvious
- Avoid the `any` type; use `unknown` when type is uncertain

## Angular Best Practices

- Always use standalone components over NgModules
- Must NOT set `standalone: true` inside Angular decorators. It's the default in Angular v20+.
- Use signals for state management
- Implement lazy loading for feature routes
- Do NOT use the `@HostBinding` and `@HostListener` decorators. Put host bindings inside the `host` object of the `@Component` or `@Directive` decorator instead
- Use `NgOptimizedImage` for all static images.
  - `NgOptimizedImage` does not work for inline base64 images.

## Accessibility Requirements

- It MUST pass all AXE checks.
- It MUST follow all WCAG AA minimums, including focus management, color contrast, and ARIA attributes.

### Components

- Keep components small and focused on a single responsibility
- Use `input()` and `output()` functions instead of decorators
- Use `computed()` for derived state
- Set `changeDetection: ChangeDetectionStrategy.OnPush` in `@Component` decorator
- Prefer inline templates for small components
- Prefer Reactive forms instead of Template-driven ones
- Do NOT use `ngClass`, use `class` bindings instead
- Do NOT use `ngStyle`, use `style` bindings instead
- When using external templates/styles, use paths relative to the component TS file.

## State Management

- Use signals for local component state
- Use `computed()` for derived state
- Keep state transformations pure and predictable
- Do NOT use `mutate` on signals, use `update` or `set` instead

## Templates

- Keep templates simple and avoid complex logic
- Use native control flow (`@if`, `@for`, `@switch`) instead of `*ngIf`, `*ngFor`, `*ngSwitch`
- Use the async pipe to handle observables
- Do not assume globals like (`new Date()`) are available.
- Do not write arrow functions in templates (they are not supported).

## Services

- Design services around a single responsibility
- Use the `providedIn: 'root'` option for singleton services
- Use the `inject()` function instead of constructor injection

## Mock API System (REQUIRED)

**IMPORTANT**: All API implementations MUST include corresponding mock data to support offline development.

### Requirements for New API Endpoints

When implementing any new API endpoint or entity:

1. **Create Mock Data File** in `src/app/api/mock-data/`:
   - Naming: `<entity-name>.mock.ts`
   - Export as `MOCK_<ENTITY_NAME>` constant
   - Include realistic data with edge cases (special characters, nulls, long text)
   - Add JSDoc with `@internal` tag

2. **Update Mock Data Index**: Add export to `src/app/api/mock-data/index.ts`

3. **Update MockApiService** in `src/app/api/mock-api.service.ts`:
   - Add Map storage for entity
   - Update `getDataMapForUrl()` with URL pattern
   - Update `getNextIdForUrl()` for auto-increment IDs
   - Update `reset()` method

4. **Create Production Stub**: Add export to `src/app/api/mock-data.production.ts`

5. **Write Tests**: Test CRUD operations and interceptor routing

### Mock Data Best Practices

- **Realistic Data**: Use actual formats, production-like values
- **Edge Cases**: Special characters, empty strings, nulls, max lengths
- **Variety**: Multiple records with different statuses/types
- **Relationships**: Foreign keys must reference actual entities
- **Timestamps**: ISO 8601 format (`2024-12-01T14:30:00Z`)

### Example

```typescript
import { Entity } from '../../models/entity.model';

/**
 * Mock entity data for local development and testing.
 * @internal
 */
export const MOCK_ENTITIES: Entity[] = [
  {
    id: 1,
    name: 'Standard Entity',
    status: EntityStatus.Active,
    createdAt: '2020-01-15T10:00:00Z',
  },
  {
    id: 2,
    name: 'Entity with Special Chars: €$£¥',
    status: EntityStatus.Active,
    createdAt: '2023-06-15T12:00:00Z',
  },
];
```

**See `documentation/mock-api.md` for complete documentation.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jlagedo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
