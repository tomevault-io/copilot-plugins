## featurize-js-metacto

> Every time you receive a new instruction or prompt, append it to prompts.txt in the project root with a timestamp (ISO 8601 format including time and timezone, example: 2026-03-26T11:52:00-05:00) and a brief summary of what you did in response. Create the file if it doesn't exist. Keep this log updated throughout all sessions.

# Project Rules

## Prompt Logging

Every time you receive a new instruction or prompt, append it to prompts.txt in the project root with a timestamp (ISO 8601 format including time and timezone, example: 2026-03-26T11:52:00-05:00) and a brief summary of what you did in response. Create the file if it doesn't exist. Keep this log updated throughout all sessions.

Log format:
[ISO-8601-timestamp-with-time] PROMPT: <one line summary> | ACTION: <what was done>

---

## Project Structure

featurize/
├── backend/ # Django REST API
│ ├── config/ # Django settings, urls, wsgi
│ ├── apps/
│ │ ├── auth_app/ # Register, login, JWT
│ │ └── features/ # Features CRUD and votes
│ ├── requirements.txt
│ └── .env
└── frontend/ # React + Vite
├── src/
│ ├── components/
│ │ ├── ui/ # Primitives: Button, Input, Badge
│ │ └── features/ # FeatureCard, VoteButton, FeatureForm
│ ├── pages/ # Home, Auth, Admin
│ ├── hooks/ # useFeatures, useVote, useAuth
│ ├── services/ # api.ts, feature.service.ts, auth.service.ts
│ ├── store/ # Zustand stores
│ ├── types/ # TypeScript interfaces
│ └── validations/ # Zod schemas
└── .env

## Tech Stack

Backend: Python 3.11, Django 5.0, djangorestframework 3.15,
djangorestframework-simplejwt 5.3, django-cors-headers 4.3,
boto3 1.34, python-decouple 3.8, bcrypt 4.1

Frontend: React 18, TypeScript 5, Vite 5,
TanStack Query 5, Zustand 4, React Hook Form 7,
Zod 3, Axios 1.6, Framer Motion 11,
Tailwind CSS 3, React Router 6, date-fns 3,
lucide-react 0.400

## Core Principles

- SOLID: single responsibility per module, class, component
- DRY: no copy-paste logic, extract shared utilities
- KISS: simplest solution that works, no premature abstraction
- YAGNI: build only what is required right now
- OOP: business logic in service classes, not in views or components
- TypeScript strict mode, no any
- Python type hints on all functions

## Validation

- Frontend: Zod schemas in src/validations/, React Hook Form with zodResolver
- Backend: DRF serializers validate all input, never trust client data
- Inline field errors on blur, disable submit while invalid or loading
- Forms must submit on Enter key press

## Error Handling

- Frontend: per-request error in hooks, toast for async errors,
  inline errors for forms
- Backend: consistent error response shape:
  { "error": "ERROR_CODE", "message": "Human readable", "field": "optional" }
- Correct HTTP status codes always

## State Management

- Server state: TanStack Query (useQuery, useMutation)
- UI state: Zustand (modals, filters)
- Form state: React Hook Form + Zod
- Query keys centralized in src/lib/queryKeys.ts

## API Design

- Base URL: http://localhost:8000/api
- Auth: JWT Bearer token in Authorization header
- POST /auth/register
- POST /auth/login
- GET /auth/me
- GET /features?status=&sort=top|new&page=
- POST /features
- POST /features/:id/vote
- PATCH /features/:id/status (admin only)

## DynamoDB Tables

- featurize-users: PK = USER#<email>
- featurize-features: PK = FEATURE#<id>, SK = METADATA
  GSI: status-createdAt-index, voteCount-index
- featurize-votes: PK = USER#<userId>, SK = FEATURE#<featureId>

## UI/UX Standards

- Dark theme: background #08080f, surface #0f0f1a, border rgba(255,255,255,0.06)
- Accent: #6366f1, gradient: linear-gradient(135deg, #6366f1, #8b5cf6)
- Geist font via @fontsource/geist
- Mobile-first responsive
- Skeleton loaders, never spinners
- Optimistic updates on vote
- Every form submits on Enter
- Smooth transitions 150ms ease on all interactive elements

## Runtime Verification

After any backend change run: python manage.py check
After any frontend change run: npx tsc --noEmit
Before declaring done always verify the feature works manually.

## Git

Conventional commits: feat:, fix:, refactor:, chore:
Commit after each completed prompt.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Juanse7793)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Juanse7793)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
