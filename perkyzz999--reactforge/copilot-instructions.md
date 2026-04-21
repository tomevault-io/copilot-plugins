## reactforge

> When working with Next.js and access to next-devtools-mcp.


# Next.js MCP Server Ruleset
# Standard-compliant AI coding rules for using the Next.js 16+ built-in MCP server via next-devtools-mcp.

- id: "nextjs-mcp.overview"
  name: "Next.js MCP usage overview"
  description: >
    High-level guidance for when and how the agent should use the Next.js MCP server and its tools.
  applies_when:
    - "The current repository is a Next.js application."
    - "A Next.js 16+ dev server is running and exposes an MCP endpoint at /_next/mcp."
    - "An MCP server such as `next-devtools` is configured in the host (for example via `.mcp.json`)."
  triggers:
    - "User asks about runtime errors, build errors, or hydration issues in the Next.js app."
    - "User asks where a page, route, layout, or Server Action is defined."
    - "User asks for help debugging unexpected runtime behaviour or failed requests."
    - "Agent needs accurate knowledge of project structure, routes, or configuration beyond static code inspection."
  tools_allowed:
    - "get_project_metadata"
    - "get_page_metadata"
    - "get_errors"
    - "get_logs"
    - "get_server_action_by_id"
  priority: 50
  guidance: |
    # Intent

    Use the Next.js MCP server to inspect the **actual** running development instance of this Next.js project, not just the static code, whenever live runtime data or project-wide metadata is needed to answer the user’s question accurately.

    Prefer MCP tools when:
    - You need to know what routes, pages, layouts, or components are currently active.
    - You are diagnosing build errors, runtime errors, or hydration issues.
    - You need to understand how Server Actions are wired up from their action IDs.
    - You need a canonical view of project structure, dev server URL, or global configuration.

    Do **not** use the MCP server:
    - As a replacement for normal file reading when simple code inspection is enough.
    - In tight loops or at very high frequency.
    - When the user explicitly indicates they are offline or the dev server is not running.

    Always:
    - Call at most one MCP tool at a time unless multiple tools clearly add separate value (for example, `get_errors` followed by `get_page_metadata` for the same page).
    - Reuse information you already retrieved instead of calling the same tool repeatedly with no code changes.
    - Explain to the user when you are basing your answer on live MCP data from their dev server.

  examples:
    good:
      - "User: 'What errors are currently in my application?' → Agent first calls `get_errors`, summarizes the result, and then proposes concrete code edits."
      - "User: 'Which layout renders this /dashboard page?' → Agent calls `get_page_metadata` for that page, inspects the returned tree, then navigates to the layout file and explains the hierarchy."
      - "User: 'I see a Server Action ID in this stack trace, where is it defined?' → Agent calls `get_server_action_by_id` with the ID, then opens the referenced file and explains the function."
    bad:
      - "Agent calls `get_errors` every time the user asks any question, including simple code style questions."
      - "Agent calls `get_logs` repeatedly within the same answer even though logs have not changed."
      - "Agent invents MCP tool names or arguments that do not exist in the tool schema."

