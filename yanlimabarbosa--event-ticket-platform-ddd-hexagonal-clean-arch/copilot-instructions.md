## event-ticket-platform-ddd-hexagonal-clean-arch

> This project follows **DDD + Hexagonal + Clean Architecture** with **NestJS**, **MikroORM**, and **TypeScript**.

# Project Guide

This project follows **DDD + Hexagonal + Clean Architecture** with **NestJS**, **MikroORM**, and **TypeScript**.

## Reference Files

@TECHNICAL.md
@TEACH-ME.md

- **TECHNICAL.md** — A starting reference for architecture patterns. **NOT law** — use it as a base but always apply real industry best practices. If TECHNICAL.md says one thing but industry practice says another, go with industry practice and explain why.

- **TEACH-ME.md** — The teaching methodology. Defines HOW to teach: the learning loop, phases 0-4 progression, challenge difficulty curve, and mentoring rules.

## Session Continuity — CRITICAL

- **SESSION-PROGRESS.md** — Current phase, completed phases with key lessons, student profile, and what's next.

### Rules for every session:
1. **At the start of every new session:** Read `SESSION-PROGRESS.md` FIRST to understand where we left off. Resume from the exact phase/step indicated.
2. **During the session:** When the student completes a phase, sub-phase, or makes significant progress — update `SESSION-PROGRESS.md` immediately. Don't wait until the end.
3. **At the end of every session** (when the student says goodbye, switches PCs, or the conversation is ending): Update `SESSION-PROGRESS.md` with:
   - Current phase and exact step
   - Key lessons learned in this session
   - Any corrections or "aha moments" the student had
   - Updated student profile if new patterns emerged (learning style, pace, struggles)
   - What's next (specific enough that the next session can resume without guessing)
4. **Student profile section** is important — it captures HOW this student learns best, what confuses them, what teaching approaches work. Update it as you learn more about the student.
5. **Never lose detail.** Each completed phase should have enough context that a new session can reference past decisions and explain WHY they were made if the student asks.

## Domain Discovery

- **DOMAIN.md** — All project-specific decisions: ubiquitous language glossary, bounded contexts, aggregate mappings, invariants, domain events, database schema, and API design. This is the living document for the Event Ticketing platform. Everything we discover goes here.

## Thinking Guides

The `how-to-map-requirements/` folder contains reusable thinking guides that teach HOW to approach domain discovery for any system. These are learning artifacts — not project-specific docs.

- **01-UBIQUITOUS-LANGUAGE-GUIDE.md** — How to discover business vocabulary: identifying roles (not "users"), replacing technical terms with business terms, naming actions with business verbs, and building a glossary.
- **02-BOUNDED-CONTEXT-GUIDE.md** — How to find system boundaries: spotting the same word with different meanings, grouping by who cares and rate of change, defining communication through events, and identifying the core domain.
- **03-AGGREGATE-MAPPING-GUIDE.md** — How to map aggregates: finding the root, identifying children and value objects, discovering invariants (using "Can it...?", "What if...?", "Who enforces this?" techniques), mapping state transitions, and finding domain events.
- **04-DATABASE-DESIGN-GUIDE.md** — How to design the database schema from the domain model: identifying tables, defining columns (identity, FKs, state, data, timestamps), relationships, denormalization decisions, indexing strategy, and common mistakes (DECIMAL for money, never delete for state changes).
- **05-API-DESIGN-GUIDE.md** — How to design REST APIs: listing user actions, choosing HTTP methods (POST for actions, PATCH for field updates), URL patterns (nouns not verbs), request bodies (minimum data), responses (full resource with human-readable data), status codes (400 vs 409), error format, and pagination.

**Important:** When creating new guides for the student, always place them in this folder with a numbered prefix to indicate the chronological order of the steps. These guides must be system-agnostic — they teach the thinking process, not this specific project's answers.

## Memories

The `.claude-memories/` folder contains persistent memories that MUST be read at the start of every session. These travel with the repo via git.

- **MEMORY.md** — Index of all memory files
- **user_learning_profile.md** — How this student learns, preferences, session continuity needs
- **feedback_production_grade.md** — Always use production tools, never toy setups
- **feedback_technical_is_base.md** — TECHNICAL.md is a base, not law — apply real industry practices
- **feedback_explicit_modifiers.md** — Always write public/private/protected, override, and return types explicitly
- **project_avenir_refactor.md** — Student wants to refactor Avenir's mesa virtual module with DDD later
- **feedback_dont_push.md** — Let the student explore questions at their own pace, don't rush to next task

### Memory sync rules:
1. **`.claude-memories/` is the source of truth** — it travels with the repo via git and works across different PCs.
2. When creating or updating memories, **always write to `.claude-memories/` first**, then sync to `~/.claude/projects/.../memory/`.
3. **At the start of every session:** Compare both MEMORY.md indexes. If `.claude-memories/` has memories that the local `~/.claude/projects/.../memory/` doesn't, copy them over. The project folder should always win since it's shared across PCs.
4. Never rely solely on the local Claude memory — always keep `.claude-memories/` up to date as the primary store.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yanlimabarbosa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
