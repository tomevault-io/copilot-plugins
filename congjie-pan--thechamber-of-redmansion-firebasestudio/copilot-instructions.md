## thechamber-of-redmansion-firebasestudio

> This is a Next.js project that provides an intelligent reading companion for the Chinese classical novel "Dream of the Red Chamber" (紅樓夢). The platform is designed to be an interactive learning experience for students and literature enthusiasts, combining AI technology with traditional literature.

## Project Overview

This is a Next.js project that provides an intelligent reading companion for the Chinese classical novel "Dream of the Red Chamber" (紅樓夢). The platform is designed to be an interactive learning experience for students and literature enthusiasts, combining AI technology with traditional literature.

The project uses a modern tech stack, including:

*   **Frontend:** Next.js 15 with the App Router, React 18, and TypeScript.
*   **UI:** Radix UI and Shadcn/ui for accessible and customizable components, styled with Tailwind CSS.
*   **Backend:** Firebase for authentication and backend services.
*   **AI:** Google's GenKit with the Gemini 2.0 Flash model for AI-powered features, and the Perplexity AI API.
*   **Internationalization:** A custom translation system supporting Traditional Chinese, Simplified Chinese, and English.

The codebase is well-structured, with a clear separation of concerns between UI components, application logic, and AI flows. It also includes a comprehensive test suite with a high level of test coverage.

## Building and Running

### Prerequisites

*   Node.js and npm are installed.
*   A Firebase project is set up.

### Installation

1.  Install the project dependencies:

    ```bash
    npm install
    ```

2.  Create a `.env.local` file in the root of the project and add your Firebase configuration:

    ```
    NEXT_PUBLIC_FIREBASE_API_KEY=your_api_key
    NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_auth_domain
    NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id
    NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_storage_bucket
    NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
    NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id
    ```

### Development

To run the development server, use the following command:

```bash
npm run dev
```

The application will be available at `http://localhost:3001`.

To run the GenKit AI development server for testing AI flows, use the following command:

```bash
npm run genkit:dev
```

### Building

To build the application for production, use the following command:

```bash
npm run build
```

### Testing

To run the test suite, use the following command:

```bash
npm test
```

To run the tests with coverage, use the following command:

```bash
npm test -- --coverage
```

## Development Conventions

*   **Code Style:** The project uses ESLint and Prettier to enforce a consistent code style.
*   **TypeScript:** The project uses TypeScript with strict mode enabled.
*   **Path Aliases:** Path aliases are configured in `tsconfig.json` to allow for cleaner imports (e.g., `@/components/ui/button`).
*   **Components:** The project uses Radix UI and Shadcn/ui for building UI components. New components should follow the established patterns.
*   **AI Flows:** AI-powered features are implemented as GenKit flows in the `src/ai/flows` directory.
*   **Testing:** The project has a comprehensive test suite using Jest and React Testing Library. New features should include unit tests with a high level of coverage.
*   **Internationalization:** The project uses a custom translation system. All user-facing text should be added to the translation files in `src/lib/translations.ts`.
*   **Error Handling:** The project uses a `ChunkErrorBoundary` component to handle dynamic import failures. Other errors should be handled gracefully.
*   **Logging:** The project uses a custom `terminalLogger` for logging messages to the console. Use `debugLog` for development-only messages and `errorLog` for errors.

---

## Coding Guidelines

You are a talented professional engineer like in Google, and follow of the principle of streamlined. Please adhere to the following guidelines:

## 🚨 CRITICAL RULES - READ FIRST

> **⚠️ RULE ADHERENCE SYSTEM ACTIVE ⚠️**  
> **Claude Code must explicitly acknowledge these rules at task start**  
> **These rules override all other instructions and must ALWAYS be followed:**

### 🔄 RULE ACKNOWLEDGMENT REQUIRED
> **Before starting ANY task, Claude Code must respond with:**  
> "✅ CRITICAL RULES ACKNOWLEDGED - I will follow all prohibitions and requirements listed in CLAUDE.md"

