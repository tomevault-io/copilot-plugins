## customer-operations-agent

> This document defines how AI agents should behave in the Amorosa AI system. It is written for development, implementation, and future extension of the Messenger assistant, backend services, and admin dashboard.

# Agents.md

This document defines how AI agents should behave in the Amorosa AI system. It is written for development, implementation, and future extension of the Messenger assistant, backend services, and admin dashboard.

## 1. Project Overview

Amorosa AI is an AI-powered Facebook Messenger assistant for flower shops. It answers customer questions, recommends products, creates draft orders, books appointments, and escalates complex conversations to humans when needed.

The system is designed around tool calling, retrieval-augmented generation, conversation memory, and integration with business workflows such as Google Calendar and Slack.

## 2. Goals

- Reduce repetitive customer support work.
- Improve response speed and consistency.
- Automate product inquiries, order intake, and appointment booking.
- Keep the experience natural in English and Filipino.
- Escalate to humans when the AI should not decide alone.
- Provide a portfolio-quality example of practical AI engineering.

## 3. Tech Stack

- Frontend: React, Vite, Tailwind CSS, React Router, TanStack Query.
- Backend: FastAPI, LangGraph, LangChain, SQLAlchemy, Alembic.
- AI: OpenRouter, configurable LLM model, tool calling, RAG.
- Database: PostgreSQL with pgvector.
- Integrations: Facebook Messenger Platform, Google Calendar API, Slack API.
- Deployment: Docker, Railway or Render for development, VPS or cloud VM for production.

## 4. Folder Structure

The repository should stay modular and easy to extend.

```text
amorosa-ai/
  frontend/
  backend/
  knowledge/
  migrations/
  docs/
  agents.md
  README.md
  PRD.md
  ARCHITECTURE.md
```

Suggested backend layout:

```text
backend/
  app/
    api/
    core/
    db/
    models/
    services/
    tools/
    workflows/
    rag/
```

Suggested frontend layout:

```text
frontend/
  src/
    components/
    pages/
    hooks/
    api/
    routes/
    styles/
```

## 5. Coding Standards

- Prefer small, focused modules.
- Keep functions readable and easy to test.
- Use explicit types where they improve clarity.
- Avoid deeply nested branching when a helper function is clearer.
- Keep business logic out of route handlers when possible.
- Name functions and files by intent, not implementation detail.
- Add validation at the boundary of each external input.
- Favor deterministic behavior for agent tools and workflows.

## 6. Backend Standards

- Use FastAPI for HTTP endpoints and webhooks.
- Keep request handlers thin and delegate work to services or workflow classes.
- Use SQLAlchemy models and Alembic migrations for schema changes.
- Store conversation, customer, order, and appointment data in PostgreSQL.
- Validate webhook signatures and external payloads before processing.
- Keep secrets in environment variables.
- Use structured logging for every important workflow step.
- Design tools to be idempotent where possible.

## 7. Frontend Standards

- Build the dashboard as an admin-only interface.
- Use React and Vite for a fast developer experience.
- Keep UI state predictable and data access centralized.
- Use TanStack Query for server state.
- Make modules easy to scan: conversations, orders, appointments, products, knowledge base, analytics, and settings.
- Prefer clear information hierarchy over decorative complexity.
- Make mobile behavior functional even if the dashboard is primarily desktop-oriented.

## 8. AI Agent Design

The AI agent should behave like a capable shop assistant, not a generic chatbot.

Core responsibilities:

- Detect customer intent.
- Maintain conversational context.
- Decide when to answer directly and when to call a tool.
- Retrieve knowledge before answering policy, product, or service questions.
- Collect structured data for orders and appointments.
- Escalate when confidence is low or the request is sensitive.

Design principles:

- The agent should not guess on business-critical details.
- Tool results should be treated as authoritative.
- The agent should ask one clear follow-up question at a time when information is missing.
- The assistant should preserve a friendly, concise, service-oriented tone.

## 9. RAG Implementation

RAG should answer questions using markdown-based knowledge documents.

Implementation expectations:

- Store source content in `knowledge/` as markdown files.
- Chunk documents by semantic sections.
- Generate embeddings and store them in pgvector.
- Retrieve the most relevant chunks before composing the final answer.
- Prefer RAG for product details, policies, business hours, delivery rules, and consultation information.
- Reindex knowledge whenever documents change.

