## mujdn-frontend

> This guide outlines a robust Angular architecture for implementing CRUD (Create, Read, Update, Delete) operations using a layered approach with abstract base classes, services, and components. The architecture promotes code reusability, consistency, and maintainability across different entities.

# Mujdn Frontend Angular CRUD Architecture Development Guide

This guide outlines a robust Angular architecture for implementing CRUD (Create, Read, Update, Delete) operations using a layered approach with abstract base classes, services, and components. The architecture promotes code reusability, consistency, and maintainability across different entities.

## Development server

To start a local development server, run:

```bash
ng serve
```

Once the server is running, open your browser and navigate to `http://localhost:4200/`. The application will automatically reload whenever you modify any of the source files.

## Code scaffolding

Angular CLI includes powerful code scaffolding tools. To generate a new component, run:

```bash
ng generate component component-name
```

For a complete list of available schematics (such as `components`, `directives`, or `pipes`), run:

```bash
ng generate --help
```

## Building

To build the project run:

```bash
ng build
```

This will compile your project and store the build artifacts in the `dist/` directory. By default, the production build optimizes your application for performance and speed.

## Architecture Components

### 1. **Model Layer**

- **Entity Model**: Extends `BaseCrudModel` for core CRUD functionality
- **Filter Model**: Defines search/filter criteria
- **Form Validation**: Built-in form building with validators

### 2. **Service Layer**

- **CRUD Service**: Extends `BaseCrudService` for HTTP operations
- **Model Interceptors**: Handle data transformation
- **URL Management**: Centralized endpoint configuration

### 3. **Component Layer**

- **List Component**: Extends `BaseListComponent` for data display
- **Popup Component**: Extends `BasePopupComponent` for add/edit operations
- **Routing & Guards**: Navigation and access control

### 4. **Supporting Elements**

- **Resolvers**: Pre-load data before route activation
- **Validators**: Custom form validation rules
- **Interceptors**: Request/response transformation

## File Structure Pattern

For each entity (e.g., "Category"), create these files:

```
models/
├── category.ts
└── category-filter.ts

services/
└── category.service.ts

model-interceptors/
└── category-interceptor.ts

resolvers/
└── categories.resolver.ts

components/
├── category-list/
│   ├── category-list.component.ts
│   ├── category-list.component.html
│   └── category-list.component.scss
└── category-popup/
    ├── category-popup.component.ts
    ├── category-popup.component.html
    └── category-popup.component.scss
```

## Step-by-Step Implementation Guide

### Step 1: Create the Entity Model

```typescript
// models/category.ts
import { BaseCrudModel } from '@/abstracts/base-crud-model';
import { CategoryService } from '@/services/category.service';
import { InterceptModel } from 'cast-response';
import { CategoryInterceptor } from '@/model-interceptors/category-interceptor';
import { Validators } from '@angular/forms';
import { CustomValidators } from '@/validators/custom-validators';

const { send, receive } = new CategoryInterceptor();

@InterceptModel({ send, receive })
export class Category extends BaseCrudModel<Category, CategoryService> {
  override $$__service_name__$$: string = 'CategoryService';

  // Entity properties
  declare nameAr: string;
  declare nameEn: string;
  declare description?: string;
  isActive: boolean = true;

  buildForm() {
    const { nameAr, nameEn, description, isActive } = this;
    return {
      nameAr: [
        nameAr,
        [
          Validators.required,
          Validators.maxLength(CustomValidators.defaultLengths.ARABIC_NAME_MAX),
          Validators.minLength(CustomValidators.defaultLengths.MIN_LENGTH),
          CustomValidators.pattern('AR_NUM'),
        ],
      ],
      nameEn: [
        nameEn,
        [
          Validators.required,
          Validators.maxLength(CustomValidators.defaultLengths.ENGLISH_NAME_MAX),
          Validators.minLength(CustomValidators.defaultLengths.MIN_LENGTH),
          CustomValidators.pattern('ENG_NUM'),
        ],
      ],
      description: [description, [Validators.maxLength(500)]],
      isActive: [isActive, []],
    };
  }
}
```

### Step 2: Create the Filter Model

```typescript
// models/category-filter.ts
export class CategoryFilter {
  declare nameAr: string | null;
  declare nameEn: string | null;
  declare description: string | null;
  declare isActive: boolean | null;
}
```

