## folder-structure

> Where new files must be created in LokalMind v2.

# Folder Structure

Full reference: `docs/development/CONVENTIONS.md` section 2 · `docs/development/FOLDER-STRUCTURE.md`.

## Feature module layout

```
src/features/{feature}/
├── presentation/  screens/ · viewmodels/ · components/
├── domain/        entities/ · repositories/ · usecases/
└── data/          repositories/ · services/
```

## Where new files go

| File type | Location |
|---|---|
| Screen | `src/features/{feature}/presentation/screens/` |
| ViewModel | `src/features/{feature}/presentation/viewmodels/` |
| Feature component | `src/features/{feature}/presentation/components/` |
| Shared UI component | `src/design-system/components/` |
| Use case | `src/features/{feature}/domain/usecases/` |
| Entity | `src/features/{feature}/domain/entities/` |
| Repository interface | `src/features/{feature}/domain/repositories/` |
| Repository impl | `src/features/{feature}/data/repositories/` |
| Infrastructure adapter | `src/infrastructure/{llm\|whisper\|storage\|downloads}/` |
| Route | `app/` (expo-router, max 30 lines, no business logic) |

---
> Source: [Sandipan006/lokalmind-app](https://github.com/Sandipan006/lokalmind-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
