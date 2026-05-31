## tuto

> - Laravel 12.x con PHP 8.2+


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

## Componentes Reutilizables

### 1. Componentes de Formularios (`resources/views/components/forms/`)

#### `<x-forms.form-field>`
Wrapper estándar para todos los campos de formulario:
```blade
<x-forms.form-field label="{{ __('panel.name') }}*" for="name" :error="$errors->first('name')">
    <flux:input
        id="name"
        wire:model="name"
        placeholder="Nombre"
        error="{{ $errors->first('name') }}"
    />
</x-forms.form-field>
```

#### `<x-forms.flatpickr-date>`
Componente de selección de fecha con Flatpickr:
```blade
<x-forms.form-field label="{{ __('panel.start_date') }}*" for="start_date" :error="$errors->first('start_date')">
    <x-forms.flatpickr-date
        name="start_date"
        wire:model="start_date"
        dateFormat="m/d/Y"
        placeholder="{{ __('panel.start_date') }}"
        minDate="today"
        error="{{ $errors->first('start_date') }}"
        required
    />
</x-forms.form-field>
```

**Propiedades disponibles:**
- `dateFormat`: Formato de fecha (por defecto 'd/m/Y')
- `placeholder`: Texto placeholder
- `minDate/maxDate`: Fechas límite
- `locale`: Idioma ('es' por defecto)
- `required`: Si es obligatorio
- `size`: Tamaño ('xs', 'sm', 'md', 'lg', 'xl')

#### `<x-forms.file-upload>`
Componente de subida de archivos con drag & drop:
```blade
<x-forms.form-field label="{{ __('panel.image') }}*" for="file" :error="$errors->first('file')">
    <x-forms.file-upload
        name="file"
        wireModel="file"
        accept="image/*"
        :error="$errors->first('file')"
        required
    />
</x-forms.form-field>
```

### 2. Componentes de Tabla (`resources/views/components/table/`)

#### `<x-table.table>`
Componente principal de tabla con paginación, búsqueda y filtros:
```blade
<x-table.table
    :data="$items"
    :perPageOptions="$perPageOptions"
    :currentPerPage="$perPage"
    :search="$search"
    searchPlaceholder="{{ __('panel.search_placeholder') }}"
>
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

#### Filtros de Tabla
Los filtros van dentro del slot `filters` usando `<flux:field>`:
```blade
<x-slot name="filters">
    <!-- Filtro de estado -->
    <flux:field class="w-full">
        <flux:label>{{ __('panel.status') }}</flux:label>
        <flux:select wire:model.live="status" size="sm" placeholder="{{ __('panel.all_statuses') }}">
            <flux:select.option value="active">{{ __('panel.active') }}</flux:select.option>
            <flux:select.option value="inactive">{{ __('panel.inactive') }}</flux:select.option>
        </flux:select>
    </flux:field>

    <!-- Filtro de fecha desde -->
    <x-forms.flatpickr-date
        name="filter_start_date"
        wire:model.live="filter_start_date"
        size="sm"
        label="{{ __('panel.filter_start_date') }}"
        dateFormat="m/d/Y"
        placeholder="{{ __('panel.filter_start_date') }}"
        locale="{{ app()->getLocale() }}"
    />

    <!-- Filtro de fecha hasta -->
    <x-forms.flatpickr-date
        name="filter_end_date"
        wire:model.live="filter_end_date"
        size="sm"
        label="{{ __('panel.filter_end_date') }}"
        dateFormat="m/d/Y"
        placeholder="{{ __('panel.filter_end_date') }}"
        locale="{{ app()->getLocale() }}"
    />
</x-slot>
```

#### Columnas de Tabla
```blade
<x-slot name="colums">
    <x-table.colum
        sortable="true"
        sortField="name"
        :currentSortBy="$sortBy"
        :currentSortDirection="$sortDirection"
    >
        {{ __('panel.name') }}
    </x-table.colum>

    <x-table.colum
        sortable="true"
        sortField="created_at"
        :currentSortBy="$sortBy"
        :currentSortDirection="$sortDirection"
    >
        {{ __('panel.created_at') }}
    </x-table.colum>

    <x-table.colum>{{ __('panel.actions') }}</x-table.colum>
</x-slot>
```

#### Filas de Tabla
```blade
<x-slot name="rows">
    @foreach($items as $item)
        <x-table.row>
            <x-table.cell>
                {{ $item->name }}
            </x-table.cell>
            <x-table.cell>
                <flux:badge color="{{ $item->status == 'active' ? 'green' : 'red' }}">
                    {{ $item->status }}
                </flux:badge>
            </x-table.cell>
            <x-table.cell>
                <flux:button.group>
                    <flux:button size="sm" icon="pencil" href="{{ route('items.edit', $item->id) }}"></flux:button>
                    <flux:button size="sm" icon="trash" wire:click="delete({{ $item->id }})" wire:confirm="¿Confirmar eliminación?"></flux:button>
                </flux:button.group>
            </x-table.cell>
        </x-table.row>
    @endforeach
</x-slot>
```

### 3. Componentes de Contenedores

#### `<x-containers.card-container>`
Contenedor principal para todas las páginas:
```blade
<x-containers.card-container>
    <!-- Contenido aquí -->
