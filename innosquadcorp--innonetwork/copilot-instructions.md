## innonetwork

> - Developer is Korean.

# Repository Guidelines

## Developer Guidelines

### Language Policy
- Developer is Korean.
- Respond in Korean unless explicitly requested otherwise.
- PR description always in Korean.

### Root Cause First Approach
- Value fundamental problem solving.
- When addressing an issue, identify root cause before proposing changes.
- Consider network-specific issues: connectivity, timeouts, SSL, response parsing.

### Code Quality Standards
- **Type Safety**: Never use `as any`, `@ts-ignore`, or similar type suppression
- **Error Handling**: No empty catch blocks
- **Concurrency**: Use actors for shared state, make types Sendable
- **Testing**: Write tests using Swift Testing framework

### Git and Version Control
- Branch name in English
- Commit message in English
- DO NOT git add unstaged changes unless specified
- Keep commits focused and logically grouped

## Project Structure

```
InnoNetwork/
├── Sources/
│   ├── InnoNetwork/              # Core (REST API)
│   │   ├── API.swift
│   │   ├── APIDefinition.swift
│   │   ├── DefaultNetworkClient.swift
│   │   ├── Model/
│   │   ├── Interceptors/
│   │   ├── NetworkError.swift
│   │   └── NetworkLogger.swift
│   │
│   └── InnoNetworkDownload/      # Download 모듈
│       ├── DownloadManager.swift
│       ├── DownloadTask.swift    # actor
│       ├── DownloadConfiguration.swift
│       ├── DownloadState.swift
│       └── DownloadSessionDelegate.swift
│
└── Tests/
    ├── InnoNetworkTests/
    └── InnoNetworkDownloadTests/
```

## Build, Test, and Development Commands

```bash
# Build all targets
swift build

# Run all tests
swift test

# Test with coverage
swift test --enable-code-coverage --parallel

# Run specific test
swift test --filter InnoNetworkTests
```

## Coding Style

- Swift 6.2 API Design Guidelines
- 4-space indentation
- Public APIs documented with `///`
- Actor-based thread safety for shared mutable state
- Protocol-based design for flexibility

## Testing Guidelines

- Framework: Swift Testing (`@Suite`, `@Test`)
- Network tests use mock URLSession
- Download tests verify state machines
- All tests are async-compatible
- Use code coverage before release PRs

## Commit & Pull Request Guidelines

- Commit messages: present-tense, English
- PR description in Korean
- Link related issues
- Update documentation if needed (README, CHANGELOG)
- Run `swift test` before submitting PR

## Key Design Decisions

### Why Actors?
- DownloadTask and internal storage use actors for thread-safe state management
- Avoids NSLock + @unchecked Sendable pattern
- Native Swift Concurrency integration

### Why Separate Download Module?
- Optional dependency via SPM product
- Users who don't need downloads don't import the module
- Clean separation of concerns

### AsyncSequence Events
- DownloadManager.events(for:) provides AsyncStream<DownloadEvent>
- Enables modern Swift concurrency patterns
- Alternative to closure-based callbacks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/InnoSquadCorp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/InnoSquadCorp)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
