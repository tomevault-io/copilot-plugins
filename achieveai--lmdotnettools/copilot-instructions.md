## realrequestsimulationtesting

> We've created DatabasedClientWrapper.cs and AnthropicClientWrapper.cs which takes first request and send it to

# Request Response Simulation

We've created DatabasedClientWrapper.cs and AnthropicClientWrapper.cs which takes first request and send it to
LLM APIs, and then caches both request and response.

As much as possible we should be using this wrapper for testing. This keeps us
as close to real request/response in a managable way.

To allow this request/response from LLM APIs, we use env.test, which normally
would contain API_KEY and API_URL to conenct to. Which we would be different for
different providers. E.g. OpenAIProvider may have a different value then
AnthropicProvider.

---
> Source: [achieveai/LmDotnetTools](https://github.com/achieveai/LmDotnetTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
