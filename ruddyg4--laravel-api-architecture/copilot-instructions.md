## laravel-api-architecture

> Complete Laravel API architecture with Repository-Service-Controller pattern. Includes BaseRepository/BaseService with inheritance, Searchable/Filterable traits, automatic pagination/filters, Laravel Data (DTOs), type safety, clean code principles, Scramble documentation. Use for: building complete Laravel backend, creating CRUD resources, implementing search/filters, refactoring architecture, ensuring SOLID principles, generating API documentation.


# Laravel API Architecture

Arquitectura completa en capas para desarrollo de APIs Laravel con patrón Repository-Service-Controller optimizado.

## 🎯 Cuándo Usar Esta Skill

**Usar automáticamente cuando se trabaje con:**
- Desarrollar backend Laravel completo con arquitectura limpia
- Crear nuevo recurso CRUD desde cero (Model, Repository, Service, Controller, DTOs)
- Implementar sistema de búsqueda y filtros dinámicos
- Configurar paginación automática con metadata
- Refactorizar código legacy a arquitectura en capas
- Eliminar código repetitivo usando herencia (BaseRepository, BaseService)
- Implementar validación con Laravel Data
- Generar documentación API automática con Scramble
- Verificar separación de responsabilidades y principios SOLID
- Escribir tests por cada capa de la arquitectura

**Palabras clave que disparan esta skill:**
- "arquitectura Laravel", "backend completo", "API Laravel"
- "crear recurso", "nuevo CRUD", "Repository", "Service", "Controller"
- "BaseRepository", "BaseService", "herencia", "código repetitivo"
- "Searchable", "Filterable", "traits", "búsqueda", "filtros"
- "paginación", "ordenamiento", "filtros dinámicos"
- "Laravel Data", "DTO", "validación", "type safety"
- "Scramble", "documentación API", "OpenAPI", "Swagger"
- "refactorizar", "separar responsabilidades", "SOLID", "Clean Architecture"
- "testing", "arquitectura en capas"

## 🏗️ Arquitectura Completa

### Componentes Principales

Esta arquitectura combina múltiples patrones y herramientas:

1. **Repository Pattern**: Abstracción de acceso a datos
2. **Service Layer Pattern**: Lógica de negocio centralizada  
3. **Laravel Data (Spatie)**: DTOs con validación y transformación
4. **Herencia con Clases Base**: BaseRepository, BaseService (elimina ~70% código repetitivo)
5. **Traits Reutilizables**: Searchable, Filterable en Models
6. **Paginación Inteligente**: Automática con búsqueda, filtros y ordenamiento
7. **Documentación Automática**: Scramble genera OpenAPI/Swagger
8. **Type Safety Completo**: PHP typed properties throughout
9. **Testing por Capas**: Unit tests independientes por Repository, Service, Controller

### Diagrama de Arquitectura

```
HTTP Request (GET /api/products?search=laptop&category=5&per_page=20)
    ↓ Validado por Laravel Data
ProductIndexQueryData {search, category, per_page, order_by}
    ↓ Controller delega
ProductService::paginate($query)
    ↓ Service usa Repository heredado
BaseRepository::paginateWithQuery($query)
    ↓ Repository usa Traits del Model
Product::search('laptop')->filter(['category'=>5])->paginate(20)
    ↓ Eloquent ejecuta
Database → Collection<Product>
    ↓ Repository retorna
LengthAwarePaginator<Product>
    ↓ Service retorna (Controller transforma para paginación)
Controller: ProductData::collect($products)
    ↓ ApiResponseTrait formatea
JsonResponse {success, data, links, meta}
```

### Principios SOLID Aplicados

1. **Single Responsibility**: Cada capa una responsabilidad clara
   - Controller: Orquestación HTTP
   - Service: Lógica de negocio + Transformación DTO
   - Repository: Acceso a datos
   - Model: Entidad + Scopes

2. **Open/Closed**: Extensible sin modificar
   - BaseRepository: Herencia para nuevos recursos
   - BaseService: Métodos comunes heredados
   - Traits: Comportamiento compartido (Searchable, Filterable)

3. **Liskov Substitution**: Interfaces intercambiables
   - ProductRepositoryInterface → ProductRepository
   - Mock repositories en tests

4. **Interface Segregation**: Interfaces específicas
   - BaseRepositoryInterface: Métodos comunes
   - ProductRepositoryInterface: Métodos específicos de Product

5. **Dependency Inversion**: Depender de abstracciones
   - Services dependen de RepositoryInterface, no implementación concreta
   - Controllers dependen de Services, no Repositories directamente

### Responsabilidades por Capa

