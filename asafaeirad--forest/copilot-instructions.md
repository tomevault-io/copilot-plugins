## forest

> This is the requirements file for the current source code. It's an app Pomodoro-based focus timer that lets users "forge" progress in a personal journey — think crafting a skill, building a digital bonsai — the deeper they focus, the more they unlock. Gamification serves attention, not addiction.

# About
This is the requirements file for the current source code. It's an app Pomodoro-based focus timer that lets users "forge" progress in a personal journey — think crafting a skill, building a digital bonsai — the deeper they focus, the more they unlock. Gamification serves attention, not addiction.

# Instructions

During your interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in [requirements.mdc](mdc:.cursor/rules/requirements.mdc) file so you will not make the same mistake again. 

The Requirements lists features you want to work on, the [x] means it's implemented, [ ] means you should implement, [n] means it's planned for next iteration, we need to work on it.

You should also use [requirements.mdc](mdc:.cursor/rules/requirements.mdc) file as a Requirements Scratchpad to organize your thoughts. Especially when you receive a new task, you should first review the content of the Requirements, clear old different task if necessary, first explain the task, and plan the steps you need to take to complete the task as a child of the main task. You can use todo markers to indicate the progress, e.g.
- [ ] Main Task
  - [X] My Task 1
  - [ ] My Task 2
  - [n] My Task 3

Also update the progress of the task in the Scratchpad when you finish a subtask.
Especially when you finished a milestone, it will help to improve your depth of task accomplishment to use the Scratchpad to reflect and plan.
The goal is to help you maintain a big picture as well as the progress of the task. Always refer to the Scratchpad when you plan the next step.

# Tools

- React
- Vite
- TypeScript
- ESLint (Linter)
- Vitest (Unit test)
- UnoCSS (Style Solution)
- PNPM (Package Manager)
- Radix (Headless UI Library)
- Storybook (Component test, A11y test)

# Lessons

## User Specified Lessons

- Use this template for stories
- Consider Accessibility
- Consider keyboard shortcuts

```typescript
import type { Meta, StoryObj } from '@storybook/react';

import { App } from './App';

const meta: Meta<typeof App> = {
  component: App,
};

export default meta;

type Story = StoryObj<typeof meta>;

export const Default: Story = {};
```

## Cursor learned

- In React 19, `forwardRef` is removed. Instead of using `forwardRef`, pass the ref directly to the underlying component. This simplifies component creation and reduces boilerplate code.
- Keep stories simple by not using `tags` and `title` properties in the meta object. The component name and file structure will provide sufficient organization.
- Use Tailwind variants with `clsx` and `tailwind-merge` for better class composition and maintainability. This helps keep component styles consistent and makes it easier to update styles across the application.
- Keep component variants in the same file as the component for better encapsulation and maintainability. This makes each component more self-contained and easier to understand.
- Use the `tailwind-variants` library for better variant management. It provides a more robust API for handling variants, default variants, and compound variants, with better TypeScript support and runtime performance.

# Requirements:
- [ ] Welcome screen
  - [ ] In this screen we want to have a input to set intention and a button to go to the focus mode.
- [n] Pomodoro timer
- [n] Session complete screen (Focused/Distracted input)
- [n] Garden item grows per session
- [n] Pomodoro timer configuration
- [n] Basic XP/streak system
- [n] Settings page
- [n] Focus Mode Noises.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ASafaeirad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