- id: "nextjs-mcp.tool-catalog"
  name: "Next.js MCP tool catalog"
  description: "Catalog of built-in Next.js MCP tools, their arguments, and common use cases."
  applies_when:
    - "Any Next.js MCP tool from this list is available in the host."
  tools_allowed:
    - "get_project_metadata"
    - "get_errors"
    - "get_page_metadata"
    - "get_logs"
    - "get_server_action_by_id"
  priority: 60
  guidance: |
    # Tool Catalog

    This rule describes the core tools exposed by the Next.js 16+ MCP endpoint as reported by `tools/list`.

    ## get_project_metadata

    - **Name:** `get_project_metadata`
    - **Description:** Returns metadata for the current Next.js project, including project path and dev server URL.
    - **Arguments:** No input fields (JSON schema: empty object).
    - **When to use:**
      - When you need a canonical understanding of the project root, dev server URL, or high-level configuration.
      - Before using other tools that benefit from knowing the dev server port or base URL.
    - **Example usage (conceptual):**
      - "Call `get_project_metadata` to learn the dev server URL, then use that knowledge to reason about where routes are served from."

    ## get_errors

    - **Name:** `get_errors`
    - **Description:** Gets the current error state from the Next.js dev server, including global Next.js errors (for example, invalid `next.config`), browser runtime errors, and build errors with source-mapped stack traces.
    - **Arguments:** No input fields (JSON schema: empty object).
    - **When to use:**
      - When the user asks about errors, hydration failures, or broken pages.
      - Before proposing large refactors that might already be failing at build-time.
    - **Example usage (conceptual):**
      - "User: 'What errors are currently in my application?' → Call `get_errors`, then summarize the error list and map each error to concrete fixes in the relevant files."

    ## get_page_metadata

    - **Name:** `get_page_metadata`
    - **Description:** Retrieves runtime metadata about what contributes to the current page render from active browser sessions (routes, components, layouts, and rendering info).
    - **Arguments:** No explicit input fields in the schema; the server uses active browser sessions to determine the page.
    - **When to use:**
      - When you need to understand which components, layouts, and data dependencies participate in rendering a given page.
      - When debugging why a page renders differently than expected, including App Router hierarchy issues.
    - **Example usage (conceptual):**
      - "User: 'Why does /settings render this extra layout?' → Call `get_page_metadata`, inspect the component tree for the /settings session, then explain which layout and nested components are involved."

    ## get_logs

    - **Name:** `get_logs`
    - **Description:** Returns the path to the Next.js development log file so the agent can read dev server logs (including browser console logs and server output).
    - **Arguments:** No required input fields; schema is an object with no required properties.
    - **When to use:**
      - When debugging request failures, 5xx responses, or unexpected server behaviour that is not explained by compile/runtime errors alone.
      - When the user reports intermittent issues that are likely to appear in logs rather than as compile-time errors.
    - **Example usage (conceptual):**
      - "User: 'My POST /api/login randomly fails' → Call `get_logs`, open the returned log file, search for relevant stack traces or error lines, then correlate them with the API route implementation."

    ## get_server_action_by_id

    - **Name:** `get_server_action_by_id`
    - **Description:** Locates a Server Action by its ID in `server-reference-manifest.json` and returns the filename and export name for that action.
    - **Arguments:**
      - `actionId` (string, required) – the Server Action ID from an error message, manifest, or stack trace.
    - **When to use:**
      - When the user shows a stack trace or error message that includes a Server Action ID and wants to know where it is defined.
      - When you need to navigate from a manifest entry or client reference back to the actual source file.
    - **Example usage (conceptual):**
      - "User: 'Where is this Server Action with id `c2f1e9b3...` defined?' → Call `get_server_action_by_id` with that ID, then open the returned file and function and explain what it does."

  examples:
    good:
      - "Agent answers 'What tools are available?' by describing `get_project_metadata`, `get_errors`, `get_page_metadata`, `get_logs`, and `get_server_action_by_id` along with when they are appropriate."
      - "Agent uses the fact that most tools have no input fields and thus can be called without arguments when the host schema indicates so."
    bad:
      - "Agent invents extra arguments for `get_errors` or `get_logs` such as a `path` or `page` parameter that do not exist in the tool schema."
      - "Agent assumes `get_page_metadata` can be used to mutate state or navigate the browser rather than just reading metadata."

- id: "nextjs-mcp.using-get-errors"
  name: "Using get_errors for diagnostics"
  description: "How the agent should use get_errors to diagnose and fix issues safely."
  applies_when:
    - "User asks about errors, failing builds, or hydration/runtime issues in a Next.js app."
  tools_allowed:
    - "get_errors"
  priority: 70
  guidance: |
    # Using `get_errors`

    When the user asks about current errors in the project, prefer calling `get_errors` rather than guessing based on partial code snippets.

    Workflow:
    1. Call `get_errors` once to fetch the current error state.
    2. Parse the returned list of errors; identify which files, components, or routes are affected.
    3. Navigate to and inspect only the relevant files instead of scanning the entire project.
    4. Propose targeted code changes that resolve the specific errors you observed.
    5. Only call `get_errors` again after suggesting changes that would plausibly alter the error state.

    Use `get_errors` especially for:
    - Hydration failed messages that reference mismatched server vs client renders.
    - Build-time type errors or bundler errors.
    - Global Next.js configuration problems (for example, invalid `next.config`).

    Avoid:
    - Calling `get_errors` repeatedly in a single answer when the code has not changed.
    - Assuming the absence of errors without checking when the user explicitly asks about them.

  examples:
    good:
      - "User: 'Fix the hydration error on /about' → Agent calls `get_errors`, finds the hydration error for /about, inspects the relevant components, and proposes concrete fixes."
    bad:
      - "Agent edits code for a reported error without ever calling `get_errors` even though the dev server is available."

- id: "nextjs-mcp.using-metadata-tools"
  name: "Using project and page metadata tools"
  description: "How to use get_project_metadata and get_page_metadata to understand structure and routing."
  applies_when:
    - "User asks how pages, layouts, or routes are structured."
    - "User asks where to place new features or components within the App Router hierarchy."
  tools_allowed:
    - "get_project_metadata"
    - "get_page_metadata"
  priority: 70
  guidance: |
    # Using `get_project_metadata` and `get_page_metadata`

    Use `get_project_metadata` when you need a project-wide view:
    - Identify the project root path and dev server URL.
    - Understand high-level configuration that affects routing or rendering.
    - Get a quick sense of the project's structure before proposing major changes.

    Use `get_page_metadata` when you need page-specific context:
    - Understand which layouts and components are currently contributing to a given page render.
    - See how nested layouts, templates, and segments are composed in the App Router.
    - Investigate why a page is rendering an unexpected layout or component tree.

    Typical pattern:
    - For general structural questions, call `get_project_metadata` once at the start of a session.
    - For page-specific questions, call `get_page_metadata` while the user is focused on a particular route or screen.

  examples:
    good:
      - "User: 'Where does the /dashboard page live and which layout wraps it?' → Agent calls `get_page_metadata`, then navigates to the route and layout files and explains the structure."
      - "User: 'Before we reorganize routes, tell me how the app is structured' → Agent calls `get_project_metadata` to understand the current layout and then proposes a refactor."
    bad:
      - "Agent guesses route locations based only on file names when `get_project_metadata` and `get_page_metadata` are available."
      - "Agent calls both tools repeatedly without using the information to guide its edits."