| Capa | Responsabilidad Principal | Retorna | NO Debe Hacer |
|------|--------------------------|---------|---------------|
| **Controller** | Orquestación HTTP, validación request, autorización | JsonResponse | Lógica de negocio, queries DB, transformar Models manualmente |
| **Service** | Lógica de negocio, transacciones, transformar Model→DTO | **DTO** | Queries directas, formateo HTTP, acceso a Request |
| **Repository** | Queries DB, acceso a datos | **Model** | Lógica de negocio, transformar a DTO, validaciones |
| **Model** | Entidad, relaciones, scopes (search, filter) | - | Lógica de negocio compleja, acceso a otros services |

### Reglas de Oro

1. **Services retornan DTOs** (no Models Eloquent)
2. **Controllers solo orquestan** (no transforman Models)
3. **Repositories retornan Models** (no DTOs)
4. **Type hints estrictos** en todos los métodos
5. **BaseRepository/BaseService** eliminan código repetitivo
6. **Traits Searchable/Filterable** automatizan búsqueda y filtros
7. **Laravel Data** centraliza validación y transformación
8. **PHPDoc completo** para Scramble (documentación automática)

## 🎁 Beneficios de Esta Arquitectura

### Eliminación de Código Repetitivo (~70%)

**Sin esta arquitectura:**
```php
// Cada Repository necesita reimplementar:
public function paginate() { ... }
public function find() { ... }
public function create() { ... }
public function update() { ... }
// Repetido 20+ veces en el proyecto
```

**Con esta arquitectura:**
```php
// Solo una vez en BaseRepository
// Todos heredan automáticamente
class ProductRepository extends BaseRepository { }
```

### Búsqueda y Filtros Automáticos

**Sin Traits:**
```php
// Repetir en cada Controller/Repository
if ($request->has('search')) {
    $query->where('name', 'like', "%{$request->search}%")
          ->orWhere('description', 'like', "%{$request->search}%");
}
```

**Con Traits:**
```php
// En Model: protected $searchableColumns = ['name', 'description'];
// Automático: Product::search($query->search)->get();
```

### Type Safety y Validación Automática

**Sin Laravel Data:**
```php
// Validación manual en cada método
$validator = Validator::make($request->all(), [...]);
if ($validator->fails()) { ... }
```

**Con Laravel Data:**
```php
// Validación automática en firma del método
public function store(ProductData $data): JsonResponse
```

### Documentación Automática con Scramble

**Sin Scramble:** Escribir documentación OpenAPI manualmente

**Con Scramble:** 
- Lee type hints y Laravel Data
- Genera OpenAPI/Swagger automáticamente
- Actualización automática al cambiar código
## 📋 Procedimiento Completo: Crear Recurso CRUD desde Cero

Este procedimiento crea un recurso completo con arquitectura limpia en ~150 líneas de código (vs ~500 sin esta arquitectura).

### Paso 1: Model con Traits Searchable y Filterable

**Objetivo:** Definir entidad con búsqueda y filtros automáticos

```php
<?php

namespace App\Models\Tenant;

use App\Traits\Models\Searchable;
use App\Traits\Models\Filterable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Product extends Model
{
    use SoftDeletes, Searchable, Filterable;
    
    // ⭐ Columnas para búsqueda global automática
    protected array $searchableColumns = ['name', 'description', 'sku'];
    
    protected $fillable = ['name', 'description', 'price', 'stock', 'category_id'];
    
    protected $casts = [
        'price' => 'decimal:2',
        'stock' => 'integer',
    ];
    
    // Relaciones
    public function category()
    {
        return $this->belongsTo(Category::class);
    }
    
    // ⭐ Filtros personalizados: filter{Name}($query, $value)
    public function filterCategory($query, int $categoryId)
    {
        return $query->where('category_id', $categoryId);
    }
    
    public function filterInStock($query, bool $inStock)
    {
        return $inStock ? $query->where('stock', '>', 0) : $query;
    }
}
```

**Beneficios:**
- ✅ Búsqueda automática en columnas definidas
- ✅ Filtros dinámicos sin código repetitivo
- ✅ URL: `?search=laptop&category=5&in_stock=true` funciona automáticamente

**Ver detalles:** [Model Guide](./references/models.md)

---

### Paso 2: Repository Interface y Repository

**Objetivo:** Abstracción de acceso a datos con herencia para eliminar código repetitivo

**Interface:**
```php
<?php

namespace App\Repositories\Contracts\Tenant;

use App\Repositories\Contracts\BaseRepositoryInterface;

interface ProductRepositoryInterface extends BaseRepositoryInterface
{
    // Solo métodos específicos (CRUD heredado de BaseRepositoryInterface)
    public function findByCode(string $code): ?Product;
    public function getActive(): Collection;
}
```

