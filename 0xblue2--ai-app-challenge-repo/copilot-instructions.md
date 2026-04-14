## ai-app-challenge-repo

> This document provides essential context for AI agents and developers working on this project.

# AGENTS.md - Working in the ai_app_challenge Repository

This document provides essential context for AI agents and developers working on this project.

## Project Overview

A **Student Readiness Inventory survey application** built entirely in Gleam:

- **Backend project** (`backend/gleam.toml`): Erlang target, runs on BEAM with Wisp web framework
  - Generates static HTML at build-time from `questions.txt`
  - Serves pre-generated HTML files
  - Receives form submissions and logs to terminal

- **Root project** (`gleam.toml` at root): Shared utilities (JavaScript target for tests)
  - Contains `questions_parser.gleam` used by backend
  - Minimal - mostly for code organization

**Key Feature**: Survey form is **generated once at build-time** from `questions.txt` using Lustre elements, producing static HTML files. No dynamic template rendering at runtime.

## Essential Commands

### Root Project (Shared utilities)

```bash
# From C:/Users/dougl/coding/ai_app_challenge/
gleam build              # Compile shared modules
gleam test               # Run tests (uses gleeunit)
gleam format --check src test  # Check formatting
```

### Backend Project

```bash
# From C:/Users/dougl/coding/ai_app_challenge/backend/
gleam deps download      # Download dependencies
gleam build              # Compile Erlang code
gleam run                # Start the HTTP server on port 8000
gleam test               # Run tests
gleam run -m build       # Generate static HTML from questions.txt
```

### Build & Deploy

```powershell
# From repository root (Windows PowerShell)
.\scripts\build_frontend.ps1
```

This script:
1. **Generates static HTML**: Reads `questions.txt`, runs `gleam run -m build` in backend
   - Generates `backend/priv/static/survey.html` with all questions pre-rendered
   - Generates `backend/priv/static/thanks.html`

Must be run before starting the backend for the first time or after changing `questions.txt`.

### CI/Verification

The GitHub Actions workflow (`.github/workflows/test.yml`) runs:
- `gleam deps download`
- `gleam test`
- `gleam format --check src test`

OTP version: 28, Gleam version: 1.15.2

## Project Structure

```
ai_app_challenge/
├── src/                          # Root project source (shared utilities)
│   ├── ai_app_challenge.gleam   # Shared modules
│   └── questions_parser.gleam   # Question parsing utility (used by backend)
├── test/
│   └── ai_app_challenge_test.gleam      # Tests using gleeunit
├── backend/                      # Backend project (main application)
│   ├── src/
│   │   ├── ai_app_challenge_backend.gleam  # Main entry
│   │   ├── backend_app.gleam           # Server bootstrap & port setup
│   │   ├── build.gleam                 # Build-time HTML generation from questions.txt
│   │   ├── graceful_shutdown.gleam     # Erlang FFI for OTP shutdown
│   │   ├── questions_parser.gleam      # Question parsing (mirrors root/src/)
│   │   └── survey/
│   │       ├── router.gleam            # HTTP request routing
│   │       ├── handlers.gleam          # Route handlers (GET /survey, POST /api/survey)
│   │       └── schema.gleam            # Form decoding & validation
│   ├── priv/static/              # Generated static files
│   │   ├── survey.html           # Pre-generated from questions.txt (build-time)
│   │   └── thanks.html           # Pre-generated thank you page
│   └── manifest.toml             # Dependency lock file
├── scripts/
│   └── build_frontend.ps1        # Build script: generates HTML from questions.txt
├── questions.txt                 # Single source of truth for survey questions
├── gleam.toml                    # Root project config
└── manifest.toml                 # Root project dependency lock
```

## Application Architecture

### Build-Time HTML Generation

```
questions.txt
    ↓
questions_parser.parse()  // Produces List(Question)
    ↓
render_survey_form()      // Creates Lustre Element tree
    ↓
element.to_document_string()  // Converts to HTML string
    ↓
simplifile.write()        // Writes to backend/priv/static/survey.html
```

