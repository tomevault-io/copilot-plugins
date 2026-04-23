## specialonerexvilledash

> Guía de desarrollo para el proyecto Hyundai Special One Rexville Dashboard


# Guía de Desarrollo - Hyundai Special One Rexville Dashboard

## Stack Tecnológico

**Backend:**
- Laravel 12.x con PHP 8.2+
- Laravel Sanctum para autenticación
- Laravel Socialite para autenticación social

**Frontend:**
- Livewire 3.6 para componentes reactivos
- Flux 2.2 para componentes de UI
- Tailwind CSS para estilos
- Vite para compilación de assets

**Base de datos:**
- MySQL/MariaDB
- Migraciones de Laravel
- Seeders para datos iniciales

## Arquitectura del Proyecto

### 1. Patrón Action-Service

El proyecto utiliza un patrón Action-Service donde:

**Actions** (`app/Actions/V1/`):
- Manejan la lógica de negocio específica
- Extienden la clase base `App\Actions\V1\Action`
- Retornan `ActionResult` para respuestas consistentes
- Incluyen validación, permisos y manejo de transacciones
- Se organizan por módulos (Admin, Client, Auth, etc.)

**Services** (`app/Services/V1/`):
- Manejan operaciones CRUD y lógica de datos
- Extienden la clase base `App\Services\V1\Service`
- Proporcionan métodos como `getPaginated()`, `findById()`, etc.
- Se configuran con modelo asociado y campos buscables

### 2. Estructura de ActionResult

```php
// Éxito
return $this->successResult(
    data: $result,
    message: 'Operación completada exitosamente'
);

// Error
return $this->errorResult(
    message: 'Error al procesar',
    errors: $errors,
    statusCode: 400
);

// Error de validación
return $this->validationErrorResult(
    errors: $validator->errors(),
    message: 'Datos inválidos'
);
```

### 3. Componentes Livewire

**Organización** (`app/Livewire/V1/`):
- `Auth/`: Componentes de autenticación
- `Panel/`: Componentes del panel administrativo
- `Components/`: Componentes reutilizables

**Patrón de componentes:**
- Utilizan el trait `HandlesActionResults` para manejo de respuestas
- Inyectan Actions y Services via dependency injection
- Manejan estados de formulario y validación en tiempo real

## Comandos Artisan Personalizados

### Crear un módulo completo:
```bash
php artisan make:module NombreModulo
```
Genera: Modelo, Migración, Service, Actions CRUD, Controlador, Seeder

### Crear Action individual:
```bash
php artisan make:action ModuleName/ActionName
php artisan make:action Admin/CreateAdmin  # Con subdirectorio
```

### Crear Service:
```bash
php artisan make:service ServiceName ModelName
```

## Patrones de UI con Flux

### 1. Layouts
- **Auth**: `v1.layouts.auth.main` - Para login/registro
- **Panel**: `v1.layouts.panel.main` - Para área administrativa

### 2. Componentes Reutilizables

**Formularios:**
```blade
<x-forms.form-field label="Nombre*" for="name">
    <flux:input wire:model="name" placeholder="Ingrese nombre" />
</x-forms.form-field>
```

**Tablas con filtros:**
```blade
<x-table.table :data="$items" searchPlaceholder="Buscar...">
    <x-slot name="filters">
        <!-- Filtros aquí -->
    </x-slot>
    <x-slot name="colums">
        <!-- Columnas aquí -->
    </x-slot>
    <x-slot name="rows">
        <!-- Filas aquí -->
    </x-slot>
</x-table.table>
```

**Contenedores:**
```blade
<x-containers.card-container>
    <!-- Contenido aquí -->
</x-containers.card-container>
```

### 3. Navegación y Breadcrumbs

```blade
@section('breadcrumbs')
<flux:breadcrumbs.item href="{{route('v1.panel.home')}}" separator="slash">
    {{ __('panel.breadcrumb_home') }}
</flux:breadcrumbs.item>
<flux:breadcrumbs.item separator="slash">
    {{ __('panel.breadcrumb_current') }}
</flux:breadcrumbs.item>
@endsection
```

## Sistema de Traducciones

### Organización:
- `lang/es/` - Español
- `lang/en/` - Inglés

### Archivos principales:
- `panel.php` - Textos del panel administrativo
- `auth.php` - Textos de autenticación
- `validation.php` - Mensajes de validación

### Uso:
```blade
{{ __('panel.create_admin') }}
{{ __('panel.admin_created_successfully') }}
```

## Rutas y Controladores

### Estructura de rutas (`routes/web.php`):
```php
// Rutas públicas
Route::get('/login', LoginComponent::class)->name('login');

// Rutas protegidas
Route::group([
    "prefix" => "v1/panel",
    "middleware" => "auth:admin",
    "as" => "v1.panel."
], function () {
    Route::get('/home', HomeComponent::class)->name("home");
    Route::get('/admins', GetAdminsComponent::class)->name("admins.index");
    // ...
});
```

## Mejores Prácticas de Desarrollo

### 1. Crear un nuevo módulo:

