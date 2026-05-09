## swiftagent

> SwiftAgent is a Swift framework for building AI agents with tool use, multi-provider LLM support, RAG, multi-agent graphs, memory, and human-in-the-loop. Requires Swift 6.0+ with strict concurrency.

# SwiftAgent - Quick Reference for LLM Coding Assistants

SwiftAgent is a Swift framework for building AI agents with tool use, multi-provider LLM support, RAG, multi-agent graphs, memory, and human-in-the-loop. Requires Swift 6.0+ with strict concurrency.

## Package Import

```swift
import SwiftAgent
```

## Core Architecture

- All providers and stores are **actors** (thread-safe by default)
- All public types conform to **Sendable**
- Uses **async/await** throughout (no Combine)
- Platform: macOS 15+, iOS 18+, watchOS 11+, tvOS 18+, visionOS 2+ (Apple Intelligence requires 26.0+, MLX requires macOS 14+)

## Providers

### Claude (Anthropic)

```swift
let provider = ClaudeProvider(apiKey: "sk-...", model: .sonnet)
// Models: .opus (.claudeOpus46), .sonnet (.claudeSonnet45), .haiku (.claudeHaiku45)
```

### OpenAI

```swift
let provider = OpenAIProvider(apiKey: "sk-...", model: .gpt54)
// Models: .gpt54, .gpt54Pro, .gpt52Pro, .gpt52, .gpt5Mini, .gpt5Nano, .gpt5
```

### Gemini (Google)

```swift
let provider = GeminiProvider(apiKey: "...", model: .gemini31Pro, thinkingLevel: .high)
// Models: .gemini31Pro, .gemini31FlashLite, .gemini31FlashImage
// ThinkingLevel: .minimal, .low, .medium, .high, .auto
```

### Apple Intelligence (On-Device)

```swift
// Requires iOS 26.0+ / macOS 26.0+
let provider = try await AppleIntelligenceProvider(instructions: "You are a helpful assistant.", maxContextTokens: 2000)
```

### MLX (Local LLM on Apple Silicon)

```swift
let provider = try await MLXProvider(model: .llama3_2_3B)
// Models: .llama3_2_3B, .llama3_2_1B, .llama3_1_8B, .qwen3_8B, .qwen3_4B,
//   .qwen2_5_7B, .qwen2_5_1_5B, .gemma3_4B, .gemma2_9B, .gemma2_2B,
//   .phi4Mini, .mistral7B, .smolLM2_1_7B, .smolLM2_360M, .deepSeekR1_8B
// Or custom: try await MLXProvider(modelId: "mlx-community/your-model")
// With progress: try await MLXProvider(model: .llama3_2_3B, progressHandler: { p in print(p.fractionCompleted) })
```

## Agent (Core)

```swift
let agent = Agent(
    name: "MyAgent",
    provider: provider,
    systemPrompt: "You are a helpful assistant.",
    tools: [WebSearchTool(), CalculatorTool()],
    maxIterations: 10,
    options: GenerationOptions(maxTokens: 4096, temperature: 0.7)
)

// Synchronous (returns final result)
let result = try await agent.run(task: "Research Swift concurrency")
print(result.output)        // String
print(result.totalTokens)   // Int
print(result.success)       // Bool

// Streaming (real-time events)
let stream = await agent.stream(task: "Explain quantum computing")
for try await event in stream {
    switch event {
    case .thinking(let text): print(text, terminator: "")
    case .toolCall(let call): print("Using: \(call.name)")
    case .toolResult(let name, let result): print("\(name): \(result)")
    case .response(let text): print(text, terminator: "")
    case .completed(let result): print("Done: \(result.output)")
    case .error(let error): print("Error: \(error)")
    }
}

// Single turn (no tool loop)
let response = try await agent.invoke(input: "Hello")

// With images
let imageData = try Data(contentsOf: imageURL)
let image = Message.ImageContent(data: imageData, mimeType: "image/jpeg")
let result = try await agent.run(task: "Describe this image", images: [image])
```