### HTTP Request Handling

1. **Routing** (`backend/src/survey/router.gleam`):
   - `GET /` → redirect to `/survey`
   - `GET /survey` → serves pre-generated `priv/static/survey.html`
   - `GET /survey/thanks` → serves pre-generated `priv/static/thanks.html`
   - `GET /favicon.ico` → 204 No Content
   - `GET /__quit?token=<token>` → graceful shutdown
   - `POST /api/survey` → form submission handler

2. **Survey Form** (`backend/priv/static/survey.html`, generated at build time):
   - Pure HTML form with all questions pre-rendered
   - No client-side generation or dynamic content
   - Submits as HTTP POST to `/api/survey`
   - Inputs are named by question ID (e.g., `q1_confidence`, `q2_time_management`)

3. **Form Submission** (`backend/src/survey/handlers.gleam` → `post_survey()`):
   - Extracts form values from `application/x-www-form-urlencoded` body
   - Calls `schema.decode()` to parse into `Submission` record
   - Logs to stdout: name, email, grade, answers (dict), notes
   - Redirects to `/survey/thanks` on success, or returns 400 on decode error

4. **Data Model** (`backend/src/survey/schema.gleam`):
   - `Submission` record: student name, email, grade, answers (Dict of question_id→Int score), notes
   - `DecodeError` type: `MissingField`, `InvalidInt`
   - Validation: name, email, grade are required; answers and notes are optional

### Data Flow

```
USER ACTION:              RUNTIME:
  │
GET /survey            handlers.get_survey()
  │                         ↓
  │                   simplifile.read("priv/static/survey.html")
  │                         ↓
  └─→ [Browser receives pre-generated HTML with all questions]
      
  User fills form and clicks Submit
      │
POST /api/survey       handlers.post_survey()
  │                         ↓
  │                   schema.decode(form.values)
  │                         ↓
  │                   io.println(submission)
  │                         ↓
  └─→ 302 Redirect to /survey/thanks
      
GET /survey/thanks     handlers.get_thanks()
      │
      └─→ [Browser receives pre-generated thanks page]
```

## Key Architectural Decisions & Gotchas

### 1. **Single Source of Truth: questions.txt**

All questions come from `questions.txt` in the repository root. Format:

```
q1_confidence
I feel confident about my ability to succeed in my classes.
1 = Strongly disagree, 5 = Strongly agree
likert_5
---
q2_time_management
I manage my time well and can keep up with deadlines.
1 = Strongly disagree, 5 = Strongly agree
likert_5
---
[more questions...]
```

At build time:
- `gleam run -m build` reads and parses `questions.txt`
- `questions_parser.parse()` creates a `List(Question)`
- `render_survey_form()` builds Lustre elements from questions
- `element.to_document_string()` converts to HTML string
- HTML is written to `backend/priv/static/survey.html`

**To add/change questions**: 
1. Edit `questions.txt`
2. Run `.\scripts\build_frontend.ps1`
3. Restart backend (or just refresh browser if already running)

### 2. **Build-Time HTML Generation (Zero Runtime Overhead)**

- HTML is generated once during build (via `gleam run -m build`), then served as static files
- No rendering or template processing at request time
- Faster response times and better for caching
- Uses type-safe Lustre elements (not string concatenation)
- **Important**: If you change `questions.txt`, you must re-run the build script before changes appear

### 3. **Build-Time HTML with Lustre Elements**

The HTML is generated using Lustre elements (type-safe, composable) rather than string concatenation.

Key functions in `backend/src/build.gleam`:
- `render_survey_form()` - main form structure using Lustre elements
- `render_question_element()` - renders each question
- `render_choice_element()` - renders radio button options
- `style_css()` - survey page styles
- `thanks_style_css()` - thanks page styles

Benefit: Type-safe HTML generation, easy to refactor, no manual HTML escaping needed.

### 4. **Serving Static HTML Files**

The backend serves pre-generated HTML files from `backend/priv/static/`:
- `survey.html` - form with all questions (generated at build-time)
- `thanks.html` - thank you page (generated at build-time)

