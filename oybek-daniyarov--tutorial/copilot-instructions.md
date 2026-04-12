## tutorial

> This guide defines coding standards and best practices for Laravel 12 using Filament 4, tailored for productivity in Cursor IDE.


## Laravel 12 + Filament 4 Standards

This guide defines coding standards and best practices for Laravel 12 using Filament 4, tailored for productivity in Cursor IDE.

### ✅ PHP 8.4+ Usage

- Use modern syntax: property hooks, asymmetric visibility, `array_find()`
- Prefer typed properties and promoted parameters
- Use null-safe operators and `readonly` where applicable
- Apply enum casting in models:

    ```php
    protected function casts(): array
    {
        return ['status' => OrderStatus::class];
    }
    ```

- Use native types in return types and parameters:

    ```php
    public function getTotal(): float { ... }
    ```

### ✅ Modular Feature-Based Structure

Each feature should have its own self-contained structure:

```
App/
├── Features/
│   └── Orders/
│       ├── Actions/
│       ├── Controllers/
│       │   └── CreateOrderController.php
│       ├── Models/
│       │   └── Order.php
│       ├── Services/
│       │   └── OrderService.php
│       ├── Providers/
│       │   └── OrderServiceProvider.php
│       ├── Routes/
│       │   └── api.php
│       ├── Database/
│       │   └── Migrations/
│       │       ├── create_orders_table.php
│       │       └── create_order_items_table.php
│       ├── Data/
│       │   ├── Payload/CreateOrderPayload.php
│       │   └── Responses/OrderResponseData.php
│       └── ...
├── Providers/
│   ├── AppServiceProvider.php (application-level services)
│   └── FeaturesServiceProvider.php (registers all feature providers)
└── bootstrap/
    └── providers.php (loads FeaturesServiceProvider)
```

- Each feature's `ServiceProvider` should register its bindings, routes, migrations, etc.
- Centralized `FeaturesServiceProvider` registers all feature providers:

    ```php
    public function register(): void
    {
        $this->app->register(\App\Features\Orders\Providers\OrderServiceProvider::class);
        $this->app->register(\App\Features\Auth\Providers\UserServiceProvider::class);
        // ... other feature providers
    }
    ```

- Load `FeaturesServiceProvider` in `bootstrap/providers.php`:

    ```php
    return [
        App\Providers\AppServiceProvider::class,
        App\Providers\FeaturesServiceProvider::class,
        App\Providers\Filament\AdminPanelProvider::class,
    ];
    ```

### ✅ API Route Organization

- Store feature routes in `App\Features\<Feature>\Routes\api.php`
- Load routes in feature service providers:

    ```php
    public function boot(): void
    {
        Route::middleware('api')
            ->prefix('api')
            ->group(base_path('app/Features/Orders/Routes/api.php'));
    }
    ```

- Keep main `routes/api.php` minimal, only for global API middleware
- Use descriptive route names and proper grouping

### ✅ Database Migration Organization