**Repository:**
```php
<?php

namespace App\Repositories\Tenant;

use App\Models\Tenant\Product;
use App\Repositories\BaseRepository;
use App\Repositories\Contracts\Tenant\ProductRepositoryInterface;

class ProductRepository extends BaseRepository implements ProductRepositoryInterface
{
    public function __construct(Product $model)
    {
        $this->model = $model;
    }
    
    // ⭐ Solo métodos NO estándar
    // Métodos heredados de BaseRepository:
    // - paginate()
    // - paginateWithQuery() ← Usa Searchable/Filterable
    // - find()
    // - create()
    // - update()
    // - delete()
    
    public function findByCode(string $code): ?Product
    {
        return $this->model->where('code', $code)->first();
    }
    
    public function getActive(): Collection
    {
        return $this->model->where('active', true)->get();
    }
}
```

**Beneficios:**
- ✅ Hereda 6+ métodos CRUD de BaseRepository
- ✅ Solo escribes métodos específicos del dominio
- ✅ `paginateWithQuery()` automático con búsqueda/filtros

**Ver detalles:** [Repository Guide](./references/repositories.md)

---

### Paso 3: Service (Retorna DTOs + Lógica de Negocio)

**Objetivo:** Centralizar lógica de negocio y transformar Models a DTOs

```php
<?php

namespace App\Services\Tenant;

use App\Data\Tenant\ProductData;
use App\Repositories\Contracts\Tenant\ProductRepositoryInterface;
use App\Services\BaseService;
use Illuminate\Support\Facades\DB;

/**
 * @property ProductRepositoryInterface $repository
 */
class ProductService extends BaseService
{
    public function __construct(ProductRepositoryInterface $repository)
    {
        $this->repository = $repository;
    }
    
    // ⭐ REGLA DE ORO: Services retornan DTOs
    public function create(array $data): ProductData
    {
        $product = DB::transaction(function () use ($data) {
            $product = $this->repository->create($data);
            
            // Lógica de negocio aquí
            // Ej: actualizar inventario, enviar notificaciones, etc.
            
            return $product;
        });
        
        return ProductData::from($product); // ← Transformación Model→DTO
    }
    
    public function update(int $id, array $data): ProductData
    {
        // Lógica de negocio
        if (isset($data['price'])) {
            $data['price'] = round($data['price'], 2);
        }
        
        $product = $this->repository->update($id, $data);
        return ProductData::from($product); // ← Siempre retorna DTO
    }
    
    // Métodos heredados de BaseService:
    // - paginate($queryData) ← Usa BaseRepository::paginateWithQuery()
    // - find($id)
    // - delete($id)
}
```

**Beneficios:**
- ✅ Lógica de negocio centralizada
- ✅ Transacciones DB seguras
- ✅ Retorna DTOs (consistencia en toda la app)
- ✅ Hereda métodos comunes de BaseService

**Ver detalles:** [Service Guide](./references/services.md) | [DTO Transformation](./references/dto-transformation.md)

---

### Paso 4: DTOs con Laravel Data (Validación + Transformación)

**Objetivo:** Type safety completo, validación centralizada y transformación automática

**Query Data (para parámetros de URL):**
```php
<?php

namespace App\Data\Tenant;

use App\Data\IndexQueryData;

class ProductIndexQueryData extends IndexQueryData
{
    public function __construct(
        // Heredados de IndexQueryData
        public int $per_page = 15,
        public int $page = 1,
        public ?string $order_by = 'created_at',
        public string $order = 'desc',
        public ?string $search = null,
        
        // ⭐ Filtros específicos de Product
        public ?int $category = null,
        public ?bool $in_stock = null,
    ) {
        parent::__construct($per_page, $page, $order_by, $order, $search);
    }
    
    protected static function allowedOrderByColumns(): array
    {
        return ['id', 'name', 'price', 'stock', 'created_at'];
    }
    
    public static function rules(...$args): array
    {
        return array_merge(parent::rules(), [
            'category' => ['nullable', 'integer', 'exists:categories,id'],
            'in_stock' => ['nullable', 'boolean'],
        ]);
    }
}
```

**Resource Data (para entrada/salida):**
```php
<?php

namespace App\Data\Tenant;

use Spatie\LaravelData\Data;
use Spatie\LaravelData\Optional;
use Spatie\LaravelData\Attributes\Hidden;

class ProductData extends Data
{
    public function __construct(
        public int|Optional $id,
        public string $name,
        public string $description,
        public float $price,
        public int $stock,
        public int $category_id,
        public string|Optional $created_at = '',
    ) {}
    
    public static function rules(...$args): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'description' => ['required', 'string'],
            'price' => ['required', 'numeric', 'min:0'],
            'stock' => ['required', 'integer', 'min:0'],
            'category_id' => ['required', 'exists:categories,id'],
        ];
    }
}
```