- id: "nextjs-mcp.using-logs"
  name: "Using get_logs for runtime debugging"
  description: "Guidance for leveraging get_logs when debugging server or network issues."
  applies_when:
    - "User reports intermittent or environment-specific issues likely to be visible in logs."
  tools_allowed:
    - "get_logs"
  priority: 70
  guidance: |
    # Using `get_logs`

    Use `get_logs` when the problem is about runtime behaviour that may not produce compile-time errors:
    - Intermittent failures in API routes or Server Actions.
    - Errors that only occur under certain input or authentication conditions.
    - Unexpected 4xx/5xx responses where the code looks superficially correct.

    Workflow:
    1. Call `get_logs` to get the path to the dev server log file.
    2. Open and scan the log file around the time or request path the user describes.
    3. Identify stack traces, error messages, or warnings related to the reported issue.
    4. Navigate to the referenced code locations and propose concrete fixes.

    Avoid:
    - Treating `get_logs` as a replacement for `get_errors` when compile-time or clear runtime errors are already present.
    - Reading the entire log file when the user has given a specific time window or endpoint; filter and focus instead.

  examples:
    good:
      - "User: 'My /api/login route fails randomly' → Agent calls `get_logs`, finds relevant error lines for /api/login, and uses them to guide the fix."
    bad:
      - "Agent repeatedly calls `get_logs` just to 'check things' without using any of the retrieved information."

- id: "nextjs-mcp.server-actions"
  name: "Using get_server_action_by_id for Server Actions"
  description: "How to map Server Action IDs back to their source definitions."
  applies_when:
    - "A stack trace, log line, or error message includes a Server Action ID."
  tools_allowed:
    - "get_server_action_by_id"
  priority: 70
  guidance: |
    # Using `get_server_action_by_id`

    Use `get_server_action_by_id` when you see a Server Action ID (for example in an error message, manifest, or log) and need to locate the corresponding source code.

    Workflow:
    1. Extract the full `actionId` string exactly as shown in the error or log.
    2. Call `get_server_action_by_id` with that `actionId`.
    3. Use the returned filename and export name to open the source file.
    4. Explain to the user what the Server Action does and how it relates to the error or behaviour they are seeing.
    5. Propose targeted changes to the action or its callers, not random edits elsewhere.

    Avoid:
    - Guessing where a Server Action is defined based on naming alone when a valid ID is available.
    - Truncating or altering the `actionId` value; it must match exactly.

  examples:
    good:
      - "User pastes an error containing 'Server Action ID: abc123...' → Agent passes that ID to `get_server_action_by_id`, opens the returned file, and explains the cause."
    bad:
      - "Agent searches the whole repo for the string 'action' instead of using `get_server_action_by_id` when an ID is provided."

- id: "nextjs-mcp.safety-and-etiquette"
  name: "MCP safety, performance, and etiquette"
  description: "Global constraints and best practices when using Next.js MCP tools."
  applies_when:
    - "Any Next.js MCP tool is available."
  tools_allowed:
    - "get_project_metadata"
    - "get_errors"
    - "get_page_metadata"
    - "get_logs"
    - "get_server_action_by_id"
  priority: 90
  guidance: |
    # Safety and Etiquette

    General rules:
    - Prefer reading and understanding before changing: use MCP tools to gather context and then propose edits.
    - Minimize calls: avoid redundant tool invocations; reuse what you already know.
    - Stay within dev context: assume the MCP endpoint is for development only and do not treat it as production.

    Performance:
    - Do not call multiple expensive tools (like `get_logs` and `get_errors`) in a tight loop.
    - Only re-call tools after edits or when the user indicates the state has changed.

    Transparency:
    - When MCP data significantly influences your answer (for example, error lists from `get_errors`), mention that your diagnosis is based on live dev-server information.
    - If the tools are unavailable or fail, fall back to static code reasoning and clearly state that limitation.

  examples:
    good:
      - "Agent calls `get_errors` once, uses the results to structure a fix plan, and only re-calls after suggesting code changes."
      - "Agent explains: 'Based on the live error data from your dev server, here is what is failing and how to fix it.'"
    bad:
      - "Agent spams MCP tools every few sentences without new information."
      - "Agent silently assumes MCP tools are available and correct when a call has actually failed."

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PerkyZZ999) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