- Store feature migrations in `App\Features\<Feature>\Database\Migrations\`
- Load migrations in feature service providers:

    ```php
    public function boot(): void
    {
        // Load feature migrations
        $this->loadMigrationsFrom(base_path('app/Features/Orders/Database/Migrations'));
    }
    ```

- Keep main `database/migrations/` minimal, only for framework-level migrations
- Use descriptive migration names with proper timestamps
- Group related migrations within each feature

### ✅ Models

- Use singular, StudlyCase (e.g., `Order`, `Customer`)
- Use `HasFactory` trait
- Eloquent relationships with descriptive method names
- Use `enum` casting and `date`/`datetime` casts appropriately

### ✅ Controllers & Services

- Thin controllers: delegate logic to Services
- Services return Data Objects, not Models (for consistency and type safety)

- Inject dependencies with property promotion:

    ```php
    public function __construct(private OrderService $service) {}
    ```

- Service Layer Pattern:

    ```php
    // Service returns Data Objects
    class OrderService
    {
        public function createOrder(CreateOrderPayload $data): OrderResponseData
        {
            $order = Order::create($data->toArray());
            return OrderResponseData::fromModel($order);
        }

        public function updateOrder(Order $order, UpdateOrderPayload $data): OrderResponseData
        {
            $order->update($data->toArray());
            return OrderResponseData::fromModel($order->fresh());
        }

        public function getOrderByUlid(string $ulid): ?OrderResponseData
        {
            $order = Order::where('ulid', $ulid)->first();
            return $order ? OrderResponseData::fromModel($order) : null;
        }
    }
    ```

- Controller Usage:

    ```php
    public function store(CreateOrderPayload $data, OrderService $service): OrderResponseData
    {
        return $service->createOrder($data);
    }
    ```

- **Actions for Multiple Services:**

    When a controller needs to use multiple services, use Actions instead:

    **Use Actions when:**
    - Controller needs 2+ services
    - Complex business logic spanning multiple domains
    - Orchestrating multiple operations
    - Transaction management across services

    **Use Services when:**
    - Single domain operations
    - Simple CRUD operations
    - Single service dependency

    ```php
    // Action class
    class CreateOrderWithPaymentAction
    {
        public function __construct(
            private OrderService $orderService,
            private PaymentService $paymentService,
            private NotificationService $notificationService,
        ) {}

        public function execute(CreateOrderPayload $data): OrderResponseData
        {
            // Create order
            $order = $this->orderService->createOrder($data);

            // Process payment
            $payment = $this->paymentService->processPayment($order);

            // Send notification
            $this->notificationService->sendOrderConfirmation($order);

            return $order;
        }
    }

    // Controller using Action
    public function store(CreateOrderPayload $data, CreateOrderWithPaymentAction $action): OrderResponseData
    {
        return $action->execute($data);
    }
    ```

- **Action Structure:**

    ```
    App/Features/Orders/Actions/
    ├── CreateOrderWithPaymentAction.php
    ├── UpdateOrderStatusAction.php
    └── CancelOrderAction.php
    ```

### ✅ Spatie Data Standards

- Store in `App\Features\<Feature>\Data`
- Annotate with `#[TypeScript]` for TS transformer
- Use generic data objects for common relationships
- Use `PaginatedDataCollection` for paginated responses

- Basic Data Object:

    ```php
    #[TypeScript]
    class CreateOrderPayload extends Data
    {
        public function __construct(
            public string $customerName,
            public OrderStatus $status,
        ) {}

        public static function rules(): array
        {
            return [
                'customerName' => ['required', 'string'],
                'status' => ['required', new Enum(OrderStatus::class)],
            ];
        }
    }
    ```

- Generic Relationship Data Objects:

    ```php
    // For simple entity references (id, name, ulid)
    #[TypeScript]
    class CompanyReferenceData extends Data
    {
        public function __construct(
            public string $ulid,
            public string $name,
        ) {}

        public static function fromModel(Company $company): self
        {
            return new self(
                ulid: $company->ulid,
                name: $company->name,
            );
        }
    }

    // For user references
    #[TypeScript]
    class UserReferenceData extends Data
    {
        public function __construct(
            public string $ulid,
            public string $name,
            public string $email,
        ) {}

        public static function fromModel(User $user): self
        {
            return new self(
                ulid: $user->ulid,
                name: $user->name,
                email: $user->email,
            );
        }
    }
    ```

- Response Data with Relationships:

    ```php
    #[TypeScript]
    class OrderResponseData extends Data
    {
        public function __construct(
            public string $ulid,
            public string $orderNumber,
            public OrderStatus $status,
            public CompanyReferenceData $company,
            public UserReferenceData $customer,
            public string $createdAt,
            public string $updatedAt,
        ) {}

        public static function fromModel(Order $order): self
        {
            return new self(
                ulid: $order->ulid,
                orderNumber: $order->order_number,
                status: $order->status,
                company: CompanyReferenceData::fromModel($order->company),
                customer: UserReferenceData::fromModel($order->customer),
                createdAt: $order->created_at->toISOString(),
                updatedAt: $order->updated_at->toISOString(),
            );
        }
    }
    ```

### ✅ Enum Standards (Filament Style Only)

- Define Enums in `App\Features\[FeatureName]\Enums`
- Implement `HasLabel`, optionally `HasColor`, `HasIcon`

    ```php
    enum OrderStatus: string implements HasLabel, HasColor {
        // ... enum cases
    }
    ```

- Usage:

    ```php
    Select::make('status')->options(OrderStatus::class)->required();
    TextColumn::make('status')->badge()
    ```

### ✅ Filament Forms

- Schema: `App\Filament\Resources\<Entity>\Schemas\<Entity>Form`
- Modularize with `Tabs`, `Fieldset`, and reusable methods

### ✅ Filament Tables

- Schema: `App\Filament\Resources\<Entity>\Schemas\<Entity>Table`
- Use `badge`, `sortable`, `searchable`, and closures for filters

