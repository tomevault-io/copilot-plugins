## pocketbase-plus

> This document provides essential information for agentic coding agents working in this Dart package repository.

# AGENTS.md - Coding Guidelines for pocketbase_plus

This document provides essential information for agentic coding agents working in this Dart package repository.

## Project Overview

Pocketbase Plus is a Dart package that automates model generation for PocketBase projects. It fetches collection schemas from PocketBase and generates type-safe Dart model classes with CRUD helper methods.

**PocketBase SDK Version:** Requires PocketBase Dart SDK >=0.23.0 (compatible with PocketBase v0.23.0+)

**Breaking Changes from 0.18.x:**
- `SchemaField` is now `CollectionField`
- `CollectionModel.schema` is now `CollectionModel.fields`
- `RecordModel.created/updated` properties are deprecated; use `r.get<String>('created')` instead
- `pb.admins` is deprecated; use `pb.collection('_superusers')` for admin authentication

**Generated Files:**
- `<collection>.dart` - Model class with fields, constructor, fromModel, toMap, copyWith, equality, toString, file URL helpers
- `<collection>_extension.dart` - Extension methods for CRUD operations (pocketbase extensions)
- `models.dart` - Barrel file exporting all models and extensions

**Generated Model Features:**
- Collection ID/name constants (`collectionId`, `collectionName`)
- Static CRUD methods (`getOne`, `getList`, `getFullList`, `getFirst`, `create`, `update`, `delete`)
- Paginated result method (`getPaginated`) - returns `PaginatedResult<T>` with metadata
- Realtime subscriptions (`subscribe`, `subscribeAll`)
- Type-safe filter builders (`Model.f.name.eq('value')`)
- Equality and hash code overrides
- `toString()` override
- `copyWith()` method
- File URL helpers (for file fields)
- Auth helpers (for auth collections)

## Build/Lint/Test Commands

### Running the Generator
```bash
# Run with default config (./pocketbase.yaml)
dart run pocketbase_plus:main

# Run with custom config file
dart run pocketbase_plus:main --config pubspec.yaml
dart run pocketbase_plus:main -c path/to/config.yaml

# Override output directory
dart run pocketbase_plus:main --output ./lib/generated
dart run pocketbase_plus:main -o ./lib/generated

# Skip extension generation (no CRUD methods)
dart run pocketbase_plus:main --no-extensions

# Skip barrel file generation
dart run pocketbase_plus:main --no-barrel

# Show help
dart run pocketbase_plus:main --help
```

### Testing
```bash
# Run all tests
dart test

# Run a specific test file
dart test test/pocketbase_plus_test.dart

# Run tests with verbose output
dart test --reporter=expanded

# Run tests with coverage
dart test --coverage=coverage
```

### Linting & Analysis
```bash
# Run static analysis
dart analyze

# Check for issues without fixing
dart analyze --fatal-infos

# Format code (check without modifying)
dart format --output=none --set-exit-if-changed .
```

### Formatting
```bash
# Format all Dart files
dart format .

# Format specific directory
dart format lib/
dart format bin/

# Format specific file
dart format lib/src/models.dart
```

### Package Management
```bash
# Get dependencies
dart pub get

# Upgrade dependencies
dart pub upgrade

# Check for outdated dependencies
dart pub outdated
```

## Code Style Guidelines

### Naming Conventions

**Files & Directories:**
- Snake_case for file names: `models.dart`, `main.dart`
- Directory names should be snake_case: `lib/src/`

**Variables & Parameters:**
- camelCase for variables and parameters: `collection`, `outputDirectory`
- Use descriptive names: `configPath` not `path`

**Classes & Types:**
- PascalCase for class names: `CollectionModel`, `UsersModel`
- Enums use PascalCase with `Enum` suffix: `GenderEnum`, `OverallStateEnum`
- Model classes end with `Model` suffix: `UsersModel`, `MatchesModel`

**Constants:**
- UPPER_CASE for static const String fields: `static const String Id = 'id';`
- Use `// ignore_for_file: constant_identifier_names` for generated code