Handlers in `backend/src/survey/handlers.gleam` read these files from disk and return them. If files don't exist, returns 500 error with helpful message.

### 5. **Graceful Shutdown Token**

The backend prints a shutdown token and URL on startup:
```
Dev shutdown token: <random-string>
Quit URL: http://127.0.0.1:8000/__quit?token=<token>
```

This is dev-only. In production, this should be:
- Removed entirely, OR
- Replaced with environment-variable-based authentication

The token is hardcoded as a dev value in `backend_app.gleam` (line 11).

### 6. **Form Validation**

- **Server side**: Only `student_name`, `student_email`, `student_grade` are required; validated in `schema.required()`
- **Client side**: HTML5 `required` attribute on form inputs
- **Email validation**: Browser-level only (type="email")
- No backend email format validation

### 7. **Questions Data Model**

- Likert scale answers are stored as `Int` (1–5)
- Form submission just parses string values to integers
- `schema.decode()` silently skips any answer that fails `int.parse()`
- If a question isn't in the form submission, it's simply absent from the answers dict
- This is intentionally permissive (allows partial submissions)

### 8. **Graceful Shutdown via Erlang FFI**

`graceful_shutdown.gleam` uses an `@external` annotation to call Erlang's `init:stop()`:
```gleam
@external(erlang, "init", "stop")
fn init_stop() -> Nil
```

This is the idiomatic way to shut down an Erlang node gracefully.

### 9. **No Persistence**

Form submissions are logged to stdout only. There's no database, file storage, or any persistent backend.

## Code Conventions & Patterns

### Gleam Style
- Module names map to file paths: `survey/router.gleam` → module `survey/router`
- Type names are CamelCase; function names are snake_case
- Functions are organized by public API first, then helpers at the bottom
- Error handling uses `Result` type extensively

### Build Module Pattern
The `build.gleam` module demonstrates:
- Reading files at build time: `simplifile.read()`
- Parsing structured data: `questions_parser.parse()`
- Rendering HTML: Lustre elements (type-safe)
- Writing output files: `simplifile.write()`
- Handling errors and logging: `io.println()`

### Form Decoding Pattern
Example from `schema.gleam`:
```gleam
required(dict, "field_name")
|> result.try(fn(value) {
  // Chain further validation or decoding
  // Use result.map or result.try
})
```

This is the standard Gleam pattern for chaining validations. Don't nest `case` statements; use `result.try()` and `result.map()`.

### HTML Rendering (Lustre Elements)

In `build.gleam`, HTML is generated using Lustre elements:
```gleam
html.div([attribute.class("container")], [
  html.h1([], [element.text("Title")]),
  html.form(
    [attribute.method("POST"), attribute.action("/api/survey")],
    [
      // form contents
    ]
  )
])
|> element.to_document_string()
```

This approach is:
- Type-safe (compiler checks valid HTML structure)
- Composable (break into smaller functions)
- Automatic escaping (no manual HTML entity handling needed)
- Still produces minified HTML

### Static Asset Serving

Wisp middleware pattern (from `router.gleam`):
```gleam
use <- wisp.serve_static(req, under: "/assets", from: static_dir)
```

This is **stacked middleware**: each `use` block is applied in order. Earlier handlers take precedence.

## Testing

### Test Framework: gleeunit

```gleam
import gleeunit

pub fn main() -> Nil {
  gleeunit.main()
}

// Test functions must end with `_test`
pub fn my_test() {
  assert 2 + 2 == 4
}
```

### Current Tests
- `test/ai_app_challenge_test.gleam`: Example test

### Run Tests
```bash
gleam test
```

### To Add More Tests
1. Add `.gleam` files in `test/` directory
2. Define functions ending in `_test`
3. Use `assert` or gleeunit matchers (see gleeunit docs)
4. No separate test runners needed; `gleam test` auto-discovers

**Note**: Currently minimal test coverage. Consider adding tests for:
- `questions_parser.parse()` with various inputs
- `schema.decode()` with valid and invalid form data