### Step 3: Create the Model Interceptor

```typescript
// model-interceptors/category-interceptor.ts
import { ModelInterceptorContract } from 'cast-response';
import { Category } from '@/models/category';

export class CategoryInterceptor implements ModelInterceptorContract<Category> {
  receive(model: Category): Category {
    // Transform data received from API
    return model;
  }

  send(model: Partial<Category>): Partial<Category> {
    // Transform data before sending to API
    return model;
  }
}
```

### Step 4: Create the Service

```typescript
// services/category.service.ts
import { BaseCrudService } from '@/abstracts/base-crud-service';
import { Category } from '@/models/category';
import { Injectable } from '@angular/core';
import { CastResponseContainer } from 'cast-response';
import { PaginatedList } from '@/models/shared/response/paginated-list';

@CastResponseContainer({
  $default: {
    model: () => Category,
  },
  $pagination: {
    model: () => PaginatedList<Category>,
    unwrap: 'data',
    shape: { 'list.*': () => Category },
  },
})
@Injectable({
  providedIn: 'root',
})
export class CategoryService extends BaseCrudService<Category> {
  serviceName: string = 'CategoryService';

  override getUrlSegment(): string {
    return this.urlService.URLS.CATEGORIES; // Add this to your URL service
  }
}
```

### Step 5: Create the Resolver

```typescript
// resolvers/categories.resolver.ts
import { ResolveFn } from '@angular/router';
import { inject } from '@angular/core';
import { CategoryService } from '@/services/category.service';
import { PaginatedList } from '@/models/shared/response/paginated-list';
import { Category } from '@/models/category';
import { PaginationParams } from '@/models/shared/pagination-params';

export const categoriesResolver: ResolveFn<PaginatedList<Category>> = () => {
  const categoryService = inject(CategoryService);
  return categoryService.loadPaginated(new PaginationParams());
};
```

### Step 6: Create the List Component

```typescript
// components/category-list/category-list.component.ts
import { Component, inject, OnInit } from '@angular/core';
import { MenuItem } from 'primeng/api';
import { Breadcrumb } from 'primeng/breadcrumb';
import { TableModule } from 'primeng/table';
import { PaginatorModule } from 'primeng/paginator';
import { InputTextModule } from 'primeng/inputtext';
import { BaseListComponent } from '@/abstracts/base-components/base-list/base-list.component';
import { CategoryPopupComponent } from '../category-popup/category-popup.component';
import { CategoryFilter } from '@/models/category-filter';
import { Category } from '@/models/category';
import { FormsModule } from '@angular/forms';
import { TranslatePipe } from '@ngx-translate/core';
import { CategoryService } from '@/services/category.service';

@Component({
  selector: 'app-category-list',
  standalone: true,
  imports: [Breadcrumb, TableModule, PaginatorModule, InputTextModule, FormsModule, TranslatePipe],
  templateUrl: './category-list.component.html',
  styleUrl: './category-list.component.scss',
})
export default class CategoryListComponent
  extends BaseListComponent<Category, CategoryPopupComponent, CategoryService, CategoryFilter>
  implements OnInit
{
  override dialogSize = {
    width: '100%',
    maxWidth: '600px',
  };

  categoryService = inject(CategoryService);
  home: MenuItem | undefined;
  filterModel: CategoryFilter = new CategoryFilter();

  override get service() {
    return this.categoryService;
  }

  override openDialog(category: Category): void {
    this.openBaseDialog(CategoryPopupComponent as any, category);
  }

  addOrEditModel(category?: Category) {
    category = category || new Category();
    this.openDialog(category);
  }
}
```

### Step 7: Create the Popup Component

