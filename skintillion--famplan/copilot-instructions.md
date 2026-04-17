## famplan

> Every time you choose to apply a rule(s), explicitly state the rule(s) in the output. You can abbreviate the rule description to a single word or phrase.

# OurDonations

Every time you choose to apply a rule(s), explicitly state the rule(s) in the output. You can abbreviate the rule description to a single word or phrase.

## Project Context

A Fundraiser Webapplication for schools and organizations to manage fundraisers, schools, organizations, students, and their associated donations.

## Code Style and Structure

-   Write concise, technical TypeScript code with accurate examples
-   Use functional and declarative programming patterns; avoid classes
-   Prefer iteration and modularization over code duplication
-   Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError)
-   Structure repository files as follows:

```
app/
├── Http/
    ├── Controllers/     # Controller files
    ├── Requests/        # Request files
├── Mail/            # Mail files for sending emails
├── Models/          # Model files
├── Providers/       # Service provider for app
├── Services/        # Service provider for models
├── Traits/            # Traits
config/              # Configuration
database/            # Database
├── factories/       # Factories
├── migrations/      # Migrations
├── seeders/         # Seeders
public/              # Public assets
resources/           # Resources
├── js/              # Javascript
    ├── helpers/         # Helper functions
    ├── plugins/         # Axios and i18n plugins
    ├── router/          # Vue Router
    ├── services/        # Vue Services for Models
    ├── stores/          # Pinia stores
    ├── stub/            # Abilities, Permissions, etc.
    ├── views/           # Vue Views
        ├── components/    # Vue Components
            ├── bar/        # Vue Nav Bars
            ├── dashboard/   # Vue Dashboard
            ├── frontend/    # Vue Frontend
            ├── shared/      # Vue Shared Components
                fundraiser/    # Vue Fundraiser Components
                student/       # Vue Student Components
                tag/           # Vue Tag Components
        ├── layouts/       # Vue Layouts
        ├── pages/         # Vue Pages
            ├── auth/        # Vue Auth Pages (Login, Register, etc.)
            ├── dashboard/   # Vue Dashboard Pages
                ├── donation/    # Vue Donation Dashboard Pages
                ├── entity/      # Vue Entity Dashboard Pages (School, Organization)
                ├── fundraiser/  # Vue Fundraiser Dashboard Pages
                ├── students/    # Vue Students Dashboard Pages
                ├── tags/       # Vue Tag Dashboard Pages
            ├── entity/      # Vue Entity Pages (School, Organization)
            ├── frontend/    # Vue Frontend Pages
                ├── donation/    # Vue Donation Pages
                ├── entity/      # Vue Entity Pages (School, Organization)
                ├── fundraiser/  # Vue Fundraiser Pages
                ├── participant/ # Vue Participant Pages
├── scss/             # SCSS
├── views/           # Blade Views
    ├── emails/          # Email blade templates
routes/              # Laravel Routes
├── api/             # Laravel API Routes
tests/               # Laravel Tests
```

## Tech Stack

-   Laravel 12
-   TypeScript
-   Vuetify 3
-   Pinia

## Naming Conventions

-   Use lowercase with dashes for directories (e.g., components/form-wizard)
-   Favor named exports for components and utilities
-   Use PascalCase for component files (e.g., VisaForm.tsx)
-   Use camelCase for utility files (e.g., formValidator.ts)

## TypeScript Usage

-   Use TypeScript for all code; prefer interfaces over types
-   Avoid enums; use const objects with 'as const' assertion
-   Use functional components with TypeScript interfaces
-   Define strict types for message passing between different parts of the extension
-   Use absolute imports for all files @/...
-   Avoid try/catch blocks unless there's good reason to translate or handle error in that abstraction
-   Use explicit return types for all functions

## State Management

-   Use Pinia for state management

## Syntax and Formatting

-   Use "function" keyword for pure functions
-   Avoid unnecessary curly braces in conditionals

## UI and Styling

-   Use Vuetify 3 components and styling
-   Use flat button backgrounds for all buttons (variant="flat")
-   Use consistent color schemes across components
-   Maintain responsive design for all screen sizes
-   Follow Material Design principles for spacing and typography

## Error Handling

-   Implement proper error boundaries
-   Log errors appropriately for debugging
-   Provide user-friendly error messages
-   Handle network failures gracefully

## Testing

-   Write unit tests for utilities and components

## Security

-   Sanitize user inputs
-   Handle sensitive data properly

## Git Usage

Commit Message Prefixes:

-   "fix:" for bug fixes
-   "feat:" for new features
-   "perf:" for performance improvements
-   "docs:" for documentation changes
-   "style:" for formatting changes
-   "refactor:" for code refactoring
-   "test:" for adding missing tests
-   "chore:" for maintenance tasks

Rules:

-   Use lowercase for commit messages
-   Keep the summary line concise
-   Include description for non-obvious changes
-   Reference issue numbers when applicable

## Documentation

-   Maintain clear README with setup instructions
-   Document API interactions and data flows
-   Keep manifest.json well-documented
-   Don't include comments unless it's for complex logic
-   Document permission requirements

## Development Workflow

-   Use proper version control
-   Implement proper code review process
-   Test in multiple environments
-   Follow semantic versioning for releases
-   Maintain changelog

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Skintillion) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