### ✅ Folder & File Structure Summary

```
App/
├── Enums/
├── Features/
│   └── Orders/
│       ├── Actions/
│       ├── Controllers/
│       ├── Models/
│       ├── Services/
│       ├── Providers/
│       ├── Routes/
│       │   └── api.php
│       ├── Database/
│       │   └── Migrations/
│       └── Data/
├── Providers/
│   ├── AppServiceProvider.php
│   └── FeaturesServiceProvider.php
├── Filament/Resources/Orders/
│   ├── OrderResource.php
│   └── Schemas/
│       ├── OrderForm.php
│       └── OrdersTable.php
└── bootstrap/
    └── providers.php
```

### ✅ Validation & Request Handling

- Only use Spatie `Data`
- Define `rules()` in Payload classes
- Use `#[TypeScript]` to generate frontend types

### ✅ Pagination with Laravel Data

- Use `PaginatedDataCollection` for paginated responses:

    ```php
    // Service method returning paginated data
    public function getAllUsers(int $perPage = 15): PaginatedDataCollection
    {
        $users = User::paginate($perPage);
        return UserResponseData::collect($users);
    }

    // Controller using paginated data
    public function index(Request $request): JsonResponse
    {
        $perPage = $request->get('per_page', 15);
        $users = $this->userService->getAllUsers($perPage);

        return response()->json($users);
    }

    // Alternative: Using PaginatedDataCollection directly
    public function getAllUsers(int $perPage = 15): PaginatedDataCollection
    {
        $users = User::paginate($perPage);
        return UserResponseData::collect($users, PaginatedDataCollection::class);
    }
    ```

### ✅ TypeScript Transformer

- Annotate classes with `#[TypeScript]` or `/** @typescript */`:

    ```php
    // Using attribute syntax
    #[TypeScript]
    class UserResponseData extends Data
    {
        public function __construct(
            public string $ulid,
            public string $name,
            public string $email,
        ) {}
    }

    // Or using docblock syntax
    /** @typescript */
    class UserResponseData extends Data
    {
        public function __construct(
            public string $ulid,
            public string $name,
            public string $email,
        ) {}
    }
    ```

- Generate TypeScript types:

    ```bash
    # Generate TypeScript types
    php artisan typescript:transform
    ```

    ```

    ```

## Laravel 12 + Filament 4 Standards

This guide defines coding standards and best practices for Laravel 12 using Filament 4, tailored for productivity in Cursor IDE.

### ✅ PHP 8.4+ Usage

- Use modern syntax: property hooks, asymmetric visibility, `array_find()`
- Prefer typed properties and promoted parameters
- Use null-safe operators and `readonly` where applicable
- Apply enum casting in models:

    ```php
    protected function casts(): array
    {
        return ['status' => OrderStatus::class];
    }
    ```

- Use native types in return types and parameters:

    ```php
    public function getTotal(): float { ... }
    ```

### ✅ Modular Feature-Based Structure

Each feature should have its own self-contained structure:

```
App/
├── Features/
│   └── Orders/
│       ├── Actions/
│       ├── Controllers/
│       │   └── CreateOrderController.php
│       ├── Models/
│       │   └── Order.php
│       ├── Services/
│       │   └── OrderService.php
│       ├── Providers/
│       │   └── OrderServiceProvider.php
│       ├── Routes/
│       │   └── api.php
│       ├── Database/
│       │   └── Migrations/
│       │       ├── create_orders_table.php
│       │       └── create_order_items_table.php
│       ├── Data/
│       │   ├── Payload/CreateOrderPayload.php
│       │   └── Responses/OrderResponseData.php
│       └── ...
├── Providers/
│   ├── AppServiceProvider.php (application-level services)
│   └── FeaturesServiceProvider.php (registers all feature providers)
└── bootstrap/
    └── providers.php (loads FeaturesServiceProvider)
```

- Each feature's `ServiceProvider` should register its bindings, routes, migrations, etc.
- Centralized `FeaturesServiceProvider` registers all feature providers:

    ```php
    public function register(): void
    {
        $this->app->register(\App\Features\Orders\Providers\OrderServiceProvider::class);
        $this->app->register(\App\Features\Auth\Providers\UserServiceProvider::class);
        // ... other feature providers
    }
    ```

- Load `FeaturesServiceProvider` in `bootstrap/providers.php`:

    ```php
    return [
        App\Providers\AppServiceProvider::class,
        App\Providers\FeaturesServiceProvider::class,
        App\Providers\Filament\AdminPanelProvider::class,
    ];
    ```

