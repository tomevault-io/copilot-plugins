## pet-registry

> 1) Check the documentation.md and project.md files first to see if the changes have already been made.

## GENERAL RULES (THESE ARE THE MOST IMPORTANT TO FOLLOW):

1) Check the documentation.md and project.md files first to see if the changes have already been made.
2) When you have completed a task, update project.md and documentation.md files with completed tasks and new learnings.
3) All changes you propose should be concise and backwards compatible.
4) Do make concise, targeted changes.
5) Do not make sweeping changes.
6) Do not propose changes to parts of the code that are unrelated to the feature we are working on.
7) Preserve the existing functionality when refactoring.

---

## ACCESSIBILITY RULES:

1) Leverage native HTML elements when possible (e.g., <button>, <header>, <nav>) instead of divs or spans for meaningful content.
2) Ensure interactive elements like buttons and inputs have descriptive labels using aria-label, aria-labelledby, or the label element.
3) Make sure all interactive elements are keyboard-navigable by setting proper tabIndex and avoiding tabIndex values above 0.
4) Use alt attributes on images to describe their purpose or mark them with alt="" if decorative.
5) Use <label> elements for inputs when possible and ensure each input has a clear description or placeholder text.
6) Apply ARIA attributes (role, aria-*) only when necessary, and prefer native HTML elements with built-in accessibility features.
- Use focus() and aria-live regions to manage focus and announce dynamic content changes.
- Ensure features can be triggered using both mouse and keyboard, avoiding reliance on hover-only actions.
7) Ensure modals trap focus within the dialog and allow exiting with the Escape key.
8) Use aria-live or aria-atomic attributes to announce updates to dynamic content like notifications.

---

## REACT RULES:

1) Ensure each component handles a single responsibility for better reusability and testability.
2) Prefer functional components with React Hooks over class components for cleaner and more modern code.
3) Leverage TypeScript to enforce type checking and improve code reliability.
4) Keep state management minimal within components and delegate complex state logic to contexts or Redux/Redux Toolkit.
5) Follow Composition Over Inheritance (Use component composition to share behavior and UI instead of inheritance).
6) Declare functions outside JSX to prevent unnecessary re-renders.
7) Optimize Rendering (Use React.memo or useMemo to prevent unnecessary re-renders and optimize performance).
8) Use meaningful and consistent naming conventions for easy maintenance and collaboration.
9) Keep business logic out of JSX by moving it to custom hooks or utility functions.
10) Use MaterialUI for styling solutions.
11) Provide default props and destructure props for cleaner and more maintainable code.
12) Write new tests, or update existing tests accordingly.

---

## RUBY ON RAILS RULES:
1) Leverage Rails’ "convention over configuration" philosophy for faster development and easier collaboration.
2) Avoid duplication by using partials, helpers, concerns, and service objects.
3) Securely handle input data in controllers with strong parameters to prevent mass-assignment vulnerabilities.
4) Keep controllers concise and move business logic to models or service objects.
5) Use Active Record validations to enforce data integrity before saving to the database.
6) Prevent N+1 query issues by using includes or preload for database queries involving associations.
7) Use Rails’ built-in protections against SQL injection, CSRF, and XSS, and secure sensitive data with Rails credentials or environment variables.
8) Use shallow nesting in routes to avoid overly complex URL structures and improve readability.
9) Implement caching strategies (e.g., fragment, page, or query caching) to improve app performance.
10) Write concise and efficient queries, use scopes, and avoid complex logic in callbacks.
11) Extract complex business logic into service objects or interactors for better organization.

---

## RSPEC TESTING RULES:

1) use rspec
2) do not use "let" syntax
3) do not use "before do" blocks
4) do not use "described_class" syntax
5) do employ contexts to describe conditions using “when,” “with,” or “without.”
6) do use factories (FactoryBot) rather than fixtures
7) do keep descriptions under 40 characters
8) do prefer single expectations in each example unless performance constraints require more
9) do stub external HTTP requests using the VCR library
10) do write clear, readable matchers (prefer expect to lambda).
11) do use .and_call_original when possible

---

## REACT TESTING RULES:
1) Do test all code using Vitest
2) Do focus on testing what the component does (e.g., renders correctly, responds to user interactions) rather than its internal implementation.
3) Do prefer React Testing Library over Enzyme for testing React components, as it encourages testing from the user's perspective.
4) Do test individual functions and logic separately to ensure reliability and isolate failures.
5) Do mock APIs, third-party libraries, and external state management to test components in isolation.
6) Do avoid dependencies between tests to ensure they can run independently and in any order.
7) Do write clear and descriptive test names to document expected behaviors.
8) do use tools like fireEvent or userEvent to test interactions, such as clicks, input changes, or form submissions.
9) Do include tests for edge cases (e.g., empty states, error handling, invalid inputs).
10) Do use snapshot tests sparingly to validate UI structure but rely on functional tests for dynamic behavior.
11) Do test for accessibility issues using tools like axe or toHaveAccessibleName.
12) Do use cleanup (automatically done in React Testing Library) to prevent memory leaks and ensure a fresh DOM for each test.
13) Do ensure critical paths are well-tested, but avoid redundant tests that make maintenance harder.
14) use "toBeDefined" instead of "toBeInTheDocument"
15) Do import functions like "beforeEffect" automatically when you decide to use them
16) Do use data-testids to identify elements if available to identify specific elements
17) Do not try to find objects by role "listbox"
18) Do not write tests for loggers

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mkrul) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
