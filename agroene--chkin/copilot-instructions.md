## chkin

> This file provides all context needed for Claude Code to continue building Chkin.

# CLAUDE.md — Context for AI Coding Assistants

This file provides all context needed for Claude Code to continue building Chkin.

## Session Continuity (READ FIRST)

**Before starting any work, check for active session state:**

1. **Check `.handoff.md`** in project root (if exists)
   - If `Last Updated` < 4 hours ago → Offer to continue previous session
   - If stale → Summarise what was in progress, ask if still relevant

2. **Check recent git history**
   ```bash
   git log --oneline -10
   git status
   ```

3. **When ending a session**, update `.handoff.md` with:
   - What was accomplished
   - What's in progress
   - Files modified
   - Next steps

4. **Check `GAP-ANALYSIS.md`** for tracked improvements
   - If you completed any items listed there, update the document
   - Mark completed items with ✅ and date
   - Update the "Overall Score" if significant progress was made
   - Check GitHub Issues for tracked gaps: `gh issue list --label "gap"`

---

# 1. Project Overview
Chkin is a healthcare digital onboarding system. The main function is to allow patients to submit personal details, medical-aid information, consent, and next-of-kin information via a mobile-friendly digital form, accessed primarily through a QR code in the waiting room.

A practice-facing dashboard allows staff to view submitted forms.

---

# 2. Current State

## 2.1 What's Built
- **Next.js 16** app with TypeScript, Tailwind CSS, ESLint
- **Three-portal route structure**: `/patient`, `/provider`, `/admin`
- **SQLite + Prisma** with Litestream replication to GCS
- **Better Auth** with multi-tenancy (organizations), roles, email verification
- **Docker** configuration for Cloud Run deployment
- **Admin console**: field library, provider/user management, audit logs
- **Provider portal**: form builder, QR codes, submissions management
- **Patient portal**: public form submission, data vault, consent management
- **Mobile app** (Expo/React Native): auth, biometric login, data vault
- **Production deployed** on Google Cloud Run (africa-south1)
- GitHub repo: https://github.com/agroene/chkin

## 2.2 Project Structure
```
app/
├── src/app/
│   ├── (admin)/admin/page.tsx      # Admin Console placeholder
│   ├── (patient)/patient/page.tsx  # Patient Portal placeholder
│   ├── (provider)/provider/page.tsx # Provider Portal placeholder
│   ├── layout.tsx                   # Root layout
│   ├── page.tsx                     # Landing page
│   └── globals.css
├── prisma/schema.prisma             # Empty, ready for models
├── docker-compose.yml               # PostgreSQL service
├── Dockerfile                       # Production build
└── .env.example                     # Environment template
```

## 2.3 Tech Stack (Decided)
| Layer      | Choice                                          |
|------------|-------------------------------------------------|
| Runtime    | Node.js + TypeScript                            |
| Framework  | Next.js 16 (App Router)                         |
| Database   | SQLite + Litestream (replicated to GCS)         |
| ORM        | Prisma (with better-sqlite3 adapter)            |
| Styling    | Tailwind CSS                                    |
| Auth       | Better Auth (multi-tenant, role-based)          |
| Hosting    | Google Cloud Run (africa-south1)                |
| CDN/Proxy  | Cloudflare (DNS, SSL, Worker reverse proxy)     |
| Mobile     | Expo / React Native                             |

---

# 3. Next Steps

1. **CI/CD Updates** - GitHub Actions for Cloud Run deployment
2. **Sprint 3: QR Scanning & Check-In (Mobile)**
3. Add missing secrets (resend-api-key, docuseal keys)

---

# 4. Key Design Decisions

## 4.1 Decided
- Rebuild from scratch (no access to original MVP code)
- Replicate functionality of MVP at **www.chkin.co.za**
- POPIA-compliant handling is non-negotiable
- Dynamic field library (admin can define new fields for providers)
- Database schema grows incrementally with features

## 4.2 Architecture Principles
- Monorepo: Single Next.js app with route groups for portals
- Dynamic fields via JSONB in PostgreSQL
- Static core tables for users, providers, consents, audit logs
- API routes within Next.js (can extract later if needed)