### ✅ API Route Organization

- Store feature routes in `App\Features\<Feature>\Routes\api.php`
- Load routes in feature service providers:

    ```php
    public function boot(): void
    {
        Route::middleware('api')
            ->prefix('api')
            ->group(base_path('app/Features/Orders/Routes/api.php'));
    }
    ```

- Keep main `routes/api.php` minimal, only for global API middleware
- Use descriptive route names and proper grouping

### ✅ Database Migration Organization

- Store feature migrations in `App\Features\<Feature>\Database\Migrations\`
- Load migrations in feature service providers:

    ```php
    public function boot(): void
    {
        // Load feature migrations
        $this->loadMigrationsFrom(base_path('app/Features/Orders/Database/Migrations'));
    }
    ```

- Keep main `database/migrations/` minimal, only for framework-level migrations
- Use descriptive migration names with proper timestamps
- Group related migrations within each feature

### ✅ Models

- Use singular, StudlyCase (e.g., `Order`, `Customer`)
- Use `HasFactory` trait
- Eloquent relationships with descriptive method names
- Use `enum` casting and `date`/`datetime` casts appropriately

### ✅ Controllers & Services

- Thin controllers: delegate logic to Services
- Services return Data Objects, not Models (for consistency and type safety)

- Inject dependencies with property promotion:

    ```php
    public function __construct(private OrderService $service) {}
    ```

- Service Layer Pattern:

    ```php
    // Service returns Data Objects
    class OrderService
    {
        public function createOrder(CreateOrderPayload $data): OrderResponseData
        {
            $order = Order::create($data->toArray());
            return OrderResponseData::fromModel($order);
        }

        public function updateOrder(Order $order, UpdateOrderPayload $data): OrderResponseData
        {
            $order->update($data->toArray());
            return OrderResponseData::fromModel($order->fresh());
        }

        public function getOrderByUlid(string $ulid): ?OrderResponseData
        {
            $order = Order::where('ulid', $ulid)->first();
            return $order ? OrderResponseData::fromModel($order) : null;
        }
    }
    ```

- Controller Usage:

    ```php
    public function store(CreateOrderPayload $data, OrderService $service): OrderResponseData
    {
        return $service->createOrder($data);
    }
    ```

- **Actions for Multiple Services:**

    When a controller needs to use multiple services, use Actions instead:

    **Use Actions when:**
    - Controller needs 2+ services
    - Complex business logic spanning multiple domains
    - Orchestrating multiple operations
    - Transaction management across services

    **Use Services when:**
    - Single domain operations
    - Simple CRUD operations
    - Single service dependency

    ```php
    // Action class
    class CreateOrderWithPaymentAction
    {
        public function __construct(
            private OrderService $orderService,
            private PaymentService $paymentService,
            private NotificationService $notificationService,
        ) {}

        public function execute(CreateOrderPayload $data): OrderResponseData
        {
            // Create order
            $order = $this->orderService->createOrder($data);

            // Process payment
            $payment = $this->paymentService->processPayment($order);

            // Send notification
            $this->notificationService->sendOrderConfirmation($order);

            return $order;
        }
    }

    // Controller using Action
    public function store(CreateOrderPayload $data, CreateOrderWithPaymentAction $action): OrderResponseData
    {
        return $action->execute($data);
    }
    ```

- **Action Structure:**

    ```
    App/Features/Orders/Actions/
    ├── CreateOrderWithPaymentAction.php
    ├── UpdateOrderStatusAction.php
    └── CancelOrderAction.php
    ```

### ✅ Spatie Data Standards

- Store in `App\Features\<Feature>\Data`
- Annotate with `#[TypeScript]` for TS transformer
- Use generic data objects for common relationships
- Use `PaginatedDataCollection` for paginated responses

- Basic Data Object:

    ```php
    #[TypeScript]
    class CreateOrderPayload extends Data
    {
        public function __construct(
            public string $customerName,
            public OrderStatus $status,
        ) {}

        public static function rules(): array
        {
            return [
                'customerName' => ['required', 'string'],
                'status' => ['required', new Enum(OrderStatus::class)],
            ];
        }
    }
    ```

