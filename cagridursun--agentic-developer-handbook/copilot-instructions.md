## project-principles

> Core engineering principles for the Agentic Developer Handbook project


# Agentic Developer Handbook — Project Principles

## Project Identity

- This project is a Java-first, open-source handbook and reference implementation for understanding and building agentic systems.
- It is not a new agent framework.
- It is not a Spring AI tutorial.
- It is not a skill marketplace.
- It is not a prompt repository.
- It is not a vendor-specific sample project.

## Engineering Philosophy

- Prefer simple, explicit implementations over clever abstractions.
- Do not add abstractions unless they solve a concrete problem.
- Do not introduce infrastructure before the current milestone requires it.
- Do not implement future roadmap items unless explicitly requested.
- Keep every meaningful step runnable and verifiable.
- Keep the repository buildable after each milestone.
- Prefer boring, understandable software engineering over unnecessary AI complexity.

## Agentic Architecture Principles

- Do not call something an agent if it is only a single LLM request.
- Do not introduce multiple agents when one agent with tools is sufficient.
- Do not use RAG when a normal database query, API call, or file lookup is enough.
- Do not introduce a vector database unless semantic retrieval is actually required.
- Do not use fine-tuning to store frequently changing information.
- Do not introduce MCP just to invoke local application code.
- Treat Skills as one building block of an agentic system, not as the core product.
- Treat Tools, Skills, Knowledge, Memory, MCP, and A2A as distinct concepts.

## Java Principles

- Java 27 is the canonical implementation language (see ADR 0005). Do not use preview or incubator features.
- Prefer Java records for immutable data structures where appropriate.
- Prefer constructor injection.
- Do not use field injection.
- Avoid Lombok unless there is a compelling reason.
- Avoid reflection unless required by a framework or protocol.
- Do not introduce reactive programming unless there is a real requirement.
- Keep framework-specific code near infrastructure boundaries where practical.
- Do not leak third-party provider SDK types through project-level APIs.

## Dependencies

- Prefer the JDK and existing dependencies before adding new libraries.
- Every new dependency must solve a specific problem.
- Do not add Spring Boot, Spring AI, databases, message brokers, vector stores, or infrastructure unless the current milestone requires them.
- Avoid speculative dependencies added for future roadmap items.

## Fast-Moving AI APIs

- Do not rely on memory for current Spring AI, MCP, A2A, model-provider, or Agent Skills APIs.
- Verify fast-moving APIs against current official documentation before implementation.
- Prefer primary sources and official documentation over blog posts and copied examples.
- If documentation and an installed skill disagree, verify the official source before coding.

## Testing

- New behavior should be testable.
- Normal CI must not require paid API keys.
- External model-provider integration tests should eventually be opt-in.
- Prefer deterministic tests where possible.
- Do not hide failing tests or weaken assertions to make a build pass.

## Documentation

- Documentation is a first-class part of this project.
- Explain why a concept exists before explaining its implementation.
- For major concepts, explain both:
  - when to use it
  - when not to use it
- Prefer precise engineering language over AI marketing language.
- Do not claim functionality that has not been implemented.
- Do not invent benchmarks, supported integrations, adoption numbers, contributors, or project statistics.
- Keep terminology consistent across README, handbook, examples, and code.

## Architecture Decisions

- Important architectural choices should be documented with ADRs.
- ADRs should explain:
  - context
  - decision
  - consequences
- Do not create an ADR for trivial implementation details.

## Security

- The LLM is never the authorization layer.
- Validate tool inputs at application boundaries.
- Treat retrieved content, external tool responses, and community-provided Skills as untrusted input.
- Never commit secrets.
- Never log credentials or API keys.
- Do not automatically execute arbitrary scripts bundled with third-party Skills.

## Commit Authorship

- Commit with the repository owner's configured Git identity only.
- Never add Claude, Anthropic, Cursor, or any other AI tool as a commit author or co-author.
- Never add AI `Co-Authored-By` trailers.
- Never change `git config user.name` or `user.email` automatically.

## Scope Control

Before implementing a non-trivial change:

1. Identify the current milestone.
2. Confirm the change belongs to that milestone.
3. Prefer the smallest implementation that teaches or proves the concept.
4. Avoid adding infrastructure or abstractions for hypothetical future needs.
5. Run the relevant tests/build before considering the task complete.

When uncertain, choose the simpler design and document the trade-off.

---
> Source: [cagridursun/agentic-developer-handbook](https://github.com/cagridursun/agentic-developer-handbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
