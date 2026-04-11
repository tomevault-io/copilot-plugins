## knowledgescroll

> Educational mobile app that converts TikTok-style scrolling into learning experiences.

# KnowledgeScroll Agent Instructions

## Project Context
Educational mobile app that converts TikTok-style scrolling into learning experiences.
Core technologies: React Native (mobile), Node.js/Express (backend), Gemini 2.5 Pro (AI), PostgreSQL + MongoDB (databases).

## Architecture Patterns
- **Mobile:** Component-based React Native with TypeScript
- **Backend:** RESTful API with controller-service-model pattern
- **AI Integration:** Prompt template system in `/backend/src/utils/promptTemplates.ts`
- **State Management:** Redux Toolkit with RTK Query
- **Database:** PostgreSQL (users/progress), MongoDB (knowledge trees JSON)

## Code Conventions
- TypeScript strict mode enabled
- ESLint + Prettier enforced
- Functional components with hooks (React)
- Async/await for promises (no `.then()`)
- Comprehensive error handling with try-catch
- Detailed JSDoc comments for public functions

## Testing Standards
- Jest + React Native Testing Library (mobile)
- Jest + Supertest (backend API)
- Minimum 80% code coverage required
- Test files colocated: `ComponentName.test.tsx`

## Key Files to Understand
- `/backend/src/utils/promptTemplates.ts` - All Gemini prompts for tree/article generation
- `/mobile/src/components/GestureRotor.tsx` - Core gesture interaction logic
- `/backend/src/services/geminiService.ts` - AI service wrapper
- `/backend/src/controllers/articleController.ts` - Content generation with 8 flag system

## Flag System (Critical Feature)
8 content generation flags: -a (article), -t (table), -ex (examples), -p (practical), -ctr (controversy), -box (unique perspective), -m (meta-secrets), -q (quiz).
Each flag triggers different prompt template in `promptTemplates.ts`.

## Environment Variables Required
- `GEMINI_API_KEY` - Google AI API key
- `DATABASE_URL` - PostgreSQL connection
- `MONGODB_URI` - MongoDB connection
- `JWT_SECRET` - Auth token secret
- `PINECONE_API_KEY` - Vector database

## Development Workflow
1. Start backend: `cd backend && npm run dev`
2. Start mobile: `cd mobile && npm start`
3. Run tests: `npm test`
4. Lint: `npm run lint`

## Common Tasks for Jules
- "Add comprehensive tests for [component/service]"
- "Refactor [file] to improve type safety"
- "Implement new prompt template for [flag] in promptTemplates.ts"
- "Add error handling and logging to [controller]"
- "Optimize database queries in [service]"
- "Document API endpoints in docs/API.md"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielpetrupascu-cell)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/danielpetrupascu-cell)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