- Generic Relationship Data Objects:

    ```php
    // For simple entity references (id, name, ulid)
    #[TypeScript]
    class CompanyReferenceData extends Data
    {
        public function __construct(
            public string $ulid,
            public string $name,
        ) {}

        public static function fromModel(Company $company): self
        {
            return new self(
                ulid: $company->ulid,
                name: $company->name,
            );
        }
    }

    // For user references
    #[TypeScript]
    class UserReferenceData extends Data
    {
        public function __construct(
            public string $ulid,
            public string $name,
            public string $email,
        ) {}

        public static function fromModel(User $user): self
        {
            return new self(
                ulid: $user->ulid,
                name: $user->name,
                email: $user->email,
            );
        }
    }
    ```

- Response Data with Relationships:

    ```php
    #[TypeScript]
    class OrderResponseData extends Data
    {
        public function __construct(
            public string $ulid,
            public string $orderNumber,
            public OrderStatus $status,
            public CompanyReferenceData $company,
            public UserReferenceData $customer,
            public string $createdAt,
            public string $updatedAt,
        ) {}

        public static function fromModel(Order $order): self
        {
            return new self(
                ulid: $order->ulid,
                orderNumber: $order->order_number,
                status: $order->status,
                company: CompanyReferenceData::fromModel($order->company),
                customer: UserReferenceData::fromModel($order->customer),
                createdAt: $order->created_at->toISOString(),
                updatedAt: $order->updated_at->toISOString(),
            );
        }
    }
    ```

### ✅ Enum Standards (Filament Style Only)

- Define Enums in `App\Features\[FeatureName]\Enums`
- Implement `HasLabel`, optionally `HasColor`, `HasIcon`

    ```php
    enum OrderStatus: string implements HasLabel, HasColor {
        // ... enum cases
    }
    ```

- Usage:

    ```php
    Select::make('status')->options(OrderStatus::class)->required();
    TextColumn::make('status')->badge()
    ```

### ✅ Filament Forms

- Schema: `App\Filament\Resources\<Entity>\Schemas\<Entity>Form`
- Modularize with `Tabs`, `Fieldset`, and reusable methods

### ✅ Filament Tables

- Schema: `App\Filament\Resources\<Entity>\Schemas\<Entity>Table`
- Use `badge`, `sortable`, `searchable`, and closures for filters

### ✅ Folder & File Structure Summary

```
App/
├── Enums/
├── Features/
│   └── Orders/
│       ├── Actions/
│       ├── Controllers/
│       ├── Models/
│       ├── Services/
│       ├── Providers/
│       ├── Routes/
│       │   └── api.php
│       ├── Database/
│       │   └── Migrations/
│       └── Data/
├── Providers/
│   ├── AppServiceProvider.php
│   └── FeaturesServiceProvider.php
├── Filament/Resources/Orders/
│   ├── OrderResource.php
│   └── Schemas/
│       ├── OrderForm.php
│       └── OrdersTable.php
└── bootstrap/
    └── providers.php
```

### ✅ Validation & Request Handling

- Only use Spatie `Data`
- Define `rules()` in Payload classes
- Use `#[TypeScript]` to generate frontend types

### ✅ Pagination with Laravel Data

- Use `PaginatedDataCollection` for paginated responses:

    ```php
    // Service method returning paginated data
    public function getAllUsers(int $perPage = 15): PaginatedDataCollection
    {
        $users = User::paginate($perPage);
        return UserResponseData::collect($users);
    }

    // Controller using paginated data
    public function index(Request $request): JsonResponse
    {
        $perPage = $request->get('per_page', 15);
        $users = $this->userService->getAllUsers($perPage);

        return response()->json($users);
    }

    // Alternative: Using PaginatedDataCollection directly
    public function getAllUsers(int $perPage = 15): PaginatedDataCollection
    {
        $users = User::paginate($perPage);
        return UserResponseData::collect($users, PaginatedDataCollection::class);
    }
    ```

### ✅ TypeScript Transformer

- Annotate classes with `#[TypeScript]` or `/** @typescript */`:

    ```php
    // Using attribute syntax
    #[TypeScript]
    class UserResponseData extends Data
    {
        public function __construct(
            public string $ulid,
            public string $name,
            public string $email,
        ) {}
    }

    // Or using docblock syntax
    /** @typescript */
    class UserResponseData extends Data
    {
        public function __construct(
            public string $ulid,
            public string $name,
            public string $email,
        ) {}
    }
    ```

- Generate TypeScript types:

    ```bash
    # Generate TypeScript types
    php artisan typescript:transform
    ```

    ```

    ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oybek-daniyarov)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/oybek-daniyarov)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
