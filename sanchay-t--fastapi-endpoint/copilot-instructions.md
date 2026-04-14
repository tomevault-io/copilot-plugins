## fastapi-endpoint

> **Goal:** We are building a RESTful API designed to analyse bank statements.

## 1. Project Overview

**Goal:** We are building a RESTful API designed to analyse bank statements.

**Core Functionality:**
*   **API Layer (`api/`):** Handles user authentication (JWT/API Key), request validation, and serves as the main interface. Built with Django REST Framework.
*   **Processing Backend (`backend/`):** Contains the core logic. It takes a PDF bank statement as input, processes it to extract relevant financial data, and returns the analysed information, typically as an Excel file.

Currently, the authentication flow is implemented. The focus is on building out the core statement processing logic and associated API endpoints.

---

## 2. Repository Structure

Here is a snapshot of our project's structure, focusing on key directories and files. Virtual environment contents, compiled files, and media are omitted for clarity.

```
.
├── api  # Main Django project directory
│   ├── backend
│   │   ├── __init__.py
│   │   ├── asgi.py
│   │   ├── celery.py
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── wsgi.py
│   ├── endpoints
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── migrations
│   │   │   ├── 0001_initial.py
│   │   │   └── __init__.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── tasks.py
│   │   ├── tests.py
│   │   ├── urls.py
│   │   └── views.py
│   ├── image-1.png
│   ├── image-2.png
│   ├── image.png
│   ├── local-cert.crt
│   ├── local-cert.key
│   ├── manage.py
│   ├── media
│   ├── .env
│   ├── nginx
│   │   └── conf.d
│   │       └── default.conf
│   └── templates
│       └── defender
│           └── lockout.html
├── backend # Primary PDF processing logic directory
│   ├── __init__.py
│   ├── account_number_ifsc_extraction.py
│   ├── code_for_extraction.py
│   ├── common_functions.py
│   ├── Customer_category.xlsx
│   ├── Final_Category.xlsx
│   ├── gather_info.py
│   ├── main.py
│   ├── old_bank_extractions.py
│   ├── pdf_to_name.py
│   ├── readme.md
│   ├── requirements.txt # Requirements specific to backend logic
│   ├── statements # Example statements (omitted specific files for brevity)
│   ├── tax_professional
│   │   ├── __init__.py
│   │   └── banks
│   │       ├── __init__.py
│   │       └── CA_Statement_Analyzer.py
│   ├── test_endpoint.py # Backend specific tests (Should be moved to backend/tests/)
│   ├── tree.py
│   └── utils.py
├── env # Virtual environment for the project (located at root)
├── django_api.md # Original detailed documentation
├── new_output.json
├── rules.md # This file!
├── TASKS.md # Current development tasks
├── start_dev.sh # Script to start development server
├── activate_env.sh # Script to validate/instruct env activation
├── test_api.py # API level tests (Should be moved into api/endpoints/tests/)
├── test_endpoint.py # More tests? (Consolidate/move)

```

---

## 3. Development Workflow

Follow these steps rigorously *every time* you start working or need to run project commands.

### 3.0. CRITICAL: Check Your Current Directory

**Before running any scripts or commands, ALWAYS verify your current location.**

Run this command:
```bash
pwd
```
This tells you where you are, which is essential for using relative paths correctly.

### 3.1. Activate the Virtual Environment (For Standalone Scripts/Commands)

*If you need to run commands directly in the activated environment (e.g., `pip install`, `python manage.py` commands other than `runserver_plus`, or running test scripts directly), you must activate it first.* The `env` directory is in the **project root**.

1.  **Navigate to Project Root:** Ensure you are in the project root (`/Users/sanchaythalnerkar/fastapi_endpoint`). Check with `pwd`.
2.  **Activate:** Run the following command **directly in your terminal**:
    ```bash
    source env/bin/activate
    ```
    Your terminal prompt should change to show `(env)` at the beginning. You are now in the activated environment for this terminal session.
    *(Note: You can run `./activate_env.sh` first to validate the  path, but it will instruct you to run the `source` command above yourself.)*
3.  **Deactivate:** When finished working in the environment, run `deactivate`.

### 3.2. Start the Development Server

**This is the standard way to run the API for development and testing.**

1.  **Navigate to Project Root:** Ensure you are in the project root (`/Users/sanchaythalnerkar/fastapi_endpoint`). Verify with `pwd`.
2.  **Run the Script:** Execute the following command:
    ```bash
    ./start_dev.sh
    ```
    This single script handles activating the environment *for the server process* and starting the Django development server (`runserver_plus`) within the `api` directory.

    Keep the terminal window running this script open; server logs will appear here. Press `CTRL+C` to stop the server.

---

## 4. Task Management & Progress Tracking

*   **Central Task List:** A file named `TASKS.md` exists in the project root (`/Users/sanchaythalnerkar/fastapi_endpoint/TASKS.md`). This file contains the breakdown of the current development goals into smaller, actionable steps, often with checkboxes.
*   **Follow Sequentially:** You must refer to `TASKS.md` to understand the current objectives and the specific steps required.
*   **Update Progress Regularly:** After completing each small step or task outlined in `TASKS.md`, you **must** update the file immediately.
    *   Mark the completed step (e.g., tick the checkbox `[x]`).
    *   Add any relevant notes or comments if necessary.
*   **Purpose:** This ensures that we maintain an accurate, up-to-date record of what has been done and what the next steps are. It is critical for collaboration and maintaining context, especially when working across multiple sessions.

