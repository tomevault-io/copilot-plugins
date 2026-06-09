## zenopsis

> <project_description>

<project_description>
You are working on Zenopsis - a Telegram bot project that monitors group chat messages, stores them in a database, and provides AI-powered conversation summaries using OpenAI's LLM.
</project_description>

<guidelines>
Project Guidelines:
1. All code must be written in TypeScript with proper type definitions
2. Use Bun.js as the runtime environment
3. Follow clean code principles and maintain consistent formatting
4. Write meaningful commit messages describing the changes
5. You must update the ROADMAP.md file with the latest changes and progress.
</guidelines>

<tech_stack>
Technology Stack:
- GramIO Framework for Telegram Bot API interactions
- Drizzle ORM for database operations
- Instructor.js for structured OpenAI API interactions
</tech_stack>

<architecture>
Project Structure:
/src
  /bot     - Telegram bot implementation using GramIO
  /db      - Database models, migrations, and Drizzle ORM setup
  /llm     - OpenAI integration and conversation summarization logic
  /types   - TypeScript type definitions and interfaces
</architecture>

<features>
Core Features:
1. Monitor and capture all messages in Telegram group chats
2. Store messages in a database using Drizzle ORM
3. Periodically summarize conversations using OpenAI's LLM
4. Post summaries back to the group chat automatically
</features>

<best_practices>
Best Practices:
1. Keep sensitive data (API keys, tokens) in environment variables
2. Write modular and reusable code
3. Handle errors gracefully with proper logging
4. Document complex logic and important functions
5. Use TypeScript's type system effectively to prevent runtime errors
</best_practices>

---
> Source: [nullptr-party/zenopsis](https://github.com/nullptr-party/zenopsis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
