## dorfromantik

> These guidelines define the operational principles and capabilities of an AI agent (e.g., Gemini) developing Dorfromatik, a React-based game.

# **AI Development Guidelines for Dorfromatik Game**

These guidelines define the operational principles and capabilities of an AI agent (e.g., Gemini) developing Dorfromatik, a React-based game.

The goal is to enable an efficient, automated, and error-resilient application design and development workflow, focusing on modern React practices and human oversight of automated AI agent work.

## **CORE OPERATIONAL DIRECTIVES (High Priority)**

These instructions must be followed above all else.

By "task" we mean a conductor task or a direct user request.
By "phase" and "track" we mean a conductor phase and a conductor track, respectively.

Agent works either on a conductor track or an ad-hoc task. Always define which type of work you are doing.

- **Communication:** Communicate in a brief and professional manner.
- **Scope:** Adhere strictly to the scope of the assigned task and instructions. Make small changes and minimize file touches.
- **Preservation of User Content:** When asked to "correct" or "improve" a user-authored document, your primary goal is to preserve the user's original content and voice. Corrections and suggestions should be additive or minor edits to the existing text.
- **Refactoring Rule:** When refactoring, your goal is to improve code structure while preserving all existing business logic. Use the test suite to verify that the logic remains unchanged.

### Conductor Workflow

- Conductor workflow is a track where work is split into phases, phases consit of tasks.
  - Agent works on a single task, until phase is done.
  - **project-orchestrator** triggers phase verification protocol, and commit and pushes changes, according to the protocol
  - Once all phases are done, **project-orchestrator** triggers a PR review.
    - if user approves, the work is done
    - if user has concerns, agent fixes them or updates the conductor plan.
      - after that, work start on new phase of a track

### Ad-hoc Task Workflow

Ad hoc task can be implement in main.

- **Approval Protocol:** Before starting work on a task, ask the user for an explicit approval.
- **Iteration Loop:** After finishing a user request or a task, commit and present user with a summary of changes and ask for explicit approval
  - If they request changes, apply them, verify them, amend the commit, present summary and ask for explicit approval again.
  - Termination: Repeat until you get explicit user approval.
  - Push changes after the approval.
- **"Done" Definition:** A coding task is only "Done" when:
  1. **Implementation:** Code is implemented.
  2. **Testing:** All changed files have adequate unit tests.
  3. **Verification:** Verification Protocol all relevant tests passes 100%.
  4. **Approval:** User approved the work with explicit LGTM.
     - If the user requests changes, apply them, verify them, and request approval again.
     - This loop continues until the user explicitly accepts the implementation (`lgtm`, `sgtm`, etc.).
     - Push only after explicit LGTM.

- **Git Tracking (ad-hoc)**
  If the user asks you to commit and push changes:
  1. **Summary:** Attach task summary to the final implementation commit using `git notes add -m "<summary>" <commit_hash>`.
     - push to `origin`; use `--no-verify` if you ran the tests immediatelly before the push.
     - When staging changes, stage only changes the AI agent made; do not interfere with the user’s work, if possible.

### **Commit and Push Strategy**

- **Ad-Hoc Tasks:**
  - Typically, do not commit, until user approves.
  - After user approval, either, commit or commit and push, based on user input.
    - remember to add notes

- **Conductor Tasks (Phase Workflow):**
  - Complete task -> Verify -> Update `plan.md` (mark `[x]`).
  - **Task Commit:** Commit the individual task immediately after it is verified to prevent data loss.
  - **Phase Completion:** Once a phase is completed, the **project-orchestrator** will perform a final verification and create a **Phase Checkpoint** commit if necessary.
    - push will happen at the end of the track
  - **Approval Tracking:** Use a moving tag `approved/<branch-name>` (e.g. `approved/worker1`, `approved/worker2`) as the pointer to the last user-approved commit on that branch. After each new approval, move the tag for that branch to the newly approved commit.

### **GitHub Integration:**

