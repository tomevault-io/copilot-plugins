## ekon-2025-agents

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a demo repository for a EKON 2025 conference talk about building AI agents. It demonstrates core concepts of LLM interactions through HTTP request examples.

## Project Structure

The repository contains educational HTTP request files demonstrating:
- `requests/01 plain llm.http` - Basic LLM API calls showing stateless nature
- `requests/02 memory.http` - Managing conversation context through message history
- `requests/03 functions.http` - Tool/function calling capabilities with LLMs

## Environment Setup

1. Copy `.env.template` to `.env`
2. Configure your LLM provider:
   - **Ollama (default)**: Pre-configured for local models
   - **OpenAI**: Uncomment and add API key
   - **OpenRouter**: Uncomment and add API key

## Required Models for Ollama

```bash
ollama pull mistral-small3.2:24b
ollama pull mistral-large:latest
ollama pull gpt-oss:20b
```

## Key Concepts Demonstrated

The HTTP files showcase progressive complexity in LLM interactions:
1. **Stateless nature** - LLMs don't remember previous interactions without explicit context
2. **Memory management** - Including conversation history in message arrays
3. **Tool usage** - Providing functions that LLMs can call to perform actions

## Testing HTTP Requests

Use VS Code REST Client extension or similar tools to execute the `.http` files directly. Environment variables are loaded from the `.env` file using `{{$dotenv %VARIABLE_NAME}}` syntax.

# Next up

We want to create a complete agent tooling, in C#, without the use of any non standard library. We only use normal .NET 9, and can use everything that is batteries includded in ASP.NET Core (e.g. Microsoft.Extensions.Configuration, Extensions.Logging, Extensions.Hosting if need be), but no non-default libraries like Serilog, Newtonsoft etc.

Use file-scoped namespaces.
Use `dotnet csharpier .` to have everything formatted correctly.

Every thing should be thoroughly commented so that a conference participant can follow and understand the code

The agent should demo the following concepts:
- LLM calls to OpenAI API compatible endpoints configured with base url and api key
- Memory management
- Complete state of the agent should be persisted (serialized to JSON with System.Text.JSON) after every LLM response and with every message send to the LLM (be it user input or tool response)
- Tools, created with reflection from classed with [Description] attributes
- Tool calls, again done by reflection
- Agent should use multiple models for different steps. Mainly a reasoning model for the first step of creating a plan to solve the issue, and a little bit faster model without reasoning for the actual doing.
- Memory management should support compacting (creating a summary and replacing the history with that summary) from time to time to save tokens
- LLM client should use prompt caching whenever possible
- LLM client should make use of streaming responses to display the current reasoning state on the UI
- UI is console based (like Gemini CLI oder Codex CLI or Claude Code)
- UI should allow inputs while the model is working, and any input should be added to the very next roundtrip
- ESC should be able to cancel the current action (so everything needs to be async and allow abortion)
- Initial tools that should be provided is a tool that allows making general web requests and returning the retrieved content to the LLM, a tool to read the current directories content and a tool to read and write files
- Every tool call should prompt a user approval, and the user can choose for each tool call individually if that should be allowed or declined, if that action should be allowed for every time the model wants to call this in the current session and if the user declines he optionally can add a text to what the model should do differently
- every action should be in a try/catch and API errors (e.g. overloaded when the model is under heavy load) should be handled with a retry, up to 5 retries
- Every step the model makes should be displayed on the UI, so that the user can follow all steps directly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thinktecture-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
