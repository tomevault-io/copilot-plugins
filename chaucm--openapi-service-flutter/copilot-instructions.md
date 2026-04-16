## openapi-service-flutter

> The goal of `openapi_service_flutter` is to generate clean, type-safe service code and DTOs that can be easily integrated into Flutter projects with well-structured service layers. The generated code follows modern Dart/Flutter patterns and provides:

# Project Overview

## Project Goal

The goal of `openapi_service_flutter` is to generate clean, type-safe service code and DTOs that can be easily integrated into Flutter projects with well-structured service layers. The generated code follows modern Dart/Flutter patterns and provides:

- **Separation of concerns**: DTOs for data modeling, services for API operations
- **Type safety**: Compile-time validation through Freezed models and strict typing
- **Error handling**: Functional error handling with Either types for robust service layers
- **Configuration**: Flexible service configuration for different environments
- **Maintainability**: Clean, readable generated code that follows Dart conventions

## Key Package Information

- **Package Name**: `openapi_service_flutter`
- **Type**: Dart code generator for Flutter
- **License**: MIT
- **Primary Dependencies**: `dio`, `either_dart`, `freezed`, `build_runner`

## Generated Code Structure

For each `.openapi.yaml` or `.openapi.json` file, the generator produces:

1. **DTOs File** (`.openapi.dtos.dart`):
   - Freezed data classes with `@freezed` annotation
   - Enum types with `@JsonValue` annotations
   - `fromJson` factory constructors
   - Part directives for `.freezed.dart` and `.g.dart`

2. **Service File** (`.openapi.service.dart`):
   - Service class with HTTP methods
   - Dio-based HTTP client integration
   - Either-based error handling
   - Import of corresponding DTOs file

## Code Generation Flow

```
.openapi.yaml/.json
    ↓
OpenApiServiceBuilder reads spec
    ↓
Uses open-api-dart to parse
    ↓
OpenApiLibraryGenerator generates:
    ├── DTOs library → .openapi.dtos.dart
    └── Service library → .openapi.service.dart
    ↓
build_runner triggers:
    ├── freezed → .freezed.dart
    └── json_serializable → .g.dart
```

## Main Components

### Core Generator
- `OpenApiServiceBuilder` - Main builder orchestrating generation
- `OpenApiLibraryGenerator` - Generates DTO and service libraries
- `CustomAllocator` - Manages imports and code organization

### Runtime Library
- `DefaultErrorHandler` - User-friendly error handling
- `ApiError` - Structured error type
- `RequestContext` - Request debugging information
- `DefaultDio` - Opinionated HTTP client setup
- `SmartUploadInterceptor` - Intelligent file upload handling

### OpenAPI Parsing
- `open-api-dart` package (internal)
- Supports OpenAPI 2.0, 3.0, and 3.1.1
- Handles schema parsing and validation

## Development Workflow

### For Generator Development
```bash
# Run tests
dart test

# Format code
dart format .

# Analyze code
dart analyze

# Generate example code
cd example && dart run build_runner build --delete-conflicting-outputs
```

### For Users (Flutter Apps)
```bash
# Place OpenAPI spec in lib/services/
# Run code generation
dart run build_runner build --delete-conflicting-outputs

# Use generated services
import 'package:your_app/services/my_api.openapi.service.dart';
```

## Key Features

1. **Type-Safe Generation**: Compile-time type checking for API calls
2. **Freezed Integration**: Immutable DTOs with copy methods
3. **Error Handling**: Either-based functional error handling
4. **File Uploads**: Binary and multipart upload support with progress
5. **Prefix Filtering**: Generate services for specific API subsets
6. **Duplicate Detection**: Automatic schema deduplication
7. **Enum Support**: Proper JSON serialization with @JsonValue
8. **Custom Error Handlers**: Extensible error handling system

## Project Structure

```
lib/
  ├── openapi_service_flutter.dart  # Main exports
  ├── runtime.dart                  # Runtime library exports
  └── src/
      ├── openapi_service_builder.dart  # Code generator
      ├── custom_allocator.dart         # Import management
      └── runtime/
          ├── error/                # Error handling
          └── dio/                  # HTTP utilities

open-api-dart/                      # Internal OpenAPI parser
  └── lib/
      ├── v2.dart                   # OpenAPI 2.0 support
      └── v3.dart                   # OpenAPI 3.0+ support

example/                            # Example usage
  └── lib/service/                  # Example OpenAPI specs

test/                               # Test suite
  ├── fixtures/                     # Test OpenAPI specs
  └── *_test.dart                   # Test files
```

## File References

- Project overview: [CLAUDE.md](mdc:CLAUDE.md)
- User documentation: [README.md](mdc:README.md)
- OpenAPI parser: [open-api-dart/claude.md](mdc:open-api-dart/claude.md)
- Build config: [build.yaml](mdc:build.yaml)
- Main builder: [openapi_service_builder.dart](mdc:lib/src/openapi_service_builder.dart)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ChauCM) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
