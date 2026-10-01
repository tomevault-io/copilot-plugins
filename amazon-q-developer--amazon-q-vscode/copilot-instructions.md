## amazon-q-vscode

> Every source file must include the Apache 2.0 license header:

# Amazon Q for VS Code - Development Guidelines

## Code Quality Standards

### File Header

Every source file must include the Apache 2.0 license header:

```typescript
/*!
 * Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
 * SPDX-License-Identifier: Apache-2.0
 */
```

### Formatting (Prettier Configuration)

-   Print width: 120 characters
-   Tab width: 4 spaces
-   Single quotes for strings
-   No semicolons
-   Trailing commas: ES5 style
-   Arrow function parentheses: always
-   Line endings: LF

### Naming Conventions

-   Classes: PascalCase (e.g., `ChatController`, `RemoteInvokeWebview`)
-   Interfaces: PascalCase with descriptive names (e.g., `ChatControllerMessagePublishers`)
-   Constants: SCREAMING_SNAKE_CASE for exported constants (e.g., `SERVERLESS_FUNCTION_TYPE`)
-   Functions/methods: camelCase (e.g., `processPromptChatMessage`)
-   Private members: camelCase, no underscore prefix
-   Type aliases: PascalCase (e.g., `ResourceType`, `ParameterType`)

## Architectural Patterns

### Controller Pattern

Controllers orchestrate feature logic and handle message passing:

```typescript
export class ChatController {
    private readonly sessionStorage: ChatSessionStorage
    private readonly messenger: Messenger

    public constructor(
        private readonly messageListeners: ChatControllerMessageListeners,
        publisher: MessagePublisher<any>,
        onDidChangeVisibility: VSCodeEvent<boolean>
    ) {
        // Initialize dependencies
        // Register message listeners
        this.messageListeners.processPromptChatMessage.onMessage((data) => {
            return this.processPromptChatMessage(data)
        })
    }
}
```

### Message Publisher/Listener Pattern

Decouple components using typed message channels:

```typescript
export interface ChatControllerMessagePublishers {
    readonly processPromptChatMessage: MessagePublisher<PromptMessage>
    readonly processTabCreatedMessage: MessagePublisher<TabCreatedMessage>
}
```

### VueWebview Pattern

Webviews extend VueWebview base class:

```typescript
export class RemoteInvokeWebview extends VueWebview {
    public static readonly sourcePath: string = 'src/lambda/vue/remoteInvoke/index.js'
    public readonly id = 'remoteInvoke'

    public constructor(
        private readonly channel: vscode.OutputChannel,
        private readonly client: LambdaClient,
        private readonly data: InitialData
    ) {
        super(RemoteInvokeWebview.sourcePath)
    }
}
```

### Error Handling

Use ToolkitError for chainable, typed errors:

```typescript
throw ToolkitError.chain(
    error,
    localize('AWS.lambda.remoteInvoke.failedToDownloadCode', 'Failed to download remote code')
)
```

Exception classes extend base service exception:

```typescript
class AccessDeniedException extends CodeWhispererStreamingServiceException {
    name = 'AccessDeniedException'
    $fault = 'client'
    reason?: string

    constructor(opts) {
        super({ name: 'AccessDeniedException', $fault: 'client', ...opts })
        Object.setPrototypeOf(this, AccessDeniedException.prototype)
        this.reason = opts.reason
    }
}
```

## Testing Patterns

### Test Structure

Use Mocha with describe/it blocks:

```typescript
describe('SmusAuthenticationProvider', function () {
    let mockAuth: any
    let smusAuthProvider: SmusAuthenticationProvider

    beforeEach(function () {
        // Setup mocks and stubs
        mockAuth = { createConnection: sinon.stub().resolves(mockConnection) }
        smusAuthProvider = new SmusAuthenticationProvider(mockAuth)
    })

    afterEach(function () {
        sinon.restore()
    })

    describe('activeConnection', function () {
        it('should return secondary auth active connection', function () {
            assert.strictEqual(smusAuthProvider.activeConnection, mockConnection)
        })
    })
})
```

### Mocking with Sinon

-   Use `sinon.stub()` for method stubs
-   Use `sinon.spy()` for call tracking
-   Always call `sinon.restore()` in afterEach
-   Stub external dependencies at module level when needed

### Assertions

Use Node.js assert module:

```typescript
assert.strictEqual(result, expected)
assert.ok(condition)
assert.rejects(
    () => asyncFn(),
    (err: ToolkitError) => err.code === 'ExpectedCode'
)
```

## Common Code Idioms

### Async/Await with Telemetry

Wrap operations in telemetry spans:

```typescript
await telemetry.lambda_invokeRemote.run(async (span) => {
    try {
        const result = await this.client.invoke(arn, input)
        span.record({ passive: false, source: source })
    } catch (e) {
        // Handle error
    }
})
```

### VS Code API Usage

```typescript
// File system operations
const exists = await fs.exists(filePath)
const content = await fs.readFile(filePath)

// User prompts
const result = await vscode.window.showQuickPick(items, { placeHolder: 'Select...' })
const input = await vscode.window.showInputBox({ prompt: 'Enter value' })

// Commands
await vscode.commands.executeCommand('workbench.action.focusFirstEditorGroup')
```

### Localization

Use vscode-nls for user-facing strings:

```typescript
import * as nls from 'vscode-nls'
const localize = nls.loadMessageBundle()

const message = localize('AWS.lambda.remoteInvoke.open', 'Open')
```

### Sensitive Data Filtering

Filter sensitive data in logs using filter functions:

```typescript
const FilterSensitiveLog = (obj) => ({
    ...obj,
    ...(obj.content && { content: SENSITIVE_STRING }),
    ...(obj.url && { url: SENSITIVE_STRING }),
})
```

## Import Conventions

-   Group imports: Node.js built-ins, external packages, internal modules
-   Use path aliases from tsconfig when available
-   Prefer named exports over default exports
-   Use `import type` for type-only imports when possible

## Documentation

-   Use JSDoc for public APIs
-   Document complex algorithms inline
-   Keep comments minimal - prefer self-documenting code
-   Update README files when adding new features

---
> Source: [Amazon-Q-Developer/amazon-q-vscode](https://github.com/Amazon-Q-Developer/amazon-q-vscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