**Beneficios:**
- ✅ Validación automática en firma del método: `public function store(ProductData $data)`
- ✅ Type safety completo (IDE autocomplete)
- ✅ Transformación bidireccional: `ProductData::from($model)`, `$data->toArray()`
- ✅ URL automática: `?search=laptop&category=5&per_page=20&order_by=price`
- ✅ Validación de query params automática

**Ver detalles:** [DTOs y Laravel Data](./references/dto-transformation.md)

---

### Paso 5: Controller (Solo Orquestación HTTP)

**Objetivo:** Punto de entrada HTTP sin lógica de negocio ni transformaciones

```php
<?php

namespace App\Http\Controllers\Tenant;

use App\Data\Tenant\ProductData;
use App\Data\Tenant\ProductIndexQueryData;
use App\Http\Controllers\Controller;
use App\Models\Tenant\Product;
use App\Services\Tenant\ProductService;
use App\Traits\ApiResponseTrait;
use Illuminate\Http\JsonResponse;

class ProductController extends Controller
{
    use ApiResponseTrait;

    public function __construct(
        protected ProductService $productService
    ) {
        // ⭐ Autorización automática usando Policy
        $this->authorizeResource(Product::class, 'product');
    }

    /**
     * Display a listing of products (paginated with search/filters).
     * 
     * ⭐ PHPDoc completo para Scramble (documentación automática)
     *
     * @param ProductIndexQueryData $query
     * @return JsonResponse
     */
    public function index(ProductIndexQueryData $query): JsonResponse
    {
        $products = $this->productService->paginate($query);

        return $this->successResponse(
            ProductData::collect($products) // Preserva paginación automáticamente
        );
    }

    /**
     * Store a newly created product.
     *
     * @param ProductData $data
     * @return JsonResponse
     */
    public function store(ProductData $data): JsonResponse
    {
        $productData = $this->productService->create($data->toArray()); // ⭐ Ya es DTO
        
        return $this->successResponse(
            $productData, // ⭐ Sin transformación
            __('resources.created_successfully', ['resource' => __('resources.names.product')]),
            JsonResponse::HTTP_CREATED
        );
    }

    /**
     * Display the specified product.
     *
     * @param Product $product
     * @return ProductData
     */
    public function show(Product $product): ProductData
    {
        return ProductData::from($product);
    }

    /**
     * Update the specified product.
     *
     * @param ProductData $data
     * @param Product $product
     * @return JsonResponse
     */
    public function update(ProductData $data, Product $product): JsonResponse
    {
        $updatedData = $this->productService->update($product->id, $data->toArray());
        
        return $this->successResponse($updatedData); // ⭐ Sin transformación
    }

    /**
     * Remove the specified product.
     *
     * @param Product $product
     * @return JsonResponse
     */
    public function destroy(Product $product): JsonResponse
    {
        $this->productService->delete($product->id);

        return $this->successResponse(
            message: __('resources.deleted_successfully', ['resource' => __('resources.names.product')])
        );
    }
}
```

**Beneficios:**
- ✅ Controller delgado (~10 líneas por método)
- ✅ NO transforma Models (Service lo hace)
- ✅ Validación automática (Laravel Data)
- ✅ Autorización automática (Policy)
- ✅ Documentación automática (Scramble lee PHPDoc)
- ✅ Respuestas consistentes (ApiResponseTrait)

**Ver detalles:** [Controller Guide](./references/controllers.md)

---

### Paso 6: Registrar Bindings

```php
// app/Providers/AppServiceProvider.php
public $bindings = [
    ProductRepositoryInterface::class => ProductRepository::class,
];
```

### Paso 7: Rutas

```php
Route::apiResource('products', ProductController::class);
```

**Genera automáticamente:**
- GET `/api/products` → index (con paginación, búsqueda, filtros)
- POST `/api/products` → store
- GET `/api/products/{product}` → show
- PUT/PATCH `/api/products/{product}` → update
- DELETE `/api/products/{product}` → destroy

---

### Paso 8: Localización y Mensajes de Retorno

**Objetivo:** Centralizar mensajes de éxito/error en archivos de traducción con soporte multiidioma

#### 8.1 Estructura de Archivos de Traducción

Crear archivos de traducción para cada idioma utilizando placeholders genéricos:

