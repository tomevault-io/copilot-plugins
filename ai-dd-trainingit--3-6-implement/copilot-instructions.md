## client-page

> Generate a complete client page


# Client Resource Rules

## Overview
A complete client page consists of:
1. Domain entity types (using [client-entity.mdc](mdc:.cursor/rules/client-entity.mdc))
2. Repository for API operations (using [client-repository.mdc](mdc:.cursor/rules/client-repository.mdc))
3. Web Components (using [client-component.mdc](mdc:.cursor/rules/client-component.mdc))
4. Component registration in main application

## Implementation Process
To create a new client resource, follow these steps:

1. **Create Domain Entity**:
   Create a client entity following [client-entity.mdc](mdc:.cursor/rules/client-entity.mdc) rules:
   ```
   src/client/domain/{resource-name}.type.ts
   ```

2. **Create Repository**:
   Generate a repository using [client-repository.mdc](mdc:.cursor/rules/client-repository.mdc) rule:
   ```
   src/client/repositories/{resource-name}.repository.ts
   ```

3. **Create Components**:
   Generate components following [client-component.mdc](mdc:.cursor/rules/client-component.mdc):
   
   a. List Component:
   ```
   src/client/components/{resource-name}/{resource-name}-list.component.ts
   ```
   
   b. Detail Component:
   ```
   src/client/components/{resource-name}/{resource-name}-detail.component.ts
   ```
   
   c. Form Component (for create/edit):
   ```
   src/client/components/{resource-name}/{resource-name}-form.component.ts
   ```

4. **Register Components**:
   Update the component registry in the main application:
   ```
   src/client/components/register-components.ts
   ```
   Add:
   ```typescript
   import { EntityListComponent } from "./entity-name/entity-name-list.component";
   import { EntityDetailComponent } from "./entity-name/entity-name-detail.component";
   import { EntityFormComponent } from "./entity-name/entity-name-form.component";
   
   // Register components
   customElements.define('entity-list', EntityListComponent);
   customElements.define('entity-detail', EntityDetailComponent);
   customElements.define('entity-form', EntityFormComponent);
   ```

5. **Add to Navigation** (optional):
   Update the navigation component to include links to the new resource.

## Component Responsibilities

1. **List Component**:
   - Display all entities in a list format
   - Handle entity selection (navigation to detail)
   - Include create new entity button
   - Support filtering/searching if appropriate

2. **Detail Component**:
   - Display detailed view of a single entity
   - Include edit and delete actions
   - Navigate back to list view
   - Refresh data as needed

3. **Form Component**:
   - Handle both create and edit operations
   - Validate input data client-side
   - Submit data to repository
   - Show validation errors
   - Support cancel operation

## State Management
- Pass entity ID via attributes for detail/edit views
- Use custom events for communication between components
- Consider using a shared state store for complex applications

## Routing (Optional)
- If using a router, update route definitions
- Handle parameterized routes for detail views

## Example Usage
```html
<!-- List view -->
<entity-list></entity-list>

<!-- Detail view -->
<entity-detail entity-id="123"></entity-detail>

<!-- Form view for create -->
<entity-form></entity-form>

<!-- Form view for edit -->
<entity-form entity-id="123"></entity-form>
```

---
> Source: [AI-DD-TrainingIT/3_6-implement](https://github.com/AI-DD-TrainingIT/3_6-implement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