**Functions:**
- camelCase for function names: `capName()`, `removeSnake()`
- Private functions prefix with underscore: `_privateFunction()`

### Imports

```dart
// Dart SDK imports first
import 'dart:io';
import 'dart:convert';

// Package imports second (alphabetically)
import 'package:args/args.dart';
import 'package:path/path.dart' as pp;
import 'package:pocketbase/pocketbase.dart';
import 'package:yaml/yaml.dart';

// Relative imports last
import 'src/models.dart';
```

**Import Aliases:**
- Use when needed to avoid conflicts: `import 'package:path/path.dart' as pp;`

### Types & Null Safety

**Explicit Typing:**
```dart
// Always specify types explicitly
final String domain;
final List<CollectionField> fields;
final Map<String, dynamic> data;

// Avoid var when type isn't obvious
String collectionName = 'users';  // Good
var collectionName = 'users';      // Avoid
```

**Nullable Types:**
```dart
// Use ? for optional/nullable fields
final String? name;
final DateTime? deletedAt;
final List<String>? mimeTypes;

// Non-nullable for required fields
final String phoneNumber;
final DateTime createdAt;
```

**Factory Constructors:**
```dart
factory UsersModel.fromModel(RecordModel r) {
  return UsersModel(
    id: r.id,
    created: DateTime.parse(r.get<String>('created')),
    name: r.data['name'],
  );
}
```

### Supported Field Types

| Field Type | Dart Type (Required) | Dart Type (Optional) |
|------------|---------------------|----------------------|
| `text` | `String` | `String?` |
| `email` | `String` | `String?` |
| `url` | `String` | `String?` |
| `editor` | `String` | `String?` |
| `number` | `num` | `num?` |
| `bool` | `bool` | `bool?` |
| `date` | `DateTime` | `DateTime?` |
| `datetime` | `DateTime` | `DateTime?` |
| `select` | Generated Enum | Generated Enum? |
| `json` | `Map<String, dynamic>` | `Map<String, dynamic>?` |
| `file` (single) | `String` | `String?` |
| `file` (multiple) | `List<String>` | `List<String>?` |
| `relation` (single) | `String` | `String?` |
| `relation` (multiple) | `List<String>` | `List<String>?` |

### Relation Fields with Expand

Generated models include **expand fields** for relation fields, allowing type-safe access to expanded records:

```dart
// Example: chat_messages collection has "sender" relation to "users" collection
class ChatMessagesModel {
  final String? id;
  final String? sender;              // Relation ID (always available)
  final UsersModel? senderExpanded;  // Expanded record (only when queried with expand)
  // ...
}

// Usage with expand:
final messages = await pb.getChatMessagesList(expand: 'sender');
for (final msg in messages) {
  print(msg.sender);              // ID string
  print(msg.senderExpanded?.name); // Expanded user data (null if not expanded)
}
```

**Expand Field Naming:**
- Single relations (`maxSelect: 1`): `{fieldName}Expanded` with type `TargetModel?`
- Multiple relations (`maxSelect > 1`): `{fieldName}Expanded` with type `List<TargetModel>?`

**Cross-Collection Imports:**
Generation automatically adds imports for related collections:
```dart
import 'package:pocketbase/pocketbase.dart';
import 'users.dart';      // Imported because "sender" relates to "users"
import 'chats.dart';      // Imported because "chat" relates to "chats"
```

**Important Implementation Notes:**
- Expand fields are always nullable (expand is optional at query time)
- The `collectionId` in the PocketBase schema maps the relation to its target collection
- Self-referential relations (collection references itself) are supported

### Static CRUD Methods

Generated model classes include static methods for common operations:

```dart
class UsersModel {
  // ... fields and constructors ...

  // Static fetch methods
  static Future<UsersModel?> getOne(PocketBase pb, String id, {String? expand, String? fields});
  static Future<List<UsersModel>> getList(PocketBase pb, {int page, int perPage, String? filter, String? sort, String? expand, String? fields});
  static Future<List<UsersModel>> getFullList(PocketBase pb, {String? filter, String? sort, String? expand, String? fields, int batch});
  static Future<UsersModel?> getFirst(PocketBase pb, String filter, {String? expand, String? fields});

  // Static mutation methods
  static Future<UsersModel> create(PocketBase pb, Map<String, dynamic> data, {String? expand, String? fields});
  static Future<UsersModel> update(PocketBase pb, String id, Map<String, dynamic> data, {String? expand, String? fields});
  static Future<void> delete(PocketBase pb, String id);

  // Filter builder accessor
  static UsersFilter get f => UsersFilter();
}
```

