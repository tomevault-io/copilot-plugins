## eino-ext

> import "github.com/cloudwego/eino-ext/components/model/gemini"

# Gemini ChatModel

```
import "github.com/cloudwego/eino-ext/components/model/gemini"
```

## Configuration

```go
client, _ := genai.NewClient(ctx, &genai.ClientConfig{APIKey: "your-key"})

chatModel, err := gemini.NewChatModel(ctx, &gemini.Config{
    Client: client,             // Required: *genai.Client
    Model:  "gemini-2.5-flash", // Required
    ThinkingConfig: &genai.ThinkingConfig{
        IncludeThoughts: true,
    },
})
```

---
> Source: [cloudwego/eino-ext](https://github.com/cloudwego/eino-ext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
