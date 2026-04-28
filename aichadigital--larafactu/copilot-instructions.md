## larafactu

> Larafactu Project - Staging for Billing Packages Development


# Larafactu - Staging Project Rules

## 🎯 Project Identity

### Core Purpose
- **Staging environment** for testing billing packages in real-world scenarios
- **Target v1.0**: Hosting companies, Spain/EU, Intracom Operator
- **Critical deadline**: December 15, 2025 - stable version for WHMCS migration
- **Philosophy**: Pragmatic, Laravel-aligned, avoid over-engineering

### Main Packages Under Development
Located at: `/Users/abkrim/development/packages/aichadigital/`

1. **larabill** (core billing) - dev-main
2. **lararoi** (EU VAT/ROI logic) - dev-main
3. **lara-verifactu** (Spain AEAT integration) - dev-main
4. **laratickets** (support tickets) - dev-main
5. **lara100** (base-100 monetary) - v1.0 stable

## 🏗️ Technical Stack

### Framework & Versions
- **PHP**: 8.4.13
- **Laravel**: 12.x
- **Filament**: 4.x
- **Pest**: 4.x (browser testing enabled)
- **Tailwind CSS**: 4.x
- **Database**: MySQL (local: Laravel Herd)

### UUID Strategy - String UUID v7

**IMPORTANTE**: Usamos UUID v7 **STRING** (char 36), NO binary.

#### Implementación

```php
// Model con UUID
use AichaDigital\Larabill\Concerns\HasUuid;

class User extends Authenticatable
{
    use HasUuid;
    // NO necesita $incrementing ni $keyType - el trait lo maneja
}
```

#### Migraciones

```php
// UUID string (36 chars)
$table->uuid('id')->primary();
$table->uuid('user_id');  // FK a users

// Para relaciones
$table->foreignUuid('user_id')->constrained()->cascadeOnDelete();
```

**Modelos con UUID**: User, Invoice, Ticket
**Modelos con Integer ID**: TaxRate, CompanyFiscalConfig, CustomerFiscalData, UnitMeasure

### UUID Binary + Filament (CRITICAL)

Si usas `uuid_binary` (binario 16 bytes), hay reglas especiales para Filament:

#### Select con relaciones de usuario

```php
// ❌ MAL - pluck directo NO aplica el cast, devuelve binario
Forms\Components\Select::make('user_id')
    ->options(fn () => User::pluck('name', 'id'))  // ERROR: binario

// ✅ BIEN - Cargar modelos PRIMERO, luego pluck (aplica cast)
Forms\Components\Select::make('user_id')
    ->options(fn () => User::all()->pluck('name', 'id'))  // OK: UUID string
```

#### Modelos con user_id FK

Usar el trait `HasUserRelation` de larabill:

```php
use AichaDigital\Larabill\Concerns\HasUserRelation;

class CustomerFiscalData extends Model
{
    use HasUserRelation;  // Añade cast EfficientUuid + relación user()
}
```

#### Filament 4 API Changes

```php
// Filament 3 → Filament 4
->actions()     → ->recordActions()
->bulkActions() → ->toolbarActions()

// Columnas con fecha nullable
->default('Texto')      // ❌ MAL - intenta parsear como fecha
->placeholder('Texto')  // ✅ BIEN - muestra texto si null
```

## 💰 Monetary Values - Base 100 (Lara100)

### CRITICAL RULE
**NEVER use float/decimal for monetary values, prices, amounts, tax rates**

### Always Use Integers (Base 100)
- €12.34 → `1234` (integer)
- 21.5% IVA → `2150` (integer)
- €0.99 → `99` (integer)

### Package
- `aichadigital/lara100` (v1.0 - stable)
- Prevents floating-point errors
- Ensures exact fiscal calculations

## 📦 Package Development Mode

### Instalador Inteligente

**Comando principal**: `php artisan larafactu:install`

```bash
# Instalación local interactiva (recomendado)
php artisan larafactu:install

# Instalación local con opciones
php artisan larafactu:install --local --fresh

# Instalación producción (sin symlinks)
php artisan larafactu:install --production

# Especificar ruta de paquetes
php artisan larafactu:install --local --packages-path=/custom/path
```

### Qué hace el instalador en modo local

1. **Crea symlinks** en `packages/aichadigital/` → `/Users/abkrim/development/packages/aichadigital/`
2. **Modifica composer.json** añadiendo path repositories
3. **Ejecuta composer update** para usar paquetes locales
4. **Ejecuta migraciones** (opción fresh o migrate)
5. **Ejecuta seeders** de desarrollo

### Reset Completo (Fresh Install)

```bash
# Preserva .cursor, .claude, .env, .mcp.json
bin/fresh-install.sh

# Luego ejecutar instalador
php artisan larafactu:install --local --fresh
```

### Local Development Setup

