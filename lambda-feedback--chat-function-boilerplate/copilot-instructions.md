## chat-function-boilerplate

> This file provides guidance to AI agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

This is a boilerplate for creating AI educational chatbots that integrate with the **Lambda-Feedback** educational platform. It deploys as an AWS Lambda function (containerized via Docker) that receives student chat messages with educational context and returns LLM-powered chatbot responses. Incoming requests follow the [muEd API](https://mued.org/) schema (`context`, `user`, `messages`).

## Commands

**Testing:**
```bash
pytest                              # Run all unit tests
python tests/manual_agent_run.py   # Test agent locally with example inputs
python tests/manual_agent_requests.py  # Test running Docker container
```

**Docker:**
```bash
docker build -t llm_chat .
docker run --env-file .env -p 8080:8080 llm_chat
```

**Manual API test (while Docker is running):**
```bash
curl -X POST http://localhost:8080/2015-03-31/functions/function/invocations \
  -H 'Content-Type: application/json' \
  -d '{"body":"{\"messages\": [{\"role\": \"USER\", \"content\": \"hi\"}]}"}'
```

**Run a single test:**
```bash
pytest tests/test_module.py        # Run specific test file
pytest tests/test_index.py::test_function_name  # Run specific test
```

## Architecture

### Request Flow

```
Lambda event → index.py (handler)
  → validates via lf_toolkit ChatRequest schema
  → src/module.py (chat_module)
    → extracts muEd API context (messages, conversationId, question context, user type)
    → parses educational context to prompt text via src/agent/context.py
    → src/agent/agent.py (BaseAgent / LangGraph)
      → routes to call_llm or summarize_conversation node
      → calls LLM provider (OpenAI / Google / Azure / Ollama)
  → returns ChatResponse (output, summary, conversationalStyle, processingTime)
```

### Key Files

| File | Role |
|------|------|
| `index.py` | AWS Lambda entry point; parses event body, validates schema |
| `src/module.py` | Transforms muEd API request → invokes agent → builds ChatResponse |
| `src/agent/agent.py` | LangGraph stateful graph; manages message history and summarization |
| `src/agent/prompts.py` | System prompts for tutor behavior, summarization, style detection |
| `src/agent/llm_factory.py` | Factory classes for each LLM provider (OpenAI, Google, Azure, Ollama) |
| `src/agent/context.py` | Converts muEd question/submission context dicts to LLM prompt text |
| `tests/utils.py` | Shared test helpers: `assert_valid_chat_request`, `assert_valid_chat_response` |
| `tests/example_inputs/` | Real muEd payloads used for end-to-end tests |

### Agent Logic (LangGraph)

`BaseAgent` maintains a state graph with two nodes:
- **`call_llm`**: Invokes the LLM with system prompt + conversation summary + conversational style preference
- **`summarize_conversation`**: Triggered when message count exceeds ~11; summarizes history and also extracts the student's preferred conversational style

Messages are trimmed after summarization to keep context window manageable. The `summary` and `conversationalStyle` fields persist across calls via the `ChatRequest` metadata.

### muEd API Format

`src/module.py` handles the muEd request format (https://mued.org/). The `context` field in `ChatRequest` contains nested educational data (question parts, student submissions, task info) that gets parsed into a tutoring prompt via `src/agent/context.py`.

### LLM Configuration

LLM provider and model are set via environment variables (see `.env.example`). The `llm_factory.py` selects the provider at runtime. The Lambda function name/identity is set in `config.json`.

The agent uses **two separate LLM instances** — `self.llm` for chat responses and `self.summarisation_llm` for conversation summarisation and style analysis. By default both use the same provider, but you can point them at different models (e.g. a cheaper model for summarisation) by changing the class in `agent.py`.

## Deployment

- Pull requests: `.github/workflows/test-lint.yml` runs pytest only
- Pushing to `main`: `.github/workflows/staging-deploy.yml` runs tests then deploys to AWS staging via the shared `lambda-feedback/chat-function-workflows` reusable workflows
- Production: `.github/workflows/production-deploy.yml` is `workflow_dispatch`-only with a `version-bump` input; redeploys staging, pauses on the `production-override` GitHub Environment for manual approval, then creates a `vX.Y.Z` tag + GitHub Release and deploys to prod
- All environment variables (API keys, model names) are injected via GitHub Actions secrets/variables — do not hardcode them

---
> Source: [lambda-feedback/chat-function-boilerplate](https://github.com/lambda-feedback/chat-function-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
