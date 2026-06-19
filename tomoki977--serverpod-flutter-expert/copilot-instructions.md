## serverpod-flutter-expert

> >


# Serverpod 3.4.x + Flutter Expert Skill

You are an expert in **Serverpod 3.4.x** (Dart backend framework) and **Flutter** (frontend). Always use Serverpod 3.4.x APIs, package versions `^3.4.0`, and modern Dart null-safety syntax.

> **Critical rule:** Always run `dart run serverpod_cli generate` after any `.spy.yaml` or endpoint change.

---

## 1. Project Structure

```
my_project/
├── my_project_server/          # Serverpod backend
│   ├── bin/
│   │   └── main.dart           # Entry point
│   ├── lib/
│   │   ├── src/
│   │   │   ├── endpoints/      # Endpoint classes (extend Endpoint)
│   │   │   ├── models/         # .spy.yaml model definitions
│   │   │   └── generated/      # Auto-generated — NEVER edit manually
│   │   └── server.dart
│   ├── migrations/             # Schema migration SQL files
│   ├── config/
│   │   ├── development.yaml
│   │   ├── staging.yaml
│   │   ├── production.yaml
│   │   └── passwords.yaml      # Secrets — NEVER commit to version control
│   └── pubspec.yaml
├── my_project_client/          # Generated Dart client — do NOT edit
└── my_project_flutter/         # Flutter frontend
    ├── lib/
    │   ├── src/
    │   │   ├── screens/
    │   │   ├── widgets/
    │   │   ├── providers/
    │   │   └── services/
    │   └── main.dart
    └── pubspec.yaml
```

### Project Creation (3.4.x)

```bash
# Install CLI
dart pub global activate serverpod_cli

# Create new project (full stack)
serverpod create my_project

# Create with specific template
serverpod create my_project --template server-only

# After creation, start Docker services then run
cd my_project/my_project_server
docker-compose up --build --detach
dart run bin/main.dart --apply-migrations
```

---

## 2. Models (.spy.yaml)

Models are defined in `lib/src/models/`. Run `dart run serverpod_cli generate` after every change.

### Basic Model

```yaml
# lib/src/models/user_profile.spy.yaml
class: UserProfile
table: user_profiles
fields:
  userId: int
  displayName: String
  bio: String?
  avatarUrl: String?
  metadata: Map<String, dynamic>?   # JSON field (3.4.x native support)
  createdAt: DateTime
  updatedAt: DateTime
indexes:
  user_profiles_user_id_idx:
    fields: userId
    unique: true
  user_profiles_display_name_idx:
    fields: displayName
```

### Enum Model

```yaml
# lib/src/models/order_status.spy.yaml
enum: OrderStatus
serialized: byName              # 3.4.x: serialize as string name, not index
values:
  - pending
  - paid
  - shipped
  - delivered
  - cancelled
```

### Model with Enum, JSON, and Composite Index

```yaml
# lib/src/models/order.spy.yaml
class: Order
table: orders
fields:
  userId: int
  status: OrderStatus
  totalCents: int
  shippingAddress: Map<String, dynamic>?  # JSON field
  tags: List<String>?                      # JSON array field
  createdAt: DateTime
  updatedAt: DateTime
indexes:
  orders_user_status_idx:
    fields: userId, status       # Composite index
  orders_created_idx:
    fields: createdAt
```

### One-to-Many Relationship

```yaml
# lib/src/models/post.spy.yaml
class: Post
table: posts
fields:
  authorId: int
  title: String
  content: String
  isPublished: bool, default=false
  publishedAt: DateTime?
  viewCount: int, default=0
indexes:
  posts_author_idx:
    fields: authorId
  posts_published_idx:
    fields: isPublished, publishedAt
```

```yaml
# lib/src/models/comment.spy.yaml
class: Comment
table: comments
fields:
  postId: int
  authorId: int
  body: String
  createdAt: DateTime
indexes:
  comments_post_idx:
    fields: postId
```

### Many-to-Many (Junction Table)

```yaml
# lib/src/models/post_tag.spy.yaml
class: PostTag
table: post_tags
fields:
  postId: int
  tagId: int
indexes:
  post_tags_unique_idx:
    fields: postId, tagId
    unique: true
```

---

## 3. Endpoints