---

## 5. Code Implementation Philosophy

### 5.1. Minimize Code Addition & File Creation

*   **Modify, Don't Just Add:** Before creating new files or functions, thoroughly check if existing code can be modified or extended to accommodate the new requirement. The goal is to keep the codebase concise and avoid unnecessary fragmentation.
*   **New Files as Last Resort:** Only create new files when it's structurally necessary (e.g., a completely new Django app, a distinct utility module with no logical home).
*   **Targeted Edits:** If working within a large file, use search tools (`grep`, IDE search) to locate the specific function or class you need to modify. Don't rely on manual scrolling.

### 5.2. **CRITICAL: Implement Heavy Logging & Print Statements**

*   **Why?** As an AI or a developer without direct debugging access, the **terminal output is the primary way to understand execution flow and diagnose issues**.
*   **What to Log:**
    *   Entry and exit points of functions.
    *   Values of important variables at key decision points.
    *   Branches taken in conditional logic (`if`/`else`).
    *   Arguments received and values returned by functions.
    *   **Crucially:** All exception blocks (`try...except`) must log the error encountered with context.
*   **How to Log:**
    *   Use Python's `logging` module or simple `print()` statements.
    *   **Clarity is Key:** Messages must be in clear, descriptive English. Don't just print variables; explain what they represent.
        *   *Bad:* `print(x)`
        *   *Good:* `print(f"Processing record ID: {record_id}, status set to: {status}")`
        *   *Good (Error):* `print(f"ERROR processing file {filename}: {e}")` or `logger.error(f"Error during payment calculation for user {user_id}: {exc}", exc_info=True)`
*   **Assume Nothing:** Log information even if it seems obvious. It helps trace the execution path precisely. Senior developers will clean up excessive prints before production, but during development, **more logging is better**.

---

## 6. Testing Your Changes

*   **Mandatory Testing:** After implementing *any* code change (feature or fix), you *must* test it.
*   **Test Structure & Locations:**
    *   **API Tests (`api/endpoints/tests/`):**
        *   **Location:** All tests related to the Django `endpoints` app (models, views, serializers, API behavior, integration within the API) **must** be placed inside the `api/endpoints/tests/` directory. (You may need to create this `tests` directory).
        *   **Organization:** Create new files within this directory for different features or modules (e.g., `test_auth.py`, `test_statement_upload_api.py`).
        *   **Framework:** Use Django's testing framework (e.g., classes inheriting from `django.test.TestCase` or `rest_framework.test.APITestCase`).
    *   **Backend Processing Tests (`backend/tests/`):**
        *   **Location:** All tests specifically for the standalone PDF processing logic contained within the root `backend` directory **must** be placed inside a `backend/tests/` directory (you may need to create this folder).
        *   **Organization:** Organize tests logically within this folder (e.g., `test_pdf_parsing.py`, `test_excel_generation.py`).
        *   **Framework:** Use a suitable test runner like `pytest` for these standalone tests.
    *   **Naming:** Use descriptive names for test files (starting with `test_`) and test methods (starting with `test_`).
*   **Running Tests:**
    *   **Django API Tests:** Activate the environment (Step 3.1), navigate to the `api/` directory, and run `python manage.py test endpoints`.
    *   **Standalone Backend Tests:** Activate the environment (Step 3.1), navigate to the `backend/` directory (or root, depending on `pytest` configuration), and run `pytest` (or your chosen test runner command).
*   **Observe Server Logs:** When running API tests that require the server (e.g., using `APITestCase` or end-to-end tools), ensure the server is running via `./start_dev.sh` and watch its terminal output for related logs and errors.

---

## 7. Git Workflow

Once your code is implemented, **thoroughly tested**, and you've observed logs confirming correct behavior:

1.  **Stage Changes:** Add all your modified/new files:
    ```bash
    git add .
    ```
2.  **Commit Changes:** Write a **detailed and descriptive** commit message. Explain *what* was changed and *why*.
    ```bash
    git commit -m "feat: Implement parsing for HDFC bank statement format [Ticket #123]"
    # OR
    git commit -m "fix: Correct calculation error in loan eligibility check for edge case [Ticket #456]"
    # OR (if extensive logging added)
    git commit -m "refactor: Add extensive logging to user authentication flow for improved debugging"
    ```
    *Bad commit message:* `"fixed stuff"`
3.  **Push Changes:** Push your changes to the `main` branch (or your feature branch if specified):
    ```bash
    git push origin main
    ```

---

## Summary: Key Mandates

*   **Check `pwd`:** Always verify your directory before running commands/scripts.
*   **Activate Env (when needed):** Use `source env/bin/activate` from root for standalone commands.
*   **Start Server:** Use `./start_dev.sh` from root.
*   **Follow & Update `TASKS.md`:** Refer to `TASKS.md` for steps and update it after completion.
*   **Minimize Additions:** Modify existing code first; create files only when necessary.
*   **Log Extensively:** Use detailed `print`/`logging` statements everywhere for debugging via terminal.
*   **Test Thoroughly:** Write tests in the designated directories (`api/endpoints/tests/` for API, `backend/tests/` for backend logic) and run them.
*   **Commit Clearly:** Use detailed messages explaining your changes.
*   **Push to `main`:** Follow the standard Git workflow.

Following these rules strictly is paramount for the project's success and maintainability. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sanchay-T)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sanchay-T)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