**Usage:**
```dart
// Fetch single record
final user = await UsersModel.getOne(pb, 'USER_ID');

// Fetch paginated list
final users = await UsersModel.getList(pb, page: 1, perPage: 30);

// Fetch all records
final allUsers = await UsersModel.getFullList(pb);

// Create new record
final newUser = await UsersModel.create(pb, {'name': 'John', 'email': 'john@example.com'});

// Update record
final updated = await UsersModel.update(pb, 'USER_ID', {'name': 'Jane'});

// Delete record
await UsersModel.delete(pb, 'USER_ID');
```

### Type-Safe Filter Builders

Each model generates a companion `Filter` class for building type-safe queries:

```dart
class UsersFilter {
  // Field filter accessors
  _UsersFieldFilter get name;      // For text/email/url/editor fields
  _UsersNumFilter get age;         // For number fields
  _UsersDateFilter get created;     // For date/datetime fields
  _UsersEnumFilter<GenderEnum> get gender;  // For select fields
  _UsersArrayFilter get tags;      // For multi-select relation/file fields

  // Logical operators
  UsersFilter operator &(UsersFilter other);  // AND
  UsersFilter operator |(UsersFilter other);  // OR
  UsersFilter get not;                         // NOT

  String build();  // Returns filter string
}
```

**Usage Examples:**
```dart
// Simple equality
final filter = UsersFilter().name.eq('John');
await UsersModel.getList(pb, filter: filter.build());

// Numeric comparison
final filter = UsersFilter().age.gt(18).age.lt(65);
await UsersModel.getList(pb, filter: filter.build());

// Date filtering
final filter = UsersFilter().created.after(DateTime(2024, 1, 1));
await UsersModel.getList(pb, filter: filter.build());

// Enum filtering
final filter = UsersFilter().gender.eq(GenderEnum.male);
await UsersModel.getList(pb, filter: filter.build());

// Combined conditions (AND)
final filter = UsersFilter().name.eq('John') & UsersFilter().age.gt(18);
await UsersModel.getList(pb, filter: filter.build());

// Combined conditions (OR)
final filter = UsersFilter().name.eq('John') | UsersFilter().name.eq('Jane');
await UsersModel.getList(pb, filter: filter.build());

// NOT condition
final filter = UsersFilter().not.name.eq('blocked');
await UsersModel.getList(pb, filter: filter.build());

// Using the shorthand accessor
final filter = UsersModel.f.name.eq('John');
await UsersModel.getList(pb, filter: filter.build());
```

**Field Filter Types:**
| Field Type | Filter Class | Available Operators |
|------------|---------------|---------------------|
| text, email, url, editor | `FieldFilter` | `eq`, `neq`, `gt`, `gte`, `lt`, `lte`, `like`, `notLike`, `isNull`, `isNotNull`, `contains` |
| number | `NumFilter` | `eq`, `neq`, `gt`, `gte`, `lt`, `lte`, `between` |
| date, datetime | `DateFilter` | `eq`, `neq`, `after`, `before`, `onOrAfter`, `onOrBefore`, `isNull`, `isNotNull` |
| select (enum) | `EnumFilter` | `eq`, `neq`, `isIn` |
| bool | `FieldFilter` | `eq`, `neq`, `isNull`, `isNotNull` |
| relation (single) | `FieldFilter` | `eq`, `neq`, `isNull`, `isNotNull` |
| relation (multiple) | `ArrayFilter` | `contains`, `containsAny`, `hasLength` |

### Paginated Results

Use `getPaginated()` to receive pagination metadata:

```dart
final result = await UsersModel.getPaginated(pb, page: 1, perPage: 30);
print(result.items);        // List<UsersModel>
print(result.page);         // Current page
print(result.perPage);      // Items per page
print(result.totalItems);   // Total items across all pages
print(result.totalPages);   // Total pages
print(result.hasNextPage);  // true if there's a next page
print(result.hasPrevPage);  // true if there's a previous page
```

### Realtime Subscriptions

Subscribe to collection changes:

```dart
// Subscribe to all changes in collection
final unsubscribe = await UsersModel.subscribeAll(pb, (record, action) {
  print('Action: $action'); // 'create', 'update', 'delete'
  print('Record: $record');
});

// Subscribe to specific record
final unsubscribe = await UsersModel.subscribe(pb, 'USER_ID', (record, action) {
  print('Record updated: $record');
});

// Later: stop receiving updates
await unsubscribe();
```

### File URL Helpers

For file fields, generated helpers build URLs:

```dart
class UsersModel {
  final String? avatar;  // Single file field
  
  // Generated helper:
  String? avatarUrl(PocketBase pb, {String? thumb}) {
    // Returns URL with optional thumbnail size
  }
}

// For multiple file fields:
class GalleryModel {
  final List<String>? images;
  
  // Generated helpers:
  String? imagesUrlAt(PocketBase pb, int index, {String? thumb});  // URL at index
  List<String> imagesUrls(PocketBase pb, {String? thumb});          // All URLs
}
```

### Auth Collection Methods

For auth-type collections (`type: 'auth'`), additional helpers are generated:

```dart
extension UsersAuthExtension on PocketBase {
  // Authentication
  Future<AuthResult<UsersModel>> authUsersWithPassword(String email, String password);
  Future<AuthResult<UsersModel>> authUsersRefresh();
  
  // Password reset
  Future<void> requestUsersPasswordReset(String email);
  Future<void> confirmUsersPasswordReset(String token, String password, String passwordConfirm);
  
  // Email verification
  Future<void> requestUsersVerification(String email);
  Future<void> confirmUsersVerification(String token);
}
```

### Collection Constants

Each model exposes collection metadata:

```dart
class UsersModel {
  static const String collectionId = 'COLLECTION_ID';
  static const String collectionName = 'users';
  // ...
}
```

### Equality & ToString

Generated models implement value equality:

```dart
final user1 = UsersModel(id: '1', name: 'John');
final user2 = UsersModel(id: '1', name: 'Jane');

print(user1 == user2);  // true (equality by ID)
print(user1.hashCode);  // Hash based on ID
print(user1.toString()); // UsersModel(id: 1, name: John, ...)
```
| file (multiple) | `ArrayFilter` | `contains`, `containsAny`, `hasLength` |
- The `collectionId` in the PocketBase schema maps the relation to its target collection
- Self-referential relations (collection references itself) are supported

### Known Limitations

**File Uploads:**
The generated `save()` method uses `toMap()` which serializes data as JSON. For file uploads, you must use the PocketBase SDK directly:

```dart
// For file uploads, use the SDK directly:
await pb.collection('users').create(
  body: {'name': 'John'},
  files: [
    http.MultipartFile.fromPath('avatar', '/path/to/file.jpg'),
  ],
);
```

Generated extensions work for all other CRUD operations.

**copyWith Semantics:**
The generated `copyWith` method uses nullable parameters for all fields, including required ones. This follows Dart conventions where passing `null` means "keep the existing value". To explicitly clear an optional field, reconstruct the model without that field.

### Error Handling

**Throw Exceptions:**
```dart
// Throw descriptive exceptions
if (pbConfig == null) {
  throw Exception('Missing "pocketbase" section in configuration.');
}

// Use ArgumentError for invalid arguments
throw ArgumentError("Invalid value: $value");
```

**Try-Catch:**
```dart
try {
  config = loadConfiguration(configPath);
} catch (e) {
  print('Error loading configuration: $e');
  printHelp(parser);
  exit(1);
}
```

### Code Organization

**File Headers (for generated code):**
```dart
// This file is auto-generated. Do not modify manually.
// Model for collection users
// ignore_for_file: constant_identifier_names
```

