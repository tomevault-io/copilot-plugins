## project-description

> Everprompt is a prompt management and sharing platform designed for AI tool users, particularly content creators. The application allows users to create, organize, and share prompts with flexible storage options and privacy controls. The prompts then can be reused across different platforms and resultss of the usage of the prompts can be also stored in the library and comprared between themselves. Prompts in the beginning include only text, maybe in markdown format, but later may be enlarged to images attachments, audio attachments, and maybe even video attachments.


# Everprompt Project Guidelines and Requirements

## Project Overview

Everprompt is a prompt management and sharing platform designed for AI tool users, particularly content creators. The application allows users to create, organize, and share prompts with flexible storage options and privacy controls. The prompts then can be reused across different platforms and resultss of the usage of the prompts can be also stored in the library and comprared between themselves. Prompts in the beginning include only text, maybe in markdown format, but later may be enlarged to images attachments, audio attachments, and maybe even video attachments.

Full minimalism. I would like the UI to be really minimal with ideally just a black or white screen and user crafting the prompt. Some clever button should be available to switch between the modes. Crafting mode should be the best one.

I see users crafting the prompt and story for 5-10 minutes in order to have a good start for the LLM request. Better prepare long and then run fast.

I foresee that this service will become popular and I will have more options added to the app, so that I want to design the schema of the database in a way which is easily extendable.

The starting audience for the prompt will be n8n community. This is a community which creates workflows and automations. Since there is a lot of LLM calls involved now, they need a lot of system prompts or user prompts. I would like Everprompt to be really good fit to this community where users will be able to store the text prompt. At later stage maybe not only text prompts, but first just text.

Free version of the app will be powerful. Paid version will just have more storage possible. This is to begin with.

I will use the app in my youtube videos. I have AI automation channel in youtube where I show how to create n8n workflows. This is a good place not just to promote it, but also to see how it works and how it can be improved.

I foresee that this service will become popular and I will have more options added to the app, so that I want to design the schema of the database in a way which is easily extendable.

## Tech Stack

- **Frontend**: Next.js 15+ with App Router, React 19, TypeScript
- **UI Framework**: shadcn/ui components with Tailwind CSS 4
- **Authentication**: Clerk
- **Database**: Neon (PostgreSQL) with Prisma ORM
- **Deployment**: Vercel
- **Domain**: everprompt.ai
- **State Management**: React Server Components + Client Components, Zustand
- **Styling**: Tailwind 4 CSS with custom themes
- **Development Tools**: TypeScript, ESLint, Prettier
- **Validation Library**: Zod
- **n8n Integration**: n8n API client for workflow integration
- **Community Features**: Public prompt library with sharing

## Development Approach

- **Incremental Development**: Always-working builds with `pnpm run dev`
- **Controlled Changes**: One feature at a time, easy rollback
- **Commit Strategy**: After each working feature, major milestone commits
- **Domain Strategy**: everprompt.ai for production, localhost:3000 for development

## Key Features

### Core Functionality

- **Prompt Management**: Create, edit, version, and organize prompts
- **Label System**: Flexible categorization with visual arc navigation
- **Workspace Support**: Multi-tenant architecture for teams
- **Real-time Sync**: Autosave and collaborative editing

### n8n Community Focus

- **Workflow Integration**: Direct n8n workflow import/export
- **Template Library**: Curated prompts for automation use cases
- **Variable System**: Dynamic prompt variables for n8n workflows
- **Community Sharing**: Public library of community-contributed prompts

### Extensibility

- **Plugin System**: Extensible architecture for future features
- **API-First**: RESTful API with GraphQL for complex queries
- **Webhook Support**: Real-time updates and integrations
- **Metadata System**: JSONB fields for extensible data

## User Interface

### Design Philosophy

- **Minimalist**: Black/white canvas with focus on content
- **Crafting-Focused**: Large text area for prompt development
- **Contextual UI**: UI appears only when needed
- **Keyboard Shortcuts**: Power user efficiency

### UI Approach: HTML/CSS with React

**Why NOT Canvas/Figma:**

- Better performance for text editing
- Superior accessibility support
- Native browser functionality (copy/paste, search)
- Easier mobile responsiveness
- Faster development iteration

**Why HTML/CSS:**

- Hardware-accelerated rendering
- Better SEO for public prompts
- Easier extensibility and customization
- Superior text input and selection

---
> Source: [mitsue-eth/everprompt-n8n-shadcn](https://github.com/mitsue-eth/everprompt-n8n-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