</x-containers.card-container>
```

### 4. Componentes de Botones

#### `<x-buttons.button-module>`
Botón estándar para acciones de módulo:
```blade
<x-buttons.button-module
    icon="plus"
    href="{{ route('items.create') }}"
    label="{{ __('panel.new_item') }}"
    variant="primary"
/>
```

## Patrones de UI con Flux

### 1. Estructura de Vista Estándar

```blade
@section('title', __('panel.items'))
@section('description', __('panel.item_management'))

@section('breadcrumbs')
<flux:breadcrumbs.item href="{{route('v1.panel.home')}}" separator="slash">
    {{ __('panel.breadcrumb_home') }}
</flux:breadcrumbs.item>
<flux:breadcrumbs.item separator="slash">
    {{ __('panel.breadcrumb_items') }}
</flux:breadcrumbs.item>
@endsection

@section('actions')
<x-buttons.button-module
    icon="plus"
    href="{{ route('v1.panel.items.create') }}"
    label="{{ __('panel.new_item') }}"
    variant="primary"
/>
@endsection

<x-containers.card-container>
    <!-- Contenido principal -->
</x-containers.card-container>
```

### 2. Estructura de Formulario Estándar

```blade
<x-containers.card-container>
    <form wire:submit.prevent="createItem">
        <div class="flex-1 space-y-6">
            <!-- Campos del formulario usando x-forms.form-field -->

            <div class="flex justify-end space-x-3 pt-0 px-6 pb-6">
                <flux:button
                    href="{{ route('v1.panel.items.index') }}"
                    type="button"
                    wire:click="cancel"
                >
                    {{ __('panel.cancel') }}
                </flux:button>
                <flux:button
                    type="submit"
                    wire:loading.attr="disabled"
                    wire:loading.class="opacity-50 cursor-not-allowed"
                >
                    <span wire:loading.remove>{{ __('panel.create_item') }}</span>
                    <span wire:loading>{{ __('panel.loading') }}</span>
                </flux:button>
            </div>
        </div>
    </form>
</x-containers.card-container>
```

### 3. Confirmaciones y Acciones

**Usar `wire:confirm` para confirmaciones simples:**
```blade
<flux:button
    wire:click="deleteItem({{ $item->id }})"
    wire:confirm="{{ __('panel.confirm_delete') }}"
>
    {{ __('panel.delete') }}
</flux:button>
```

**NO usar modales para confirmaciones simples**, seguir el patrón establecido con `wire:confirm`.

### 4. Estados de Carga y Feedback

```blade
<!-- Estados de carga en botones -->
<flux:button wire:loading.attr="disabled">
    <span wire:loading.remove>{{ __('panel.save') }}</span>
    <span wire:loading>{{ __('panel.saving') }}</span>
</flux:button>

<!-- Feedback con session flash -->
@if(session()->has('success'))
    <flux:alert variant="success">{{ session('success') }}</flux:alert>
@endif
```

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

### 4. Patrón de Vista de Listado:

```blade
@section('title', __('panel.clientes'))
@section('description', __('panel.cliente_management'))

@section('breadcrumbs')
<flux:breadcrumbs.item href="{{route('v1.panel.home')}}" separator="slash">
    {{ __('panel.breadcrumb_home') }}
</flux:breadcrumbs.item>
<flux:breadcrumbs.item separator="slash">
    {{ __('panel.breadcrumb_clientes') }}
</flux:breadcrumbs.item>
@endsection

@section('actions')
<x-buttons.button-module
    icon="plus"
    href="{{ route('v1.panel.clientes.create') }}"
    label="{{ __('panel.new_cliente') }}"
    variant="primary"
/>
@endsection

<x-containers.card-container>
    <x-table.table
        :data="$clientes"
        :perPageOptions="$perPageOptions"
        :currentPerPage="$perPage"
        :search="$search"
        searchPlaceholder="{{ __('panel.search_clientes_placeholder') }}"
    >
        <x-slot name="filters">
            <!-- Filtros usando flux:field -->
        </x-slot>
        <x-slot name="colums">
            <!-- Columnas usando x-table.colum -->
        </x-slot>
        <x-slot name="rows">
            <!-- Filas usando x-table.row y x-table.cell -->
        </x-slot>
    </x-table.table>
</x-containers.card-container>
```

### 5. Patrón de Vista de Formulario:

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
            <x-forms.form-field label="{{ __('panel.name') }}*" for="name" :error="$errors->first('name')">
                <flux:input wire:model="name" placeholder="Nombre del cliente" />
            </x-forms.form-field>

            <x-forms.form-field label="{{ __('panel.email') }}*" for="email" :error="$errors->first('email')">
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
4. **Usar componentes reutilizables** para mantener consistencia
5. **Usar traducciones** para todos los textos de la interfaz
6. **Seguir la convención de nombres** establecida
7. **Usar wire:confirm** para confirmaciones simples, NO modales
8. **Implementar loading states** en formularios
9. **Manejar errores** apropiadamente con el trait HandlesActionResults
10. **Usar breadcrumbs** para navegación clara
11. **Usar x-forms.flatpickr-date** para campos de fecha
12. **Usar x-forms.file-upload** para subida de archivos
13. **Mantener estructura de filtros consistente** en tablas

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
alwaysApply: true
---


# Migraciones
php artisan migrate
php artisan db:seed

# Tests
composer test
```
alwaysApply: true
---

---
> Source: [Forreal360/SpecialOneRexvilleDash](https://github.com/Forreal360/SpecialOneRexvilleDash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
