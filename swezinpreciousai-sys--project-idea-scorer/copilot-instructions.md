## project-idea-scorer

> Build a single-page application that helps users score project ideas before building them.

## Project Name

Project-Idea-Scorer

## Project Goal

Build a single-page application that helps users score project ideas before building them.

The app should be beginner-friendly, clean, responsive, and deployable on Vercel.

## Tech Stack

* Next.js
* React
* Tailwind CSS
* Vercel deployment
* No API
* No database
* Local state only

## Important Rules

* Do not create backend API routes.
* Do not add database integration.
* Do not add authentication.
* Do not use external paid services.
* Keep all data in React local state.
* Keep the project simple and beginner-friendly.
* Use reusable components.
* Use clean Tailwind CSS styling.
* Make the page mobile responsive.
* Keep code easy for a new learner to understand.

## App Description

The user enters a project idea and manually scores it using 5 criteria.

The app calculates the total score and shows a recommendation.

## Scoring Criteria

Use these 5 criteria:

1. Real User Problem
2. AI Value
3. Scope
4. Market Demand
5. Team Excitement

Each criterion should have a score from 1 to 10.

Maximum total score: 50

## Score Result Logic

Use this logic:

* 40–50: Excellent idea
* 30–39: Good MVP idea
* 20–29: Needs improvement
* Below 20: Not recommended yet

## Required Page Sections

### 1. Hero Section

Show:

* App title: AI Project Idea Scorer
* Subtitle: Score your project idea before you build it
* Short description
* Start button or scroll button

### 2. Idea Input Form

Fields:

* Project Idea
* Target Users
* Problem
* AI Usage
* Team Size
* Build Time

### 3. Scoring Section

Use sliders or number inputs for:

* Real User Problem
* AI Value
* Scope
* Market Demand
* Team Excitement

Each field should show the selected score.

### 4. Result Section

After clicking "Score My Idea", show:

* Project idea title
* Total score
* Status badge
* Score breakdown
* Recommendation text
* MVP suggestion
* Risk warning
* Next step

### 5. Sample Ideas Section

Add sample idea buttons:

* AI resume checker for job seekers
* Hospital visitor appointment guide
* Yangon delivery assistant
* AI interview practice app
* Waste collection mapping system

When the user clicks a sample idea, auto-fill the form.

## UI Style

Use a modern dashboard style.

Design direction:

* Light background
* White cards
* Rounded corners
* Soft shadow
* Blue or purple accent color
* Clean spacing
* Mobile-first layout

## Suggested Components

Create components if useful:

* HeroSection
* IdeaForm
* ScoreSlider
* ResultCard
* SampleIdeas
* Footer

## Beginner-Friendly Coding Rules

* Use clear variable names.
* Add simple comments where useful.
* Avoid over-engineering.
* Avoid complex state management.
* Use `useState` only.
* Keep everything understandable.

## Deployment

The project must work on Vercel.

Before deployment, make sure:

```bash
npm run build
```

passes successfully.

## GSD MCP Usage

Use GSD MCP to break the project into small tasks.

Recommended task flow:

1. Create Next.js project
2. Install Tailwind CSS
3. Build static layout
4. Build input form
5. Add scoring sliders
6. Add score calculation logic
7. Build result card
8. Add sample idea buttons
9. Improve responsive design
10. Test locally
11. Deploy to Vercel

## Local Development Commands

Create project:

```bash
npx create-next-app@latest ai-project-idea-scorer
```

Recommended choices:

```text
TypeScript: Yes
ESLint: Yes
Tailwind CSS: Yes
src directory: Yes
App Router: Yes
Turbopack: Yes
Import alias: Yes
```

Run project:

```bash
cd ai-project-idea-scorer
npm run dev
```

Build project:

```bash
npm run build
```

## Claude Code Instruction

When helping with this project:

* Follow this CLAUDE.md file.
* Build only a frontend single-page app.
* Do not add backend, API, database, or login.
* Keep the code simple.
* Explain changes clearly for a beginner.
* Use Next.js App Router.
* Use Tailwind CSS for styling.
* Make the final app ready for Vercel deployment.

## Final Expected Result

A polished one-page web app where users can:

1. Enter a project idea
2. Score the idea
3. View total score
4. Understand whether the idea is worth building
5. See MVP suggestion, risk, and next step

---
> Source: [swezinpreciousai-sys/Project-Idea-Scorer](https://github.com/swezinpreciousai-sys/Project-Idea-Scorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