**`lang/en/resources.php`:**
```php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Resource Language Lines
    |--------------------------------------------------------------------------
    |
    | Generic messages for resource operations. Use the :resource placeholder.
    | Example: __('resources.created_successfully', ['resource' => __('resources.names.empresa')])
    |
    */

    'created_successfully' => ':resource created successfully.',
    'updated_successfully' => ':resource updated successfully.',
    'deleted_successfully' => ':resource deleted successfully.',
    'restored_successfully' => ':resource restored successfully.',
    'permanently_deleted_successfully' => ':resource permanently deleted.',
    'not_found' => ':resource not found.',

    /*
    |--------------------------------------------------------------------------
    | Resource Names
    |--------------------------------------------------------------------------
    |
    | Resource names for translation
    |
    */

    'names' => [
        'empresa' => 'Company',
        'sucursal' => 'Branch',
        'product' => 'Product',
        'user' => 'User',
        'role' => 'Role',
        'customer' => 'Customer',
        'category' => 'Category',
        'order' => 'Order',
        // Agregar nombres de recursos según necesidad
    ]
];
```

**`lang/es/resources.php`:**
```php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Resource Language Lines
    |--------------------------------------------------------------------------
    |
    | Mensajes genéricos para operaciones de recursos. Usa el placeholder :resource.
    | Ejemplo: __('resources.created_successfully', ['resource' => __('resources.names.empresa')])
    |
    */

    'created_successfully' => ':resource creado exitosamente.',
    'updated_successfully' => ':resource actualizado exitosamente.',
    'deleted_successfully' => ':resource eliminado exitosamente.',
    'restored_successfully' => ':resource restaurado exitosamente.',
    'permanently_deleted_successfully' => ':resource eliminado permanentemente.',
    'not_found' => ':resource no encontrado.',

    /*
    |--------------------------------------------------------------------------
    | Resource Names
    |--------------------------------------------------------------------------
    |
    | Nombres de recursos para traducción
    |
    */

    'names' => [
        'empresa' => 'Empresa',
        'sucursal' => 'Sucursal',
        'product' => 'Producto',
        'user' => 'Usuario',
        'role' => 'Rol',
        'customer' => 'Cliente',
        'category' => 'Categoría',
        'order' => 'Orden',
        // Agregar nombres de recursos según necesidad
    ]
];
```

#### 8.2 Uso en Controllers

Utilizar la función `__()` de Laravel con placeholders para mensajes dinámicos:

```php
<?php

namespace App\Http\Controllers\Tenant;

use App\Data\Tenant\ProductData;
use App\Http\Controllers\Controller;
use App\Services\Tenant\ProductService;
use App\Traits\ApiResponseTrait;
use Illuminate\Http\JsonResponse;

class ProductController extends Controller
{
    use ApiResponseTrait;

    public function __construct(
        protected ProductService $productService
    ) {
        $this->authorizeResource(Product::class, 'product');
    }

    /**
     * Store a newly created product.
     */
    public function store(ProductData $data): JsonResponse
    {
        $productData = $this->productService->create($data->toArray());
        
        return $this->successResponse(
            $productData,
            __('resources.created_successfully', [
                'resource' => __('resources.names.product')
            ]),
            JsonResponse::HTTP_CREATED
        );
    }

    /**
     * Update the specified product.
     */
    public function update(ProductData $data, Product $product): JsonResponse
    {
        $updatedData = $this->productService->update($product->id, $data->toArray());
        
        return $this->successResponse(
            $updatedData,
            __('resources.updated_successfully', [
                'resource' => __('resources.names.product')
            ])
        );
    }

    /**
     * Remove the specified product.
     */
    public function destroy(Product $product): JsonResponse
    {
        $this->productService->delete($product->id);

        return $this->successResponse(
            message: __('resources.deleted_successfully', [
                'resource' => __('resources.names.product')
            ])
        );
    }
}
```

#### 8.3 Beneficios del Sistema de Localización

- ✅ **Mensajes centralizados**: Un solo lugar para todos los mensajes
- ✅ **Multiidioma**: Soporte automático para múltiples idiomas
- ✅ **Reutilizables**: Mismos mensajes para todos los recursos
- ✅ **Mantenibles**: Cambiar un mensaje actualiza toda la aplicación
- ✅ **Consistentes**: Formato uniforme en todas las respuestas
- ✅ **Genéricos**: Un mensaje sirve para cualquier recurso usando placeholders

#### 8.4 Respuesta JSON Resultante

**Request:** `POST /api/products` (con locale `en`)
```json
{
    "success": true,
    "message": "Product created successfully.",
    "data": {
        "id": 1,
        "name": "Laptop",
        "price": 999.99
    }
}
```

**Request:** `POST /api/products` (con locale `es`)
```json
{
    "success": true,
    "message": "Producto creado exitosamente.",
    "data": {
        "id": 1,
        "name": "Laptop",
        "price": 999.99
    }
}
```