1. **Ejecutar comando de módulo:**
```bash
php artisan make:module Cliente
```

2. **Configurar el Service** en `app/Services/V1/ClienteService.php`:
```php
$this->searchableFields = ['name', 'email', 'phone'];
$this->per_page = 10;
```

3. **Implementar Actions** en `app/Actions/V1/Cliente/`:
- `CreateClienteAction.php`
- `UpdateClienteAction.php`
- `GetClienteAction.php`
- etc.

4. **Crear componentes Livewire:**
```bash
php artisan make:livewire V1/Panel/Cliente/GetClientesComponent
php artisan make:livewire V1/Panel/Cliente/CreateClienteComponent
```

5. **Crear vistas** en `resources/views/v1/panel/cliente/`

6. **Agregar traducciones** en `lang/es/panel.php` y `lang/en/panel.php`

7. **Agregar rutas** en `routes/web.php`

### 2. Patrón de Action:

```php
class CreateClienteAction extends Action
{
    public function __construct(
        private ClienteService $clienteService
    ) {}

    public function handle($data): ActionResult
    {
        // 1. Validar permisos
        $this->validatePermissions(['clientes.create']);

        // 2. Validar datos
        $validated = $this->validateData($data, [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:clientes,email',
        ], [
            'name.required' => 'El nombre es obligatorio',
            'email.required' => 'El email es obligatorio',
        ]);

        // 3. Lógica de negocio con transacción
        return DB::transaction(function () use ($validated) {
            $cliente = $this->clienteService->create($validated);

            return $this->successResult(
                data: $cliente,
                message: 'Cliente creado exitosamente'
            );
        });
    }
}
```

### 3. Patrón de Componente Livewire:

```php
class CreateClienteComponent extends Component
{
    use HandlesActionResults;

    public $name;
    public $email;

    private $createClienteAction;

    public function boot(CreateClienteAction $createClienteAction)
    {
        $this->createClienteAction = $createClienteAction;
    }

    public function createCliente()
    {
        $result = $this->executeAction($this->createClienteAction, [
            'name' => $this->name,
            'email' => $this->email,
        ], true);

        if ($result->isSuccess()) {
            return redirect()->route('v1.panel.clientes.index');
        }
    }

    public function render()
    {
        return view('v1.panel.cliente.create-cliente-component');
    }
}
```

### 4. Patrón de Vista:

```blade
@section('title', __('panel.clientes'))
@section('description', __('panel.create_cliente'))

@section('breadcrumbs')
<flux:breadcrumbs.item href="{{route('v1.panel.home')}}" separator="slash">
    {{ __('panel.breadcrumb_home') }}
</flux:breadcrumbs.item>
<flux:breadcrumbs.item href="{{route('v1.panel.clientes.index')}}" separator="slash">
    {{ __('panel.breadcrumb_clientes') }}
</flux:breadcrumbs.item>
<flux:breadcrumbs.item separator="slash">
    {{ __('panel.breadcrumb_create') }}
</flux:breadcrumbs.item>
@endsection

<x-containers.card-container>
    <form wire:submit.prevent="createCliente">
        <div class="flex-1 space-y-6">
            <x-forms.form-field label="{{ __('panel.name') }}*" for="name">
                <flux:input wire:model="name" placeholder="Nombre del cliente" />
            </x-forms.form-field>

            <x-forms.form-field label="{{ __('panel.email') }}*" for="email">
                <flux:input wire:model="email" type="email" placeholder="email@ejemplo.com" />
            </x-forms.form-field>

            <div class="flex justify-end space-x-3 pt-0 px-6 pb-6">
                <flux:button href="{{route('v1.panel.clientes.index')}}" type="button">
                    {{ __('panel.cancel') }}
                </flux:button>
                <flux:button type="submit" wire:loading.attr="disabled">
                    <span wire:loading.remove>{{ __('panel.create_cliente') }}</span>
                    <span wire:loading>{{ __('panel.loading') }}</span>
                </flux:button>
            </div>
        </div>
    </form>
</x-containers.card-container>
```

## Notas Importantes

1. **Siempre usar ActionResult** para respuestas consistentes
2. **Validar permisos** en Actions cuando sea necesario
3. **Usar transacciones** para operaciones críticas
4. **Implementar búsqueda y filtros** en componentes de listado
5. **Usar traducciones** para todos los textos de la interfaz
6. **Seguir la convención de nombres** establecida
7. **Usar los componentes Flux** para mantener consistencia visual
8. **Implementar loading states** en formularios
9. **Manejar errores** apropiadamente con el trait HandlesActionResults
10. **Usar breadcrumbs** para navegación clara
11. **Errores en acciones**: es importante que en la accion cuando hay un error, utilizar ValidationErrorException y no utlizar try catch si no es necesario porque ya la clase Action ya tiene un bloque de manejo de errores.

## Comandos Útiles

```bash
# Desarrollo
composer run dev

# Limpiar cache
php artisan config:clear
php artisan view:clear
php artisan route:clear

# Migraciones
php artisan migrate
php artisan db:seed

# Tests
composer test
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Forreal360) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