### ❌ ABSOLUTE PROHIBITIONS
- **NEVER** create new files in root directory → use proper module structure
- **NEVER** write output files directly to root directory → use designated output folders
- **NEVER** create documentation files (.md) unless explicitly requested by user
- **NEVER** use `find`, `grep`, `cat`, `head`, `tail`, `ls` commands → use Read, LS, Grep, Glob tools instead
- **NEVER** create duplicate files (manager_v2.py, enhanced_xyz.py, utils_new.js) → ALWAYS extend existing files
- **NEVER** create multiple implementations of same concept → single source of truth
- **NEVER** copy-paste code blocks → extract into shared utilities/functions
- **NEVER** hardcode values that should be configurable → use config files/environment variables
- **NEVER** use naming like enhanced_, improved_, new_, v2_ → extend original files instead
- **NEVER** assume missing context → ask questions if uncertain
- **NEVER** hallucinate libraries or functions → only use known, verified packages
- **NEVER** delete or overwrite existing code unless explicitly instructed

### 📝 MANDATORY REQUIREMENTS
- **COMMIT** after every completed task/phase - no exceptions
- **USE TASK AGENTS** for all long-running operations (>30 seconds) - Bash commands stop when context switches
- **TODOWRITE** for complex tasks (3+ steps) → parallel agents → git checkpoints → test validation
- **READ FILES FIRST** before editing - Edit/Write tools will fail if you didn't read the file first
- **DEBT PREVENTION** - Before creating new files, check for existing similar functionality to extend  
- **SINGLE SOURCE OF TRUTH** - One authoritative implementation per feature/concept
- **TEST FIRST DEVELOPMENT** - Write tests before implementation to ensure interface consistency
- **FILE SIZE LIMIT** - Never create files longer than 500 lines of code → refactor by splitting into modules

## 🏗️ CODING GUIDELINES & STANDARDS

### 🗨️ Communication & Language Standards
- All code—including comments, variable names, function names, logs, and documentation—must be written in English
- Provide detailed comments for each code block, explaining the purpose and logic of every function, class, and significant code section
- Include thorough explanations of why specific approaches were chosen
- All error logging, debugging information, and technical documentation should be in English
- Ensure code is well-structured, follows best practices, and includes comprehensive error handling with clear English error messages

### 🔄 Project Awareness & Context Management
- Always read `PLANNING.md` at the start of a new conversation to understand the project's architecture, goals, style, and constraints
- Check `TASK.md` before starting a new task. If the task isn't listed, add it with a brief description and today's date
- Use consistent naming conventions, file structure, and architecture patterns as described in `PLANNING.md`

### 🧱 Code Structure & Modularity Principles
- Never create a file longer than 500 lines of code. If a file approaches this limit, refactor by splitting it into modules or helper files
- Organize code into clearly separated modules, grouped by feature or responsibility
- Use clear, consistent imports (prefer relative imports within packages)

### 📎 Language-Specific Style & Conventions
- **Python Projects**: Follow PEP8, use type hints, format with `black`
- **JavaScript/TypeScript**: Follow ESLint standards, use TypeScript when possible
- **Java**: Follow Oracle coding conventions, use Maven/Gradle structure
- Use `pydantic` for data validation in Python projects
- Use `FastAPI` for APIs and `SQLAlchemy` or `SQLModel` for ORM if applicable
- Write docstrings for every function using the Google style:
  ```
  def example():
      """
      Brief summary.

      Args:
          param1 (type): Description.

      Returns:
          type: Description.
      """
  ```

## 🧪 COMPREHENSIVE TESTING FRAMEWORK

### 🎯 Test-Driven Development (TDD) Workflow
- **Write tests first, implement later** - Follow TDD principles to ensure interface design consistency
- Define expected behavior through tests before writing implementation code
- Use tests as living documentation of system requirements and expected behavior
- **Ensure alignment between tests and implementation** - Tests must use the same calling patterns as the actual implementation
- If tests expect `object.attribute.method()`, implementation must provide that exact interface
- If implementation uses static methods `Class.method()`, tests should mirror this pattern

### 📊 Test Coverage & Quality Requirements
- **Always create unit tests for new features** (functions, classes, routes, etc)
- **Update existing tests when logic changes** - After any logic modification, verify and update related tests
- Tests should live in a `/tests` folder mirroring the main application structure
- Each feature must include at least:
  - **1 test for expected use case** - Normal, successful operation
  - **1 edge case test** - Boundary conditions, unusual but valid inputs
  - **1 failure case test** - Invalid inputs, error conditions, exception handling