#### 8.5 Configuración de Locale

Laravel detecta automáticamente el idioma del usuario mediante:

**Opción 1: Header HTTP**
```
Accept-Language: es
```

**Opción 2: Middleware personalizado**
```php
// app/Http/Middleware/SetLocale.php
public function handle($request, Closure $next)
{
    $locale = $request->header('Accept-Language', config('app.locale'));
    app()->setLocale($locale);
    
    return $next($request);
}
```

**Opción 3: Usuario autenticado**
```php
if (auth()->check() && auth()->user()->locale) {
    app()->setLocale(auth()->user()->locale);
}
```

---

## ✅ Checklist Completo de Validación

### Arquitectura General
- [ ] Separación clara de responsabilidades por capa
- [ ] Inyección de dependencias (interfaces, no implementaciones concretas)
- [ ] Type hints estrictos en todos los métodos públicos
- [ ] PHPDoc completo (especialmente en Controllers para Scramble)
- [ ] Bindings registrados en ServiceProvider

### Model
- [ ] Usa traits `Searchable` y `Filterable`
- [ ] Define `$searchableColumns` para búsqueda global
- [ ] Métodos `filter{Name}()` para filtros personalizados
- [ ] Relaciones Eloquent definidas
- [ ] `$fillable`, `$casts`, `$hidden` configurados

### Repository
- [ ] Extiende `BaseRepository`
- [ ] Implementa `{Resource}RepositoryInterface`
- [ ] Solo tiene métodos NO estándar (específicos del dominio)
- [ ] Retorna Models Eloquent (NO DTOs)
- [ ] No tiene lógica de negocio
- [ ] No accede a otros repositories directamente

### Service
- [ ] Extiende `BaseService`
- [ ] Inyecta `{Resource}RepositoryInterface` en constructor
- [ ] **Retorna DTOs en TODOS sus métodos públicos** (no Models)
- [ ] **Transforma Model → DTO antes de retornar**
- [ ] Lógica de negocio centralizada aquí
- [ ] Usa `DB::transaction()` para operaciones múltiples
- [ ] No hace queries directas (usa Repository)
- [ ] PHPDoc `@property` para IDE autocomplete

### Controller
- [ ] Usa `ApiResponseTrait`
- [ ] **NO transforma Models a DTOs** (Service lo hace)
- [ ] Solo orquestación HTTP (delega todo a Service)
- [ ] Autorización con `authorizeResource()` o gates
- [ ] Type hints con Laravel Data: `ProductIndexQueryData`, `ProductData`
- [ ] PHPDoc completo para Scramble
- [ ] Respuestas consistentes (`successResponse`, `errorResponse`)
- [ ] Internacionalización: `__('resources.created_successfully', ['resource' => __('resources.names.product')])`
- [ ] Métodos ~10 líneas máximo

### DTOs (Laravel Data)
- [ ] `IndexQueryData` hereda de base para query params
- [ ] Define `allowedOrderByColumns()`
- [ ] `{Resource}Data` para entrada/salida
- [ ] Método `rules()` implementado
- [ ] Usa `Optional` para campos opcionales
- [ ] Usa `#[Hidden]` para campos sensibles

### Localización
- [ ] Archivos de traducción creados: `lang/en/resources.php`, `lang/es/resources.php`
- [ ] Mensajes genéricos con placeholder `:resource`
- [ ] Array `names` con nombres de recursos traducidos
- [ ] Controllers usan `__('resources.created_successfully', ['resource' => __('resources.names.product')])`
- [ ] Locale configurado (middleware o header Accept-Language)

### Paginación y Filtros
- [ ] URLs funcionan: `?search=x&category=5&per_page=20&order_by=price`
- [ ] Búsqueda global usa `$searchableColumns`
- [ ] Filtros usan `filter{Name}()` del Model
- [ ] Paginación retorna links y meta automáticamente

### Testing
- [ ] Tests de Repository validan queries
- [ ] **Tests de Service validan que retorna DTOs**
- [ ] Tests de Controller validan respuestas JSON
- [ ] Coverage > 80% en Services (lógica de negocio crítica)

### Documentación
- [ ] Scramble genera OpenAPI automáticamente
- [ ] Endpoints documentados en `/docs/api`
- [ ] Esquemas de request/response claros

---

## 🚫 Anti-Patrones Comunes y Soluciones

### ❌ 1. Controller Transforma Models

```php
// ❌ INCORRECTO
public function store(ProductData $data): JsonResponse
{
    $product = $this->service->create($data->toArray()); // Model
    return $this->successResponse(ProductData::from($product)); // ← Transformación en controller
}

// ✅ CORRECTO
public function store(ProductData $data): JsonResponse
{
    $productData = $this->service->create($data->toArray()); // Ya es DTO
    return $this->successResponse($productData);
}
```

