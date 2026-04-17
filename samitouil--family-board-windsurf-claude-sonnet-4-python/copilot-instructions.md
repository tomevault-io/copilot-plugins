## family-board-windsurf-claude-sonnet-4-python

> Alright AI, let's establish the ground rules for our "vibe coding" sessions. Your primary role is to assist me in iterating rapidly and creatively, while ensuring we maintain high code quality, stability, and my ultimate control over the project. Please internalize and adhere to the following operational guidelines:


Alright AI, let's establish the ground rules for our "vibe coding" sessions. Your primary role is to assist me in iterating rapidly and creatively, while ensuring we maintain high code quality, stability, and my ultimate control over the project. Please internalize and adhere to the following operational guidelines:

**1. Environment Management: Local Speed, Docker Truth**
* You are encouraged to help me run and test code locally when I'm aiming for quick feedback, especially during initial 'vibe coding' experiments and UI development.
* However, it is crucial that you **always remind me and actively assist in verifying** that all code we develop runs correctly and as expected within our project's official Docker container. Be careful using « docker compose » and not «  docker-compose » as it is more reliable.
* Before we consider any feature stable or ready for a commit, prompt me to test it thoroughly in the Docker environment. Regularly suggest switching to Docker for validation to catch any environment-specific issues early.

**2. Version Control & Git Workflow: My Command, Your Assistance**
* **This is critical: You MUST ALWAYS obtain my explicit confirmation BEFORE you execute any `git commit` or `git push` command.** No exceptions.
* You will only assist me in pushing code changes to GitHub (or our chosen remote repository) via **Pull Requests (PRs)** originating from feature branches.
* **You are NEVER to attempt to commit or push code directly to `main`, `develop`, or any other protected, long-lived branches.**
* I expect you to **proactively assist me by:**
    * Staging relevant changes for a commit.
    * Suggesting clear and descriptive commit messages based on the changes.
    * Helping me draft PR descriptions.
* Remember, I will always perform the final review and give the explicit command for any Git action that modifies the repository's history or state.

**3. Testing Strategy: Smart Coverage, Perfect Pass Rate, Local CI Checks**
* Assist me in writing **meaningful and effective tests.** Our initial focus should be on covering critical paths, core business logic, and essential functionality.
* As features mature and stabilize, help me identify areas where we can increase test coverage. You can suggest and help generate diverse test cases (unit, integration, E2E).
* While 100% code coverage is a desirable long-term goal for stable code, **do not treat it as an immediate blocker during the highly iterative and experimental phases of vibe coding.** Our priority is functional, well-tested core features.
* **Crucially, ensure ALL tests (Unit and E2E) achieve a 100% pass rate** before we consider merging a PR or deem a feature substantially complete. If tests fail, actively help me diagnose the cause and suggest potential fixes.
* Whenever we work on CI/CD workflow files (e.g., GitHub Actions), remind me to run the full test suite and workflows locally using tools like `act` in **non-interactive mode.** If you help me modify these workflow files, ensure your suggestions are compatible with such local simulation.
* When using act locally, always stop Docker containers first to avoid ports conflicts

**4. Development Mode & Database Management: Agile Data, Controlled Evolution**
* During the initial, highly experimental "vibe coding" phases where data models are rapidly evolving, if I explicitly instruct you, you can assist me in wiping the development database or making direct schema modifications **to maximize speed and iteration.** Clearly confirm these actions with me due to their destructive nature.
* However, once a data model begins to stabilize, or if I indicate that we need to preserve test data or specific database states, **you must transition to helping me generate and use database migrations** for all subsequent schema changes.
* **You are NEVER to create or apply a database migration autonomously.** Always present any generated migration script to me for careful review and wait for my explicit instruction to proceed with its application.

**5. AI Interaction, Code Quality & Hygiene: Precision, Minimalism, Cleanliness**
* When you generate or are asked to execute terminal commands, **you MUST ALWAYS ensure that these commands use appropriate non-interactive flags** (e.g., `-y`, `--force`, `--assume-yes`, `--quiet`, or tool-specific equivalents) to prevent the process from hanging or requiring manual input.
* I expect you to strive for and help me achieve **minimal, clear, and efficient code.** If you generate code that appears verbose or overly complex, be prepared to refactor it for conciseness and simplicity upon my request.
* Proactively assist me in **maintaining excellent code hygiene.** This includes:
    * Helping me identify and remove unused variables, functions, methods, classes, and imports.
    * Flagging potentially dead or redundant code blocks.
    * Assisting in cleaning up experimental code that is no longer needed.
    * This is especially important after intensive experimentation phases common in vibe coding.

By adhering to these guidelines, you will function as an invaluable and trusted partner in our development process. Let's get to work!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SamiTouil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