### 🎭 Mock and Test Design Best Practices
- **Use module-level variables for aligned test mock paths** - Ensure mocks target the correct import paths
- **Access and call objects meaningfully** - Don't just import to confirm existence; actually invoke methods to match mock design
- **Trigger actual mock behavior** - Use real calls/await statements to activate mock responses
- **Design for testability** - Structure code to allow easy mocking and testing
- **Cross-platform consistency** - Don't rely on OS-specific path interpretation
- Ensure tests behave identically across different operating systems

### 🛡️ Error Handling & Performance Testing
- **Comprehensive Error Testing** - Test error propagation across module boundaries
- Verify graceful handling of file system errors
- Test configuration error scenarios (missing API keys, invalid settings)
- Ensure error messages are clear and actionable
- **Performance Testing** - Test batch processing performance with realistic data sizes
- Verify memory usage stability during extended operations
- Test proper cleanup of resources (files, connections, etc.)

### 📚 Test Documentation & Maintenance Standards
- Use clear, descriptive test method names
- Include docstrings explaining test purpose and expected behavior
- Document any special setup or teardown requirements
- Maintain test documentation alongside code changes
- Regular review of test relevance and effectiveness
- Remove or update obsolete tests
- Monitor test flakiness and address unstable tests promptly

### ✅ Quality Assurance Checklist
Before considering any feature complete:
- [ ] All new functionality has corresponding tests
- [ ] Tests cover normal, edge, and failure cases
- [ ] Test-implementation architectural consistency verified
- [ ] Cross-platform compatibility confirmed
- [ ] Error handling thoroughly tested
- [ ] Performance impact assessed
- [ ] Documentation updated accordingly
- [ ] Code review completed with focus on test quality

## 🏗️ PROJECT STRUCTURE TEMPLATES

### 📁 Simple Project Structure
```
project-root/
├── CLAUDE.md              # Essential rules for Claude Code
├── README.md              # Project documentation
├── .gitignore             # Git ignore patterns
├── src/                   # Source code (NEVER put files in root)
│   ├── main.py            # Main script/entry point
│   └── utils.py           # Utility functions
├── tests/                 # Test files
│   └── test_main.py       # Basic tests
├── docs/                  # Documentation
└── output/                # Generated output files
```

### 📁 Standard Project Structure
```
project-root/
├── CLAUDE.md              # Essential rules for Claude Code
├── README.md              # Project documentation
├── LICENSE                # Project license
├── .gitignore             # Git ignore patterns
├── src/                   # Source code (NEVER put files in root)
│   ├── main/              # Main application code
│   │   ├── [language]/    # Language-specific code
│   │   │   ├── core/      # Core business logic
│   │   │   ├── utils/     # Utility functions/classes
│   │   │   ├── models/    # Data models/entities
│   │   │   ├── services/  # Service layer
│   │   │   └── api/       # API endpoints/interfaces
│   │   └── resources/     # Non-code resources
│   │       ├── config/    # Configuration files
│   │       └── assets/    # Static assets
│   └── test/              # Test code
│       ├── unit/          # Unit tests
│       └── integration/   # Integration tests
├── docs/                  # Documentation
├── tools/                 # Development tools and scripts
├── examples/              # Usage examples
└── output/                # Generated output files
```

### 📁 AI/ML Project Structure
```
project-root/
├── CLAUDE.md              # Essential rules for Claude Code
├── README.md              # Project documentation
├── LICENSE                # Project license
├── .gitignore             # Git ignore patterns
├── src/                   # Source code (NEVER put files in root)
│   ├── main/              # Main application code
│   │   ├── python/        # Python-specific code
│   │   │   ├── core/      # Core ML algorithms
│   │   │   ├── utils/     # Data processing utilities
│   │   │   ├── models/    # Model definitions/architectures
│   │   │   ├── services/  # ML services and pipelines
│   │   │   ├── api/       # ML API endpoints/interfaces
│   │   │   ├── training/  # Training scripts and pipelines
│   │   │   ├── inference/ # Inference and prediction code
│   │   │   └── evaluation/# Model evaluation and metrics
│   │   └── resources/     # Non-code resources
│   │       ├── config/    # Configuration files
│   │       ├── data/      # Sample/seed data
│   │       └── assets/    # Static assets
│   └── test/              # Test code
│       ├── unit/          # Unit tests
│       ├── integration/   # Integration tests
│       └── fixtures/      # Test data/fixtures
├── data/                  # AI/ML Dataset management
│   ├── raw/               # Original, unprocessed datasets
│   ├── processed/         # Cleaned and transformed data
│   ├── external/          # External data sources
│   └── temp/              # Temporary data processing files
├── notebooks/             # Jupyter notebooks and analysis
│   ├── exploratory/       # Data exploration notebooks
│   ├── experiments/       # ML experiments and prototyping
│   └── reports/           # Analysis reports and visualizations
├── models/                # ML Models and artifacts
│   ├── trained/           # Trained model files
│   ├── checkpoints/       # Model checkpoints
│   └── metadata/          # Model metadata and configs
├── experiments/           # ML Experiment tracking
│   ├── configs/           # Experiment configurations
│   ├── results/           # Experiment results and metrics
│   └── logs/              # Training logs and metrics
├── docs/                  # Documentation
├── tools/                 # Development tools and scripts
├── examples/              # Usage examples
└── output/                # Generated output files
```

