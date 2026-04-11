## petjournal

> This project is a Flutter application called "PetJournal". This app allows users to store and manage important information about their pets. It currently stores data in a local sqlite database using Drift.

### Project Overview

This project is a Flutter application called "PetJournal". This app allows users to store and manage important information about their pets. It currently stores data in a local sqlite database using Drift.

### General

Rules:

- Use the Dart programming language.
- Use the Flutter framework.
- The `pubspec.yaml` file indicates the following SDK constraint: `sdk: ^3.8.0`. Adhere to this version.
- When creating a new method, provide a doc comment (///) to document the method and arguments.
- Use Riverpod for state management.
- Use Drift for local database storage.

### Dependencies

The following dependencies are used in this project. Please use them as specified in the `pubspec.yaml` file.

- `cupertino_icons: ^1.0.8`
- `drift: ^2.26.0`
- `drift_flutter: ^0.2.4`
- `go_router: ^14.8.1`
- `flutter_riverpod: ^2.6.1`
- `riverpod_annotation: ^2.6.1`
- `material_color_utilities: ^0.11.1`
- `flutter_markdown: ^0.7.7`
- `fl_chart: ^0.70.2`
- `path_provider: ^2.1.5`
- `intl: ^0.20.2`
- `url_launcher: ^6.3.1`
- `package_info_plus: ^8.3.0`
- `freezed_annotation: ^3.0.0`
- `json_annotation: ^4.9.0`

**Development Dependencies:**

- `flutter_test`
- `flutter_lints: ^5.0.0`
- `riverpod_generator: ^2.6.5`
- `build_runner: ^2.4.15`
- `custom_lint: ^0.7.5`
- `riverpod_lint: ^2.6.5`
- `mockito: ^5.4.5`
- `test: ^1.25.15`
- `matcher: ^0.12.17`
- `drift_dev: ^2.26.0`
- `freezed: ^3.0.6`
- `json_serializable: ^6.9.5`

### UI

Rules:

- Make any UI you generate beautiful and user-friendly.

### Context 7

Rules:
-Always use the Context7 MCP server to reference documentation for libraries like Flutter, Riverpod, Drift, and Mockito.

### Tests

Rules:

- Write unit tests for any code you generate.
- Write integration tests for any code you generate.
- If mocks are required, use the mockito library.
- Tests should be generated in the /test directory.
- Use the test directory structure to mirror the lib directory structure.
- When writing tests to get, list or delete a record, write additional tests to check that the records not found are handled correctly.
- When writing tests to check that a record is created, updated or deleted, test all the fields of a record that have been updated or set in the test contain the expected values.
- Use "[method name] should [expected behaviour] when [condition]" for the test description.
- Use the test name to describe the test case. For example, "GetUserById Should return User When user exists" instead of "testGetUserById".
- Use the AAA pattern for test cases. Arrange, Act, Assert. Use comments to separate the sections of the test.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zjcz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zjcz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