---

# 5. Constraints & Guidelines

## 5.1 Regulatory
- POPIA compliance
- Sensitive personal information (PHI) must be handled securely
- Consent must be explicit and logged
- Access to patient data must be auditable

## 5.2 Operational
- Practices have minimal IT sophistication
- Must work on unstable mobile networks
- QR onboarding must work instantly

### 5.3 Must Do
- Run tests before deploying changes
- Commit with meaningful messages (feat:, fix:, docs:, chore:)
- Update this file when adding significant features

### 5.4 Must Not Do
- Commit `.env` or secrets
- Push directly to production without testing
- Modify database schema without migration plan

### 5.5 Sensitive Data
- `.env` contains API keys — never commit
- Database contains personal transaction data
- Card numbers partially visible in `users` table

---

# 6. Reference Documentation
- `docs/mvp-analysis.md` — Comprehensive analysis of existing MVP
- `.handoff.md` — Original project handoff context

---

# 7. Commands

```bash
# Fresh install (installs deps, creates DB, seeds all data)
cd app && npm run setup

# Run dev server
cd app && npm run dev

# Build for production
cd app && npm run build

# Seed only (if DB already exists)
cd app && npx prisma db seed
```

---

## CRITICAL: Database Reset Protocol

**NEVER reset the database without explicit user permission.**

If a database reset is absolutely necessary (migration drift, etc.):

1. **ASK THE USER FIRST** - Database contains test data that takes time to recreate
2. After reset, run the setup script which handles everything:

```bash
cd app && npm run setup
```

This runs `scripts/setup.mjs` which:
- Installs npm dependencies
- Copies `.env.example` → `.env` (if missing)
- Generates Prisma client
- Pushes schema to SQLite
- Seeds: admin user, field library, categories, field priorities, PMS adapter, test practice, second practice, pre-screening form

**IMPORTANT:** On Windows, `npm run seed` does NOT work - it opens the file in Notepad++ instead of executing it. Always use `npx tsx` to run TypeScript files directly.

The user will still need to manually re-register any test providers through `/provider/register`.
---

## Development Standards (14-Point Checklist)

All development on this project must adhere to these principles:

### 1. Plan Before Programming
- No coding without clear requirements
- Document decisions in this file or docs/
- Create architecture sketches before implementation

### 2. Keep It Simple (KISS)
- Prefer simple solutions over clever ones
- New developers should understand code in < 30 mins
- Avoid unnecessary abstractions

### 3. Don't Repeat Yourself (DRY)
- Extract common logic into shared utilities
- Centralise configuration
- Single source of truth for constants

### 4. You Aren't Gonna Need It (YAGNI)
- Only build what's immediately needed
- No "future-proofing" without clear requirements
- Remove dead code promptly

### 5. SOLID Principles
- **Single Responsibility:** One reason to change per module
- **Open/Closed:** Extend via composition, not modification
- **Liskov Substitution:** Subtypes must honor parent contracts
- **Interface Segregation:** Small, focused interfaces
- **Dependency Inversion:** Depend on abstractions

### 6. Version Control
- Meaningful commit messages (feat:, fix:, docs:, chore:)
- Feature branches for new work
- Never commit secrets or credentials

### 7. Automated Testing
- Write tests for new features
- Minimum: happy path + major edge cases
- Tests must pass before merging

### 8. Clean, Readable Code
- Meaningful names for variables, functions, files
- Functions < 50 lines
- Consistent formatting (use project linters)

### 9. Security First
- Validate all inputs
- Secrets in environment variables only
- PHI/PII handling per compliance requirements
- Regular dependency updates

### 10. Performance Awareness
- No N+1 queries
- Pagination for large datasets
- Cache where appropriate
- Profile before optimising

### 11. Continuous Refactoring
- Leave code better than you found it
- Track tech debt in TODO.md
- Regular cleanup sprints

### 12. Error Handling
- Graceful error handling, no crashes
- Helpful error messages
- Structured logging
- Log context for debugging

### 13. CI/CD Workflow