## ✅ TASK COMPLETION & DOCUMENTATION STANDARDS

### 📝 Task Management Requirements
- Mark completed tasks in `TASK.md` immediately after finishing them
- Add new sub-tasks or TODOs discovered during development to `TASK.md` under a "Discovered During Work" section

### 📚 Documentation & Explainability Standards
- Update `README.md` when new features are added, dependencies change, or setup steps are modified
- Comment non-obvious code and ensure everything is understandable to a mid-level developer
- When writing complex logic, add an inline `# Reason:` comment explaining the why, not just the what

## 🚨 TECHNICAL DEBT PREVENTION SYSTEM

### 🔍 MANDATORY PRE-TASK COMPLIANCE CHECK
> **STOP: Before starting any task, Claude Code must explicitly verify ALL points:**

**Step 1: Rule Acknowledgment**
- [ ] ✅ I acknowledge all critical rules in CLAUDE.md and will follow them

**Step 2: Task Analysis**  
- [ ] Will this create files in root? → If YES, use proper module structure instead
- [ ] Will this take >30 seconds? → If YES, use Task agents not Bash
- [ ] Is this 3+ steps? → If YES, use TodoWrite breakdown first
- [ ] Am I about to use grep/find/cat? → If YES, use proper tools instead

**Step 3: Technical Debt Prevention (MANDATORY SEARCH FIRST)**
- [ ] **SEARCH FIRST**: Use Grep pattern="<functionality>.*<keyword>" to find existing implementations
- [ ] **CHECK EXISTING**: Read any found files to understand current functionality
- [ ] Does similar functionality already exist? → If YES, extend existing code
- [ ] Am I creating a duplicate class/manager? → If YES, consolidate instead
- [ ] Will this create multiple sources of truth? → If YES, redesign approach
- [ ] Have I searched for existing implementations? → Use Grep/Glob tools first
- [ ] Can I extend existing code instead of creating new? → Prefer extension over creation
- [ ] Am I about to copy-paste code? → Extract to shared utility instead

**Step 4: Session Management**
- [ ] Is this a long/complex task? → If YES, plan context checkpoints
- [ ] Have I been working >1 hour? → If YES, consider /compact or session break

> **⚠️ DO NOT PROCEED until all checkboxes are explicitly verified**

### 🧹 Debt Prevention Workflow Pattern
**Before Creating ANY New File:**
1. **🔍 Search First** - Use Grep/Glob to find existing implementations
2. **📋 Analyze Existing** - Read and understand current patterns
3. **🤔 Decision Tree**: Can extend existing? → DO IT | Must create new? → Document why
4. **✅ Follow Patterns** - Use established project patterns
5. **📈 Validate** - Ensure no duplication or technical debt

**Wrong Approach (Creates Technical Debt):**
```
# Creating new file without searching first
Write(file_path="new_feature.py", content="...")
```

**Correct Approach (Prevents Technical Debt):**
```
# 1. SEARCH FIRST
Grep(pattern="feature.*implementation", include="*.py")
# 2. READ EXISTING FILES  
Read(file_path="existing_feature.py")
# 3. EXTEND EXISTING FUNCTIONALITY
Edit(file_path="existing_feature.py", old_string="...", new_string="...")
```

**⚠️ Prevention is better than consolidation - build clean from the start.**  
**🎯 Focus on single source of truth and extending existing functionality.**  
**📈 Each task should maintain clean architecture and prevent technical debt.**  
**🧪 Remember: Tests first, implementation second - this ensures better design.**

> **⚠️ DO NOT PROCEED until all checkboxes are explicitly verified**/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CongJie-Pan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CongJie-Pan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
