## moderationapi

> Auto-generated C# SDK for [ModerationAPI](https://moderationapi.com/) -- multi-modal content moderation for text, images, video, and audio in 200+ languages.

# CLAUDE.md -- ModerationAPI SDK

## Overview

Auto-generated C# SDK for [ModerationAPI](https://moderationapi.com/) -- multi-modal content moderation for text, images, video, and audio in 200+ languages.
OpenAPI spec from `https://docs.moderationapi.com/openapi.json` (3.1.0).

## Build & Test

```bash
dotnet build ModerationAPI.slnx
dotnet test src/tests/IntegrationTests/
```

## Auth

API key via Bearer token. Constructor accepts API key directly:

```csharp
var client = new ModerationAPIClient(apiKey); // MODERATIONAPI_API_KEY env var
```

## Key Files

- `src/libs/ModerationAPI/openapi.yaml` -- Downloaded OpenAPI spec
- `src/libs/ModerationAPI/generate.sh` -- Downloads spec and runs autosdk generate
- `src/libs/ModerationAPI/Generated/` -- **Never edit** -- auto-generated code
- `src/libs/ModerationAPI/Extensions/ModerationAPIClient.Tools.cs` -- MEAI `AIFunction` tools
- `src/tests/IntegrationTests/Examples/` -- Example tests (also generate docs)

## Sub-client Pattern

```csharp
var client = new ModerationAPIClient(apiKey);

// Moderate content (text, image, video, audio, object)
client.NewModerateModerateAsync(request)

// Account
client.Account.AccountGetAsync()

// Actions
client.Actions.ActionsListAsync()
client.Actions.ActionsCreateAsync(request)
client.Actions.ActionsGetAsync(id)
client.Actions.ActionsExecuteAsync(id, request)

// Authors
client.Author.AuthorOpenListAuthorsAsync()
client.Author.AuthorOpenCreateAsync(request)
client.Author.AuthorOpenGetAuthorDetailsAsync(id)

// Review Queues
client.ReviewQueues.QueueViewOpenGetQueueAsync(id)
client.ReviewQueues.QueueViewOpenGetStatsAsync(id)
client.ReviewQueues.QueueViewOpenGetItemsAsync(id)
client.ReviewQueues.QueueViewOpenResolveItemAsync(id, itemId, request)
client.ReviewQueues.QueueViewOpenUnresolveItemAsync(id, itemId, request)

// Wordlists
client.Wordlist.WordlistListAsync()
client.Wordlist.WordlistGetAsync(id)
client.Wordlist.WordlistAddWordsAsync(id, request)
client.Wordlist.WordlistRemoveWordsAsync(id, request)
```

## MEAI Integration

AIFunction tools for use with any `IChatClient`:
- `AsModerateTextTool()` -- Moderate text for toxicity, profanity, PII, spam, violence, hate, etc.
- `AsModerateImageTool()` -- Moderate image by URL for NSFW, violence, hate symbols, etc.
- `AsGetQueueStatsTool()` -- Get review queue statistics
- `AsListActionsTool()` -- List available moderation actions

## Content Type Construction

The `Content` property uses nested `OneOf` types. Construct content like this:

```csharp
// Text content
OneOf<NewModerateModerateRequestContentContentLeafText, NewModerateModerateRequestContentContentLeafImage, NewModerateModerateRequestContentContentLeafVideo, NewModerateModerateRequestContentContentLeafAudio>?
    textContent = new NewModerateModerateRequestContentContentLeafText("text to moderate");

var request = new NewModerateModerateRequest { Content = textContent };

// Image content
OneOf<...>? imageContent = new NewModerateModerateRequestContentContentLeafImage("https://example.com/image.jpg");
```

## Spec Notes

- Base URL: `https://api.moderationapi.com/v1`
- Auth: Bearer token via `--security-scheme Http:Header:Bearer` in generate.sh
- Single moderation endpoint (`POST /moderate`) handles all content types via discriminated union
- Response includes evaluation (flagged, severity_score), recommendation (allow/review/reject), and policy results

---
> Source: [tryAGI/ModerationAPI](https://github.com/tryAGI/ModerationAPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
