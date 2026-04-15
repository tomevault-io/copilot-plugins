## slack-to-faq

> This is the context and rules for all responses about our "Slack to FAQ" app. The app turns Slack messages into easy-to-search FAQs for teams. We're building v1 to start. Use this to guide all your answers. Always follow the rules exactly.

# Project Context and Rules for LLMs

This is the context and rules for all responses about our "Slack to FAQ" app. The app turns Slack messages into easy-to-search FAQs for teams. We're building v1 to start. Use this to guide all your answers. Always follow the rules exactly.
- Stick strictly to the rules and tools listed below.
- Keep answers clear, simple, and focused on this project.
- If unsure about something, ask a short question for clarity.

## Frontend
The frontend is the part users see in their browser, like buttons and forms.

**Rules**
1. Style everything with Tailwind only.
2. Use Tailwind Flex properties for layouts. Do not use Tailwind Grid system.
3. Component naming follows CamelCase.
4. At the top of every file, just below the imports, document with a comment following this example:
```text
/**
* [Name of file / function / component]
* @author [if done by LLM, then name should be **LLM**]
*
* @description [very brief - 1 / 2 lines]
*
* @todo
* - xxx
* - [list of things remaining / to improve]
*/
```
5. Above every code logic (or on the side), comment with: // [simple explanatory phrase]

**Context**
- Built with React fully type safe (.tsx) and Next.js App Router.
- Handles user login via Clerk.
- Priority: Keep the user interface simple and easy to use.

## Backend
The backend handles data, storage, and connections to microservices behind the scenes.

**Rules**
1. Follow the same documentation and commenting convention established in the Frontend section of this guide.
2. Build microservices with Flask for simple tasks or FastAPI for faster ones.
3. Use Supabase for all database work and file storage.
4. Handle user login only with Clerk.
5. Process payments only with Stripe.

**Context**
- Overall app type: T3 stack (TypeScript, Tailwind, tRPC, Next.js).
- Database: Supabase.
- File storage: Supabase (like an S3 bucket).
- User login: Clerk.
- Payments: Stripe.
- Connects to Slack for pulling messages.
- Priority: Focus on secure handling of Slack data and user info.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/endpoint-data) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
