## vortex-crud

> - Create custom repository methods

# AGENTS Instructions

## ⚠️ CRITICAL: This is vortex-crud, NOT vanilla Spring Boot!

### DO NOT:
- Create custom repository methods
- Create `@Service` classes for data access
- Use `@Transactional` for CRUD operations
- Create custom implementations like `UserDetailsService`

### DO:
- **Use `VortexCrudDataStore` directly for ALL data access**
- Look at existing implementations: `FormRouteFactory`, `FormDialogFactory`, `SignUpView`
- **Use `RouteRenderer.dataStoreInstance()` or `DataStoreConfig.dataStoreInstance()` to get DataStores**
- Use `ReflectionService` for field access, not direct field manipulation

---

## Key Pattern

```java
// Get DataStore from configuration
VortexCrudDataStore<FieldType, Object> dataStore = routeRenderer.dataStoreInstance();

// Create/save entities
Object entity = dataStore.newInstance();
// ... set values with reflectionService.setValue() ...
dataStore.insertRecord(entity);
```

---

## Common Pitfalls (Why Previous Agents Failed)

- Saw `UserRepository` exists → thought "I need to add methods to it" ❌
- Saw Spring Security → thought "I need a UserDetailsService" ❌
- Didn't look at `FormRouteFactory` to understand the data access pattern ❌
- Tried to create separate service classes instead of handling logic directly in Views ❌

**The framework is elegant once you understand it, but easy to get wrong coming from vanilla Spring Boot!**

### Golden Rule for New Features:
**When adding functionality (like authentication), implement it directly in the View using the same pattern as existing Views (SignUpView, FormRouteFactory).** Do NOT create separate `@Service` classes or traditional Spring components.

---

## Task Completion Checklist

1. ✅ Use VortexCrudDataStore (NOT custom repos/services)
2. ✅ Update examples (jpa-sqlite-example AND jooq-sqlite-example) if adding features
3. ✅ Update AGENTS.md if you learned something universally valuable

---
> Source: [appreciated/vortex-crud](https://github.com/appreciated/vortex-crud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
