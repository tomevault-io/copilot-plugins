## patient-registration-ui

> # Patient Registration UI - Code Style Guide


# Patient Registration UI - Code Style Guide

## 🚀 Project Overview

This project is a modern, responsive patient registration web application built with React. The application provides an intuitive interface for patient registration, form handling, and data validation, with a focus on accessibility and user experience.

## 💻 Tech Stack

- **Core:** React 18.x (Web)
- **State Management:** React Query for server state, React Context API for global UI state, useState for local component state
- **Form Handling:** React Hook Form with Yup for validation
- **UI Library:** Material-UI (MUI) components with custom theme
- **Styling:** Emotion (CSS-in-JS) with MUI's styled API
- **API Interaction:** Axios with request/response interceptors
- **Routing:** React Router v6
- **Testing:** Jest, React Testing Library, and MSW for API mocking
- **Linting/Formatting:** ESLint, Prettier, and TypeScript for type safety
- **Build Tool:** Vite for fast development and production builds

## ✍️ Coding Style & Conventions

- **Language:** JavaScript (ES6+)
- **Component Structure:**
  - Prefer functional components with Hooks.
  - Keep components small and focused on a single responsibility.
  - Organize files by feature or component type (e.g., `src/components/`, `src/screens/`, `src/features/`).
- **Naming Conventions:**
  - Components: `PascalCase` (e.g., `UserProfile.js`, `SettingsScreen.tsx`)
  - Variables/Functions: `camelCase` (e.g., `WorkspaceUserData`, `isLoading`)
  - Constants: `UPPER_SNAKE_CASE` (e.g., `API_BASE_URL`)
- **Styling:**
  - **Web:** Standard CSS with component-specific CSS files
  - **Mobile (React Native):** React Native StyleSheet API with platform-specific styling where needed
  - Maintain consistent styling across platforms while respecting platform-specific UX conventions
- **State Management:**
  - Use React Context API for global state (auth, user data)
  - Use useState hook for local component state
  - Avoid prop drilling by leveraging Context where appropriate
- **API Calls:**
  - Centralize API call logic in services or hooks (e.g., `src/services/api.js` or `src/hooks/useApi.js`).
  - Handle loading states and errors gracefully.
- **Error Handling:**
  - Implement comprehensive error handling for API requests and user interactions.
  - Use try-catch blocks for asynchronous operations.
- **Comments:**
  - Write clear and concise comments for complex logic or non-obvious code.
  - Use JSDoc for function and component prop descriptions if using JavaScript

## 🌐 Web-Specific Considerations

- **Responsive Design:**
  - Use MUI's responsive utilities and breakpoints for all components
  - Test on mobile, tablet, and desktop viewports
  - Implement touch-friendly UI elements with appropriate sizing
- **Accessibility (a11y):**
  - Follow WCAG 2.1 AA standards
  - Use semantic HTML elements
  - Ensure proper keyboard navigation and focus management
  - Include ARIA attributes where necessary
  - Test with screen readers
- **Performance:**
  - Implement code splitting using React.lazy and Suspense
  - Optimize bundle size with tree-shaking
  - Use virtualization for long lists (react-window or react-virtualized)
  - Optimize images and assets
- **Browser Support:**
  - Support latest 2 versions of major browsers (Chrome, Firefox, Safari, Edge)
  - Include appropriate polyfills for IE11 if required

## ✅ Testing Guidelines

- **Unit Tests:** Write unit tests for individual functions, components, and hooks using Jest, React Testing Library.
- **Integration Tests:** Test interactions between components and workflow processes.
- **End-to-End (E2E) Tests (Mobile):** Use Detox for automated E2E testing on mobile simulators/devices.
- **Test Coverage:** Aim for 80%+ coverage on critical paths and business logic.
- **Test Organization:**
  - Co-locate test files with implementation files (e.g., `Component.js` and `Component.test.js`)
  - Group tests using `describe` blocks for logical organization
  - Use meaningful test descriptions that document the expected behavior

## 🧑‍🤝‍🧑 Collaboration & Git Workflow

- **Branching:** Feature branches from `main`, using GitHub Flow
  - Branch naming convention: `feature/feature-name` or `bugfix/issue-description`
- **Commits:**
  - Write clear and descriptive commit messages
  - Start with a verb in present tense (e.g., "Add patient registration form")
  - Reference issue numbers when applicable (e.g., "Fix form validation #42")
- **Pull Requests:**
  - All code changes must go through Pull Requests with at least one reviewer
  - Include screenshots for UI changes
  - Provide testing instructions
- **Code Reviews:**
  - Focus on constructive feedback regarding code quality and adherence to conventions
  - Use GitHub's suggestion feature for small changes
- **Documentation:**
  - Update documentation (README, JSDoc comments) as part of the PR process
  - Keep copilot-instructions.md updated with any new conventions or patterns

## ❌ Things to Avoid

- **Component Anti-patterns:**
  - Avoid large, monolithic components (keep components focused and small)
  - Avoid prop drilling (use Context or state management when appropriate)
  - Avoid direct DOM manipulation (use refs sparingly and correctly)
  - Don't use index as key in lists

- **State Management:**
  - Avoid overusing Context for state that's only needed locally
  - Don't store derived state that can be computed
  - Avoid deeply nested state objects

- **Performance Pitfalls:**
  - Don't forget to clean up event listeners and subscriptions
  - Avoid inline function definitions in render props
  - Be cautious with large dependencies
  - Don't ignore React's warning messages

- **Security:**
  - Never commit sensitive information (API keys, secrets) to version control
  - Sanitize all user inputs
  - Use environment variables for configuration
  - Implement proper CORS policies
  - Protect against XSS and CSRF attacks

- **Accessibility:**
  - Don't ignore a11y warnings
  - Avoid non-semantic elements when semantic ones are available
  - Don't rely solely on color to convey information
  - Ensure sufficient color contrast

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deepak-sekarbabu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