### Prompt from File

```swift
let agent = try Agent(
    name: "Agent",
    provider: provider,
    promptFile: "/path/to/prompt.md",
    promptVariables: ["name": "Alice", "role": "assistant"],
    tools: []
)
```

## Tools

### Built-in Tools

```swift
WebSearchTool()           // DuckDuckGo search
CalculatorTool()          // Math expressions via NSExpression
DateTimeTool()            // Date/time operations
FileSystemTool()          // File read/write/list/delete
HTTPRequestTool()         // HTTP requests (GET/POST/PUT/DELETE/PATCH)
JSONParserTool()          // JSON parse/extract/validate/pretty_print
CalendarTool()            // Local EventKit calendar
GoogleCalendarTool(accessToken: "...")  // Google Calendar API
ImageAnalysisTool()       // Vision (delegates to LLM)
OCRTool()                 // OCR (delegates to LLM)
```

### Custom Tool

```swift
struct MyTool: Tool {
    let name = "my_tool"
    let description = "Does something useful"
    let parameters = ToolParameters(
        properties: [
            "query": ParameterProperty(type: "string", description: "Search query"),
            "limit": ParameterProperty(type: "integer", description: "Max results")
        ],
        required: ["query"]
    )

    func execute(arguments: [String: Any]) async throws -> String {
        guard let query = arguments["query"] as? String else {
            throw ToolError.invalidArguments("Missing query")
        }
        let limit = arguments["limit"] as? Int ?? 10
        return "Results for '\(query)' (limit: \(limit))"
    }
}
```

## RAG (Retrieval-Augmented Generation)

### Embeddings

```swift
let openAIEmbeddings = OpenAIEmbeddingProvider(apiKey: "sk-...")
// Models: .textEmbedding3Small (default), .textEmbedding3Large, .textEmbeddingAda002

let geminiEmbeddings = GeminiEmbeddingProvider(apiKey: "...")
// Models: .embedding001 (text, default), .embedding2 (multimodal)
// Supports taskType: .retrievalQuery, .retrievalDocument, .semanticSimilarity, etc.
```

### Vector Stores

```swift
// In-memory (SIMD-optimized)
let store = InMemoryVectorStore(embeddingProvider: openAIEmbeddings)

// Pinecone (cloud)
let store = PineconeVectorStore(apiKey: "...", host: "index-xxx.pinecone.io", embeddingProvider: openAIEmbeddings)
```

### Document Chunking & Search

```swift
// Chunk text into documents
let docs = DocumentChunker.createDocuments(from: longText, chunkSize: 500, overlap: 50, sourceMetadata: ["source": "wiki"])
try await store.add(documents: docs)

// Search
let results = try await store.search(query: "quantum computing", topK: 5)

// Use as agent tool
let agent = Agent(
    provider: provider,
    systemPrompt: "Answer from the knowledge base.",
    tools: [VectorSearchTool(vectorStore: store)]
)
```

## Multi-Agent Graphs

```swift
let graph = AgentGraph(maxIterations: 50)

// Add nodes (agents or functions)
await graph.addNode("researcher", agent: researchAgent)
await graph.addNode("writer", agent: writerAgent)
await graph.addNode("transform") { state in
    var s = state
    s.setValue(.string("processed"), forKey: "status")
    return s
}

// Define edges
await graph.addEdge(from: .START, to: "researcher")
await graph.addEdge(from: "researcher", to: "transform")
await graph.addEdge(from: "transform", to: "writer")
await graph.addEdge(from: "writer", to: .END)

// Conditional routing
await graph.addConditionalEdge(from: "classifier") { state in
    let category = state.getValue(forKey: "category")?.stringValue ?? "default"
    return category  // Returns next node name
}

// Execute
var state = GraphState()
state.addMessage(.user("Research topic"))
let result = try await graph.invoke(input: state)
print(result.messages.last?.textContent ?? "")

// Streaming
let events = await graph.stream(input: state)
for try await event in events {
    switch event {
    case .nodeStarted(let name): print("Starting: \(name)")
    case .nodeCompleted(let name, _): print("Completed: \(name)")
    case .edgeTraversed(let from, let to): print("\(from) -> \(to)")
    case .graphCompleted(let state): print("Done")
    case .error(let error): print("Error: \(error)")
    }
}
```

