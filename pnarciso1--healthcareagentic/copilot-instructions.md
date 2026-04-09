## healthcareagentic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MyCareClaim (HealthcareAgentic) is an AI-powered healthcare cost management platform. Users upload medical bills for analysis, generate dispute letters, and appeal insurance denials through 4 AI agents (Ava, Sam, Riley, John).

**Tech Stack:** Vanilla JS frontend on Firebase Hosting, Python Cloud Functions on GCP, Firestore database, Stripe payments.

## Common Commands

```bash
# Development
npm run dev                           # Local Firebase emulator (firebase serve)

# Deployment
npm run build                         # Generate articles + deploy to Firebase
npm run deploy                        # Deploy frontend only (firebase deploy)
bash scripts/deploy-cloud-function.sh # Deploy document analysis Cloud Function
bash scripts/deploy-security-rules.sh # Deploy Firestore security rules

# Article generation
npm run generate-articles             # Generate HTML articles from articles-data.json
```

### Cloud Function Monitoring
```bash
gcloud functions logs read process-medical-bill --region=us-central1 --limit=50
```

## Architecture

### Frontend (public/)
- **index.html** - SPA shell with all UI components
- **app.js** (~6,700 lines) - All application logic: auth, agents, payments, analytics
- **style.css** (~9,500 lines) - Dark theme, mobile-first styling

The frontend uses Firebase SDK via CDN imports (no build step). Client-side routing handles all navigation.

### Backend
- **main.py** - Flask app on Cloud Run for Stripe webhooks and subscription management
- **document_analysis_function/main.py** - Cloud Function triggered on GCS uploads; uses DocumentAI for OCR and VertexAI for analysis

### Database (Firestore)
```
users/{uid}
├── analyses/{id}     # Bill analysis results
├── chat_history/{id} # Agent conversations
├── disputes/{id}     # Dispute letters and tracking
│   └── activity/{id} # Status change timeline
└── cases/{id}        # Per-case billing records
```

### Key Patterns
- All user data is isolated by `request.auth.uid == userId` in firestore.rules
- Dispute status must be one of 9 valid values (enforced in rules and app.js: draft, submitted, in_review, awaiting_response, escalated, appealed, resolved_adjusted, resolved_denied, closed)
- Server-side timestamps required for audit compliance

## Agent System

| Agent | Purpose |
|-------|---------|
| Ava | Quick answers (free: 3 questions, premium: unlimited) |
| Sam | Bill analysis with DocumentAI + VertexAI error detection |
| Riley | Dispute letter generation and status tracking |
| John | Insurance appeal packet creation |

Navigation uses "Team Tabs" system with consistent UI across all pages.

## Environment Variables

Required in `.env`:
- `STRIPE_API_KEY`, `STRIPE_WEBHOOK_SECRET`
- `GCP_PROJECT_ID`, `DOCAI_PROCESSOR_ID`, `DOCAI_LOCATION`
- `GCS_UPLOAD_BUCKET`

Firebase config is hardcoded in app.js (public keys only).

## Testing

No automated test framework. Test files in `tests/browser/` are browser console scripts:
1. Deploy to Firebase
2. Open browser DevTools console
3. Paste and execute test script from `tests/browser/`

## Documentation

Project documentation is in `docs/`:
- `docs/architecture.md` - Platform architecture
- `docs/deployment.md` - Deployment procedures
- `docs/stripe-setup.md` - Stripe integration
- `docs/security-rules.md` - Firestore security rules
- `docs/archive/` - Historical fix reports and implementation docs

## Deployment Targets

Firebase has two hosting targets in `.firebaserc`:
- `live` - Primary production
- `agent1-chat-improvements` - Feature branch testing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pnarciso1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pnarciso1)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
