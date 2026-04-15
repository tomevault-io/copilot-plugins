## ui-development-kit

> This file provides rules and patterns for Cursor AI when working with the SailPoint UI Development Kit codebase.

# UI Development Kit - Cursor Rules

This file provides rules and patterns for Cursor AI when working with the SailPoint UI Development Kit codebase.

## Creating New API Endpoints / Electron IPC Handlers

When creating new endpoints that need to communicate between the Angular frontend and Electron main process, you MUST follow the 5-step pattern documented at: https://developer.sailpoint.com/docs/tools/ui-development-kit/extending-services

### Step-by-Step Pattern

#### Step 1: Add the IPC Handler in Electron Main Process

**File:** `app/main.ts`

Add a new `ipcMain.handle()` inside the `//#region Custom IPC handlers` section:

```typescript
ipcMain.handle('my-new-method', async (event, param1: string, param2?: number) => {
  // Implement your logic here
  // You can:
  // - Make API calls using fetch
  // - Access the filesystem using fs
  // - Call other helper functions
  // - Return data to the renderer process
  
  try {
    const result = await someOperation(param1, param2);
    return { success: true, data: result };
  } catch (error) {
    console.error('Error in my-new-method:', error);
    return { success: false, error: error.message };
  }
});
```

**Naming Convention for IPC Channels:**
- Use kebab-case for channel names: `'my-new-method'`, `'get-user-data'`, `'save-config'`
- Be descriptive about the action: `'fetch-colab-posts'`, `'deploy-workflow'`, `'validate-transform'`

#### Step 2: Expose the Method in Preload Script

**File:** `app/preload.ts`

Add the method to the `contextBridge.exposeInMainWorld()` call:

```typescript
contextBridge.exposeInMainWorld('electronAPI', {
  // ... existing methods ...
  
  // Add your new method - match the channel name from Step 1
  myNewMethod: (param1: string, param2?: number) => ipcMain.invoke('my-new-method', param1, param2),
});
```

**Important:**
- The method name exposed (e.g., `myNewMethod`) should be camelCase
- Parameters must match what the IPC handler expects
- Use `ipcMain.invoke()` to call the handler defined in Step 1

#### Step 3: Add Type Definition to ElectronAPIInterface

**File:** `projects/sailpoint-components/src/lib/services/web-api.service.ts`

Add the method signature to the `ElectronAPIInterface`:

```typescript
export interface ElectronAPIInterface {
  // ... existing methods ...
  
  // Add your new method with full type definitions
  myNewMethod: (param1: string, param2?: number) => Promise<MyNewMethodResponse>;
}
```

**Type Definitions:**
- Always define response types for better type safety
- Define types in the same file or import from a shared types file:

```typescript
export type MyNewMethodResponse = {
  success: boolean;
  data?: MyDataType;
  error?: string;
};
```

#### Step 4: Implement the Web API Fallback (Optional but Recommended)

**File:** `projects/sailpoint-components/src/lib/services/web-api.service.ts`

If the application should work in web mode (without Electron), implement the web version:

```typescript
export class WebApiService implements ElectronAPIInterface {
  // ... existing methods ...
  
  async myNewMethod(param1: string, param2?: number): Promise<MyNewMethodResponse> {
    // Implement web-compatible version or proxy through backend
    return this.apiCall<MyNewMethodResponse>('my-new-method', 'POST', { param1, param2 });
  }
}
```

**Note:** The `apiCall` method proxies requests to the backend server defined in the `server/` directory.

#### Step 5: Use the API in Angular Components

**File:** Any Angular component or service (e.g., `*.component.ts`)

```typescript
import { Component } from '@angular/core';
import { ElectronApiFactoryService } from 'sailpoint-components';

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.html'
})
export class MyComponent {
  constructor(private apiFactory: ElectronApiFactoryService) {}

  async performAction(): Promise<void> {
    try {
      const result = await this.apiFactory.getApi().myNewMethod('test', 42);
      
      if (result.success) {
        console.log('Success:', result.data);
      } else {
        console.error('Failed:', result.error);
      }
    } catch (error) {
      console.error('Error calling myNewMethod:', error);
    }
  }
}
```

---

## Common Response Patterns

Always use consistent response patterns for IPC handlers:

### Success/Error Pattern
```typescript
// For operations that succeed or fail
return { success: true, data: resultData };
return { success: false, error: 'Error message' };
```

### Validation Pattern
```typescript
// For validation operations
return { isValid: true };
return { isValid: false, errors: ['Error 1', 'Error 2'] };
```

### Status Pattern
```typescript
// For status checks
return { 
  status: 'active' | 'inactive' | 'pending',
  details: { /* additional info */ }
};
```

---

## File Organization

When adding new features that require IPC handlers, organize code as follows:

```
app/
├── main.ts                      # IPC handlers go here
├── preload.ts                   # Expose handlers here
├── my-feature/                  # Feature-specific logic
│   ├── my-feature.ts            # Main feature logic
│   └── types.ts                 # TypeScript types

projects/sailpoint-components/src/lib/
├── services/
│   └── web-api.service.ts       # Interface + Web implementation
├── my-feature/                  # Angular component
│   ├── my-feature.component.ts
│   ├── my-feature.component.html
│   └── my-feature.service.ts    # Feature-specific service (optional)
```

---

## Component Development Guidelines

### Use Angular Material for All Components

**CRITICAL:** When creating new Angular components, you MUST use Angular Material components wherever possible instead of custom HTML elements or other UI libraries.

#### Required Angular Material Modules

