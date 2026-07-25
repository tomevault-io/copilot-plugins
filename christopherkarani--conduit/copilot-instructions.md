## conduit

> Use Google’s native Gemini API through a Conduit provider.

# GeminiProvider

Use Google’s native Gemini API through a Conduit provider.

**Requires:** `Gemini` trait (`#if CONDUIT_TRAIT_GEMINI`)

## Overview

`GeminiProvider` uses Gemini’s native `generateContent` request shape: `contents`, `systemInstruction`, `generationConfig`, tools, function calls, inline image data, and JSON structured output.

```swift
let provider = GeminiProvider(apiKey: geminiAPIKey)

let result = try await provider.generate(
    messages: [.system("Be concise."), .user("Explain Swift actors.")],
    model: .gemini("gemini-3-flash-preview"),
    config: .default.maxTokens(300)
)
```

## Structured output

```swift
@Generable
struct Summary {
    let title: String
    let bullets: [String]
}

let result = try await provider.generate(
    messages: [.user("Summarize this document.")],
    model: .gemini("gemini-3-flash-preview"),
    config: .default.responseFormat(.jsonSchema(name: "summary", schema: Summary.generationSchema))
)
```

Conduit maps the schema into Gemini `generationConfig.responseSchema` and requests `application/json`.

## Typed Gemini options

```swift
var config = GenerateConfig.default
config[custom: GeminiProvider.self] = GeminiOptions(
    thinkingConfig: ["thinkingLevel": "low"],
    toolConfig: ["includeServerSideToolInvocations": true],
    serverTools: [["googleSearch": [:]]]
)
```

Gemini-specific thinking, tool, and server-tool fields stay provider-owned instead of expanding global `GenerateConfig`.

## Multimodal input

```swift
let image = Message.ImageContent(base64Data: encodedPNG, mimeType: "image/png")
let message = Message(role: .user, content: .parts([
    .text("Describe this image."),
    .image(image)
]))
```

`GeminiProvider` serializes images as Gemini `inlineData`.

---
> Source: [christopherkarani/Conduit](https://github.com/christopherkarani/Conduit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