## Memory System

```swift
let memoryStore = InMemoryMemoryStore()
// Or persistent: let memoryStore = try SwiftDataMemoryStore(configuration: .local)
// Or iCloud: let memoryStore = try SwiftDataMemoryStore(configuration: .iCloud)

let memoryManager = MemoryManager(
    store: memoryStore,
    embeddingProvider: openAIEmbeddings,  // optional, for semantic search
    llmProvider: provider
)

let config = MemoryConfig(manager: memoryManager, userId: "user123", autoSave: true)
let result = try await agent.runWithMemory(task: "Remember I prefer dark mode", config: config, threadId: "thread1")

// Recall memories
let memories = try await memoryManager.recallMemories(userId: "user123", query: "preferences")
// memories.profile - UserProfile
// memories.episodes - [Episode]
// memories.semanticMemories - [SemanticMemory]
```

## Human-in-the-Loop

```swift
let config = InterruptConfig(
    checkpointStore: InMemoryCheckpointStore(),
    interruptBefore: ["dangerous_tool"],  // Tool names to pause before
    interruptAfter: ["important_tool"]    // Tool names to pause after
)

let result = try await agent.invoke(task: "Delete all files", threadId: "t1", config: config)

if let interrupt = result.pendingInterrupt {
    print("Approval needed: \(interrupt.message)")
    // User approves
    let response = InterruptResponse(requestId: interrupt.id, action: .approve)
    let resumed = try await agent.updateState(response, threadId: "t1", config: config)
}
```

## Key Types Reference

| Type | Description |
|------|-------------|
| `LLMProvider` | Protocol: `generate(messages:tools:options:)`, `stream(messages:tools:options:)` |
| `GenerationOptions` | `maxTokens`, `temperature`, `topP`, `stopSequences`, `thinkingLevel` |
| `LLMResponse` | `content`, `toolCalls`, `stopReason`, `usage` |
| `LLMChunk` | `.content(String)`, `.toolCall(ToolCall)`, `.done(StopReason)` |
| `Message` | `.system(String)`, `.user(String)`, `.assistant(String)`, `.tool(String, toolCallId:)` |
| `ToolCall` | `id`, `name`, `arguments: [String: AnyCodable]` |
| `AnyCodable` | `.bool`, `.int`, `.double`, `.string`, `.array`, `.dictionary`, `.null` |
| `Tool` | Protocol: `name`, `description`, `parameters`, `execute(arguments:)` |
| `AgentResult` | `output`, `state`, `totalTokens`, `success`, `error` |
| `AgentEvent` | `.thinking`, `.toolCall`, `.toolResult`, `.response`, `.completed`, `.error` |
| `GraphState` | `messages`, `data`, `currentNode`, `visitedNodes` |
| `GraphEvent` | `.nodeStarted`, `.nodeCompleted`, `.edgeTraversed`, `.graphCompleted`, `.error` |
| `VectorStore` | Protocol: `search(query:topK:)`, `add(documents:)`, `delete(ids:)`, `clear()`, `count()` |
| `EmbeddingProvider` | Protocol: `embed(text:)`, `embedBatch(texts:)` |
| `Document` | `id`, `content`, `metadata` |
| `SearchResult` | `id`, `content`, `score`, `metadata` |

---
> Source: [juraskrlec/SwiftAgent](https://github.com/juraskrlec/SwiftAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