### ❌ 2. Service Retorna Models

```php
// ❌ INCORRECTO
public function create(array $data): Product
{
    return $this->repository->create($data);
}

// ✅ CORRECTO
public function create(array $data): ProductData
{
    $product = $this->repository->create($data);
    return ProductData::from($product);
}
```

### ❌ 3. Repository con Lógica de Negocio

```php
// ❌ INCORRECTO
public function createWithDiscount(array $data): Product
{
    // Lógica de negocio en Repository
    if ($data['quantity'] > 10) {
        $data['price'] *= 0.9; // 10% descuento
    }
    return $this->model->create($data);
}

// ✅ CORRECTO - Lógica en Service
// Repository:
public function create(array $data): Product
{
    return $this->model->create($data);
}

// Service:
public function createWithDiscount(array $data): ProductData
{
    if ($data['quantity'] > 10) {
        $data['price'] *= 0.9;
    }
    $product = $this->repository->create($data);
    return ProductData::from($product);
}
```

### ❌ 4. Controller con Queries Directas

```php
// ❌ INCORRECTO
public function index(): JsonResponse
{
    $products = Product::where('active', true)->paginate(15);
    return $this->successResponse(ProductData::collect($products));
}

// ✅ CORRECTO
public function index(ProductIndexQueryData $query): JsonResponse
{
    $products = $this->productService->paginate($query);
    return $this->successResponse(ProductData::collect($products));
}
```

### ❌ 5. No Usar Herencia (BaseRepository/BaseService)

```php
// ❌ INCORRECTO - Reimplementar en cada Repository
class ProductRepository implements ProductRepositoryInterface
{
    public function paginate() { /* código repetido */ }
    public function find($id) { /* código repetido */ }
    public function create($data) { /* código repetido */ }
    // ... repetido 20 veces en el proyecto
}

// ✅ CORRECTO - Heredar de Base
class ProductRepository extends BaseRepository implements ProductRepositoryInterface
{
    // Hereda automáticamente: paginate(), find(), create(), update(), delete()
    // Solo agregar métodos específicos
}
```

### ❌ 6. No Usar Traits Searchable/Filterable

```php
// ❌ INCORRECTO - Código repetitivo en Repository
public function search(string $search): Collection
{
    return $this->model
        ->where('name', 'like', "%{$search}%")
        ->orWhere('description', 'like', "%{$search}%")
        ->get();
}

// ✅ CORRECTO - Usar Trait en Model
// Model:
use Searchable;
protected $searchableColumns = ['name', 'description'];

// Uso automático:
Product::search($query->search)->get();
```

### ❌ 7. Validación Manual sin Laravel Data

```php
// ❌ INCORRECTO
public function store(Request $request): JsonResponse
{
    $validator = Validator::make($request->all(), [
        'name' => 'required|string',
        'price' => 'required|numeric',
    ]);
    
    if ($validator->fails()) {
        return $this->errorResponse($validator->errors());
    }
    // ...
}

// ✅ CORRECTO
public function store(ProductData $data): JsonResponse
{
    // Validación automática por Laravel Data
    $productData = $this->service->create($data->toArray());
    return $this->successResponse($productData);
}
```

### ❌ 8. Sin PHPDoc para Scramble

```php
// ❌ INCORRECTO - Sin documentación
public function index($query) // Sin tipos
{
    return $this->service->paginate($query);
}

// ✅ CORRECTO - Con PHPDoc y tipos
/**
 * Display a listing of products (paginated).
 *
 * @param ProductIndexQueryData $query
 * @return JsonResponse
 */
public function index(ProductIndexQueryData $query): JsonResponse
{
    $products = $this->service->paginate($query);
    return $this->successResponse(ProductData::collect($products));
}
// Scramble genera documentación automática ✓
```

---

## 🔍 Detección Automática de Problemas

Si encuentras código con estos síntomas, requiere refactorización:

### Síntomas en Controller
- `::from($model)` → Service debe retornar DTO
- `Product::where()` → Debe usar Service
- `DB::table()` → Debe usar Service
- Sin type hints → Agregar `ProductData`, `ProductIndexQueryData`
- Sin PHPDoc → Agregar para Scramble

### Síntomas en Service
- Retorna `Product` (Model) → Cambiar a `ProductData`
- `Product::where()` → Debe usar Repository
- Sin `DB::transaction()` en operaciones múltiples → Agregar
- Sin type hint de retorno → Agregar `: ProductData`

