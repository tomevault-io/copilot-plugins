## ceremony-field-catalog

> <!-- Synced with CLAUDE.md - keep in sync when updating -->

# GEMINI.md

<!-- Synced with CLAUDE.md - keep in sync when updating -->

A dynamic field observation catalog that tracks XML field usage patterns across business contexts. Spring Boot + MongoDB backend, React + TypeScript frontend.

---

## Quick Navigation by Task

### Changing Search/Autocomplete/Filtering
→ `docs/how-to/search.md`

### Changing Contexts/Metadata Rules
→ `docs/how-to/contexts.md`

### Changing Observation Merge/Upload Parsing/SDK
→ `docs/how-to/observations.md`

### Changing Schema Export (XSD/JSON)
→ `docs/how-to/schema-export.md`

### Running/Writing Tests
→ `docs/how-to/testing.md`

### Understanding the Domain
→ `docs/domain/motivation.md`, `docs/domain/glossary.md`

### System Design & Data Flow
→ `docs/ARCHITECTURE.md` (integration, deployment, data model, scaling)

### Code Navigation (where things are)
→ `docs/reference/backend.md` (Java packages, classes)
→ `docs/reference/frontend.md` (React components, hooks)

### Configuration/Environment Variables
→ `docs/reference/configuration.md`

### Integrating SDKs (.NET/Python)
→ `sdks/dotnet/net48/README.md`, `sdks/python/README.md`

### API Details (endpoints, request/response)
→ Run server, visit `/swagger-ui.html` for OpenAPI spec
→ Or inspect `@Operation` annotations in `*Controller.java`

### Project Backlog/Known Issues
→ `plans/BACKLOG.md`

### Maintaining Documentation
→ `docs/meta/DOCUMENTATION.md`

---

## Essential Commands

```bash
# Backend
docker-compose up --build        # Full stack with MongoDB
mvn spring-boot:run              # Local (needs MongoDB on :27017)
mvn clean test                   # Tests (needs Docker for Testcontainers)

# UI (from ui/ directory)
npm run dev                      # Development server
npm run typecheck                # MUST pass before committing
npm run build                    # Production build
```

---

## UI Navigation

| Route | Tab | Page Component | Purpose |
|-------|-----|----------------|---------|
| `/` | Discover Fields | `DiscoverFieldsPage` | Reactive field exploration with facets |
| `/schema` | Explore Schema | `ExploreSchemaPage` | Generate schemas for export |
| `/submit` | Submit Data | `SubmitDataPage` | Upload XML to extract observations |
| `/contexts` | Manage Contexts | `ManageContextsPage` | Create/edit context schemas |

---

## Core Invariants

- **Metadata normalization:** All metadata keys/values normalized to lowercase
- **Field identity:** `hash(contextId + requiredMetadata + fieldPath)` - optional metadata doesn't affect identity
- **Required metadata immutability:** Cannot change after context creation
- **Active context filtering:** Only active contexts appear in search/dropdowns by default
- **Multi-value metadata:** `metadata.key=a&metadata.key=b` → OR within field, AND between fields

---

## Key File Locations

### Backend
- Domain models: `src/main/java/com/ceremony/catalog/domain/`
- Services: `src/main/java/com/ceremony/catalog/service/`
- Controllers: `src/main/java/com/ceremony/catalog/api/`
- Custom queries: `src/main/java/com/ceremony/catalog/persistence/CatalogCustomRepositoryImpl.java`

### Frontend
- Pages: `ui/src/pages/`
- Hooks: `ui/src/hooks/`
- Components: `ui/src/components/`
- XML parser: `ui/src/utils/xmlParser.ts`
- Schema generators: `ui/src/lib/schema/`

### Configuration
- Backend: `src/main/resources/application.yml`
- UI: `ui/src/config.ts`
- Docker: `docker-compose.yml`

---

## Type Safety

The UI uses strict TypeScript. All type errors must be resolved before committing:
- `noUncheckedIndexedAccess`: Array indexing returns `T | undefined`
- `noImplicitReturns`: All code paths must return
- Run `npm run typecheck` to verify

---

## Testing

- Backend uses Testcontainers (real MongoDB) - Docker must be running
- Manual API tests: `tests/CatalogSmokeTests.http` (VS Code REST Client)
- See `docs/how-to/testing.md` for patterns

---

## Documentation Structure

```
docs/
├── ARCHITECTURE.md   # System architecture overview
├── how-to/           # Task-oriented guides
│   ├── search.md
│   ├── contexts.md
│   ├── observations.md
│   ├── schema-export.md
│   └── testing.md
├── reference/        # Deep architecture
│   ├── backend.md
│   ├── frontend.md
│   └── configuration.md
├── domain/           # Business context
│   ├── motivation.md
│   ├── vision.md
│   └── glossary.md
├── sdk/              # SDK index
│   └── README.md
├── samples/          # Sample XML files
│   └── README.md
├── history/          # Historical archives
│   └── releases/     # Release primers
└── meta/             # Doc maintenance
    └── DOCUMENTATION.md
sdks/                 # Co-located SDK docs
├── dotnet/net48/README.md
└── python/README.md
plans/
└── BACKLOG.md        # Project backlog
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thefluxatlas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/thefluxatlas)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
