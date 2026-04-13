## filament-hrms

> This document outlines the standard conventions to be followed when creating Eloquent models and Filament Resources in this project.

# CursorRules –  Conventions

This document outlines the standard conventions to be followed when creating Eloquent models and Filament Resources in this project.

---

## 🧱 Model Structure Rules

1. **Universal Fields for All Models**
   - Every model **must include** the following attributes:
     - `id` (Primary Key, auto-incrementing)
     - `uuid` (Universally Unique Identifier)
     - Laravel's default timestamps:
       - `created_at`
       - `updated_at`

2. **Traits**
   - All models **must use the `HasUuid` trait** to ensure UUID functionality is automatically handled.
     ```php
     use App\Traits\HasUuid;

     class YourModel extends Model
     {
         use HasUuid;
     }
     ```

3. **Enums**
   - Any field requiring an enumeration of fixed values **must** use a dedicated **Enum class**.
     - Enum classes should be stored in a well-organized directory (e.g., `App\Enums`)
     - Always type-cast the enum in the model where it's used
     - Example:
       ```php
       protected $casts = [
           'status' => \App\Enums\StatusEnum::class,
       ];
       ```

---

Please ensure these conventions are followed across all modules to maintain consistency and facilitate code maintenance.

# Cursor Rules for Modular Filament Resources

## ✅ Mandatory Command

All Filament resources **must** be created using the custom Artisan command:

```bash
php artisan make:modular-filament-resource <ModelName> --fields="GroupOne,GroupTwo,GroupThree" [--generate] [--model] [--migration] ...
```

---

## 🧠 AI Model Responsibilities

### 1. **Always Use the Modular Command**
Do not use the default `make:filament-resource`. Every resource should follow the modular pattern provided by the `make:modular-filament-resource` command.

### 2. **Infer and Group Fields from Model**
- Parse the Laravel model located in `app/Models/<ModelName>.php`
- Analyze fillable fields, casts, nullable columns, and relationships
- Group related fields into logical sections (e.g. "Basic Info", "Contact Info")
- Pass these group names using the `--fields` option in the command

### 3. **Field Group Example**

```bash
php artisan make:modular-filament-resource User --fields="Basic Info,Personal Info,Account Settings,Contact Info"
```

---

## 📁 Output Structure

For a model named `User`, the following structure should be created:

```
app/
└── Filament/
    └── Resources/
        ├── UserResource.php
        └── UserResource/
            ├── FormSchema.php
            ├── TableSchema.php
            ├── Actions.php
            └── Fields/
                ├── BasicInfo.php
                ├── PersonalInfo.php
                ├── AccountSettings.php
                └── ContactInfo.php
```

---

## 🧩 Field Group Files

Each group in `--fields` generates a corresponding file inside `Fields/`:

- Use `Filament\Forms\Components\Section::make('GroupName')`
- Wrap related fields inside `->schema([])`
- Apply layout options like `->columns(2)`

---

## 📊 Table Schema Rules

### Generate in `TableSchema.php`:
- Columns should be based on the model’s database columns or `$fillable`
- Each column should be:
  - **Toggleable**:
    ```php
    ->toggleable()
    ```
  - **Hidden by default if nullable**:
    ```php
    ->hidden()
    ```

### Example:
```php
TextColumn::make('email')
    ->toggleable()
    ->hidden();
```

---

## 🛠 Table Actions

Use a centralized actions handler:

```php
->actions(Actions::getActions())
->bulkActions(Actions::getBulkActions())
```

Define them in `Actions.php`.

---

## 🚫 What Not to Do

- ❌ Don’t put form/table logic directly inside `UserResource.php`
- ❌ Don’t define custom actions inline — always use `Actions.php`
- ❌ Don’t skip `--fields` argument; even a single group must be named

---

## 📌 Best Practices Summary

| Element         | Rule                                                             |
|----------------|------------------------------------------------------------------|
| Resource File  | Only basic method overrides, no logic inside                     |
| Field Groups   | Each group = 1 file under `Fields/`, should return a schema      |
| Table Columns  | Auto-generated, toggleable, hidden if nullable                   |
| Actions        | Defined in `Actions.php` using `getActions()` and `getBulkActions()` |
| Consistency    | All files must follow PSR-4 naming and proper namespaces         |

---

By following these rules, your Filament resources will remain clean, maintainable, and modular—ready for future microservice conversion if needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gitwithravi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
