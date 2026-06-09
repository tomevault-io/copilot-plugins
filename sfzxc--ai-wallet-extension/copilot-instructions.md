## ai-wallet-extension

> Build a Chrome MV3 browser extension that works like a wallet for AI API access.

# AI Wallet Extension Project

## Project Goal

Build a Chrome MV3 browser extension that works like a wallet for AI API access.

Unlike crypto wallets such as MetaMask or OKX Wallet, this extension does not hold private keys or sign transactions. It holds user-owned AI API keys and exposes approved AI model access to websites without revealing the real API key.

Websites request permission through a connect modal and protocol. Users approve or reject access, choose allowed models, then websites can run AI conversations through an approved session.

## MVP Scope

The MVP has three parts:

1. `ConnectModal` library
   - Website-facing library.
   - Shows connect UI.
   - Calls the injected provider protocol.
   - Requests permission to access AI models.
   - Provides helper APIs for streaming responses.

2. Demo website
   - Simple conversation UI.
   - Embeds `ConnectModal`.
   - Lets user connect to the AI wallet extension.
   - Sends a sample prompt such as `1+2=?`.
   - Renders streamed response, expected result: `3`.

3. Chrome MV3 extension
   - Stores user OpenAI API key.
   - Has hardcoded supported model list for first verification.
   - Injects provider into websites, similar to `window.ethereum`.
   - Shows approve/reject UI when website requests access.
   - Lets user choose which models the website can use.
   - Brokers requests from website to OpenAI Responses API.
   - Streams model output back to website.

## Initial Provider Target

MVP targets OpenAI Responses API only.

Endpoint:

```text
POST https://api.openai.com/v1/responses
```

Streaming request shape:

```json
{
  "model": "gpt-4.1-mini",
  "input": "1+2=?",
  "stream": true
}
```

Important stream events:

```text
response.output_text.delta
response.completed
response.error
```

## Hardcoded MVP Models

Start with a hardcoded model list to verify end-to-end behavior quickly.

Example list:

```text
gpt-4.1-mini
gpt-4.1
o4-mini
```

Custom model input is out of scope for first MVP.

## Protocol Concept

The extension injects an AI wallet provider into the website page.

Expected global:

```ts
window.aiWallet
```

Initial methods:

```ts
ai_requestAccounts
ai_getPermissions
ai_requestResponseStream
ai_disconnect
```

Example connect request:

```ts
await window.aiWallet.request({
  method: "ai_requestAccounts",
  params: {
    models: ["gpt-4.1-mini"],
    reason: "Demo conversation needs AI response access"
  }
})
```

Example approved result:

```ts
{
  "sessionId": "session_123",
  "origin": "https://demo.localhost",
  "models": ["gpt-4.1-mini"],
  "expiresAt": "2026-06-07T12:00:00.000Z"
}
```

Example stream request:

```ts
await window.aiWallet.request({
  method: "ai_requestResponseStream",
  params: {
    sessionId: "session_123",
    model: "gpt-4.1-mini",
    input: "1+2=?"
  }
})
```

Actual stream delivery should use a long-lived connection, not a single request response. For MV3, use `chrome.runtime.Port` between content script and background service worker so streaming can stay alive long enough to forward deltas.

## Permission Model

Permissions are scoped by:

- Website origin.
- Session ID.
- Allowed model list.
- Expiration time.

Website must not directly know or receive the OpenAI API key.

Extension must validate every AI request against the active permission before calling OpenAI.

Approval UI must show:

- Requesting origin.
- Requested models.
- Requested reason.
- Approve button.
- Reject button.
- Model selection controls.

## Data Flow

Connect flow:

```text
Demo website -> ConnectModal -> window.aiWallet -> content script -> background -> extension approval UI -> background -> content script -> website
```

Streaming flow:

```text
Demo website -> ConnectModal -> window.aiWallet -> content script port -> background -> OpenAI /v1/responses stream -> background -> content script port -> website
```

## Security Requirements

- API key is stored only inside extension storage.
- Website never receives API key.
- Website cannot choose unapproved models.
- Website cannot reuse permission from another origin.
- Website cannot call AI API after session expires.
- Extension must reject calls when API key is missing.
- Extension must reject calls when model is not in approved list.
- Extension must reject calls when origin does not match session origin.

## MV3 Constraints

Chrome MV3 background runs as a service worker, not a persistent background page.

Implications:

- Long streams need an active `runtime.Port` or equivalent keep-alive strategy.
- State needed for validation should be persisted or reconstructable.
- Approval flow must tolerate service worker restart.
- No remote executable code should be loaded.

## Non-Goals For First MVP

- Firefox support.
- Multi-provider adapters.
- Anthropic/Gemini support.
- Custom model entry.
- Usage metering or billing.
- Prompt privacy guarantees beyond extension/browser boundary.
- Production-grade key encryption.
- Extension store publishing.
- Full protocol standardization.

## Feasibility Notes

This project is feasible as a Chrome MV3 prototype.

Key feasible parts:

- Extension can store API key locally.
- Extension can inject a provider into website pages.
- Website can request permissions through content script/background messaging.
- Background can call OpenAI API without exposing key to website.
- Streaming can be forwarded through a long-lived port.

Main risks:

- MV3 service worker lifecycle can interrupt long streams if keep-alive handling is weak.
- Extension approval UI may need careful UX because it is not identical to MetaMask transaction signing.
- Website trust boundary depends on origin validation and session enforcement.
- API key storage security depends on browser profile security.
- OpenAI stream event formats may evolve, so parsing should be defensive.

## Recommended First Verification

Build smallest end-to-end path:

1. Extension stores one OpenAI API key.
2. Extension injects `window.aiWallet`.
3. Demo website requests access to `gpt-4.1-mini`.
4. User approves.
5. Demo website sends `1+2=?`.
6. Extension calls OpenAI Responses API with streaming enabled.
7. Demo website renders streamed answer.

---
> Source: [SFzxc/ai-wallet-extension](https://github.com/SFzxc/ai-wallet-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