Endpoint classes live in `lib/src/endpoints/` and extend `Endpoint`. Public methods are automatically exposed as RPC calls.

### Standard Endpoint Pattern

```dart
// lib/src/endpoints/post_endpoint.dart
import 'package:serverpod/serverpod.dart';
import '../generated/protocol.dart';

class PostEndpoint extends Endpoint {
  /// Public: no authentication required
  Future<List<Post>> getPublishedPosts(
    Session session, {
    int limit = 20,
    int offset = 0,
  }) async {
    return await Post.db.find(
      session,
      where: (t) => t.isPublished.equals(true),
      orderBy: (t) => t.publishedAt,
      orderDescending: true,
      limit: limit,
      offset: offset,
    );
  }

  /// Requires login — override requireLogin for the entire endpoint
  @override
  bool get requireLogin => true;

  Future<Post> createPost(
    Session session,
    String title,
    String content,
  ) async {
    if (title.trim().isEmpty) throw ArgumentError('Title cannot be empty');
    if (content.trim().isEmpty) throw ArgumentError('Content cannot be empty');

    final userId = await session.auth.authenticatedUserId;
    if (userId == null) throw ServerpodUnauthenticatedException();

    final now = DateTime.now().toUtc();
    final post = Post(
      authorId: userId,
      title: title.trim(),
      content: content.trim(),
      isPublished: false,
      createdAt: now,
      updatedAt: now,
    );
    return await Post.db.insertRow(session, post);
  }

  Future<Post?> getPost(Session session, int postId) async {
    return await Post.db.findById(session, postId);
  }

  Future<Post> updatePost(
    Session session,
    int postId,
    String title,
    String content,
  ) async {
    final userId = await session.auth.authenticatedUserId;
    if (userId == null) throw ServerpodUnauthenticatedException();

    final post = await Post.db.findById(session, postId);
    if (post == null) throw NotFoundException('Post $postId not found');
    if (post.authorId != userId) throw ForbiddenException('Not your post');

    return await Post.db.updateRow(
      session,
      post.copyWith(
        title: title.trim(),
        content: content.trim(),
        updatedAt: DateTime.now().toUtc(),
      ),
    );
  }

  Future<void> deletePost(Session session, int postId) async {
    final userId = await session.auth.authenticatedUserId;
    if (userId == null) throw ServerpodUnauthenticatedException();

    final post = await Post.db.findById(session, postId);
    if (post == null) throw NotFoundException('Post $postId not found');
    if (post.authorId != userId) throw ForbiddenException('Not your post');

    await Post.db.deleteRow(session, post);
  }
}
```

### Per-Method Auth (Mixed Endpoint)

```dart
class ArticleEndpoint extends Endpoint {
  // Public method
  Future<List<Article>> listPublic(Session session) async {
    return await Article.db.find(
      session,
      where: (t) => t.isPublished.equals(true),
    );
  }

  // Authenticated method — check manually
  Future<Article> createDraft(Session session, String title) async {
    final userId = await session.auth.authenticatedUserId;
    if (userId == null) throw ServerpodUnauthenticatedException();
    // ...
  }
}
```

### Logging in Endpoints

```dart
Future<void> someAction(Session session) async {
  session.log('Starting someAction', level: LogLevel.debug);
  try {
    // ... business logic
    session.log('someAction completed successfully');
  } catch (e, stackTrace) {
    session.log(
      'someAction failed: $e',
      level: LogLevel.error,
      exception: e,
      stackTrace: stackTrace,
    );
    rethrow;
  }
}
```

---

## 4. ORM Queries (3.4.x)

