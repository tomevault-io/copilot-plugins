## client-component

> Generate a client web component


# Client Web Component Rules

## Naming Conventions
- File naming: Use `{component-name}.component.ts` in kebab-case
- Class naming: Use PascalCase with Component suffix (e.g., `ToolListComponent`)
- Component tag: Use kebab-case for custom element name (e.g., `tool-list`)
- Internal properties: Use camelCase prefixed with `_` for private fields
- Template property: Use `private template` for HTML template string
- Event handlers: Use on{Event} naming (e.g., `onClick`, `onSubmit`)

## Required Structure
1. **Component Class**:
   - Extend HTMLElement
   - Include a constructor that initializes the component
   - Export the class, but don't define the custom element in the file

2. **Lifecycle Methods**:
   - Implement `connectedCallback()` for setup and event listeners
   - Implement `disconnectedCallback()` for cleanup and event listener removal
   - Optionally implement `attributeChangedCallback()` for handling attribute changes

3. **Template Management**:
   - Use a private `private template` property for HTML content
   - Use the html helper tag for template literals
   - Add the template to innerHTML in the constructor

4. **Event Handling**:
   - Use private methods for event handlers
   - Properly bind 'this' context in event listeners
   - Remove event listeners in disconnectedCallback

5. **Web Rules**:
   - Respect general web rules from [web.mdc](mdc:.cursor/rules/web.mdc)     

## Imports
- Import domain entities or other types
- Import repository functions for data access
- Import other components if needed (for composition)

## State Management
- Use private class fields for component state
- Implement render method for updating the DOM based on state
- Consider using observables pattern for complex state management

## Documentation
- Add JSDoc comments to the component class
- Document public methods and properties
- Document events emitted by the component

## Example Structure
```typescript
import { EntityName } from "../domain/entity-name.type";
import { fetchAllEntities } from "../repositories/entity-name.repository";

const html = String.raw;

/**
 * Component for displaying and managing entities
 * Usage: <entity-list></entity-list>
 */
export class EntityListComponent extends HTMLElement {
  private template: string;
  private entities: EntityName[] = [];
  private deleteButton: HTMLButtonElement | null = null;

  constructor() {
    super();
    
    this.#template = html`
      <section>
        <h2>Entities</h2>
        <ul id="entity-list"></ul>
        <button id="refresh-button">Refresh</button>
      </section>
    `;
    
    this.innerHTML = this.#template;
    this.loadEntities();
  }
  
  private async loadEntities(): Promise<void> {
    try {
      this.entities = await fetchAllEntities();
      this.render();
    } catch (error) {
      console.error('Failed to load entities:', error);
    }
  }
  
  private render(): void {
    const listElement = this.querySelector('#entity-list');
    if (!listElement) return;
    
    listElement.innerHTML = '';
    
    this.#entities.forEach(entity => {
      const li = document.createElement('li');
      li.textContent = entity.name;
      listElement.appendChild(li);
    });
  }
  
  onRefreshClick = (): void => {
    this.loadEntities();
  };
  
  connectedCallback(): void {
    const refreshButton = this.querySelector('#refresh-button');
    if (refreshButton) {
      refreshButton.addEventListener('click', this.#onRefreshClick);
    }
  }
  
  disconnectedCallback(): void {
    const refreshButton = this.querySelector('#refresh-button');
    if (refreshButton) {
      refreshButton.removeEventListener('click', this.#onRefreshClick);
    }
  }
}

// Note: The component is not defined here
// Define in parent module with: customElements.define('entity-list', EntityListComponent);
```

---
> Source: [AI-DD-TrainingIT/3_6-implement](https://github.com/AI-DD-TrainingIT/3_6-implement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
