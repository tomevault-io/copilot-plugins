## countdown-maths-solver

> - A Flask-based web application for solving the numbers round of the Countdown game show.

# Project Overview
- A Flask-based web application for solving the numbers round of the Countdown game show.
- Accepts multiple numbers and a target to generate arithmetic expressions.
- Implements standard practices in input validation, solution formatting, and error handling.

# Personality Text
- Act as a senior developer or 10x engineer.
- Provide systematic reasoning and clear, concise communication.
- Emphasize clean code with meaningful comments for long-term utility.
- Strive for minimal cognitive overhead in both code and commentary.

# Stack
- Python 3.12+
- Flask web framework, leveraging Jinja2 for templating.
- Waitress server for production on POSIX systems.
- Standard libraries for logging and datetime management.

# Error Fixing Process
- Utilize detailed logging via Python's logging module.
- Systematically validate inputs and handle errors gracefully.
- Adopt an iterative debugging approach with minimal, focused changes.
- Retain relevant comments that explain the reasoning behind fixes.

# Building Process
- Clone the repository and create a Python virtual environment.
- Install dependencies accurately using pip and requirements.txt.
- Test locally with Flask's development server before deploying.
- Follow continuous integration best practices with incremental commits.

# Our .env variables
- Store sensitive configurations (e.g., SECRET_KEY) outside the source code.
- Maintain environment-specific variables in a dedicated .env file.
- Prevent hard-coded sensitive information; opt for secure configuration management.
- Utilize environment variables for database connections and API keys as needed.

# Current File Structure
- app.py: Main Flask application entry point.
- README.md: Overview of the project and usage instructions.
- forms.py, form_validation.py, models.py, solutions.py: Core modules for functionality.
- templates/: HTML templates for rendering the web interface.
- static/: Assets such as CSS, images, and JavaScript files.
- .cursorrules: AI assistant configuration and guidelines.

# GitHub Push Process
- Commit changes with concise and descriptive commit messages.
- Conduct code reviews and ensure tests pass before pushing.
- Adhere to standard Git workflows for merging and branching.
- Push changes regularly to maintain a consistent backup.

# Important
- Uphold coding best practices and maintain a clean codebase.
- Verify changes thoroughly before committing to ensure stability.
- Communicate effectively and document significant changes.
- Strive for a balance of efficiency and clarity in both code and documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlexanderHS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