```dart
// ── FIND ─────────────────────────────────────────────────────────────────────

// All rows
final all = await Post.db.find(session);

// Filter
final published = await Post.db.find(
  session,
  where: (t) => t.isPublished.equals(true),
);

// Compound filter
final recent = await Post.db.find(
  session,
  where: (t) =>
      t.isPublished.equals(true) &
      t.publishedAt.greaterThan(
        DateTime.now().toUtc().subtract(const Duration(days: 7)),
      ),
  orderBy: (t) => t.publishedAt,
  orderDescending: true,
  limit: 10,
  offset: 0,
);

// By primary key
final post = await Post.db.findById(session, id);

// First match
final first = await Post.db.findFirstRow(
  session,
  where: (t) => t.title.like('%flutter%'),
);

// ── INSERT ────────────────────────────────────────────────────────────────────

final saved = await Post.db.insertRow(session, post);
final savedMany = await Post.db.insert(session, [post1, post2, post3]);

// ── UPDATE ────────────────────────────────────────────────────────────────────

final updated = await Post.db.updateRow(session, post.copyWith(title: 'New'));
await Post.db.update(session, manyPosts);

// ── DELETE ────────────────────────────────────────────────────────────────────

await Post.db.deleteRow(session, post);
await Post.db.deleteWhere(
  session,
  where: (t) =>
      t.isPublished.equals(false) &
      t.createdAt.lessThan(
        DateTime.now().toUtc().subtract(const Duration(days: 90)),
      ),
);

// ── COUNT ─────────────────────────────────────────────────────────────────────

final total = await Post.db.count(
  session,
  where: (t) => t.authorId.equals(userId),
);

// ── TRANSACTIONS ──────────────────────────────────────────────────────────────

await session.db.transaction((tx) async {
  final post = await Post.db.insertRow(session, newPost, transaction: tx);
  await PostTag.db.insert(
    session,
    tagIds.map((tid) => PostTag(postId: post.id!, tagId: tid)).toList(),
    transaction: tx,
  );
});
```

---

## 5. Authentication (3.4.x)

### Server Setup

```dart
// bin/main.dart
import 'package:serverpod/serverpod.dart';
import 'package:serverpod_auth_server/serverpod_auth_server.dart' as auth;
import 'package:my_project_server/src/generated/endpoints.dart';
import 'package:my_project_server/src/generated/protocol.dart';

void main(List<String> args) async {
  // Configure auth module before starting
  auth.AuthConfig.set(auth.AuthConfig(
    sendValidationEmail: (session, email, validationCode) async {
      // Send via your email provider (SendGrid, SES, etc.)
      session.log('Sending validation email to $email');
      return true;
    },
    sendPasswordResetEmail: (session, userInfo, validationCode) async {
      session.log('Sending password reset to ${userInfo.email}');
      return true;
    },
    // 3.4.x: Configure allowed origins for web
    allowedRequestOrigins: ['https://myapp.com', 'http://localhost:3000'],
  ));

  final pod = Serverpod(
    args,
    Protocol(),
    Endpoints(),
  );

  await pod.start();
}
```

### Checking Auth

```dart
// In endpoint methods
final userId = await session.auth.authenticatedUserId;
if (userId == null) throw ServerpodUnauthenticatedException();

// Get full user info
final userInfo = await session.auth.authenticatedUserInfo;
```

### Flutter Client Authentication

```dart
import 'package:serverpod_auth_client/serverpod_auth_client.dart';
import 'package:serverpod_auth_email_flutter/serverpod_auth_email_flutter.dart';
import 'package:serverpod_auth_google_flutter/serverpod_auth_google_flutter.dart';

// ── Sign up ───────────────────────────────────────────────────────────────────
Future<void> signUp(String email, String password) async {
  final success = await EmailAuth.createAccount(
    email: email,
    password: password,
    // Display name optional but recommended
    displayName: 'New User',
  );
  if (!success) throw Exception('Sign up failed');
}

// ── Email sign-in ─────────────────────────────────────────────────────────────
Future<void> signIn(String email, String password) async {
  final response = await EmailAuth.signIn(
    email: email,
    password: password,
  );
  if (!response.success) {
    throw Exception(response.failReason?.name ?? 'Sign in failed');
  }
  await SessionManager.instance.registerSignedInUser(
    response.userInfo!,
    response.keyId!,
    response.key!,
  );
}

// ── Google Sign-In ────────────────────────────────────────────────────────────
Future<void> signInWithGoogle() async {
  final response = await GoogleSignIn.authenticate();
  if (response == null) return; // User cancelled
  await SessionManager.instance.registerSignedInUser(
    response.userInfo,
    response.keyId,
    response.key,
  );
}

// ── Sign out ──────────────────────────────────────────────────────────────────
Future<void> signOut() async {
  await SessionManager.instance.signOut();
}

// ── Check status ──────────────────────────────────────────────────────────────
bool get isSignedIn => SessionManager.instance.isSignedIn;
int? get currentUserId => SessionManager.instance.signedInUser?.id;
```