## Dependencies & Versions

### Root Project (Shared utilities)
- `gleam_stdlib >= 0.44.0` – standard library
- `gleeunit >= 1.0.0` (dev) – testing framework

### Backend Project (Application)
- `gleam_stdlib >= 0.44.0`
- `gleam_erlang >= 1.0.0` – Erlang VM bindings (process, atom, etc.)
- `gleam_http >= 4.0.0` – HTTP types
- `filepath >= 1.1.0` – path manipulation
- `logging >= 1.0.0` – structured logging (currently unused)
- `mist >= 6.0.0` – HTTP server
- `wisp >= 2.2.0` – web framework (routing, middleware)
- `simplifile >= 3.0.0` – file I/O (build-time and runtime file reading)
- `lustre >= 5.6.0` – HTML element generation (build-time only, for type-safe HTML)
- `gleeunit >= 1.0.0` (dev)

**Note on Lustre in backend**: Used only at build-time to generate HTML using type-safe elements. Not used at runtime.

**Why Wisp + Mist?** Wisp is a lightweight Gleam web framework. Mist is the HTTP adapter. Together they provide routing and middleware without heavy dependencies.

## Common Tasks

### Add a New Survey Question

1. Edit `questions.txt` in the repository root:
   ```
   q6_new_id
   New prompt text
   Help text / instructions
   likert_5
   ---
   ```

2. Rebuild:
   ```powershell
   .\scripts\build_frontend.ps1
   ```

3. Restart backend or refresh browser
   (Backend serves pre-generated HTML, so changes require rebuild)

### Modify Survey Styling

The CSS is embedded in `backend/src/build.gleam` in the `style_css()` function. 

To change styles:
1. Edit the CSS inline in `build.gleam`
2. Re-run `gleam run -m build` in backend to regenerate HTML
3. Refresh browser

### Change the Server Port

Edit `backend/src/backend_app.gleam` (line 21):
```gleam
|> mist.port(8000)  // Change to 8001, 9000, etc.
```

Then rebuild and run.

### Store Form Submissions Persistently

Currently submissions are logged to stdout. To add persistence:

1. Add a database dependency (e.g., `pgsql` or SQLite driver)
2. Modify `handlers.post_survey()` to insert the `Submission` record into a database instead of `io.println()`
3. Add a migration or schema setup

The `Submission` type and validation are already in place; you just need to store them.

### Run Locally

```bash
# Build static HTML from questions.txt
.\scripts\build_frontend.ps1

# Start backend
cd backend
gleam run
# Prints: Dev shutdown token: ... Quit URL: http://127.0.0.1:8000/__quit?token=...
```

Then open `http://localhost:8000/survey` in browser.

Submissions are logged to the terminal where the backend is running.

## Troubleshooting

### "Failed to read questions.txt" when running build

The build module looks for `questions.txt` relative to the backend working directory.

**Fix**: 
1. Ensure `questions.txt` exists in the repository root
2. Run `gleam run -m build` from the backend directory, or
3. Use the build script which handles this: `.\scripts\build_frontend.ps1`

### "Survey HTML not found" error (500) when accessing /survey

The backend couldn't find `backend/priv/static/survey.html`.

**Fix**: Run the build script:
```powershell
.\scripts\build_frontend.ps1
```

### Form submission shows "Bad request" (400)

The server couldn't decode the form. Check:
1. Required fields (`student_name`, `student_email`, `student_grade`) are filled in
2. Browser console for any client-side form errors
3. Backend logs in the terminal for the specific decode error

### Port 8000 already in use

Change the port in `backend/src/backend_app.gleam` (line 21):
```gleam
|> mist.port(8000)  // Change to 8001, 9000, etc.
```

Then rebuild and run.

---

**Last Updated**: 2026-04-11  
**Project Status**: 
- Backend-only application (frontend folder removed)
- Static HTML generation at build-time using Lustre elements
- No dynamic rendering at runtime
- No persistence (logs to stdout only)
- Dev shutdown token enabled

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0xBlue2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