```typescript
// components/category-popup/category-popup.component.ts
import { BasePopupComponent } from '@/abstracts/base-components/base-popup/base-popup.component';
import { Component, Inject, inject, OnInit } from '@angular/core';
import { InputTextModule } from 'primeng/inputtext';
import { FormBuilder, FormGroup, ReactiveFormsModule } from '@angular/forms';
import { MAT_DIALOG_DATA } from '@angular/material/dialog';
import { Category } from '@/models/category';
import { RequiredMarkerDirective } from '@/directives/required-marker.directive';
import { Observable } from 'rxjs';
import { AlertService } from '@/services/shared/alert.service';
import { CategoryService } from '@/services/category.service';

@Component({
  selector: 'app-category-popup',
  imports: [InputTextModule, ReactiveFormsModule, RequiredMarkerDirective],
  templateUrl: './category-popup.component.html',
  styleUrl: './category-popup.component.scss',
})
export class CategoryPopupComponent extends BasePopupComponent<Category> implements OnInit {
  declare model: Category;
  declare form: FormGroup;
  alertService = inject(AlertService);
  service = inject(CategoryService);
  fb = inject(FormBuilder);

  constructor(@Inject(MAT_DIALOG_DATA) public data: any) {
    super();
  }

  override saveFail(error: Error): void {
    // Handle save error
    console.error('Save failed:', error);
  }

  override prepareModel(model: Category, form: FormGroup): Category | Observable<Category> {
    this.model = Object.assign(model, { ...form.value });
    return this.model;
  }

  override initPopup() {
    this.model = this.data.model;
  }

  override buildForm() {
    this.form = this.fb.group(this.model.buildForm());
  }

  beforeSave(model: Category, form: FormGroup) {
    return form.valid;
  }

  afterSave() {
    const successObject = { messages: ['COMMON.SAVED_SUCCESSFULLY'] };
    this.alertService.showSuccessMessage(successObject);
  }
}
```

### Step 8: Add Route Configuration

```typescript
// Add to app.routes.ts
{
  path: 'categories',
  canActivate: [authGuard],
  data: { roles: [ROLES_ENUM.ADMIN] },
  resolve: { list: categoriesResolver },
  loadComponent: () =>
    import('./components/category-list/category-list.component'),
}
```

### Step 9: Update URL Service

```typescript
// Add to url.service.ts
URLS = {
  // ... existing URLs
  CATEGORIES: '/api/categories',
};
```

## Key Architecture Benefits

### 1. **Code Reusability**

- Base classes handle common CRUD operations
- Consistent patterns across all entities
- Reduced boilerplate code

### 2. **Type Safety**

- Generic types ensure compile-time safety
- Strong typing throughout the application
- IntelliSense support

### 3. **Separation of Concerns**

- Clear separation between models, services, and components
- Easy to test and maintain
- Modular architecture

### 4. **Extensibility**

- Easy to override base behavior
- Add entity-specific functionality
- Flexible validation and transformation

## Advanced Features

### Custom Validators

```typescript
// In your model's buildForm method
nameEn: [
  nameEn,
  [
    Validators.required,
    CustomValidators.pattern('ENG_NUM'),
    CustomValidators.uniqueName(this.service), // Custom async validator
  ],
],
```

### Advanced Filtering

```typescript
// In your filter model
export class CategoryFilter {
  declare nameAr: string | null;
  declare nameEn: string | null;
  declare createdDateFrom: Date | null;
  declare createdDateTo: Date | null;
  declare isActive: boolean | null;
}
```

### Custom Actions

```typescript
// In your service
@CastResponse()
activateCategory(id: number): Observable<Category> {
  return this.http.patch<Category>(`${this.getUrlSegment()}/${id}/activate`, {});
}

// In your list component
activateCategory(category: Category) {
  this.service.activateCategory(category.id).subscribe({
    next: () => this.loadList(),
    error: (error) => console.error('Activation failed:', error)
  });
}
```

## Best Practices

1. **Always extend base classes** - Don't create components from scratch
2. **Use consistent naming** - Follow the established naming conventions
3. **Implement proper validation** - Use the buildForm method for validation rules
4. **Handle errors gracefully** - Implement proper error handling in all components
5. **Use interceptors wisely** - Transform data consistently using model interceptors
6. **Keep services thin** - Business logic should be in the model layer
7. **Test thoroughly** - Write unit tests for custom behavior and overrides

## Common Pitfalls

1. **Forgetting to register services** - Ensure services are provided correctly
2. **Missing route resolvers** - Always pre-load data for list components
3. **Incorrect generic types** - Maintain type consistency across the inheritance chain
4. **Skipping validation** - Always implement proper form validation
5. **Hardcoded strings** - Use translation keys for all user-facing text

This architecture provides a solid foundation for building scalable Angular applications with consistent CRUD operations across different entities.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/3pillars-projects)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/3pillars-projects)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