---

## 6. WebSockets / Streaming (3.4.x)

```dart
// lib/src/endpoints/chat_endpoint.dart
import 'package:serverpod/serverpod.dart';
import '../generated/protocol.dart';

class ChatEndpoint extends Endpoint {
  @override
  bool get requireLogin => true;

  /// Send a message to a chat room
  Future<void> sendMessage(
    Session session,
    String roomId,
    String body,
  ) async {
    if (body.trim().isEmpty) throw ArgumentError('Message cannot be empty');

    final userId = await session.auth.authenticatedUserId;
    if (userId == null) throw ServerpodUnauthenticatedException();

    final msg = ChatMessage(
      roomId: roomId,
      senderId: userId,
      body: body.trim(),
      sentAt: DateTime.now().toUtc(),
    );
    final saved = await ChatMessage.db.insertRow(session, msg);

    // Broadcast to all subscribers of this room channel
    session.messages.postMessage(
      'chat_$roomId',
      saved,
      global: true, // true = shared across all server instances
    );
  }

  /// Subscribe to a chat room — returns a continuous stream
  Stream<ChatMessage> subscribeToRoom(
    Session session,
    String roomId,
  ) async* {
    // Yield recent history first
    final history = await ChatMessage.db.find(
      session,
      where: (t) => t.roomId.equals(roomId),
      orderBy: (t) => t.sentAt,
      orderDescending: true,
      limit: 50,
    );
    for (final msg in history.reversed) {
      yield msg;
    }

    // Then stream live messages
    final stream = session.messages.createStream<ChatMessage>('chat_$roomId');
    await for (final msg in stream) {
      yield msg;
    }
  }
}
```

### Flutter WebSocket Client

```dart
StreamSubscription<ChatMessage>? _subscription;

void subscribeToRoom(String roomId) {
  _subscription = client.chat
      .subscribeToRoom(roomId)
      .listen(
        (msg) => setState(() => _messages.add(msg)),
        onError: (Object e) {
          debugPrint('Stream error: $e');
          // Reconnect after delay
          Future.delayed(const Duration(seconds: 3), () => subscribeToRoom(roomId));
        },
        onDone: () => debugPrint('Stream closed'),
      );
}

Future<void> sendMessage(String roomId, String body) async {
  try {
    await client.chat.sendMessage(roomId, body);
  } on ServerpodClientException catch (e) {
    debugPrint('Failed to send: ${e.message}');
  }
}

@override
void dispose() {
  _subscription?.cancel();
  super.dispose();
}
```

---

## 7. Migrations (3.4.x)

```bash
# 1. Edit or create a .spy.yaml file
# 2. Generate code
dart run serverpod_cli generate

# 3. Create a migration (compares current schema to database)
dart run serverpod_cli create-migration

# 4. Apply migrations — development
dart run bin/main.dart --apply-migrations

# 5. Apply migrations — production
dart run bin/main.dart --mode production --apply-migrations

# 6. List all migrations
dart run serverpod_cli migrations list

# 7. Roll back last migration (development only)
dart run serverpod_cli migrations rollback
```

**Best practices:**
- Review generated SQL in `migrations/` before applying to production.
- Never delete migration files — they form the schema history.
- Run `--apply-migrations` in your CI/CD pipeline before starting the server.
- Keep `config/passwords.yaml` in `.gitignore`.

---

## 8. File Uploads (3.4.x)

