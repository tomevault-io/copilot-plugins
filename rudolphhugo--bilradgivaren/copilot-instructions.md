## bilradgivaren

> These rules apply to all work in this repository. Follow them without exception.

# Project Rules for Claude

These rules apply to all work in this repository. Follow them without exception.

---

## ⚠️ TEMPLATE INTEGRITY GUARD — READ THIS FIRST

**This repository is a clean starter template. It is not a project.**

Its purpose is to be copied as a starting point. The stack is fixed and intentional. You are working on the template itself, not a product built from it.

### The stack is:

| Layer | Technology |
|---|---|
| Framework | **Next.js 15** (App Router) |
| Styling | **Tailwind CSS v4** |
| Components | **shadcn/ui** |
| Icons | **Lucide React** |
| Language | **TypeScript** |

### Stack replacement rule

If the user asks you to replace any part of the core stack — you **must stop and say this clearly before doing anything**:

> "You're about to change a foundational part of this template. Specifically: [what they're replacing] → [what they're replacing it with]. This would change the template for all future projects copied from it. Are you sure this is what you intend, or are you working in the wrong repo?"

**You may only proceed if the user explicitly confirms** they understand they are modifying the template itself and still want to continue. One clear confirmation is enough — do not keep blocking them after that.

---

## 1. Plan Before Coding

**Always present a plan and wait for confirmation before writing any code.**

- Before starting any non-trivial task, outline your approach: what files you'll touch, what you'll create, and why.
- Ask clarifying questions upfront if anything is ambiguous. Offer a **"Skip for now"** option so the user can proceed if they feel the context is clear enough.
- Once the plan is confirmed, begin execution. Do not deviate from the confirmed plan without flagging it.
- "Plan means stop" — presenting a plan is not permission to start. Wait for explicit approval.

---

## 2. Always Use the Existing Stack

**This project uses Next.js 15, shadcn/ui, Lucide React, and Tailwind v4. Use them.**

- When building UI, always reach for shadcn/ui components first — even when the user hasn't specified which components to use.
- If the required shadcn/ui components are not yet installed, ask before adding:
  > "These components are needed: `[list]`. Should I install them with `npx shadcn@latest add`?"
- Offer a "code from scratch" fallback only as an option — **never build components from scratch without explicit confirmation**.
- Use Lucide React for all iconography unless the user specifies otherwise.
- Use Tailwind v4 utility classes for all styling — no inline styles, no CSS modules unless specifically requested.
- All theme tokens live in `app/globals.css` under `@theme {}` — do not use a `tailwind.config.ts`.

---

## 3. Conserve Tokens — Ask Before Sweeping the Codebase

**Never perform broad codebase searches without checking context first.**

- Before running wide searches (e.g., grepping the whole repo, reading multiple files speculatively), pause and ask:
  > "I'd need to scan [scope] to answer this. Should I proceed, or can you point me to the relevant area?"
- If the answer is likely already in the conversation context or in a file already read, use that — don't re-fetch.
- Prefer targeted, specific reads over broad sweeps. Fewer tokens = faster, cheaper, better.
- When in doubt: ask, don't assume.

---

## 4. Capture New Context

**If you learn something important about this project during a session, ask if it should be saved.**

- When you discover a pattern, decision, constraint, or convention that would help future sessions, flag it:
  > "I've learned [X] about this project. Should I add it to CLAUDE.md?"
- Do not silently discard useful context. Surface it and let the user decide.
- Keep additions concise — one rule or note per thing learned, no padding.

---
> Source: [rudolphhugo/bilradgivaren](https://github.com/rudolphhugo/bilradgivaren) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