Always import the necessary Angular Material modules in your component's `imports` array:

```typescript
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatIconModule } from '@angular/material/icon';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatProgressSpinnerModule } from '@angular/material/progress-spinner';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatSnackBarModule } from '@angular/material/snack-bar';
import { MatDialogModule } from '@angular/material/dialog';
import { MatTooltipModule } from '@angular/material/tooltip';
import { MatTableModule } from '@angular/material/table';
// ... and other Material modules as needed

@Component({
  selector: 'app-my-component',
  standalone: true,
  imports: [
    CommonModule,
    MatButtonModule,
    MatCardModule,
    MatIconModule,
    // ... other Material modules
  ],
  templateUrl: './my-component.html',
  styleUrl: './my-component.scss'
})
```

#### Angular Material Component Usage

**Use Material components instead of native HTML:**

✅ **DO:**
- `<button mat-raised-button>` instead of `<button>`
- `<mat-card>` instead of `<div class="card">`
- `<mat-icon>` instead of `<i>` or `<span>`
- `<mat-form-field>` with `<input matInput>` instead of plain `<input>`
- `<mat-spinner>` instead of custom loading spinners
- `<mat-toolbar>` instead of `<header>` or `<nav>`
- `<mat-snack-bar>` for notifications
- `<mat-dialog>` for modals

❌ **DON'T:**
- Use plain HTML buttons, inputs, or form elements
- Create custom card/container components when `mat-card` exists
- Use third-party UI libraries (Bootstrap, PrimeNG, etc.) without justification
- Mix Material with other UI frameworks

#### Examples

**Buttons:**
```html
<!-- ✅ Correct -->
<button mat-raised-button color="primary">Submit</button>
<button mat-button>Cancel</button>
<button mat-icon-button><mat-icon>delete</mat-icon></button>

<!-- ❌ Incorrect -->
<button class="btn btn-primary">Submit</button>
```

**Forms:**
```html
<!-- ✅ Correct -->
<mat-form-field>
  <mat-label>Email</mat-label>
  <input matInput type="email" [(ngModel)]="email">
  <mat-error>Invalid email</mat-error>
</mat-form-field>

<!-- ❌ Incorrect -->
<input type="email" [(ngModel)]="email" class="form-control">
```

**Cards:**
```html
<!-- ✅ Correct -->
<mat-card>
  <mat-card-header>
    <mat-card-title>Title</mat-card-title>
  </mat-card-header>
  <mat-card-content>Content</mat-card-content>
  <mat-card-actions>
    <button mat-button>Action</button>
  </mat-card-actions>
</mat-card>

<!-- ❌ Incorrect -->
<div class="card">
  <div class="card-header">Title</div>
  <div class="card-body">Content</div>
</div>
```

**Loading States:**
```html
<!-- ✅ Correct -->
<mat-spinner diameter="40" *ngIf="loading"></mat-spinner>

<!-- ❌ Incorrect -->
<div class="spinner" *ngIf="loading"></div>
```

#### When Material Components Are Not Available

If you need functionality that Angular Material doesn't provide:

1. **Check Material Extensions** - Look for `@angular/material` extensions or community packages
2. **Use Material Design Principles** - Style custom components to match Material Design guidelines
3. **Document the Exception** - Add a comment explaining why a Material component wasn't used

#### Theming

All Material components automatically respect the app's theme configuration. Use Material's theming system rather than custom CSS for colors, spacing, and typography when possible.

---

## Security Considerations

1. **Never expose Node.js APIs directly** - Always use the IPC pattern
2. **Validate all inputs** in the IPC handler before processing
3. **Sanitize file paths** when accessing the filesystem
4. **Use contextIsolation** - This is already enabled in the app
5. **Don't store secrets in renderer process** - Handle sensitive data in main process only

---

## Testing New Endpoints

1. **Unit Test the IPC Handler** - Test the main process logic independently
2. **Test the Angular Integration** - Use the `ElectronApiFactoryService` mock
3. **E2E Test** - Use Playwright to test the full flow

---

## Common Mistakes to Avoid

❌ **Don't** use `ipcRenderer` directly in Angular components
❌ **Don't** forget to add the type definition to `ElectronAPIInterface`
❌ **Don't** use inconsistent naming between handler and preload
❌ **Don't** forget error handling in async operations
❌ **Don't** expose sensitive data in error messages
❌ **Don't** use plain HTML elements when Angular Material equivalents exist
❌ **Don't** mix Angular Material with other UI frameworks (Bootstrap, PrimeNG, etc.)
❌ **Don't** create custom components for functionality Material already provides

✅ **Do** follow the 5-step pattern consistently
✅ **Do** use TypeScript types for all parameters and returns
✅ **Do** handle errors gracefully with informative messages
✅ **Do** test both Electron and Web modes if supporting both
✅ **Do** use Angular Material components for all UI elements where possible
✅ **Do** import Material modules in the component's `imports` array
✅ **Do** follow Material Design principles for custom styling

---

## Quick Reference: File Locations

| Step | File | What to Add |
|------|------|-------------|
| 1 | `app/main.ts` | `ipcMain.handle('channel-name', ...)` |
| 2 | `app/preload.ts` | `methodName: () => ipcMain.invoke('channel-name', ...)` |
| 3 | `projects/.../web-api.service.ts` | Interface method signature |
| 4 | `projects/.../web-api.service.ts` | Web implementation (optional) |
| 5 | `*.component.ts` | `this.apiFactory.getApi().methodName()` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sailpoint-oss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
