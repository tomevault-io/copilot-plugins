## portfoliomanu

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Portfolio website for Manuel Ferrer built with **Ruby on Rails 7.1.3** (Ruby 3.3.6). Includes an AI chatbot that answers questions about Manuel's professional background using RAG, and a **personal finance assistant** (PWA) at `/finance` powered by `ruby_llm` with AI tool calling.

## Common Commands

```bash
# Start development server
rails server                    # or: rails s

# Database
rails db:create db:migrate      # Setup database (requires PostgreSQL with vector extension)
rails db:seed                   # Seed skills, sample projects and experiences
rails db:reset                  # Drop, create, migrate and seed

# Console
rails console                   # or: rails c

# Assets
rails assets:precompile         # Precompile for production

# Linting
rubocop                         # Run RuboCop (config in .rubocop.yml, max line length: 120)
rubocop -a                      # Auto-fix offenses

# Tests
rails test                      # Run test suite (Minitest + Capybara)

# Background jobs (Solid Queue)
# Solid Queue runs via Puma plugin in production (see config/puma.rb)
```

## Architecture

### Stack
- **Backend:** Rails 7.1, PostgreSQL (with pgvector extension), Devise authentication
- **Frontend:** Hotwire (Turbo + Stimulus), Bootstrap 5.2, Importmap (no Node/Webpack)
- **Assets:** Sprockets + SASSC for CSS, Importmap for JS modules
- **Jobs:** Solid Queue (runs as Puma plugin)
- **Storage:** Cloudinary (images), Active Storage
- **AI (Portfolio):** OpenAI API (ruby-openai gem), LangChainRB (custom fork: puzzle404/langchainrb)
- **AI (Finance):** ruby_llm gem (~1.13) with tool calling (acts_as_chat, RegisterExpenseTool, ListExpensesTool, GetBalanceTool)

### AI Chatbot / RAG System

The core differentiator of this portfolio. Flow:

1. **Chunking** (`app/models/concerns/chunkable.rb`): Projects/Experiences are split into text chunks using LangChain's RecursiveText chunker (1535 tokens, 200 overlap). Each chunk is embedded via `text-embedding-ada-002` and stored in the `chunks` table (1536-dim vectors).

2. **Question handling** (`app/controllers/questions_controller.rb`): User question saved with a `session_id`, then `ChatbotJob` is enqueued.

3. **ChatbotJob** (`app/jobs/chatbot_job.rb`): Embeds the question with `text-embedding-3-small`, finds nearest chunks via pgvector (`Chunk.nearest_neighbors`), builds system prompt with context, calls `gpt-3.5-turbo`, updates the question record.

4. **Real-time response**: Answer is broadcast to the user via Turbo Streams / Action Cable.

Key model: `Chunk` (`app/models/chunk.rb`) - polymorphic association via `chunkable_type/chunkable_id`, uses `neighbor` gem for vector search.

### Finance Assistant (PWA)

Personal finance chat at `/finance`, protected by Devise login. WhatsApp-style interface.

- **ruby_llm integration**: `Chat` model with `acts_as_chat`, `Message` with `acts_as_message`, `ToolCall` with `acts_as_tool_call`
- **Tool calling**: AI parses natural language ("pagué 500 de luz") and calls `RegisterExpenseTool` to create `Finance::Expense` records automatically
- **Tools** (`app/tools/`): `RegisterExpenseTool`, `ListExpensesTool`, `GetBalanceTool` - each initialized with `user` context
- **Job** (`app/jobs/finance/chat_response_job.rb`): Configures system prompt + tools, calls `chat.ask()`, broadcasts response via Turbo Streams
- **Layout**: Separate mobile-first layout (`app/views/layouts/finance.html.erb`)
- **PWA**: Installable via `public/manifest.json` + `public/service-worker.js`, start_url points to `/finance`

### Data Model

- **User** (Devise) - admin authentication
- **Project** has_many Skills (through ProjectSkill), includes Chunkable, has_many_attached photos
- **Experience** has_many Skills (through ExperienceSkill), has_many_attached photos
- **Skill** - predefined list (Skill::SKILLS constant), many-to-many with Projects and Experiences
- **Question** - stores user_question, ai_answer, session_id
- **Chunk** - polymorphic (chunkable), stores content + embedding vector
- **Chat** (ruby_llm) - belongs_to User, acts_as_chat with Messages/ToolCalls
- **Message** (ruby_llm) - belongs_to Chat, acts_as_message, has_many_attached attachments
- **ToolCall** (ruby_llm) - belongs_to Message, acts_as_tool_call
- **Finance::Category** - predefined expense categories (10 defaults, table: finance_categories)
- **Finance::Expense** - belongs_to User + Category + Message(optional), amount/description/date/currency

### Routes

```
root              -> pages#home
resources         projects (full CRUD)
resources         experiences > photos (nested destroy)
resources         questions (index, create)
resources         contacts (index, new, create)
devise_for        users
/letter_opener    -> development mail preview
/finance          -> finance/chats#show (PWA chat, requires login)
/finance/chat/messages -> finance/messages#create
```

### Environment Variables

Required in `.env`:
- `OPENAI_ACCESS_TOKEN` - OpenAI API key (used for embeddings + chat)
- `CLOUDINARY_URL` - Cloudinary credentials for image storage

## Code Style

- RuboCop enabled with relaxed rules (see `.rubocop.yml`)
- Max line length: 120 characters
- Style/Documentation disabled (no mandatory class docs)
- Use double quotes for strings (StringLiterals disabled)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/puzzle404)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/puzzle404)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
