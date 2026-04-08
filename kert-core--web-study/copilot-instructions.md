## web-study

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **web study repository** for 2025-2 KERT (경북대학교 멋쟁이 사자처럼), designed for a 90-minute JavaScript, TypeScript, and React fundamentals study session.

**Study Format:**
- Total duration: 1 hour 30 minutes
- Theory:Practice ratio: 4:6 (36 minutes theory, 54 minutes practice)
- Target audience: Web frontend development beginners

## Repository Structure

```
web-study/
├── README.md                           # Main study plan and curriculum
├── references/                         # Reference materials
│   └── 프론트엔드_2차시_JS.pdf        # Korean PDF reference material
├── session1-js-ts/                     # Session 1: JavaScript & TypeScript
│   ├── 01-variables-and-types.md
│   ├── 02-functions-and-arrays.md
│   ├── 03-async-programming.md
│   ├── 04-typescript-basics.md
│   └── exercises/                      # Practice exercises
└── session2-react/                     # Session 2: React
    ├── 01-react-fundamentals.md
    ├── 02-hooks-and-state.md
    ├── 03-api-integration.md
    ├── 04-todo-app-tutorial.md
    └── todo-app/                       # Complete Todo app code
```

## Study Session Architecture

### Session 1: JavaScript & TypeScript (45 minutes)

**1.1 JavaScript Core Concepts (20 min - 8 min theory, 12 min practice)**
- Variables (const, let, var, hoisting)
- Data types (primitive vs reference)
- Operators (arithmetic, comparison, logical, ternary)
- Template literals
- Arrow functions vs regular functions
- Array methods (map, filter, reduce, find, some, every, forEach)
- Object shorthand notation
- Destructuring
- Spread/rest operators
- Control flow (if-else, switch, for, while)
- Truthy/Falsy values and short-circuit evaluation
- Promise with then-catch
- async/await with try-catch
- Fetch API (GET, POST, PUT, DELETE)
- JSON methods (stringify, parse)
- Optional chaining (?.) and nullish coalescing (??)

**1.2 TypeScript Basics (15 min - 6 min theory, 9 min practice)**
- Basic types (string, number, boolean, array, object)
- Interfaces and type aliases
- Union types and literal types
- Generic basics
- Type guards and type assertions

**1.3 Comprehensive Practice (10 min)**
- Type-defined user data processing functions
- Async data fetching and transformation

### Session 2: React Basics (45 minutes)

**2.1 React Core Concepts (18 min - 8 min theory, 10 min practice)**
- Components and JSX syntax
- Props (parent-child data passing)
- State (useState)
- Event handling (onClick, onChange, etc.)
- Conditional rendering (&&, ternary operator)
- List rendering (map and key)
- useEffect (side effects)
- API communication (fetch vs axios)

**2.2 Practical Project: Todo List Application (27 min - 6 min theory, 21 min practice)**
- Features: add, toggle, delete, filter, LocalStorage persistence
- Concepts: useState, component separation, Props, useEffect, TypeScript types

## Content Creation Guidelines

When creating study materials for this repository:

### Markdown Files
1. **Structure**: Keep theory sections concise (few sentences) followed by practical code examples
2. **Code Examples**: Focus on real-world, practical examples over theoretical concepts
3. **Language**: Content should be in Korean for Korean students
4. **Time Allocation**: Maintain 4:6 theory-to-practice ratio as specified in README.md

### Code Examples
1. **JavaScript/TypeScript**:
   - Use modern ES6+ syntax (const/let, arrow functions, destructuring)
   - Include both correct examples and common mistakes to avoid
   - Reference PDF material in `/references/프론트엔드_2차시_JS.pdf` for consistency

2. **React**:
   - Use functional components with Hooks (not class components)
   - Always use TypeScript for React examples
   - Follow the React + TypeScript + Vite project structure

### Exercise Files
- Create practical, hands-on exercises that students can run immediately
- Include TODO comments to guide students
- Provide solution files or solution sections

## Development Environment Setup

When creating example projects (especially for Session 2):

```bash
# Create React + TypeScript project with Vite
npm create vite@latest [project-name] -- --template react-ts
cd [project-name]
npm install

# Install axios for API communication
npm install axios

# Run development server
npm run dev
```

## Key Reference Materials

- **Main Curriculum**: `/README.md` - Complete study plan with all topics and time allocations
- **PDF Reference**: `/references/프론트엔드_2차시_JS.pdf` - Original Korean teaching material covering:
  - Variables (var, let, const, hoisting)
  - Data types and operators
  - Control flow structures
  - Async programming (Promise, fetch, async/await)
  - React-specific JS patterns (arrow functions, map, filter, destructuring, spread, ternary operator)

## Content Organization Principles

1. **Progressive Difficulty**: Start with fundamentals, build to complex applications
2. **Theory Density**: Keep explanations brief (2-3 sentences per concept)
3. **Code-First**: Emphasize working code examples over lengthy explanations
4. **Practical Focus**: Every concept should have immediate practical application
5. **Time-Boxed**: Each section has strict time limits - content must fit within allocated time

## TODO List Management

When creating content for this study:
- Session 1 materials are planned but not yet created
- Session 2 materials are planned but not yet created
- Todo app project code needs to be implemented
- Exercise files need to be created with solutions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KERT-core)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KERT-core)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