- Most tasks should have a GitHub Issue. Ask user for the issue or if you can proceed without GitHub issue.
- **Issue Linkage:**
  - **Development Commits:** Use `refs #123` to link a task to an issue without closing it (e.g., `feat: add hex logic (refs #123)`).
  - **Closing Commits:** ONLY use `fixes #123` or `closes #123` in the final commit of a Track or when the entire feature/bug is verified as complete.
- **Backlog:** Maintain a `low priority` label on GitHub for non-critical backlog items.
- **Tooling:** Use the `gh` CLI for creating, listing, and managing issues.

## **MULTI-WORKTREE & PRAGMATIC BRANCHING PROTOCOL (High Priority)**

This project uses a "Track-Based" branching model to maintain focus and ensure a clean history.

- **Branching Strategy:**
  - **New Tracks:** When starting a new Conductor Track (e.g., `feat/tile-rotation`), create a dedicated feature branch or reuse a universal branch, such as `coding-agent/worker1`.
  - **Merging:** When working in a branch, all PRs to sync with `main` MUST use a **rebase** strategy (no squash) to preserve a linear and detailed history. All PRs must be reviewed and approved by the user.
  - **Cross-Branch Synchronization:** When asked to implement a track or task from `main` while staying on a current working branch (e.g., `worker1`), ALWAYS synchronize the branch by running `git fetch origin main && git rebase origin/main`.
    - NEVER surgically fetch or checkout specific files/folders from another branch, as this bypasses version control history and creates untracked file conflicts.
  - **Minor Fixes, Ad-hoc tasks & Chores:** For documentation updates, configuration tweaks, or minor verified bug fixes, you may work directly on the current branch (including `main`).
- **Synchronization:** Changes must reach `origin` when appropriate.
  - **Ad-hoc:** push only after user **LGTM** / **approved** (or explicit push instruction).
  - **Conductor:** routine **`git push`** for the track happens at end of the track or during phase checkpoint, depending on user input.

### **Core Engineering Mandates**

- **Contextual Precedence:** Instructions within an active **Gemini Skill** (e.g., `task-conductor`, `project-orchestrator`) take absolute precedence over the general workflows described below. When executing a Conductor Track, strictly adhere to the methodology defined in the skill file.
- **Refactoring:** Always analyze files in the context of the entire project. Check all usages of a class or function to ensure its public API remains valid after refactoring.
- **Verification:**
  1. **Development Loop (Logic):** If you are modifying pure TypeScript logic (`src/models`, `src/utils`, etc.), run `npx putout <changed_files>`, `npm run typecheck`, and **`npm run test:unit`** for fast feedback.
  2. **Development Loop (UI):** If you are modifying React components or styling (`.tsx`, `.css`), run `npx putout <changed_files>`, `npm run typecheck`, `npm run test:unit`, AND **`npm run test:ui`** to ensure visual components render correctly.
  3. **Phase/Track Gates:** Run `npm run test:e2e` ONLY when completing a Conductor Phase or finalizing an Ad-hoc Task.
  4. **Targeted E2E:** You may run specific E2E tests (e.g., `npx playwright test e2e/main-menu.spec.ts`) if your changes directly affect visual rendering or complex UI flows, but avoid running the full suite unnecessarily.
     - **Port Pinning (Multi-Worktree):** To avoid port conflicts when running multiple agents in separate worktrees, each worktree MUST have its own local `.env` file pinning the `PORT` variable (e.g., `PORT=5173` for main, `PORT=5174` for worker1, `PORT=5175` for worker2). Playwright and Vite will automatically honor this variable.
     - **E2E/CI Port:** Dedicated port `5179` is reserved for isolated E2E and CI test runs to prevent collision with active development servers.
     - **Inquiry:** When in doubt, ask the user for the port number.
- **Maintenance:** The AI is responsible for keeping project documentation up-to-date.
  - **Architecture Sync:** Whenever a refactoring or architectural change occurs (e.g., adding a new renderer, changing a design pattern), the AI must update the relevant `ARCHITECTURE.md` file (`tech-stack.md`, `CHANGELOG.md` or equivalent) and ensure `conductor/product.md` remains consistent.
  - **Code Comments:** Maintain clear, concise JSDoc (or equivalent) for all public classes and methods. Ensure comments explain _intent_ and _parameters_, especially after refactoring.
    - comment should explain why
    - do not state obvious in comment, if code is simple
    - for complex methods, feel free to provide overview in comments
  - **Brevity:** Updates should be concise yet sufficient for a future AI agent to understand the system structure and design decisions without needing to re-analyze the entire codebase.