**Class Structure:**
```dart
class ExampleModel {
  // 1. Static constants
  static const String Id = 'id';
  
  // 2. Instance fields
  final String? id;
  final String name;
  
  // 3. Constructor
  const ExampleModel({
    this.id,
    required this.name,
  });
  
  // 4. Factory constructor
  factory ExampleModel.fromModel(RecordModel r) { ... }
  
  // 5. Instance methods
  Map<String, dynamic> toMap() { ... }
  
  // 6. copyWith method
  ExampleModel copyWith({ ... }) { ... }
}
```

### Documentation

**Library-Level Documentation:**
```dart
/// Support for doing something awesome.
///
/// More dartdocs go here.
library;
```

**Function Documentation:**
```dart
/// Authenticates an admin user with PocketBase.
/// Note: pb.admins is deprecated; prefer pb.collection('_superusers')
Future<void> authenticate(PocketBase pb, String email, String password) async {
  // ignore: deprecated_member_use
  await pb.admins.authWithPassword(email, password);
}

/// Capitalizes the first letter of a string.
String capName(String str) {
  if (str == 'date_time' || str == 'datetime' || str == 'dateTime') {
    return 'DateTimez';
  }
  return str[0].toUpperCase() + str.substring(1);
}
```

### Special Patterns

**Enum with Value:**
```dart
enum GenderEnum {
  male("male"),
  female("female"),
  ;

  final String value;
  const GenderEnum(this.value);

  static GenderEnum fromValue(String value) {
    return GenderEnum.values.firstWhere(
      (enumValue) => enumValue.value == value,
      orElse: () => throw ArgumentError("Invalid value: $value"),
    );
  }
}
```

**DateTime Handling:**
```dart
// Parsing from JSON
DateTime.parse(r.data['date_field'])

// Serializing to JSON
dateTime?.toIso8601String()

// Null-safe handling
r.data['deleted_at'] != null ? DateTime.parse(r.data['deleted_at']) : null
```

**String Buffer for Code Generation:**
```dart
final buffer = StringBuffer();
buffer.writeln('// Comment');
buffer.writeln('class Example {');
buffer.writeln('  final String name;');
buffer.writeln('}');
return buffer.toString();
```

## Project Structure

```
pocketbase_plus/
├── bin/
│   └── main.dart           # CLI entry point
├── lib/
│   ├── pocketbase_plus.dart # Library exports
│   └── src/
│       └── models.dart      # Internal implementation
├── test/
│   └── pocketbase_plus_test.dart
├── example/
│   └── lib/
│       └── models/          # Generated model examples
├── dart-sdk/                # SDK documentation
├── analysis_options.yaml    # Linter configuration
├── pubspec.yaml            # Package configuration
└── README.md
```

## Linting Rules

This project uses `package:lints/recommended.yaml` which includes:

- `avoid_dynamic_calls` - Avoid dynamic method calls
- `avoid_returning_null_for_void` - Don't return null for void
- `avoid_type_to_string` - Avoid .toString() on types
- `camel_case_types` - Use camelCase for types
- `constant_identifier_names` - Use UPPER_CASE for constants
- `curly_braces` - Always use curly braces
- `empty_catches` - Don't use empty catch blocks
- `file_names` - Use snake_case for file names
- `no_duplicate_case_values` - No duplicate case values
- `non_constant_identifier_names` - Use camelCase for identifiers
- `prefer_const_constructors` - Use const constructors
- `prefer_final_fields` - Make fields final
- `prefer_single_quotes` - Use single quotes for strings
- `sort_child_properties_last` - Sort child properties last
- `unnecessary_null_comparison` - Avoid unnecessary null checks
- `use_key_in_widget_constructors` - Use key in widget constructors

## Before Committing

1. Run `dart analyze` and ensure no errors
2. Run `dart format .` to format code
3. Run `dart test` to ensure all tests pass
4. Check for any new dependencies with `dart pub outdated`
5. Ensure generated code in `example/lib/models/` is up to date

---
> Source: [seifalmotaz/pocketbase_plus](https://github.com/seifalmotaz/pocketbase_plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