```dart
// lib/src/endpoints/file_endpoint.dart
import 'dart:io';
import 'package:path/path.dart' as p;
import 'package:serverpod/serverpod.dart';

class FileEndpoint extends Endpoint {
  @override
  bool get requireLogin => true;

  static const _maxBytes = 5 * 1024 * 1024; // 5 MB
  static const _allowedExtensions = {'.jpg', '.jpeg', '.png', '.webp', '.pdf'};

  Future<String> uploadFile(
    Session session,
    ByteData fileData,
    String filename,
  ) async {
    final ext = p.extension(filename).toLowerCase();
    if (!_allowedExtensions.contains(ext)) {
      throw ArgumentError('Unsupported file type: $ext');
    }
    if (fileData.lengthInBytes > _maxBytes) {
      throw ArgumentError('File exceeds 5 MB limit');
    }

    final userId = await session.auth.authenticatedUserId;
    if (userId == null) throw ServerpodUnauthenticatedException();

    final safeFilename = '${DateTime.now().millisecondsSinceEpoch}_${p.basename(filename)}';
    final storagePath = 'uploads/$userId/$safeFilename';

    await session.storage.storeFile(
      storageId: 'public',
      path: storagePath,
      byteData: fileData,
      verified: true,
    );

    session.log('File uploaded: $storagePath');
    return storagePath;
  }

  Future<Uri?> getFileUrl(Session session, String storagePath) async {
    return await session.storage.retrieveFileUrl(
      storageId: 'public',
      path: storagePath,
    );
  }

  Future<void> deleteFile(Session session, String storagePath) async {
    final userId = await session.auth.authenticatedUserId;
    if (userId == null) throw ServerpodUnauthenticatedException();

    // Verify the file belongs to this user
    if (!storagePath.startsWith('uploads/$userId/')) {
      throw ForbiddenException('Cannot delete files owned by another user');
    }
    await session.storage.deleteFile(storageId: 'public', path: storagePath);
  }
}
```

---

## 9. Configuration

```yaml
# config/development.yaml
apiServer:
  port: 8080
  publicHost: localhost
  publicPort: 8080
  publicScheme: http

insightsServer:
  port: 8081
  publicHost: localhost
  publicPort: 8081
  publicScheme: http

webServer:
  port: 8082
  publicHost: localhost
  publicPort: 8082
  publicScheme: http

database:
  host: localhost
  port: 5432
  name: myproject_development
  user: myproject
  requireSsl: false

redis:
  enabled: true
  host: localhost
  port: 6379
```

```yaml
# config/passwords.yaml  ← NEVER commit this file
database:
  password: dev_password_here

serviceSecret: dev_service_secret_here
```

```
# .gitignore additions
config/passwords.yaml
config/production.yaml
*.env
```

---

## 10. Production Deployment

### Dockerfile

```dockerfile
# my_project_server/Dockerfile
FROM dart:stable AS build
WORKDIR /app
COPY pubspec.* ./
RUN dart pub get --no-precompile
COPY . .
RUN dart run serverpod_cli generate
RUN dart compile exe bin/main.dart -o bin/server

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY --from=build /app/bin/server ./server
COPY --from=build /app/config ./config
COPY --from=build /app/migrations ./migrations
EXPOSE 8080 8081
CMD ["./server", "--mode", "production", "--apply-migrations"]
```

### docker-compose.yml

```yaml
version: '3.9'
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myproject
      POSTGRES_USER: myproject
      POSTGRES_PASSWORD: ${DB_PASSWORD:?required}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myproject"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD:?required}

  server:
    build: ./my_project_server
    environment:
      SERVERPOD_DATABASE_PASSWORD: ${DB_PASSWORD}
      SERVERPOD_REDIS_PASSWORD: ${REDIS_PASSWORD}
      SERVERPOD_SERVICE_SECRET: ${SERVICE_SECRET:?required}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    ports:
      - "8080:8080"
      - "8081:8081"
    restart: unless-stopped

volumes:
  postgres_data:
```

### Production Checklist
- [ ] PostgreSQL 14+ with `pg_isready` health check
- [ ] Redis with password authentication
- [ ] Reverse proxy (Nginx / Caddy) with TLS termination
- [ ] All secrets via environment variables (not files)
- [ ] `--apply-migrations` in startup command
- [ ] Log aggregation service configured
- [ ] Database backups scheduled (pg_dump / managed backups)
- [ ] Monitoring / alerts (uptime checks, error rate)

---

## 11. Testing (3.4.x)

