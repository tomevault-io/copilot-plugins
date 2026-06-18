## telemetry-skill

> Integrate Latitude telemetry into any backend project for LLM observability. Use when the user wants to add Latitude tracing, logging, or observability to their LLM calls. Covers three approaches based on language and needs -- SDK telemetry wrapper (Python, TypeScript), raw OpenTelemetry OTLP (any language with OTel), or the Create Log API (any language with HTTP). Triggers on tasks involving Latitude telemetry, tracing LLM calls, sending traces to Latitude, or adding LLM observability.


# Latitude Telemetry Integration

## Prerequisite

A Latitude account and project are required for all telemetry approaches. Confirm the user has both before proceeding (Y/N). Without them, none of the methods work. Sign up at https://app.latitude.so.

## Environment Variables

Prompt the user interactively for each value:

- `LATITUDE_API_KEY` -- Ask the user for this (paste API key). It can be found in the **Settings** tab of their Latitude account.
- `LATITUDE_PROJECT_ID` -- Ask the user for this (number from URL). It is the number in the URL when inside a project, e.g. in `https://app.latitude.so/projects/30403/versions/.../documents/...` the project ID is `30403`.
- `LATITUDE_PROMPT_PATH` -- Ask the user: if they have a prompt saved in Latitude, use its path (e.g. `wikipedia-article-generator`). If not, use any string as a tag to group traces from that part of the code together in the Latitude project (e.g. `nps-categorizer`).
- `LATITUDE_PROMPT_VERSION_UUID` -- Defaults to `"live"`. Only prompt if the user needs a specific version (UUID or "live").

The LLM provider API key (OpenAI, OpenRouter, Anthropic, etc.) depends on the user's project and is not prescribed here.

## Choosing an Approach

Ask the user these questions in order:

### 1. Do you want to use Latitude as the gateway for LLM inference? (Y/N)

If yes, Latitude handles the LLM call -- the app sends the prompt path and parameters to Latitude, which calls the provider and streams back the response. Telemetry is handled automatically. Works in **any language**: use the SDK's `prompts.run()` (Python/TypeScript) or the HTTP API "Run a Prompt" endpoint for all other languages. See [references/gateway.md](references/gateway.md). No further telemetry setup is needed.

If no, continue.

### 2. Are you using Latitude as a prompt manager? (Y/N)

If yes, they need the Latitude SDK (Python/TypeScript only) to fetch and render prompts from Latitude before calling the LLM directly. This applies alongside whichever telemetry approach they pick below. The sdk-telemetry reference includes instructions for this. If no, they construct messages directly in code.

### 3. How should telemetry be sent to Latitude?

Pick based on the user's language and observability needs:

- **Language has a Latitude SDK? (Y/N)** (currently Python and TypeScript)
  - Use **SDK Telemetry**. See [references/sdk-telemetry.md](references/sdk-telemetry.md).

- **No SDK available -- does the user need token/cost tracking? (Y/N)**
  - **Yes, need token and cost tracking** -- Use **Raw OpenTelemetry (OTLP)**. This is significantly more complex: requires OpenTelemetry dependencies, manual two-span hierarchy, and ~20 span attributes. But it gives Latitude full observability including token counts, cost data, and latency breakdown. See [references/raw-opentelemetry.md](references/raw-opentelemetry.md).
  - **No, want the simplest integration** -- Use **Create Log API**. Just one HTTP POST with the conversation after the LLM call. No OpenTelemetry dependencies, no span management. Trade-off: Latitude only receives the conversation as a log entry -- no token counts, no cost data, no latency breakdown. See [references/log-api.md](references/log-api.md).

Be explicit about this trade-off with the user: the Log API is far simpler to implement (a single HTTP call, no extra dependencies) but sacrifices usage and cost observability. If they care about tracking tokens and costs, they must use Raw OTLP despite the added complexity.

## Approach Summaries

### Gateway Mode (Any language)

Latitude acts as the LLM gateway -- the app sends the prompt path and parameters, Latitude calls the provider and returns the response. Telemetry is automatic. Use the SDK (`prompts.run()`) for Python/TypeScript, or the HTTP API "Run a Prompt" endpoint for any other language.

### SDK Telemetry (Python, TypeScript)

The official SDK handles instrumentation automatically. Wrap LLM calls with a capture decorator/function and the SDK sends traces with full token/cost data. Minimal code required.

- **Python**: `latitude-telemetry` package, `@telemetry.capture()` decorator
- **TypeScript**: `@latitude-data/telemetry` package, `telemetry.capture()` wrapper

### Raw OpenTelemetry OTLP (Any language with OTel)

Send spans directly to `https://gateway.latitude.so/api/v4/traces` using any OpenTelemetry library. Create a parent span for the capture context and a child span for each LLM completion with `latitude.*` and `gen_ai.*` attributes. Full attribute reference in [references/span-attributes.md](references/span-attributes.md).

### Create Log API (Any language with HTTP)

POST the full conversation to `/api/v3/projects/{id}/versions/{uuid}/documents/logs` after the LLM call completes. Messages must be in PromptL format. No extra dependencies needed.

## Latitude API Base URL

All API calls use `https://gateway.latitude.so`.

---
> Source: [latitude-dev/telemetry-skill](https://github.com/latitude-dev/telemetry-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
