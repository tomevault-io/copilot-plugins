## clinicalscribe

> This repository is part of **Qoder (AgentForge)**.

# ClinicalScribe + Qoder Agents

## Project Context
This repository is part of **Qoder (AgentForge)**.  
It powers **ClinicalScribe**, an ambient medical documentation system designed for clinicians.  
Qoder orchestrates multiple agents that work together to process, guide, and structure information.

## Agents in Qoder
- **ContentForge** → Generates structured medical documentation and transforms scrolls into usable formats.
- **QuizMaster** → Creates adaptive tests and assessments for learning from medical cases.
- **AdaptiveGuide** → Provides contextual explanations, adapting to the user's knowledge and role (clinician, patient, admin).
- **AgentOrchestra** → The conductor: coordinates agent workflows, routes tasks, and manages execution order.

## Firestore Scrolls
Qoder agents can access information stored in Firestore through utility functions:

- **Client-side**: `getScroll(id)` in [lib/utils.ts](file:///c%3A/Users/Kodjo%20AdjeeLaa/clinicalscribe/lib/utils.ts) - Fetches scrolls by ID from Firestore
- **Server-side**: `getScroll(id)` in [lib/serverUtils.ts](file:///c%3A/Users/Kodjo%20AdjeeLaa/clinicalscribe/lib/serverUtils.ts) - Fetches scrolls by ID using Firebase Admin
- **Server-side**: `getAllScrolls()` in [lib/serverUtils.ts](file:///c%3A/Users/Kodjo%20AdjeeLaa/clinicalscribe/lib/serverUtils.ts) - Fetches all scrolls using Firebase Admin

## Goals
- Provide **real-time transcription** and **SOAP note generation** for clinicians.
- Integrate **Firebase** (Auth + Firestore + Storage) for authentication and data persistence.
- Use **Firebase** for folders, reports, and analytics.
- Generate **PDFs** with Puppeteer for exporting notes and reports.
- Manage billing and plans via **Stripe**.

## Known Issues
- **Firebase**: Service account permission errors and auth sync issues.
- **PDF Rendering**: Puppeteer inconsistencies (local dev vs Vercel serverless).
- **Session Management**: Dashboard sessions not always syncing between Firebase services.
- **Storage Costs**: Decision to prioritize PDFs over audio storage.

## Tech Stack
- **Framework**: Next.js 15
- **Auth & DB**: Firebase Auth + Firestore + Firebase Storage
- **Storage**: Firebase Storage buckets (reports, recordings, folders)
- **Payments**: Stripe subscriptions
- **PDF Engine**: Puppeteer with serverless Chromium
- **Hosting**: Vercel

## Guidance for Gemini
When assisting:
- Prioritize **agent collaboration logic** (how they share tasks).
- Suggest improvements for **auth/session sync**.
- Optimize **Firebase queries** for reports/folders.
- Help debug **PDF generation issues** in `api/pdf/render/route.ts`.
- Provide ideas for **agentic orchestration**, especially around ClinicalScribe's workflows.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/techshare101)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/techshare101)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
