## horta-hub

> This is a Ruby on Rails application that serves as a starting point for a web application. It includes user authentication, password reset, and a basic home page.

# Project Overview

This is a Ruby on Rails application that serves as a starting point for a web application. It includes user authentication, password reset, and a basic home page.

**Main Technologies:**

*   **Backend:** Ruby on Rails 8.1
*   **Database:** SQLite3
*   **Frontend:** Hotwire (Turbo, Stimulus), TailwindCSS
*   **Authentication:** Session-based authentication with `has_secure_password`.
*   **Deployment:** Kamal

**Architecture:**

The application follows a standard Rails MVC architecture.

*   **Models:** `User` and `Session` models for authentication.
*   **Views:** ERB templates with TailwindCSS classes.
*   **Controllers:** `UsersController`, `SessionsController`, `PasswordsController`, and `HomeController` to handle web requests.
*   **Authentication:** The `Authentication` concern in `app/controllers/concerns/authentication.rb` provides session-based authentication.

# Building and Running

**Prerequisites:**

*   Ruby
*   Bundler

**Setup:**

1.  Install dependencies:
    ```bash
    bundle install
    ```
2.  Create the database:
    ```bash
    rails db:create
    ```
3.  Run database migrations:
    ```bash
    rails db:migrate
    ```

**Running the application:**

```bash
./bin/dev
```

This will start the Rails server, and you can access the application at `http://localhost:3000`.

**Running tests:**

```bash
rails test
```

# Development Conventions

*   **Code Style:** The project uses `rubocop-rails-omakase` for code style enforcement. Run `rubocop` to check for violations.
*   **Testing:** The project uses Minitest for testing. Tests are located in the `test` directory.
*   **UI Components:** The project uses `ruby_ui` for some UI components.

# Customization

When responding to technical prompts, always adopt the persona of a "senior developer expert." This means providing complete, technical, and well-founded explanations. Your answers should include:

*   Clear code examples.
*   Best practices.
*   Considerations for security, performance, and maintainability.
*   Alternative solutions when applicable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LucassF5)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LucassF5)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
