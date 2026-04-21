## labrandswiftui

> Guidelines for using Apple's on-device LLM framework for generative AI features.

# Foundation Models - On-Device LLM

Guidelines for using Apple's on-device LLM framework for generative AI features.

---

## 🎯 OVERVIEW

Foundation Models provides access to on-device LLMs powering Apple Intelligence. Features:
- Text generation and understanding
- Content summarization
- Structured data generation
- Custom tool integration
- **No cloud connectivity required**

---

## ✅ CHECK AVAILABILITY

Always check model availability first:

```swift
struct GenerativeView: View {
    private var model = SystemLanguageModel.default

    var body: some View {
        switch model.availability {
        case .available:
            Text("Model available")
        case .unavailable(.deviceNotEligible):
            Text("Device not eligible")
        case .unavailable(.appleIntelligenceNotEnabled):
            Text("Please enable Apple Intelligence")
        case .unavailable(.modelNotReady):
            Text("Model downloading...")
        case .unavailable(let other):
            Text("Unavailable: \(other)")
        }
    }
}
```

---

## 💬 BASIC USAGE

### Create a Session

```swift
// Basic session
let session = LanguageModelSession()

// With instructions
let session = LanguageModelSession(instructions: """
    You are a helpful assistant.
    Keep responses under 100 words.
    Focus on clarity.
    """)
```

### Generate Response

```swift
let response = try await session.respond(to: "What's a good month to visit Paris?")
print(response.content)

// With options
let options = GenerationOptions(temperature: 0.7)
let response = try await session.respond(to: prompt, options: options)
```

---

## 📋 INSTRUCTIONS

Good instructions specify:
- Model's role
- What to do
- Style preferences
- Safety measures

```swift
let instructions = """
    You are a cooking assistant.
    Provide recipe suggestions based on ingredients.
    Keep suggestions brief and practical.
    Include approximate cooking time.
    """

let session = LanguageModelSession(instructions: instructions)
```

---

## 🏗️ GUIDED GENERATION (Structured Output)

### 1. Define a Generable Type

```swift
@Generable(description: "Basic profile information about a cat")
struct CatProfile {
    var name: String

    @Guide(description: "The age of the cat", .range(0...20))
    var age: Int

    @Guide(description: "A one sentence profile")
    var profile: String
}
```

### 2. Request Structured Response

```swift
let response = try await session.respond(
    to: "Generate a cute rescue cat",
    generating: CatProfile.self
)

// Access typed properties - use .content NOT .output
print("Name: \(response.content.name)")
print("Age: \(response.content.age)")
```

### 3. Arrays with Count

```swift
@Generable
struct CookbookSuggestions {
    @Guide(description: "Cookbook Suggestions", .count(3))
    var suggestions: [String]
}
```

---

## 🔄 SNAPSHOT STREAMING

Stream partial results as they generate:

```swift
@Generable
struct TripIdeas {
    @Guide(description: "Ideas for upcoming trips")
    var ideas: [String]
}

let stream = session.streamResponse(
    to: "What are some trip ideas?",
    generating: TripIdeas.self
)

for try await partial in stream {
    // partial is TripIdeas.PartiallyGenerated
    // Properties are optional, fill in as generated
    print(partial)
}
```

### SwiftUI Integration

```swift
struct ContentView: View {
    @State private var partial: TripIdeas.PartiallyGenerated?
    
    var body: some View {
        VStack {
            ForEach(partial?.ideas ?? [], id: \.self) { idea in
                Text(idea)
            }
        }
        .task {
            let stream = session.streamResponse(to: prompt, generating: TripIdeas.self)
            for try await result in stream {
                partial = result
            }
        }
    }
}
```

---

## 🛠️ TOOL CALLING

### 1. Create a Tool

```swift
struct RecipeSearchTool: Tool {
    struct Arguments: Codable {
        var searchTerm: String
        var numberOfResults: Int
    }
    
    func call(arguments: Arguments) async throws -> ToolOutput {
        let recipes = await searchRecipes(term: arguments.searchTerm, 
                                         limit: arguments.numberOfResults)
        return .string(recipes.map { "- \($0.name)" }.joined(separator: "\n"))
    }
}
```

### 2. Use with Session

```swift
let session = LanguageModelSession(tools: [RecipeSearchTool()])
let response = try await session.respond(to: "Find pasta recipes")
```

### 3. Handle Errors

```swift
do {
    let answer = try await session.respond("Find a recipe for soup.")
} catch let error as LanguageModelSession.ToolCallError {
    print(error.tool.name)
    if case .databaseIsEmpty = error.underlyingError as? RecipeSearchToolError {
        // Handle specific error
    }
}
```

---

## ⚠️ CONTEXT LIMITS

- **4,096 tokens** per session (~3-4 characters per token)
- Instructions + prompts + outputs count toward limit
- Error: `LanguageModelSession.GenerationError.exceededContextWindowSize`
- **Solution:** Break large tasks into smaller chunks across sessions

---

## 🔧 GENERATION OPTIONS

```swift
let options = GenerationOptions(temperature: 2.0)  // Higher = more creative
let response = try await session.respond(to: prompt, options: options)

// Access transcript for debugging
let transcript = session.transcript
```

---

## 💡 PROMPT ENGINEERING TIPS

1. **Be specific** about what you want
2. **Specify output constraints** ("in three sentences")
3. **Break complex tasks** into multiple prompts
4. **Use examples** in instructions

---

## ✅ BEST PRACTICES

1. **Check availability** before using
2. **Use structured output** (`@Generable`) for reliable parsing
3. **Stream for better UX** - show progress
4. **Keep sessions focused** - create new for unrelated tasks
5. **Handle errors gracefully** - device/model may be unavailable
6. **Respect context limits** - break up large inputs

---

## 📚 REFERENCES

- [Generating content with Foundation Models](https://developer.apple.com/documentation/FoundationModels/generating-content-and-performing-tasks-with-foundation-models)
- [Generating Swift data structures](https://developer.apple.com/documentation/FoundationModels/generating-swift-data-structures-with-guided-generation)
- [Expanding with tool calling](https://developer.apple.com/documentation/FoundationModels/expanding-generation-with-tool-calling)
- [HIG: Generative AI](https://developer.apple.com/design/human-interface-guidelines/technologies/generative-ai)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LaBrand23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