- Automated tests on push
- Linting enforced
- Automated deployments (when ready)

**After every `git push`, verify CI status:**

1. Check GitHub Actions: `https://github.com/agroene/<repo>/actions`
2. If failed:
   - Review the error logs
   - Fix locally
   - Push fix and verify again
3. Never merge or consider work "done" until CI passes

**CI must pass before:**
- Merging any PR
- Deploying to production
- Moving to next task

**If using Claude Code**, after pushing ask:
> "Check if the GitHub Action passed"

Claude Code can check the workflow status and help fix failures.

### 14. Documentation
- README.md: Getting started
- CLAUDE.md: AI assistant context
- API documentation
- Update docs when code changes

---

## 15. UI/UX Design Standards

### Design-First Principle

**No UI code until design is approved.** The sequence is:

1. **Requirements** → What problem are we solving?
2. **User Research** → Who are the users? What are their goals?
3. **Wireframes** → Low-fidelity layout and flow
4. **Mockups** → High-fidelity visual design
5. **Prototype** → Interactive clickable prototype (optional)
6. **Approval** → Stakeholder sign-off
7. **Implementation** → Now write code

### Required Design Artifacts

Before implementing any UI feature, ensure these exist:

| Artifact | Purpose | Tool Suggestions |
|----------|---------|------------------|
| **User Personas** | Who are we designing for? | Markdown doc |
| **User Flows** | How do users accomplish goals? | Mermaid diagrams, FigJam |
| **Wireframes** | Layout and information architecture | Figma, Excalidraw, paper |
| **Component Inventory** | Reusable UI elements | Figma, Storybook |
| **Style Guide** | Colors, typography, spacing | Figma, Tailwind config |

### Design Checklist

Before coding any UI:

- [ ] User persona identified for this feature?
- [ ] User flow documented?
- [ ] Wireframe/mockup exists and approved?
- [ ] Mobile-first design (responsive)?
- [ ] Accessibility considered (contrast, keyboard nav, screen readers)?
- [ ] Loading states designed?
- [ ] Error states designed?
- [ ] Empty states designed?
- [ ] Edge cases considered (long text, missing data)?

### UI Implementation Standards

Once design is approved:

- [ ] Use existing components from component library first
- [ ] Match design exactly (spacing, colors, typography)
- [ ] Implement mobile breakpoint first, then scale up
- [ ] Test on actual devices, not just browser resize
- [ ] Verify accessibility (run Lighthouse, test keyboard nav)

### Design-Dev Handoff

Designers (or design phase) must provide:
- Exportable assets (icons, images)
- Exact spacing values
- Color codes
- Typography specs (font, size, weight, line-height)
- Interaction notes (hover states, transitions)

### When No Designer Available

If working solo or without a dedicated designer:

1. **Start with wireframes** — even hand-drawn sketches
2. **Use a design system** — Tailwind UI, Shadcn, Material UI
3. **Reference quality apps** — screenshot patterns you like
4. **Get feedback early** — show wireframes before coding
5. **Iterate in design tool, not code** — faster to change Figma than React

### Tools

| Purpose | Recommended |
|---------|-------------|
| Wireframes/Mockups | Figma (free tier), Excalidraw |
| Prototyping | Figma, Framer |
| User Flows | Mermaid (in markdown), FigJam |
| Icons | Heroicons, Lucide, Phosphor |
| Design System | Tailwind + Shadcn/ui |

---

---

## Code Review Checklist

Before committing, verify:
- [ ] Does this follow KISS? Is there a simpler way?
- [ ] Any duplicated code that should be extracted?
- [ ] Am I building something not yet needed?
- [ ] Are there tests for new functionality?
- [ ] Are error cases handled?
- [ ] Is this secure? Inputs validated? No hardcoded secrets?
- [ ] Is the code readable without comments explaining "what"?
- [ ] Documentation updated if needed?
- [ ] Pushed to GitHub and CI workflow passed?
- [ ] UI matches approved design/wireframe?
- [ ] Mobile responsive?
- [ ] Loading, error, and empty states implemented?

---
> Source: [agroene/chkin](https://github.com/agroene/chkin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
