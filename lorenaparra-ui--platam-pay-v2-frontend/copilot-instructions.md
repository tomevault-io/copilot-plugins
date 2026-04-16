## platam-pay-v2-frontend

> Structure and conventions for feature modules in features/


# Feature Modules

Each feature (`features/<feature>/`) is self-contained and follows this structure:

```
features/
  onboarding/
    components/   # Feature-specific components (not transversal)
    constants/    # Form field configs + default values
    interfaces/   # Feature-scoped types (e.g. option sets)
    schemas/      # Zod validation schemas
  partners/
    services/     # API/service calls
```

## Constants (`constants/`)

Each form entity has two exports:

```ts
// 1. Default values — every field initialized (no undefined)
export const defaultValuesNaturalPerson = { first_name: "", ... }

// 2. Form field config — array of FormStep[] using FieldType enum
export const naturalPersonFormFields: FormStep[] = [
  {
    step: 1,
    sections: [
      {
        section: "Datos del Cliente",
        columns: 2,
        fields: [
          { name: "first_name", label: "Nombres", typefield: FieldType.Input, type: "text", rules: { required: "..." } }
        ]
      }
    ]
  }
]
```

- ✅ Always use `FieldType` enum (never plain strings like `"input"`)
- ✅ Conditional fields use `dependency` + `dependencyValue`
- ✅ Currency fields include `numberFormatLocale: 'es-CO'`, `prefix: '$'`, `integerOnly: true`

## Schemas (`schemas/`)

Use **Zod** for validation. Export the schema and its inferred type:

```ts
export const naturalPersonSchema = z.object({ ... })
export type NaturalPersonSchema = z.infer<typeof naturalPersonSchema>
```

- Schema field names must match `defaultValues` keys exactly
- Optional/conditional fields: `.optional()` or `.optional().or(z.literal(""))`

## Services (`services/`)

```ts
export const myEntityService = {
  getAll: () => { ... },
  getById: (id: number) => { ... },
}
```

- Services are plain objects with typed methods
- Import the API client from `@/infrastructure/api/`

## Feature components (`components/`)

- Receive `formFields`, `schema`, `onSubmit`, `defaultValues` as props
- Use `useForm` with `zodResolver` and pass `control` down to transversal form components
- ❌ No business logic in pages — delegate to feature components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lorenaparra-ui) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
