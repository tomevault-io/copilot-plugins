## easy-code-lab

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## Primary Objective

**The #1 goal of EasyCode is to teach and ensure students grasp the basics and fundamentals of web development.** Every feature, lesson, and UI decision must serve this objective. Content should not just show syntax — it should explain *why* things work the way they do, connect concepts to real-world browser behavior, and reinforce web standards conformance. When in doubt, prioritize understanding over coverage.

### Content Formatting

- **Use HTML, never Markdown, in lesson content strings** (`body`, `analogy`, `infoBoxes[].body`). Since EasyCode teaches HTML, the content itself must use HTML tags for formatting — `<strong>`, `<em>`, `<code>`, `<ul>/<li>`, `<ol>/<li>`, `<br>`, etc. Never use Markdown syntax (`**bold**`, `*italic*`, `` `code` ``, `- list item`). The InstructionPanel and InfoBox components render these strings as HTML.

### Content Guidelines

- **Explain why it exists**: Every concept should include its purpose — *why* was it created, *what problem* does it solve? Don't just show how to use a tag; explain why the web needs it.
- **Give real-world examples**: Connect concepts to websites and apps students already know (Google, YouTube, Wikipedia, Amazon, Instagram, etc.). Students should recognize these patterns in the wild.
- **Make it relatable**: Use analogies drawn from everyday life — school, shipping packages, concerts, phone apps — so abstract concepts feel familiar before the technical explanation.
- **Include doc links**: Reference MDN Web Docs for every HTML element, attribute, and CSS property. Conform to web standards — teach the *right* way from the start.
- **Add info boxes**: Use "Web Standard" boxes for accessibility, spec conformance, and browser behavior. Use "Tip" boxes for common mistakes and practical gotchas — placed on practice steps where students are most likely to hit the issue.
- **Include a full example lesson**: Every module should end with a capstone lesson that combines all taught concepts into building something recognizable and complete. Students should walk away having built a real thing, not just practiced isolated skills.
- **Include codelabs**: Every module must include step-by-step codelab lessons that guide students through building real projects in a code editor and browser. Codelabs bridge the gap between the interactive tool and real-world development. Assume Linux, no build tools (just HTML/CSS files opened in a browser). Don't assume prior knowledge — explain every terminal command and editor action. End each codelab with a free-edit checkpoint. Progressive difficulty: start simple, build on previous codelabs.

## Overview

EasyCode is an interactive, browser-based learning tool that teaches HTML and CSS to complete beginners through visual, hands-on exercises. No backend — all progress is persisted client-side via IndexedDB.

## Tech Stack

- **Framework:** Next.js 16 (App Router, React Compiler enabled)
- **UI:** React 19, Tailwind CSS v4 (inline `@theme`), Motion (`motion/react`)
- **State:** Zustand stores (`src/stores/`)
- **Code Editor:** CodeMirror 6 (`@uiw/react-codemirror`)
- **3D:** Three.js + React Three Fiber + Drei (box-model module only)
- **Persistence:** IndexedDB via `idb` wrapper
- **Drag & Drop:** `@dnd-kit/core` + `@dnd-kit/sortable`

## Architecture

- App Router with `src/` directory structure
- Route group `(modules)` for learning module pages
- All interactive components are client components (`"use client"`)
- Lesson content defined as typed TypeScript objects in `src/content/`
- Custom hooks in `src/hooks/` for lesson progress, code validation, IndexedDB

## Component Organization

```text
src/components/
  ui/          — Primitives: Button, Card, Slider, ProgressIndicator, StepDots, Tooltip
  layout/      — Header, InstructionPanel, LessonStepper, SplitPane
  editor/      — CodeEditor, GapFillEditor, HtmlPreview, SplitPane
  tag-builder/ — TagBuilderLesson, TagVisualizer
  box-model/   — BoxModelLesson, BoxModelCanvas, BoxModel2D, BoxLayer, ValueControls, TargetChallenge, BoxModelCodePanel
```

## Styling

- **Tailwind CSS v4** with design tokens defined via `@theme inline` in `src/app/globals.css`
- No CSS modules — utility-first approach only
- Animations via `motion/react` (Framer Motion)
- Custom keyframe animations in `globals.css` for domain-specific effects
- Fonts: Geist Sans + Geist Mono (loaded via `next/font/google` in root layout)
- HeroUI-inspired visual design: gradient accents (blue-to-violet), generous rounding, glassmorphism header, cool neutral palette

## Code Style

- TypeScript strict mode
- Types defined in `src/types/` (lesson.ts, tag-builder.ts, box-model.ts)
- Prefer named exports for types, default exports for components
- Keep component interfaces stable — avoid breaking prop changes

## Testing

- No test framework currently configured
- Validation logic lives in `src/lib/` (html-parser, tag-validator, lesson-engine, box-model-calc)

---
> Source: [mwakidenis/Easy-Code-Lab](https://github.com/mwakidenis/Easy-Code-Lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
