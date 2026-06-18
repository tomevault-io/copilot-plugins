## ultimotp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

### Backend (.NET 9 API)
```bash
# Build solution
dotnet build BurgerShop.sln

# Run API (serves on http://localhost:5021)
dotnet run --project src/BurgerShop.API

# Swagger UI available at http://localhost:5021/swagger (Development only)
# Health check: GET http://localhost:5021/api/health
```

### Frontend (React + Vite)
```bash
cd frontend/burgershop-web

npm install        # Install dependencies
npm run dev        # Dev server on http://localhost:5173
npm run build      # TypeScript check + production build
npm run lint       # ESLint
```

### Database
- PostgreSQL with EF Core + Npgsql. Auto-migrates on API startup.
- Connection string in `src/BurgerShop.API/appsettings.json`
- Add migration: `dotnet ef migrations add <Name> --project src/BurgerShop.Infrastructure --startup-project src/BurgerShop.API`

## Architecture

Clean Architecture with 4 backend layers + React frontend:

```
BurgerShop.Domain        → Entities, repository interfaces (no external dependencies)
BurgerShop.Application   → Services, DTOs, business logic (depends on Domain)
BurgerShop.Infrastructure→ EF Core DbContext, repository implementations, migrations (depends on Domain + Application)
BurgerShop.API           → Controllers, DI registration, CORS config (depends on Application + Infrastructure)
frontend/burgershop-web  → React 19 + TypeScript + Tailwind CSS + Axios
```

**Dependency flow:** API → Infrastructure → Application → Domain

## Business Modules

The application is organized into 3 business modules, each with its own entities, services, DTOs, repositories, and controllers:

1. **Catalogo** — Categorias, Productos, Combos (ComboDetalle for many-to-many)
2. **Ventas** — Pedidos, LineaPedido, Cliente. Pedido has 5 states: Pendiente → Asignado → EnCamino → Entregado | Cancelado. Cancelar requires MotivoCancelacion.
3. **Logistica** — Zonas, Repartidores, RepartidorZona (many-to-many). Includes repartidor auth via CodigoAcceso

## Key Patterns

- **Generic Repository:** `IRepository<T>` / `Repository<T>` for basic CRUD; specialized repositories (e.g., `IPedidoRepository`) extend it
- **Service Layer:** Each module has service interfaces in Application and implementations that orchestrate repository calls
- **DI Registration:** Each module has an extension method in `src/BurgerShop.API/Extensions/` (e.g., `AddCatalogoServices()`, `AddVentasServices()`, `AddLogisticaServices()`)
- **DTOs:** Separate Create/Update/Read DTOs per entity in `Application/{Module}/DTOs/`
- **Entity Configuration:** Fluent API configurations in `Infrastructure/Data/Configurations/`

## API & Frontend Integration

- API base: `http://localhost:5021/api/`
- Frontend Axios client configured in `frontend/burgershop-web/src/api/client.ts`
- CORS allows `localhost:5173` and `localhost:3000`
- API modules per endpoint in `frontend/burgershop-web/src/api/` (categorias, productos, combos, pedidos, entregas, repartidores)

## Seed Data

The initial migration seeds: 4 categorías (Hamburguesas, Bebidas, Acompañamientos, Postres), 8 productos, and 2 repartidores (codes: 1234, 5678).


## Aclaraciones
- Al pedirte que subas cambios al repositorio de github asociado, deberas generar el commit y luego ejecutar este comando para generar el push a main ! git push origin main

---
> Source: [nicofloress/ultimoTP](https://github.com/nicofloress/ultimoTP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