## **Iterative Development & User Interaction**

The AI's workflow is iterative, transparent, and responsive to user input.

- **Prompt Understanding:** The AI will interpret user prompts to understand the desired changes, new features, bug fixes, or questions. It will ask clarifying questions if the prompt is ambiguous.
- **Track Generation & Plan Management:** Each time the user requests a significant change or feature, the AI will propose a new **Track** or update an existing one using **planning-architect** skill.
- **Contextual Responses:** The AI will provide conversational and contextual responses, explaining its actions, progress, and any issues encountered. It will summarize changes made.
- **Error Checking Flow:**
  1.  **Code Change:** AI applies a code modification.
  2.  **Lint/Format:** AI runs `eslint . --fix`.
  3.  **Dependency Check:** If `package.json` was modified, AI runs `npm install`.
  4.  **Compile & Analyze:** AI monitors the terminal for Vite and linter errors.
  5.  **Test Execution:** The AI runs the appropriate verification checks:
      - **Always:** `npm run typecheck` and `npm run test:unit`.
      - **UI Changes:** `npm run test:ui`.
      - **At Gates only:** `npm run test:e2e:ci` (unless the task specifically requires visual verification).
  6.  **Preview Check:** AI observes the browser preview for visual and runtime errors.
  7.  **Remediation/Report:** If errors are found, AI attempts automatic fixes. If unsuccessful, it reports details to the user.

## **Adversarial Review Loop**

We use a two-phase development cycle: **Implement -> Review -> Fix**.

User can trigger AI or human review at the end of a task, phase or a track to ensure high quality of the code.
Therefore it is important to ask user if we can move to the next task.

### **Standard Review (On-Demand)**

1.  **Implementation Phase:**
    - The coding agent implements the feature/fix.
    - **"Done" Signal:** When the code is ready and verified (tests pass), the agent stops and says: "Ready for Review."

2.  **Review Phase (User-Driven):**
    - The user runs `.gemini/review-loop.sh`.
    - This script invokes the **Adversarial Senior Architect** (configured in `.gemini/reviewer.md`) to analyze the changes.
    - The reviewer outputs a `REVIEW_FEEDBACK.md` file.
    - The user manually filters this feedback to ensure relevance.

3.  **Fix Phase:**
    - The user passes the filtered `REVIEW_FEEDBACK.md` back to the coding agent.
    - **Mandate:** The coding agent must address _every_ item in the feedback file.
    - Once fixes are applied and verified, the loop is complete.

### **Thorough Review (Deep Dive / Audit)**

