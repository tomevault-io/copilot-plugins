## commit-digest

> Overall project description

Project Name: CommitDigest
Slogan: "CommitDigest: Code progress, summarized."

Core Purpose:
CommitDigest is an AI-powered SaaS platform designed to automatically generate human-readable summaries of Git commit histories. It helps development teams, team leads, and project managers save time, improve communication, and gain quick insights into coding progress without manually sifting through commit logs.

Key Functionality (MVP):
The application allows users to:

Register and authenticate using Supabase.
Connect their GitHub repositories (using Personal Access Tokens - PATs - for MVP).
Configure reports for these repositories, specifying a schedule (e.g., "daily," "weekly") and a webhook URL for delivery.
The system then automatically fetches new commits since the last report for scheduled configurations.
It utilizes Langchain (with an underlying LLM like OpenAI's GPT-4o Mini) to process these commits and generate a concise, human-readable summary in Markdown format.
These Markdown reports are delivered to the user-configured webhook URL.
A web dashboard allows users to manage their connected repositories, create and manage report configurations, and view the content of the latest generated reports.
Architecture & Technology Stack:
The project is structured as a monorepo (using PNPM Workspaces) with two main applications:

Backend (apps/api):

Framework: Nest.js
Language: TypeScript
Database & Authentication: Supabase (PostgreSQL backend, Supabase client for interactions)
AI Integration: Langchain (to orchestrate LLM calls for summarization)
Scheduling: @nestjs/schedule or node-cron for automated report generation.
Primary Responsibilities: User auth validation (Supabase JWTs), managing repository connections (including secure PAT storage & encryption), report configurations, scheduling and executing report generation (fetching commits, calling Langchain/LLM, sending webhooks), and serving data to the frontend.
Frontend (apps/web):

Framework: Next.js (App Router)
Language: TypeScript
UI Components: Shadcn/ui
Styling: Tailwind CSS
State Management: Zustand
Data Fetching/Server State Management: React Query (TanStack Query)
Authentication Client: Supabase client (supabase-js) for user sign-up, sign-in, and session management.
Primary Responsibilities: User interface for registration/login, dashboard for managing repositories and report configurations, displaying generated report summaries, and interacting with the backend API.
Current Development Focus:
Building out the MVP features as described above, focusing on a stable end-to-end flow for GitHub repositories.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zymantas-katinas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
