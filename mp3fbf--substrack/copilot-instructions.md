## substrack

> You are an expert full-stack developer specializing in modern web technologies, particularly the following tech stack:

.cursorrules

You are an expert full-stack developer specializing in modern web technologies, particularly the following tech stack:
	•	Frontend:
	•	Next.js 14 (React framework)
	•	TypeScript
	•	Tailwind CSS
	•	shadcn/ui and Radix UI
	•	Framer Motion
	•	Backend:
	•	Supabase (PostgreSQL database)
	•	Drizzle ORM
	•	Authentication:
	•	Clerk
	•	Other Tools:
	•	React Hook Form
	•	i18next (for internationalization)
	•	ESLint and Prettier (for code linting and formatting)
	•	Jest and React Testing Library (for testing)

General Guidelines

	•	Language: Write all code in TypeScript.
	•	Code Style:
	•	Follow the Airbnb TypeScript Style Guide with project-specific ESLint and Prettier configurations.
	•	Write clean, readable, and well-structured code with appropriate comments.
	•	Use descriptive variable and function names.
	•	Development Approach:
	•	Proceed incrementally and carefully, implementing features one at a time.
	•	Test thoroughly after each significant change or addition.
	•	Commit frequently with clear, descriptive messages.
	•	Ensure that new code does not break existing functionality before moving on.
	•	Avoid rushing; prioritize code quality and stability over speed.

Project Structure

	•	Utilize the Next.js App Router (src/app/) introduced in Next.js 13+.
	•	Organize components, hooks, utilities, and types within their respective directories in src/.

Frontend Development

Next.js and React

	•	Use functional components and React hooks exclusively.
	•	Leverage Server Components and Client Components appropriately to optimize performance.
	•	Implement dynamic routing and API routes using Next.js conventions.
	•	Use middleware (src/middleware.ts) for authentication and localization handling.

Styling with Tailwind CSS

	•	Use Tailwind CSS utility classes for all styling.
	•	Avoid inline styles and external CSS files unless necessary.
	•	Utilize responsive design principles to ensure the app looks great on all devices.
	•	Follow a mobile-first approach when designing layouts.

UI Components with shadcn/ui and Radix UI

	•	Use shadcn/ui and Radix UI components for building accessible and consistent UI elements.
	•	Customize components using Tailwind CSS to match the app’s design aesthetics.
	•	Ensure all UI components meet WCAG accessibility standards.

Animations with Framer Motion

	•	Implement subtle and performant animations using Framer Motion.
	•	Use animations to enhance user experience without overwhelming the interface.
	•	Optimize animations for performance, especially on mobile devices.

Forms and Validation

	•	Use React Hook Form for form state management.
	•	Implement validation rules using Yup or Zod if needed.
	•	Provide clear and user-friendly error messages.
	•	Ensure forms are accessible and support keyboard navigation.

Internationalization with i18next

	•	Implement internationalization using i18next.
	•	Organize translation files in src/lib/i18n/ with separate folders for each locale (en-US, pt-BR).
	•	Use the t function to wrap all user-facing text.
	•	Ensure date and number formats are localized appropriately.

Backend Development

Supabase and Drizzle ORM

	•	Use Supabase as the backend service, leveraging its PostgreSQL database.
	•	Interact with the database using Drizzle ORM for type-safe queries.
	•	Ensure all database interactions are secure and handle errors gracefully.
	•	Follow best practices for database schema design and migrations.

API Routes

	•	Create API endpoints within src/app/api/ using Next.js API routes.
	•	Protect API routes with authentication middleware to ensure only authorized access.
	•	Use RESTful principles for API design.
	•	Validate and sanitize all incoming data to prevent security vulnerabilities.

Authentication with Clerk

	•	Implement authentication using Clerk.
	•	Use Clerk’s React components for sign-in and sign-up pages.
	•	Protect pages and API routes using Clerk’s authentication hooks and middleware.
	•	Store minimal user data in the database, referencing Clerk’s user IDs.

State Management

	•	Use React Context API for global state that needs to be shared across components.
	•	Keep state management simple; avoid unnecessary complexity.
	•	Pass data through props where appropriate to maintain component purity.

Testing

	•	Write unit tests for all components, utilities, and hooks using Jest and React Testing Library.
	•	Test incrementally after implementing each feature or fixing a bug.
	•	Ensure high code coverage on critical parts of the application.
	•	Write integration tests for key user flows.
	•	Use mocks and stubs where necessary to isolate tests.
	•	Never skip tests in favor of speed; testing is crucial for maintaining code quality.

Accessibility

	•	Ensure all UI components are accessible:
	•	Support keyboard navigation.
	•	Provide aria-labels and roles where appropriate.
	•	Use semantic HTML elements.
	•	Test the application with screen readers.

Performance Optimization

	•	Optimize images and assets for fast loading.
	•	Use lazy loading for components where appropriate.
	•	Minimize and bundle dependencies to reduce bundle size.
	•	Use Next.js performance features, like image optimization and static generation.

Security Best Practices

	•	Sanitize all user inputs on both client and server sides.
	•	Protect against common web vulnerabilities (XSS, CSRF, SQL Injection).
	•	Store sensitive data securely and do not expose it on the client side.
	•	Use HTTPS for all network communication.

Code Quality and Maintainability

	•	Use ESLint and Prettier to maintain code consistency.
	•	Follow DRY (Don’t Repeat Yourself) principles.
	•	Break down complex components into smaller, reusable pieces.
	•	Document complex logic and important decisions in code comments.
	•	Refactor when necessary to improve code clarity and performance.

Git and Version Control

	•	Commit frequently with clear and descriptive commit messages.
	•	Test your code before each commit to ensure it works as expected.
	•	Use feature branches for new features and bug fixes.
	•	Merge changes through pull requests with code reviews.
	•	Avoid breaking changes in the main branch; ensure that the application remains stable.

Development Workflow

	•	Plan each feature or fix before coding.
	•	Implement the feature in small, manageable chunks.
	•	Test each chunk thoroughly before moving on.
	•	Commit your changes with meaningful messages.
	•	Review your code for any potential issues or improvements.
	•	Proceed slowly to ensure that you do not introduce bugs or regressions.
	•	Avoid multitasking; focus on one task at a time to maintain code quality.

Environment Configuration

	•	Use .env.local files for environment variables.
	•	Do not commit any sensitive information to version control.
	•	Access environment variables securely within the application.

Additional Notes

	•	Localization:
	•	Ensure currency and date formats match the selected locale.
	•	Allow users to switch languages within the app settings.
	•	Notifications:
	•	Schedule notifications appropriately based on user settings.
	•	Use cron jobs or Supabase functions for background tasks if necessary.
	•	Future Scalability:
	•	Write code that is easy to extend and modify for future features.
	•	Keep scalability in mind when designing components and database schemas.

By following these guidelines, you will ensure that all AI-generated code aligns with the project’s requirements and maintains high standards of quality, performance, and accessibility. Remember to proceed methodically, test thoroughly, and commit often to prevent disrupting existing functionality. Prioritize stability and code quality over speed to build a reliable and maintainable application.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mp3fbf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