### Síntomas en Repository
- `ProductData::from()` → No debe transformar, retornar Model
- Lógica de negocio (validaciones, cálculos) → Mover a Service
- No extiende `BaseRepository` → Extender para heredar métodos
- Métodos duplicados (`find`, `create`) → Eliminar, están en base

### Síntomas en Model
- Sin traits `Searchable`, `Filterable` → Agregar
- Sin `$searchableColumns` → Definir para búsqueda
- Lógica de negocio compleja → Mover a Service

---

## 📚 Referencias Detalladas por Componente

Para información completa sobre cada componente, consulta:

- **[Models con Traits](./references/models.md)** - Searchable, Filterable, filtros personalizados
- **[Repositories](./references/repositories.md)** - BaseRepository, métodos CRUD, herencia
- **[Services](./references/services.md)** - Transformación DTO, lógica de negocio, transacciones
- **[Controllers](./references/controllers.md)** - ApiResponseTrait, orquestación, PHPDoc
- **[DTOs entre Capas](./references/dto-transformation.md)** - Principios SOLID, transformación Model→DTO, ejemplos completos

## 🎓 Resumen Ejecutivo

### Esta Arquitectura Proporciona

1. **~70% Menos Código** - Herencia elimina repetición (BaseRepository, BaseService)
2. **Búsqueda/Filtros Automáticos** - Traits Searchable/Filterable
3. **Type Safety Completo** - Laravel Data + PHP typed properties
4. **Documentación Automática** - Scramble lee tipos y genera OpenAPI
5. **Código Limpio** - SOLID principles aplicados consistentemente
6. **Testing Sencillo** - Capas independientes, fácil de mockear
7. **Escalabilidad** - Agregar recursos es rápido (~150 líneas vs ~500)
8. **Mantenibilidad** - Separación clara, cambios localizados

### Flujo Completo de Datos

```
HTTP Request: GET /api/products?search=laptop&category=5&per_page=20
    ↓ Laravel Data valida automáticamente
ProductIndexQueryData {search, category, per_page, order_by}
    ↓ Controller delega (sin lógica)
ProductService::paginate($query)
    ↓ Service usa BaseRepository
BaseRepository::paginateWithQuery($query)
    ↓ Repository usa Traits del Model
Product::search('laptop')->filter(['category'=>5])->paginate(20)
    ↓ Eloquent ejecuta query
Database → Collection<Product>
    ↓ Service transforma (o Controller para paginación)
ProductData::collect($products)
    ↓ ApiResponseTrait formatea
JsonResponse {success: true, data: [...], links: {...}, meta: {...}}
```

### Componentes Clave

| Componente | Función Principal | Beneficio |
|------------|------------------|-----------|
| **BaseRepository** | Métodos CRUD heredables | Elimina código repetitivo |
| **BaseService** | Métodos comunes heredables | Centraliza lógica común |
| **Searchable Trait** | Búsqueda automática | `?search=x` funciona sin código extra |
| **Filterable Trait** | Filtros dinámicos | `?category=5` usa `filterCategory()` automático |
| **Laravel Data** | DTOs con validación | Type safety + validación automática |
| **Scramble** | Documentación OpenAPI | Genera docs sin escribir YAML |
| **ApiResponseTrait** | Respuestas consistentes | {success, data, message} estandarizado |

### Reglas de Oro (Recordatorio)

1. ✅ **BaseRepository/BaseService** - Siempre extender (nunca reimplementar CRUD)
2. ✅ **Searchable/Filterable** - Siempre usar en Models (búsqueda/filtros automáticos)
3. ✅ **Services retornan DTOs** - NUNCA Models (preparar datos para presentación)
4. ✅ **Controllers solo orquestan** - NUNCA transforman ni tienen lógica
5. ✅ **Repositories retornan Models** - NUNCA DTOs
6. ✅ **Type hints estrictos** - En TODOS los métodos públicos
7. ✅ **PHPDoc completo** - Para Scramble (documentación automática)
8. ✅ **Laravel Data** - Para validación y transformación (no manual)

### Resultado Final

Código que es:
- **Limpio**: Responsabilidades claras por capa
- **Rápido de escribir**: ~150 líneas por recurso completo
- **Fácil de testear**: Capas independientes
- **Bien documentado**: Scramble genera docs automáticamente
- **Mantenible**: Cambios localizados, sin efectos colaterales
- **Escalable**: Agregar recursos es copy-paste template + ajustes mínimos

## 📖 Documento Fuente Completo

El documento detallado de arquitectura con ejemplos extensos está en: `laravel-api-architecture.md`

---

> **"Hacer las cosas bien desde el inicio es más barato que refactorizar después."** - Robert C. Martin

---
> Source: [RuddyG4/laravel-api-architecture](https://github.com/RuddyG4/laravel-api-architecture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