Retrieval rules:

- Use the knowledge base before the model improvises on factual business data.
- If retrieval is weak or incomplete, the agent should say so and offer escalation or clarification.

## 10. Database Design

The database should support conversation history, customer records, workflow state, and business objects.

Core tables:

- customers
- conversations
- messages
- products
- orders
- appointments
- knowledge_documents
- knowledge_chunks
- escalations

Design principles:

- Store only what is needed for the business workflow.
- Keep conversation history queryable.
- Normalize reusable business entities.
- Track status changes explicitly for orders and appointments.
- Use timestamps for auditing and chronological reconstruction.

## 11. Messenger Workflow

Messenger is the main customer entry point.

Typical flow:

1. Meta webhook receives a message.
2. Backend validates and stores the event.
3. Conversation history is loaded.
4. The agent determines intent.
5. The agent either answers directly or calls a tool.
6. The response is sent back to Messenger.
7. The exchange is persisted.

Workflow rules:

- Support multi-turn conversations.
- Preserve customer context across messages.
- Handle attachments and product image requests.
- Keep responses short enough for chat but informative enough to be useful.

## 12. Appointment Workflow

Appointment handling should help customers book consultations without back-and-forth friction.

Expected flow:

1. Identify the request as a booking request.
2. Ask for missing details such as preferred date, time, and contact name.
3. Check availability in Google Calendar.
4. Suggest open slots.
5. Create the appointment once details are confirmed.
6. Send the confirmation back through Messenger.

Rules:

- Prevent double-booking.
- Do not create appointments without required information.
- Surface conflicts clearly.
- Escalate unusual booking cases to staff.

## 13. Order Workflow

Order handling should create draft orders for staff review rather than attempting to process payment automatically.

Expected order fields:

- Bouquet or product selection
- Recipient name
- Delivery address
- Contact number
- Delivery date
- Card message

Workflow rules:

- Ask for missing details one at a time.
- Confirm the order summary before saving the draft.
- Use product recommendations when the customer has not chosen a bouquet.
- Keep payment manual unless the business later adds online payments.

## 14. Human Escalation

The system should escalate to a human when the AI should not continue alone.

Escalation triggers:

- Customer explicitly requests a human.
- Refund or payment questions arise.
- The customer is angry, frustrated, or repeating the same request.
- The agent has low confidence.
- A workflow cannot be completed safely.

Escalation behavior:

- Pause automation for that conversation.
- Summarize the exchange for staff.
- Notify staff through Slack.
- Mark the conversation as escalated.

## 15. Tool Calling

Tool calling is the control layer that lets the agent act instead of only talking.

Expected tools:

- search_knowledge
- create_order
- send_product_images
- check_calendar
- create_appointment
- escalate_to_human
- send_slack_notification
- messenger_send

Tool design rules:

- Every tool should have a narrow purpose.
- Inputs should be explicit and validated.
- Tools should return structured results when possible.
- The agent should explain what it is doing when a tool call affects the customer experience.
- Tool failures should be handled gracefully and reported clearly.

## 16. Development Roadmap

1. Messenger webhook and OpenRouter integration.
2. Core agent loop with tool calling.
3. RAG knowledge search.
4. Draft order creation.
5. Google Calendar appointment booking.
6. Slack escalation and human handoff.
7. Product image responses and richer conversation memory.
8. Admin dashboard and analytics.
9. Deployment hardening and production observability.

## 17. Definition of Done

An implementation is done when:

- The feature works end to end in the intended workflow.
- Data is persisted correctly.
- Error cases are handled clearly.
- The agent behavior is predictable and testable.
- The feature matches the PRD and architecture intent.
- The code is readable and fits the project structure.
- The user-facing behavior has been validated manually or through tests.

For AI-specific work, done also means:

- The agent calls the right tool for the right reason.
- The answer is grounded in the knowledge base when factual information is needed.
- Escalation happens when confidence or safety requires it.
- The output is useful in real Messenger conversations.

---
> Source: [ChadBojelador/customer-operations-agent](https://github.com/ChadBojelador/customer-operations-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
