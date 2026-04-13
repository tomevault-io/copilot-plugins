## cine-search

> 🏗️ Clean Architecture Structure


🏗️ Clean Architecture Structure
Data Layer

All data access must be in core/data/datasources/
All models must be in core/data/models/
All repository implementations must be in core/data/repositories/
Separate datasources by responsibility (local, remote)
Implement abstract interfaces for each datasource

Domain Layer

All abstract repositories must be in core/domain/repositories/
All entities must be in core/domain/entities/
All use cases must be in core/domain/usecases/
Repositories must define only method signatures
Use Either<Failure, Success> return types in repositories and use cases

Presentation Layer

All controllers must be in features/{feature}/presentation/controllers/
All pages must be in features/{feature}/presentation/pages/
All feature-specific widgets in features/{feature}/presentation/widgets/
Reusable widgets in core/presentation/widgets/

📁 Core Services

Any external services must be in core/services/
Examples: HttpService, StorageService, NotificationService
Always implement interfaces for easier testing
Register in dependency injection container

🎯 Model Requirements

Models must extend their respective entity
Must implement: fromMap(), toMap(), fromJson(), toJson()
Models handle serialization/deserialization
Keep entities pure (no external dependencies)

🔧 Use Case Pattern

One use case = one responsibility
Must implement UseCase<Type, Params> interface
Controllers call use cases, never repositories directly
Handle business logic validation

📦 Dependency Injection

Use GetIt for dependency injection
Register dependencies in core/injection/injection.dart
Follow order: datasources → repositories → usecases → controllers
Use interfaces for loose coupling

🚨 Error Handling

Create core/errors/failures.dart with error types
Use Either<Failure, Success> pattern
Implement global error handling
Never throw exceptions in domain layer

🧪 Testing Structure

test/unit/ - Unit tests (usecases, repositories)
test/widget/ - Widget tests
test/integration/ - Integration tests
All use cases must have unit tests
Mock external dependencies

🔍 Naming Conventions

Controllers: {Feature}Controller
UseCases: {Action}{Entity}UseCase
Repositories: {Entity}Repository (interface) / {Entity}RepositoryImpl
Models: {Entity}Model
Entities: {Entity}Entity or just {Entity}
Datasources: {Entity}{Type}DataSource

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Snowman-Labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