**CRITICAL**: Larafactu usa **path repositories con symlinks** para desarrollo local.

```
/Users/abkrim/
├── development/packages/aichadigital/  # Paquetes SOURCE
│   ├── larabill/
│   ├── lararoi/
│   ├── lara-verifactu/
│   └── laratickets/
└── SitesLR12/larafactu/                # App STAGING
    └── packages/aichadigital/          # Symlinks
```

### Workflow de Desarrollo

Ver `docs/WORKFLOW.md` para flujo completo.

**Resumen**:
1. Editar en `packages/aichadigital/` (symlinks)
2. Cambios reflejados inmediatamente
3. Commit SEPARADO: primero paquete, luego app
4. `composer update aichadigital/*` después de commits en paquetes

## 🧪 Testing Standards

### Coverage Goals (Pragmatic)
- **Packages**: 80-95% coverage
- **Staging app**: 60-70% coverage
- **Priority**: Feature tests > Unit tests

### Comandos

```bash
php artisan test                    # Todos los tests
php artisan test --filter=Invoice   # Filtrar
vendor/bin/pint --dirty             # Formatear
```

## 📚 Gestión de Documentación

### Estructura de docs/

```
docs/
├── ADR_*.md              # Decisiones arquitectónicas (permanentes)
├── PRODUCTION_*.md       # Guías de producción (permanentes)
├── DEVELOPMENT_*.md      # Guías de desarrollo (permanentes)
├── WORKFLOW.md           # Flujo de trabajo (permanente)
└── in-progress/          # Temporales (limpiar regularmente)
```

### Reglas CRÍTICAS

**PERMITIDO**:
- ✅ ADR_XXX_*.md (decisiones arquitectónicas)
- ✅ Guías de producción/desarrollo
- ✅ WORKFLOW.md

**PROHIBIDO** (NO COMMITEAR):
- ❌ HOTFIX_*.md → aplicar y eliminar
- ❌ BUG_*.md → resolver y eliminar
- ❌ RESUMEN_*.md → no commitear
- ❌ SESION_*.md → no commitear
- ❌ Documentos de versiones antiguas

## 🐛 Debugging Strategy (CRITICAL)

### ALWAYS Read Logs First

**RULE**: Before assuming the cause of ANY error, **READ THE ACTUAL LOGS**.

```bash
# Read recent Laravel logs
tail -100 storage/logs/laravel.log

# Clear logs before testing (to get clean output)
rm storage/logs/laravel.log && touch storage/logs/laravel.log

# Then reproduce the error and read fresh logs
cat storage/logs/laravel.log
```

### Why This Matters

Browser error messages (like "Malformed UTF-8 characters") are often **symptoms**, not root causes. The actual error in logs might be:

1. **Filament API changes** (`actions()` → `recordActions()` in Filament 4)
2. **Date parsing errors** (`default()` vs `placeholder()` on date columns)
3. **Missing relationships** (model doesn't have `user()` method)
4. **UUID binary issues** (only after ruling out other causes)

### Log Reading is Cheaper Than Playwright

- ✅ `tail storage/logs/laravel.log` - **instant**, **free**
- ❌ Browser automation - slow, complex, resource-intensive

**Use browser tools for verification, not debugging.**

## 🚫 Anti-Patterns (Critical)

### DON'T Do This
❌ Over-engineer for hypothetical scenarios
❌ Use float/decimal for money (always base100)
❌ Bypass Laravel conventions
❌ Create complex abstractions "just in case"
❌ Commitear documentos temporales/obsoletos
❌ Assume error cause without reading logs

### DO This Instead
✅ Follow Laravel 12 conventions strictly
✅ Test in staging before package commits
✅ Document breaking changes in CHANGELOG
✅ Prioritize delivery over perfection
✅ Limpiar docs/ regularmente
✅ **Read logs FIRST when debugging**

## 🔧 Arquitectura Fiscal (ADR-001)

### Modelos Principales

```
CompanyFiscalConfig    → Configuración fiscal empresa (temporal)
CustomerFiscalData     → Datos fiscales cliente (histórico)
Invoice                → Factura (inmutable una vez emitida)
```

### Validez Temporal

- `valid_from`: Inicio de vigencia
- `valid_until`: Fin de vigencia (null = actual)
- Facturas capturan snapshot al crearse

## 📌 Quick Reference

**UUID Models**: User, Invoice, Ticket (string UUID v7)
**Integer Models**: TaxRate, CompanyFiscalConfig, CustomerFiscalData
**Money Format**: Always base 100 integers (lara100)
**Package Mode**: All in `dev-main` with symlinks
**Deadline**: December 15, 2025 (v1.0 stable)
**Target**: Hosting + Spain + Intracom + WHMCS migration

---

**Remember**: Pragmatism and delivery speed are priorities. Over-engineering is the enemy. Laravel conventions are the guide. Keep docs/ clean.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AichaDigital) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
