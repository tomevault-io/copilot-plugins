## streaming-tracker

> Streaming Tracker is a web application built with Go, HTMX, Postgres, and PGX that helps manage and track TV shows across different streaming platforms. The application allows users to organize their viewing habits, maintain watchlists, and coordinate what family members are watching.

# Streaming Tracker Application

## Overview

Streaming Tracker is a web application built with Go, HTMX, Postgres, and PGX that helps manage and track TV shows across different streaming platforms. The application allows users to organize their viewing habits, maintain watchlists, and coordinate what family members are watching.

## Core Functionality

### Show Management
- **Track viewing status**: Shows can be marked as "Want to Watch", "Watching", or "Finished"
- **Season progression**: Track current season and automatically advance or mark complete
- **Multi-user support**: Multiple family members can watch the same show 
- **Platform organization**: Shows are categorized by streaming platform (Netflix, Hulu, etc.)

### User Features
- **Dashboard**: Main view showing currently watching and want-to-watch shows grouped by viewers
- **Show management**: Add, edit, and delete shows with search and pagination
- **Authentication**: Simple password-based authentication system

### Database Schema
The application uses the following entities:
- **Accounts**: An account houses one or more users
- **Users**: Friends and family members who watch shows
- 
**Watchers**: People watching shows. Every user is a "watcher", but not all watchers are users. This allows users to add people watching who may not want to sign up for an account.
- **Platforms**: Streaming services (Netflix, Hulu, etc.)
- **Shows**: TV series with season information

## Technical Stack

- **Framework**: Go and [Adam Go Kit](https://github.com/adampresley/adamgokit)
- **Database**: Postgres with PGX
- **Styling**: PicoCSS and custom CSS
- **Authentication**: Session-based auth with password protection

## Deployment

The application supports both traditional deployment and Docker containerization, with environment variables for database path, authentication, and pagination settings.

## Codebase Architecture & Conventions

### Project Structure
```
cmd/streaming-tracker/          # Main application
├── internal/                   # Internal handlers/controllers
│   ├── home/                  # Dashboard handlers (home-handlers.go:66)
│   ├── show/                  # Show management handlers (show-handlers.go:493)
│   ├── identity/              # Auth handlers
│   └── viewmodels/            # View data structures
├── app/                       # Frontend assets
│   ├── layouts/               # HTML layout templates
│   ├── pages/                 # Page templates
│   ├── components/            # Reusable components
│   └── static/                # CSS, JS, images
pkg/                           # Reusable packages
├── models/                    # Data structures
├── services/                  # Business logic
├── shows/                     # Show-specific services
├── identity/                  # Auth services
└── watchers/                  # Watcher services
```

### Database Structure (sql-migrations/commit00001.sql)

#### Core Entities
- **accounts**: Container for household/family units with join tokens
- **users**: Authenticated users with email/password and activation codes
- **watchers**: People who watch shows (includes both users and non-users)
- **platforms**: Streaming services (Netflix, Hulu, Disney+, etc.) with icons
- **shows**: TV series with season tracking and cancellation status
- **show_status**: Links shows to watchers with progress tracking
- **watch_status**: Enum values (1="Want To Watch", 2="Watching", 3="Finished")
- **watchers_to_show_statuses**: Many-to-many relationship table

#### Key Relationships
- Every user belongs to an account
- Watchers can be users or non-users within an account
- Shows have many-to-many relationships with watchers through show_status
- Shows track current season progress and completion status

### Go Code Conventions

#### Code Style Patterns
- **Error handling**: `err` variable declared at top of functions
- **Variable declarations**: Grouped at top using `var (...)` blocks
- **Interface-driven design**: Services implement interfaces for testability
- **Configuration injection**: Services receive config structs
- **Session management**: User sessions stored in cookies with auth2 middleware

#### Handler Patterns
- Controllers follow interface-based design with dependency injection
- Handlers use consistent error handling with logging and user feedback
- HTMX partial rendering for dynamic updates without full page reloads
- Session-based authentication with excluded paths for public routes

### Frontend Structure

#### HTML Templates
- **Layouts**: Base layout (layouts/layout.html) with navigation and main content
- **Pages**: Full page templates extending layouts (pages/home.html)
- **Components**: Reusable UI pieces (components/dashboard-shows.html)
- **Template functions**: Custom includes for CSS/JS assets

#### CSS Conventions (static/css/styles.css)
- **PicoCSS framework**: Clean, semantic CSS framework as base
- **CSS nesting**: Modern nested syntax for component styling
- **CSS variables**: Theme-based color system with custom properties
- **Component classes**: `.shows`, `.call-to-action`, tertiary buttons
- **Responsive design**: Flexbox layouts with gap and flex-wrap

#### JavaScript Patterns (static/js/pages/)
- **Modular JS**: Page-specific modules with event-driven validation
- **Form validation**: Custom validation with accessibility (aria-invalid)
- **HTMX integration**: Server-driven dynamic updates with hx-* attributes
- **DOM manipulation**: Event listeners for interactive form validation

### Key Features Implementation

#### Dashboard (home.html, dashboard-shows.html)
- Shows grouped by watcher and status using ordered maps
- Interactive HTMX buttons for status changes (Start Watching, Finish Season)
- Server-side rendering with partial updates for performance

#### Show Management (show-handlers.go:493)
- CRUD operations with validation and error handling
- Platform and watcher assignment with multi-select
- Season progression tracking with automatic advancement
- Search and pagination using query parameters

#### Multi-user Support
- Account-based data isolation with session.AccountID
- Watcher management allowing non-user family members
- Session-based authentication with secure cookie storage

## AI Rules

1. Before you change any code, first display a plan outlining what components and code is going to change, the nature of each change, and a short summary of why you are going to change it. Wait for approval before making any code changes.
2. Do not introduce new 3rd party libraries or dependencies without first consulting me, and telling my why you have to add a new dependency
3. Once you complete a task, always execute both `goimports` and `go build` to ensure the changes do not break anything. The application lives in the `cmd/streaming-tracker` directory, so that is where `go build` needs to be run from.
4. In functions that call things and checks for errors, the variable must be named `err`, and it must be defined at the top of the function
5. Prefer variable declarations at the top of functions (in a var block in parenthesis) over var declarations spread throughout the function, and over short variable declaration form
6. When calling a method that returns a value and an error, prefer single-line function call/`if err != nil` if the function being called doesn't have more than 3 arguments. If it has more than 3 arguments, it is ok to split the function call and `if err != nil`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adampresley)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adampresley)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
