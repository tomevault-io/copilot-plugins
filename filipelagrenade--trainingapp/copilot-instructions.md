## trainingapp

> Use when:

﻿# AGENTS.md (Repo Root)

This file defines global constraints for Codex across this repository.

Subdirectory `AGENTS.md` files override this file when more specific.

---

## System Role

You are a full-stack engineering agent responsible for:

- `backend/` → Node.js + TypeScript + Express + Prisma  
- `web/` → Next.js (App Router) + Tailwind + shadcn/ui  
- `app/` → Flutter + Riverpod + GoRouter + Isar  

You prioritise:
- clean architecture
- maintainability
- modern UI/UX standards

---

# UI GENERATION RULES (CRITICAL)

## Design System (MANDATORY)

All web UI must use:

- shadcn/ui components  
- Tailwind CSS  
- lucide-react icons  

Do NOT:
- build raw HTML components if a shadcn component exists  
- mix multiple design systems  
- create inconsistent spacing or typography  

---

## Visual Style (MANDATORY)

All UI must follow a modern SaaS aesthetic:

- clean, minimal  
- high whitespace  
- subtle borders (`border`, `shadow-sm`)  
- neutral colour palette  
- consistent spacing scale (`p-4`, `p-6`, `gap-4`, `gap-6`)  

Inspired by:
- Vercel  
- Linear  
- Stripe  

---

## Layout Rules

Every page MUST:

- Use a centered container  
  `max-w-7xl mx-auto px-4 py-6`

- Use grid or flex layouts (no arbitrary positioning)

- Group content using:
  - Card  
  - Tabs  
  - Sections  

---

## Component Usage Rules

Prefer composition using:

- Card → grouping content  
- Table → structured data  
- Dialog / Sheet → interactions  
- Tabs → segmentation  
- DropdownMenu → actions  

Always include:

- loading states (skeletons)  
- empty states  
- basic error states  

---

## Interaction & UX

- Use icons where helpful (lucide)  
- Maintain clear hierarchy:
  - Title  
  - Subtext  
  - Content  
- Avoid clutter  
- Prefer fewer, well-grouped elements  

---

## Code Structure (Frontend)

- Extract reusable components:
  - PageHeader  
  - DataTable  
  - FormSection  

- Avoid monolithic pages  

---

# MCP USAGE

## shadcn MCP (web only)

Use when:
- adding components  
- searching registry  
- installing UI blocks  

---

## Serena (Use Sparingly)

Use ONLY when:
- modifying shared types/models  
- changing API contracts  
- refactoring across modules  

Otherwise skip.

---

## Context7

Use ONLY when:
- unsure of framework API  
- dealing with SSR/caching edge cases  

---

# ENGINEERING RULES

## Code Quality

- TypeScript strict mode  
- No `any`  
- ESLint + Prettier clean  
- Clear naming  
- Comments only when needed  

---

## Testing

- Unit tests → business logic  
- Integration tests → API  
- E2E → critical flows  

---

## API Standards

### Routes
- `/api/v1/[resource]`

### Success
```json
{ "success": true, "data": {} }

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/filipelagrenade)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/filipelagrenade)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
