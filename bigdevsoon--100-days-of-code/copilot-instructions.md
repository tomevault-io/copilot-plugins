## 100-days-of-code

> This repository is structured for AI-assisted development. Each of the 100 challenges provides the exact context coding agents need - design mockups, user stories, and acceptance criteria.

# AI Agent Guide

This repository is structured for AI-assisted development. Each of the 100 challenges provides the exact context coding agents need - design mockups, user stories, and acceptance criteria.

## Challenge Structure

Every challenge folder (`day-01` through `day-100`) contains:

```
day-XX/
  README.md    - User story, acceptance criteria, tips
  design.png   - Professional design mockup to replicate
  index.html   - Starter HTML file
```

The README includes:
- **User Story** - Who the user is and what they need
- **Acceptance Criteria** - Specific, testable requirements
- **Design Preview** - Screenshot of the target UI

## How to Use with AI Agents

### Provide Full Context

When starting a challenge, give your AI agent:
1. The user story and acceptance criteria from the README
2. The design mockup image (if your agent supports vision)
3. Your preferred tech stack (React, Vue, vanilla JS, etc.)

**Example prompt:**
```
I'm building the "Profile Card" challenge. Here are the requirements:

User Story: As a user, I want a profile card that showcases my professional
details and personality, making it easy for viewers to connect with me.

Acceptance Criteria:
- Profile image in a dynamically shaped frame
- Name, username, and professional bio displayed clearly
- Functional social media icons
- Responsive Follow and Message buttons
- Works across different screen sizes

Build this with React and Tailwind CSS. Start with the component structure.
```

### Iterate, Don't Generate Everything at Once

Break the work into steps:
1. Component structure and layout
2. Styling to match the design
3. Interactivity and state management
4. Responsive design adjustments
5. Accessibility and edge cases

### Validate Against Acceptance Criteria

After building, have your agent review each criterion:
```
Check our implementation against each acceptance criterion.
What's missing or incomplete?
```

## Challenge Categories

**Components** (Days 1-20) - Cards, forms, navigation, carousels. Good for learning layout and styling.

**Interactive UI** (Days 21-50) - Dashboards, planners, media players. State management and user interaction.

**Complex Layouts** (Days 51-80) - Multi-section pages, e-commerce, data displays. Responsive design and composition.

**Advanced Patterns** (Days 81-100) - Real-time features, data visualization, full applications. Architecture decisions.

## Tips for Better Results

- **Be specific about tech stack** - "React 18 with TypeScript" beats "make a web app"
- **Reference the design** - Describe colors, spacing, and layout if your agent can't see images
- **Ask for explanations** - "Why did you use flexbox here instead of grid?" helps you learn
- **Request tests** - "Write unit tests for the tip calculation logic" ensures correctness
- **Iterate on accessibility** - "Add ARIA labels and keyboard navigation" after the initial build

## BigDevSoon Integration

These challenges are also available on [BigDevSoon](https://bigdevsoon.me) with **Merlin AI** - a coding assistant built into the editor that understands your full project context. Merlin helps you vibe code through challenges with context-aware suggestions, code generation, and real-time guidance.

Try it with a [7-day free trial](https://app.bigdevsoon.me) or use the [free Demo Editor](https://app.bigdevsoon.me/demo/custom/browser) to get started.

---
> Source: [bigdevsoon/100-days-of-code](https://github.com/bigdevsoon/100-days-of-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