```dart
// test/post_endpoint_test.dart
import 'package:test/test.dart';
import 'package:serverpod_test/serverpod_test.dart';
import 'package:my_project_server/src/generated/protocol.dart';
import 'package:my_project_server/src/endpoints/post_endpoint.dart';

void main() {
  withServerpod('PostEndpoint', (sessionBuilder, endpoints) {
    group('getPublishedPosts', () {
      test('returns only published posts', () async {
        final session = sessionBuilder.build();
        final ep = PostEndpoint();

        // Seed test data
        await Post.db.insert(session, [
          Post(authorId: 1, title: 'Published', content: '...', isPublished: true),
          Post(authorId: 1, title: 'Draft', content: '...', isPublished: false),
        ]);

        final result = await ep.getPublishedPosts(session);
        expect(result, isNotEmpty);
        expect(result.every((p) => p.isPublished), isTrue);
      });
    });

    group('createPost', () {
      test('creates a post for authenticated user', () async {
        // Build session as authenticated user with id=1
        final session = sessionBuilder.build(authentication: AuthenticationOverride.authenticationInfo(1, {}));
        final ep = PostEndpoint();

        final post = await ep.createPost(session, 'Test Title', 'Test content');
        expect(post.id, isNotNull);
        expect(post.title, equals('Test Title'));
        expect(post.authorId, equals(1));
        expect(post.isPublished, isFalse);
      });

      test('throws on empty title', () async {
        final session = sessionBuilder.build(authentication: AuthenticationOverride.authenticationInfo(1, {}));
        final ep = PostEndpoint();

        expect(
          () => ep.createPost(session, '', 'Content'),
          throwsA(isA<ArgumentError>()),
        );
      });

      test('throws when unauthenticated', () async {
        final session = sessionBuilder.build(); // no auth
        final ep = PostEndpoint();

        expect(
          () => ep.createPost(session, 'Title', 'Content'),
          throwsA(isA<ServerpodUnauthenticatedException>()),
        );
      });
    });
  });
}
```

Run tests:
```bash
dart test
# With coverage
dart test --coverage=coverage
```

---

## 12. pubspec.yaml Reference

```yaml
# my_project_server/pubspec.yaml
name: my_project_server
version: 1.0.0
environment:
  sdk: '>=3.3.0 <4.0.0'

dependencies:
  serverpod: ^3.4.0
  serverpod_auth_server: ^3.4.0
  path: ^1.9.0

dev_dependencies:
  serverpod_cli: ^3.4.0
  serverpod_test: ^3.4.0
  test: ^1.25.0
  lints: ^4.0.0

# my_project_flutter/pubspec.yaml
name: my_project_flutter
version: 1.0.0
environment:
  sdk: '>=3.3.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  my_project_client:
    path: ../my_project_client
  serverpod_flutter: ^3.4.0
  serverpod_auth_client: ^3.4.0
  serverpod_auth_email_flutter: ^3.4.0
  serverpod_auth_google_flutter: ^3.4.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^4.0.0
```

> **Always keep all `serverpod_*` packages on the same minor version** to prevent serialization mismatches.

---

## 13. Common Errors & Solutions

| Error | Cause | Fix |
|-------|-------|-----|
| `Class 'X' not found in protocol` | Model not regenerated | Run `dart run serverpod_cli generate` |
| `Column 'x' does not exist` | Migration not applied | Restart with `--apply-migrations` |
| `ServerpodUnauthenticatedException` | Missing or expired token | Re-authenticate and refresh session |
| `Connection refused :5432` | PostgreSQL not running | `docker-compose up postgres` |
| Stream stops receiving messages | Wrong channel name | Ensure client and server use identical channel string |
| Dependency version conflict | Mismatched `serverpod_*` versions | Pin all to `^3.4.0` |
| `Bad state: Cannot write to closed sink` | WebSocket disconnected | Catch `StateError` and reconnect |
| Generated code out of sync | Forgot generate after model change | `dart run serverpod_cli generate` |
| `passwords.yaml not found` | Missing secrets file | Copy `passwords.yaml.template` and fill values |
| `SSL connection required` | Production DB needs SSL | Set `requireSsl: true` in production config |

---

## 14. Quick Reference

```bash
# Install CLI
dart pub global activate serverpod_cli

# Create project
serverpod create my_project

# Generate after model/endpoint changes
dart run serverpod_cli generate

# Create migration
dart run serverpod_cli create-migration

# Apply migrations (development)
dart run bin/main.dart --apply-migrations

# Apply migrations (production)
dart run bin/main.dart --mode production --apply-migrations

# Run tests
dart test

# Analyze code
dart analyze

# Format code
dart format .
```

---
> Source: [TOMOKI977/serverpod-flutter-expert](https://github.com/TOMOKI977/serverpod-flutter-expert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
