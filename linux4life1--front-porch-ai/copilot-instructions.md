## front-porch-ai

> This document provides comprehensive guidelines for AI coding agents (Claude, Cursor, GitHub Copilot, etc.) working on the Front Porch AI codebase. Follow these rules to ensure high-quality, consistent contributions.

# AI Agent Guidelines for Front Porch AI

This document provides comprehensive guidelines for AI coding agents (Claude, Cursor, GitHub Copilot, etc.) working on the Front Porch AI codebase. Follow these rules to ensure high-quality, consistent contributions.

## Table of Contents

- [Project Architecture Overview](#project-architecture-overview)
- [Preferred Riverpod Patterns](#preferred-riverpod-patterns)
- [Flutter Widget & State Rules](#flutter-widget--state-rules)
- [Python Sidecar Conventions](#python-sidecar-conventions)
- [Code Style & Naming](#code-style--naming)
- [Files/Areas Requiring Discussion](#filesareas-requiring-discussion)
- [Testing Expectations](#testing-expectations)
- [Examples: Good vs Bad Changes](#examples-good-vs-bad-changes)

## Project Architecture Overview

Front Porch AI is a Flutter desktop application for AI-powered character chat and storytelling. Key architectural components:

### Core Structure
```
lib/
├── main.dart              # App entry point with service initialization
├── app_version.dart       # Version information
├── database/              # Drift database schema and migrations
├── models/                # Data models (characters, chats, etc.)
├── providers/             # State management (currently Provider, migrating to Riverpod)
├── services/              # Business logic and external integrations
├── ui/                    # Flutter UI components
│   ├── dialogs/           # Modal dialogs
│   ├── layout/            # Main app layout
│   ├── pages/             # Screen pages
│   └── widgets/           # Reusable UI components
└── utils/                 # Helper utilities
```

### Key Services
- **ChatService**: Core chat logic and AI integration
- **CharacterRepository**: Character data management
- **EmbeddingSidecar**: Rust-based local embedding server
- **TTSService**: Text-to-speech with multiple engines
- **StorageService**: File and data persistence
- **CloudSyncService**: Google Drive/WebDAV synchronization

### External Dependencies
- **Python sidecars**: TTS (kokoro_tts.py), STT (whisper_stt.py)
- **Rust embedding server**: ONNX-based text embeddings
- **KoboldCpp**: Local LLM backend
- **Database**: SQLite via Drift ORM

## Preferred Riverpod Patterns

**Note**: The codebase currently uses Provider but is migrating to Riverpod. When adding new state management:

### Provider Setup
```dart
@riverpod
class CharacterList extends _$CharacterList {
  @override
  Future<List<Character>> build() async {
    return ref.watch(characterRepositoryProvider).getAllCharacters();
  }

  Future<void> addCharacter(Character character) async {
    await ref.read(characterRepositoryProvider).save(character);
    ref.invalidateSelf();
  }
}
```

### Widget Usage
```dart
class CharacterListWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final characters = ref.watch(characterListProvider);

    return characters.when(
      data: (data) => ListView.builder(...),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

### Best Practices
- Use `AsyncNotifier` for async operations
- Prefer `ref.watch` for reactive dependencies
- Use `ref.read` for one-time actions
- Implement proper error handling with `AsyncValue`

## Flutter Widget & State Rules

### Widget Patterns
- Use `ConsumerWidget` or `StatefulWidget` appropriately
- Prefer composition over inheritance
- Keep widgets focused on single responsibilities
- Use `const` constructors where possible

### State Management
```dart
// Good: Reactive state with Provider
class ChatPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<ChatState>(
      builder: (context, chatState, child) {
        return ListView.builder(
          itemCount: chatState.messages.length,
          itemBuilder: (context, index) => MessageWidget(
            message: chatState.messages[index],
          ),
        );
      },
    );
  }
}
```

### Performance Considerations
- Use `const` for static widgets
- Implement `shouldRebuild` for custom `Consumer`
- Avoid unnecessary rebuilds with `select()`
- Use `AnimatedBuilder` for complex animations

## Python Sidecar Conventions

Python scripts run as subprocesses for TTS, STT, and other media processing.

### Input/Output Protocol
```python
# Read JSON request from stdin
import sys
import json

def main():
    try:
        line = sys.stdin.readline().strip()
        if not line:
            print("No input", file=sys.stderr)
            sys.exit(1)

        request = json.loads(line)

        # Process request...

        # Output result to stdout (JSON)
        result = {"success": True, "data": output_data}
        print(json.dumps(result))

    except Exception as e:
        # Errors to stderr
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

### Error Handling
- Always catch exceptions and exit with non-zero code
- Log errors to stderr, not stdout
- Validate input JSON thoroughly
- Handle missing dependencies gracefully

### Subprocess Management (Dart side)
```dart
// Good: Proper subprocess handling
class TTSEngine {
  Future<String> generateAudio(String text, String voice) async {
    final process = await Process.start('python', ['kokoro_tts.py']);

    // Send JSON request
    final request = {
      'text': text,
      'voice': voice,
      'output': outputPath,
    };
    process.stdin.writeln(jsonEncode(request));
    await process.stdin.close();

    // Read response
    final output = await process.stdout.transform(utf8.decoder).join();
    final errorOutput = await process.stderr.transform(utf8.decoder).join();

    final exitCode = await process.exitCode;
    if (exitCode != 0) {
      throw Exception('TTS failed: $errorOutput');
    }

    final result = jsonDecode(output);
    return result['path'];
  }
}
```

## Code Style & Naming

### Dart Conventions
- Follow `flutter_lints` rules (see `analysis_options.yaml`)
- Use camelCase for variables/methods, PascalCase for classes
- Prefix private members with `_`
- Use meaningful, descriptive names
- Prefer `final` over `var` when possible

### File Organization
- One class per file (except small related classes)
- Group related functionality in directories
- Use snake_case for file names
- Import order: Dart SDK, packages, local imports

### Import Style
```dart
// Good: Organized imports
import 'dart:async';
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../models/character.dart';
import '../services/chat_service.dart';
import 'message_bubble.dart';
```

## Files/Areas Requiring Discussion

### Never Touch Without Discussion
- `database/migrations/`: Database schema changes require migration planning
- `lib/main.dart`: Core service initialization order
- `pubspec.yaml`: Dependencies and version changes
- `analysis_options.yaml`: Linting rules
- Release scripts in `scripts/`

### Sensitive Areas
- Authentication and API key handling
- Database queries (performance implications)
- UI layout changes (affects all platforms)
- Network request patterns
- File system operations

### Require Architecture Review
- New services or major refactors
- State management changes
- External API integrations
- Performance-critical code paths

## Testing Expectations

### Unit Tests
```dart
// Good: Comprehensive unit test
void main() {
  group('CharacterRepository', () {
    late CharacterRepository repository;
    late AppDatabase db;

    setUp(() async {
      db = await AppDatabase.instance();
      repository = CharacterRepository(db);
    });

    tearDown(() async {
      await db.close();
    });

    test('should save and retrieve character', () async {
      final character = Character(name: 'Test Character', description: 'Test desc');

      final id = await repository.save(character);
      final retrieved = await repository.getById(id);

      expect(retrieved.name, equals(character.name));
      expect(retrieved.description, equals(character.description));
    });

    test('should handle missing character gracefully', () async {
      expect(() => repository.getById(-1), throwsA(isA<NotFoundException>()));
    });
  });
}
```

### Test Coverage Requirements
- Aim for 80%+ coverage on new code
- Test error conditions and edge cases
- Mock external dependencies
- Test async operations properly

### Integration Testing
- Test service interactions
- Verify database operations
- Test UI workflows (manual verification)

## Examples: Good vs Bad Changes

### Good: Proper Error Handling
```dart
// GOOD: Comprehensive error handling
Future<void> loadCharacter(int id) async {
  try {
    final character = await _repository.getById(id);
    if (character == null) {
      _showError('Character not found');
      return;
    }
    _currentCharacter = character;
    notifyListeners();
  } catch (e) {
    _showError('Failed to load character: $e');
    debugPrint('Character load error: $e');
  }
}
```

### Bad: Silent Failures
```dart
// BAD: No error handling
Future<void> loadCharacter(int id) async {
  _currentCharacter = await _repository.getById(id);
  notifyListeners();
}
```

### Good: Clear Naming and Documentation
```dart
/// Calculates the semantic similarity between two text embeddings.
/// Returns a value between 0.0 (completely dissimilar) and 1.0 (identical).
///
/// Uses cosine similarity on normalized embeddings.
double calculateSimilarity(List<double> embedding1, List<double> embedding2) {
  // Implementation...
}
```

### Bad: Unclear Purpose
```dart
double calc(List<double> a, List<double> b) {
  // Magic similarity calculation
  return 0.0; // TODO: implement
}
```

### Good: Proper State Management
```dart
class ChatState extends ChangeNotifier {
  List<Message> _messages = [];
  bool _isLoading = false;

  List<Message> get messages => _messages;
  bool get isLoading => _isLoading;

  Future<void> sendMessage(String text) async {
    _isLoading = true;
    notifyListeners();

    try {
      final response = await _chatService.sendMessage(text);
      _messages.add(response);
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
}
```

### Bad: Direct State Mutation
```dart
// BAD: Exposing mutable state
class ChatState extends ChangeNotifier {
  List<Message> messages = []; // Public mutable list!

  void addMessage(Message msg) {
    messages.add(msg); // No notification!
  }
}
```

### Good: Python Sidecar Implementation
```python
import sys
import json
from pathlib import Path

def validate_request(req):
    required = ['text', 'voice', 'output_path']
    for field in required:
        if field not in req:
            raise ValueError(f"Missing required field: {field}")

def main():
    try:
        line = sys.stdin.readline().strip()
        if not line:
            raise ValueError("No input received")

        request = json.loads(line)
        validate_request(request)

        # Generate audio...
        output_path = Path(request['output_path'])
        # ... audio generation logic ...

        result = {
            'success': True,
            'output_path': str(output_path),
            'duration_seconds': 2.5
        }
        print(json.dumps(result))

    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

### Bad: Poor Error Handling
```python
# BAD: No validation or proper error handling
import sys
import json

data = json.loads(sys.stdin.read())
text = data['text']  # KeyError if missing!
voice = data['voice']
# ... proceed without error checking ...
```

Remember: Always prioritize code quality, maintainability, and user experience. When in doubt, ask for clarification or review from maintainers.

---
> Source: [linux4life1/front-porch-AI](https://github.com/linux4life1/front-porch-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