For complex features or comprehensive code audits, we use a structured Conductor-based process (refs #20).

1.  **Initiation:** Triggered by asking for a "Thorough Review" or using `/code-review --against main`.
2.  **Track Setup:** A dedicated Conductor Track is created. Every target file (or changed file) is listed as a discrete task in `plan.md`.
3.  **Execution:** The AI processes the plan file-by-file, auditing each against the criteria in `.gemini/reviewer.md`.
4.  **Reporting:** Findings are appended to a root `REVIEW_FEEDBACK.md`.
5.  **Completion:** Once all tasks are marked as `[x]`, the track is closed, and the findings are handed over to the **Fix Phase**.

## **Environment & Context Awareness**

The AI operates within the Firebase Studio development environment, which provides a Code OSS-based IDE with deep integration for React and Firebase services.

- **Project Structure:** The AI assumes a standard React project structure, initialized with Vite. The primary application entry point is typically `src/main.jsx` or `src/main.tsx`.
- **`dev.nix` Configuration:**
  - The `.idx/dev.nix` file is the declarative source of truth for the workspace environment. The AI understands its role in defining:
    - Required system tools (e.g., `pkgs.nodejs_20`).
    - IDE extensions.
    - Environment variables.
    - Startup commands (`idx.workspace.onStart`).
  - The AI should leverage `dev.nix` to ensure environment consistency and to automatically configure necessary tools or verify their presence.
- **Preview Server:**
  - Firebase Studio provides a running preview server for the web, configured in `dev.nix`.
  - The `web` preview is configured to run `npm run dev -- --port $PORT --host 0.0.0.0`, meaning the Vite dev server is already running and available on a specific port.
  - The AI will continuously monitor the output of the preview server for real-time feedback on changes.
- **Firebase Integration:** The AI recognizes standard Firebase integration patterns in React, including the use of a `firebase.js` or `firebase.ts` configuration file and interactions with various Firebase SDKs.
- **Environment Variables Strategy (.env vs .env.local):**
  - **`.env` (Worktree Specific):** This file is strictly for worktree-specific configurations like `PORT` (e.g., `PORT=5174` for worker1). It must remain unique to each folder to prevent port collisions.
  - **`.env.local` (Shared Secrets):** This file stores shared project secrets like Firebase API keys. To ensure consistency, there should only be one physical `.env.local` file located in the `main` directory. All other worktrees (`worker1`, `worker2`) should use a symlink pointing to the physical file (e.g. `ln -s ../dorfromantik/.env.local .env.local`).
  - **`.env.example` (Tracked Template):** This is the only environment file tracked by Git. It must only contain placeholder values and document the required keys for the project.
- **Architecture:** The AI can read about the architecture of a given folder from an `ARCHITECTURE.md` file.

## **Code Modification & Dependency Management**

The AI is empowered to modify the React codebase and manage its dependencies autonomously based on user requests and detected issues.

- **Core Code Assumption:** When a user requests a change (e.g., "Add a button to navigate to a new page"), the AI will primarily focus on modifying the JSX/TSX code. `src/App.jsx` (or `tsx`) is assumed to be the main component, and the AI will infer other relevant files (e.g., creating new component files, updating `package.json`).
- **Package Management:** If a new feature requires an external package, the AI will identify the most suitable and stable package from npm.
  - To add a regular dependency, it will execute `npm install <package_name>`.
  - To add a development dependency (e.g., for testing or linting), it will execute `npm install -D <package_name>`.
- **Code Quality:** The AI aims to adhere to React best practices, including:
  - Clean code structure and separation of concerns (e.g., UI logic separate from business logic).
  - Meaningful and consistent naming conventions.
  - Effective use of functional components and hooks.
  - Appropriate state management solutions (e.g., component state, context, or a dedicated library like Zustand or Redux Toolkit).
  - Proper use of `async/await` for asynchronous operations with robust error handling.

## **Automated Error Detection & Remediation**

A critical function of the AI is to continuously monitor for and automatically resolve errors to maintain a runnable and correct application state.

- **Post-Modification Checks:** After _every_ code modification (including adding packages or modifying existing files), the AI will:
  1.  Monitor the IDE's diagnostics (problem pane) and the terminal output for compilation errors, linting warnings, and runtime exceptions.
  2.  Check the Vite dev server's output for rendering issues, application crashes, or unexpected behavior.
- **Automatic Error Correction:** The AI will attempt to automatically fix detected errors. This includes, but is not limited to:
  - Syntax errors in JSX/TSX code.
  - Type mismatches (if using TypeScript).
  - Unresolved imports or missing package references.
  - Linting rule violations (the AI will automatically run `eslint . --fix`).
  - Common React-specific issues such as incorrect hook usage or invalid component returns.
- **Problem Reporting:** If an error cannot be automatically resolved (e.g., a logic error requiring user clarification or an environment issue), the AI will clearly report the specific error message, its location, and a concise explanation with a suggested manual intervention or alternative approach to the user.

## **Modern React Practices**

### **React Compiler**

The AI will leverage the React Compiler for automatic memoization, reducing the need for manual `useMemo` and `useCallback` hooks. This leads to cleaner and more performant code. The AI will assume the compiler is enabled for the project.

### **React Server Components (RSC)**

For projects using a framework that supports RSC (like Next.js), the AI will adopt the following principles:

- **Server-Side Logic:** Components that fetch data or perform server-side logic will be implemented as Server Components.
- **Client-Side Interactivity:** Components that require user interaction or client-side state will be marked with the `"use client"` directive.
- **Data Fetching:** The AI will use `async/await` directly within Server Components for clean and efficient data fetching.

## **Architectural Learnings & Patterns**

### **Canvas Controller Pattern**

For high-performance 2D rendering, we use a **Controller Pattern** that decouples React from the Canvas API.

- **React's Role:** Manage the high-level lifecycle of the `<canvas>` element and host DOM-based UI overlays (HUD, buttons).
- **Controller's Role:** A pure TypeScript class (`CanvasController`) that owns the game state, handles the `requestAnimationFrame` loop, and orchestrates specialized renderers.
- **Why:** This avoids React's reconciliation overhead and "effect hell" when dealing with 60fps rendering logic.

### **Rotated Hexagonal Coordinate System**

This project uses a custom orientation for "Flat-Top" hexagons:

- **North is (-1, 0, 1)** instead of the standard `(0, -1, 1)`.
- This requires careful mapping in `HexUtils.ts` (specifically swapping/negating `q` and `r` in pixel-to-hex conversions).

### **E2E Stability (Canvas)**

When writing Playwright tests for a canvas:

- **UI Overlays:** Prefer taking screenshots of the `canvas` locator specifically (e.g., `expect(canvas).toHaveScreenshot()`) rather than the full page to avoid flaky failures caused by UI buttons or overlays.
- **MaxDiff:** Use a small `maxDiffPixels` (e.g., 150) to account for minor anti-aliasing variations between headless environments.

### **"Tiles are Turns" Philosophy**

To ensure state synchronization and a clean game loop, the game follows a "Tiles are Turns" philosophy.

- **Single Source of Truth:** `remainingTurns` must be a derived property (getter) of `tileQueue.length`.
- **Benefit:** This prevents synchronization errors between the turn counter and the actual resources available to the player.

### Canvas -> React State Synchronization

For the Controller Pattern to work with React-based UI (HUD), we use a synchronization callback:

- **Pattern:** `CanvasView` (React) passes a callback or attaches a listener to `CanvasController` (Plain TS).
- **Update:** When the game state changes (score, turns), the Controller invokes the callback, triggering a `useState` update in React.
- **Why:** This keeps the heavy rendering loop out of React while allowing the UI to remain reactive to state changes.

### Vitest & requestAnimationFrame

In a JSDOM environment, `requestAnimationFrame` is not natively supported.

- **Gotcha:** Recursive `setTimeout` mocks for `requestAnimationFrame` can lead to "leaking" timeouts that cause `ReferenceError` or infinite loops in subsequent tests.
- **Best Practice:** Use a static mock `vi.fn().mockReturnValue(1)` in `setup.ts`. This allows the first frame of a Controller (triggered in the constructor) to run while preventing the background loop from interfering with other tests.

### **Explicit Domain Accessors**

Prefer explicit methods like `Tile.getTerrain(direction)` or `Game.rotateQueuedTileClockwise()` over direct property access or boolean-flagged methods (e.g., `rotate(true)`).

- **Reason:** This improves readability for non-TypeScript developers, ensures clearer intent, and prevents "boolean blindness" in API usage.

### **Fail Fast in Game Logic**

Avoid wrapping core game logic (like placement or rotation) in `try/catch` blocks that merely log errors.

- **Reason:** Domain errors (e.g., rotating an empty queue) represent logic bugs or UI state desyncs. Silent catching masks these issues; allowing them to throw ensures they are visible and fixed early.

### **Geometric Logic Centralization**

All geometric and directional logic (finding neighbors, determining opposite sides) must reside in the `Navigation` class.

- **Service Pattern:** Treat `Navigation` as a geometric service that other models (`Game`, `Board`) use to query the world's structure, rather than embedding coordinates math directly into business logic.

### **Graphics Type Safety (Browser Imports)**

When importing types from models into graphics/renderer files (e.g., `TerrainType`):

- **Requirement:** Use `import type { ... }` to ensure the type is stripped during compilation.
- **Why:** Importing a type as a value can lead to a `SyntaxError` in the browser if the underlying model file does not provide a corresponding runtime export for that specific name.

### **Fail-Fast Session State**

The `CanvasController` requires a valid `Session` with an `activeGame` to function.

- **Pattern:** Throw an explicit `Error` in the constructor or `render` loop if `session.activeGame` is missing.
- **Why:** This ensures that synchronization issues between React and the Controller are caught immediately during development rather than failing silently with blank screens.

## **Styling**

The AI will use a consistent styling approach, preferring modern solutions like Tailwind CSS or CSS-in-JS libraries (e.g., styled-components, Emotion) if they are already present in the project. If no styling solution is present, the AI will default to using CSS Modules.

## **Visual Design**

**Aesthetics:** The AI always makes a great first impression by creating a unique user experience that incorporates modern components, a visually balanced layout with clean spacing, and polished styles that are easy to understand.

1.  Build beautiful and intuitive user interfaces that follow modern design guidelines.
2.  Ensure your app is mobile responsive and adapts to different screen sizes, working perfectly on mobile and web.
3.  Propose colors, fonts, typography, iconography, animation, effects, layouts, texture, drop shadows, gradients, etc.
4.  If images are needed, make them relevant and meaningful, with appropriate size, layout, and licensing (e.g., freely available). If real images are not available, provide placeholder images.
5.  If there are multiple pages for the user to interact with, provide an intuitive and easy navigation bar or controls.

**Bold Definition:** The AI uses modern, interactive iconography, images, and UI components like buttons, text fields, animation, effects, gestures, sliders, carousels, navigation, etc.

1.  Fonts - Choose expressive and relevant typography. Stress and emphasize font sizes to ease understanding, e.g., hero text, section headlines, list headlines, keywords in paragraphs, etc.
2.  Color - Include a wide range of color concentrations and hues in the palette to create a vibrant and energetic look and feel.
3.  Texture - Apply subtle noise texture to the main background to add a premium, tactile feel.
4.  Visual effects - Multi-layered drop shadows create a strong sense of depth. Cards have a soft, deep shadow to look "lifted."
5.  Iconography - Incorporate icons to enhance the user’s understanding and the logical navigation of the app.
6.  Interactivity - Buttons, checkboxes, sliders, lists, charts, graphs, and other interactive elements have a shadow with elegant use of color to create a "glow" effect.

## **Accessibility or A11Y Standards:** The AI implements accessibility features to empower all users, assuming a wide variety of users with different physical abilities, mental abilities, age groups, education levels, and learning styles.

## **Routing and Navigation**

For routing, the AI will use `react-router-dom` as the default.

- **Basic Routing:**

```
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Home from "./pages/Home";
import About from "./pages/About";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}
```

- **Navigation:**

```
import { Link, useNavigate } from "react-router-dom";

function MyComponent() {
  const navigate = useNavigate();

  return (
    <div>
      <Link to="/about">About</Link>
      <button onClick={() => navigate("/")}>Go Home</button>
    </div>
  );
}
```

## **Component Library Selection**

The AI will assist in selecting and integrating a component library that best fits the project's needs. The following are popular choices that the AI is familiar with:

- **MUI (formerly Material-UI):** A comprehensive suite of components that implement Google's Material Design. Ideal for a wide range of projects, from simple websites to complex enterprise applications.
- **Ant Design:** A top-tier library for enterprise-level applications, with a focus on a consistent design language and internationalization support.
- **React-Bootstrap:** A great choice for quickly building responsive and familiar-looking interfaces, especially if the team is already familiar with Bootstrap.
- **Chakra UI:** Known for its focus on accessibility and developer experience, providing a set of composable and accessible components that are easy to theme and customize.
- **Shadcn/ui:** A collection of reusable components that you can copy and paste into your project, built on top of Tailwind CSS and Radix UI. This provides maximum control over the code and avoids adding another dependency to your project.
- **Mantine:** A comprehensive library with over 100 customizable components and 50 hooks, known for its ease of use and extensive feature set.

## **Best Practices for Implementation**

### **Theming and Customization**

- **Favor Theming:** Instead of directly overriding component styles with CSS, the AI will leverage the library's built-in theming capabilities. This leads to more maintainable and consistent styling.
- **Wrapper Components:** The AI will create wrapper components around the library's components to enforce project-specific logic and styles. This also simplifies potential future migrations to a different library.

### **Performance Optimization**

- **Tree Shaking:** The AI will import only the components that are needed to reduce the application's bundle size.
- **Lazy Loading:** The AI will use `React.lazy` and `Suspense` to lazy load components, improving the initial load time of the application.

### **Accessibility**

The AI will prioritize using component libraries that follow WAI-ARIA standards to ensure the application is usable by a wider audience.

### **Code Consistency**

The AI will maintain a consistent coding style and structure throughout the project, including naming conventions and file organization.

## **Example: Using Material-UI**

### **Installation**

```shell
npm install @mui/material @emotion/react @emotion/styled
```

### **Theming**

```
import { createTheme, ThemeProvider } from '@mui/material/styles';
import { deepPurple } from '@mui/material/colors';

const theme = createTheme({
  palette: {
    primary: {
      main: deepPurple[500],
    },
  },
});

function App() {
  return (
    <ThemeProvider theme={theme}>
      {/* ... your components */}
    </ThemeProvider>
  );
}
```

### **Using Components**

```
import Button from '@mui/material/Button';

function MyComponent() {
  return <Button variant="contained">Hello World</Button>;
}
```

## **State Management**

The AI will use the simplest appropriate state management solution for the job.

- **Component State (`useState`, `useReducer`):** For local state that is not shared with other components.
- **Context API (`useContext`):** For state that needs to be shared with a few components, but not globally.
- **Zustand or Redux Toolkit:** For complex, global state that is shared across many components. The AI will recommend and use one of these libraries if the application's state management needs become complex.

## **Test Generation & Execution**

When requested, the AI will facilitate the creation and execution of tests, ensuring code reliability and validating functionality.

- **Test Writing:**
  - Upon user request for tests (e.g., "Write tests for this new feature"), the AI will generate appropriate test files (e.g., `src/components/MyComponent.test.jsx`).
  - The AI will use Vitest as the testing framework and React Testing Library for rendering and interacting with components.
  - Tests will be designed to cover different component states, user interactions, and edge cases.
  - **Keep tests DRY** (shared helpers, setup, fixtures where it helps). **Using `for` loops in tests is discouraged** (prefer clear, explicit cases).
  - **Test formatting conventions:**
    - For short tests (roughly 3-6 lines in the test body), do not add `// Arrange`, `// Act`, `// Assert` comments.
    - For very short tests (about 3 lines), keep the body compact with no extra blank lines.
    - Use `// Arrange`, `// Act`, `// Assert` comments only when the test body is medium/long and the sections improve readability.
    - Inside a single test body, keep blank lines only when they separate conceptual blocks.
    - Keep standard structural spacing between tests and describe blocks (one blank line between adjacent `it(...)` and `describe(...)` blocks).
- **Automated Test Execution:**
  - After generating or modifying tests, and after any significant code change, the AI will automatically execute the relevant tests using `npm test` in the terminal.
  - The AI will report test results (pass/fail, with details on failures) to the user.
- **Test Updates**
  - AI will update only tests and cases related to the current task and stick strictly to the task at hand.

# Firebase MCP

When requested for Firebase add the following the server configurations to .idx/mcp.json. Just add the following and don't add anything else.

{
"mcpServers": {
"firebase": {
"command": "npx",
"args": [
"-y",
"firebase-tools@latest",
"experimental:mcp"
]
}
}
}

## Component Visualizer

Task: Whenever I ask for a new component, first output a Component Contract:

Props Interface: (Exactly what goes in)

State Ownership: (Who "owns" the data?)

Visual Hierarchy: (Where does this sit in the tree?)

## Misc

SGTM means sounds good to me, like LGTM Looks good to me.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kursjan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kursjan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
