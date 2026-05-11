## ml-outliers

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ML Outliers** Most people use ML Models like "Black Box" —they throw data in, get a result, and hope it’s right. Our vision is to make learners peer inside the box.

We want to build the place where you go to develop your "Data Sense." Through our platform, they should be able to look at a messy data and immediately say, "I know exactly which model to use, how to use and why to use?."

**Core Philosophy**: "Syntax is easy, Intuition is hard." The goal is to develop "Data Sense" - the ability to look at messy data and immediately identify why a model will fail in production.

## Project Vision: Exemplary Architecture

This project is specifically engineered to demonstrate **SYSTEM DESIGN**, **MODULARITY**, **SCALABILITY**, and **DEPLOYABILITY** - the hallmarks of senior engineering excellence.

**Key Architectural Goals:**
- **System Design**: Open-Closed Principle - open for extension, closed for modification
- **Modularity**: Each challenge is a self-contained package with zero coupling
- **Scalability**: Architecture supports 100+ challenges without refactoring
- **Deployability**: Production-ready patterns with clean separation of concerns

## Development Commands

```bash
# Development server (runs on http://localhost:3000)
npm run dev

# Production build
npm run build

# Production server
npm start

# Linting
npm run lint
```

## Architecture: The Registry Pattern

This codebase uses a **Modular Registry Architecture** to ensure scalability to 100+ challenges without code coupling.

### Core Structure

```
/src
├── /app/solve/[id]        # Dynamic route shell that pulls problems from Registry
├── /problems              # THE CORE ENGINE
│   ├── index.js           # THE REGISTRY - maps IDs to Problem Modules
│   └── /[problem-slug]    # Self-contained problem modules
│       ├── info.js        # Metadata: title, expert answer, hints
│       ├── Visual.jsx     # Custom animated visualization (Framer Motion)
│       └── data.json      # Rigged dataset specific to this challenge
├── /api/judge             # Next.js Route Handler
│   └── route.js           # Uses Gemini API as semantic judge
└── /components/shared     # Reusable UI components
```

### How the Registry Works

1. User navigates to `/solve/[id]` (e.g., `/solve/log-transform`)
2. Dynamic route reads the `id` parameter
3. Registry (`/problems/index.js`) maps `id` → Problem Module
4. Problem Module provides: `Visual.jsx`, `data.json`, and `info.js`
5. User submits diagnosis → Judge compares with expert answer

**Key Principle**: Each problem is completely isolated. Adding complex animations or custom logic for one challenge does not affect others.

### Why This Architecture Matters

**Encapsulation**: Each problem module is isolated. A complex animation or unique data transformation in one challenge doesn't bloat the global bundle or risk breaking other challenges.

**Zero Coupling**: Problems can be added, modified, or removed without touching shared code. Teams can contribute asynchronously by following the folder template.

**Scalability**: The registry acts as a plugin system. Adding the 100th challenge is as simple as adding the first.

**Testability**: Each module can be tested in isolation. Integration tests only validate the registry mapping.

## Creating a New Challenge

When building a new challenge (e.g., "multicollinearity-hunt"):

1. **Create folder**: `/src/problems/multicollinearity-hunt/`
2. **Rig the data**: Create `data.json` with pathological patterns (e.g., two 99% correlated features)
3. **Build visualization**: Create `Visual.jsx` with appropriate plot (heatmap, scatter plot, etc.)
4. **Define expert answer**: In `info.js`, write the conceptual answer describing what the user should identify
5. **Register it**: Import and add to `/src/problems/index.js`

**Important**: Each challenge must be self-contained with its own data, visualization, and validation logic.

## AI Integration Strategy

The Gemini API is used as a **Semantic Judge**, not a code generator. It evaluates whether a user's conceptual reasoning matches the ground-truth expert answer, allowing for variations in phrasing while checking for understanding.

## Technical Stack

- **Next.js 16** (App Router)
- **React 19** with React Compiler enabled
- **Framer Motion** for animations (visualizations)
- **Gemini API** for semantic evaluation

## Key Configuration

- React Compiler is enabled (`reactCompiler: true` in `next.config.mjs`)
- ESLint uses Next.js core-web-vitals configuration
- Uses Geist font family (Sans and Mono variants)

## Design Principles for Development

When working in this codebase, always maintain these architectural principles:

### 1. Never Break Encapsulation
- Challenge-specific logic belongs ONLY in `/problems/[challenge-slug]/`
- Shared UI components go in `/components/shared/`
- Never create dependencies between problem modules

### 2. Registry Pattern Integrity
- The Registry (`/problems/index.js`) is the ONLY file that knows about all problems
- Problem modules export a standard interface (Visual, data, info)
- Dynamic routes consume problems through the Registry, never direct imports

### 3. Semantic Versioning for Modules
- Each problem module can evolve independently
- Breaking changes to the problem interface require Registry updates
- Maintain backward compatibility when possible

### 4. Code Splitting & Performance
- Each problem's Visual component is lazy-loaded
- Problem data files are loaded on-demand, not bundled globally
- This ensures fast initial page loads even with 100+ challenges

### 5. Deployment Readiness
- All API routes follow Next.js Route Handler conventions
- Environment variables for API keys (Gemini) must be configured
- Static exports possible for problem metadata (for CDN caching)

## Interview Talking Points

When discussing this project in technical interviews, emphasize:

1. **Strategic Architecture**: "I used the Registry Pattern to ensure the codebase could scale from 10 to 100+ challenges without refactoring. Each challenge is a plugin."

2. **Production Mindset**: "The modular design allows for independent deployment pipelines. Problems could be contributed via separate PRs and merged without conflicts."

3. **AI Integration Strategy**: "Rather than using LLMs for code generation, I used Gemini as a semantic judge - validating conceptual understanding against expert answers, allowing for phrasing variations."

4. **Performance Optimization**: "Dynamic imports and code splitting ensure users only download the code for challenges they access. The 100th challenge doesn't slow down the first."

---
> Source: [PriyamJChakrabarty/ML-Outliers](https://github.com/PriyamJChakrabarty/ML-Outliers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
