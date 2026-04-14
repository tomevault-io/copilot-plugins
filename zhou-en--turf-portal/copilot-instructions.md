## turf-portal

> - name: Development Philosophy

# ~/DevRules/.cursorrules

rules:

- name: Development Philosophy
  pattern: "\*_/_.go"
  content: |

  - Prefer small, incremental, compiling commits over large "big bang" changes.
  - Study and learn from existing code before writing new functionality.
  - Choose pragmatic, clear, and maintainable code over clever abstractions.
  - Each function or struct should have one clear responsibility.
  - Avoid premature abstractions or unnecessary complexity.
  - When in doubt, choose the simplest solution that clearly conveys intent.

- name: Implementation Workflow
  pattern: "\*_/_.go"
  content: |

  - Follow a test-driven flow when practical: red → green → refactor.
  - Break complex work into documented stages in `IMPLEMENTATION_PLAN.md`.
  - Every commit must:
    - Compile successfully
    - Pass all tests
    - Include new tests for new functionality
    - Conform to project formatting/linting
  - Commit messages should explain _why_ changes are made.

- name: Problem-Solving Guidelines
  pattern: "\*\*"
  content: |

  - Maximum 3 failed attempts before reassessing approach.
  - Document what failed, why, and alternatives you explored.
  - When multiple valid approaches exist, prioritize:
    1. Testability
    2. Readability
    3. Consistency
    4. Simplicity
    5. Reversibility

- name: Architecture & Code Quality
  pattern: "\*_/_.go"
  content: |

  - Use composition and interfaces, not inheritance or singletons.
  - Keep dependencies explicit; avoid hidden coupling.
  - Fail fast with descriptive errors that include context.
  - Never disable tests — fix or adapt them.
  - Write clear, idiomatic Go with descriptive naming and comments when necessary.

- name: Testing & Quality Gates
  pattern: "tests/\*_/_.go"
  content: |

  - Test behavior, not implementation details.
  - Keep tests deterministic and isolated.
  - One clear assertion per test where possible.
  - Follow existing testing utilities and conventions.
  - Definition of Done:
    - Tests passing
    - Code linted and formatted
    - No TODOs without issue references
    - Implementation matches the documented plan

- name: Documentation
  pattern: "\*\*"
  content: |

  - Use Mermaid diagrams to visualize complex flows, architecture, or relationships.
  - Prefer diagrams over lengthy textual descriptions when they add clarity.
  - Common diagram types: flowcharts, sequence diagrams, class diagrams, state diagrams.
  - Keep diagrams simple and focused on one concept at a time.
  - Always accompany diagrams with brief explanatory text.

- name: Development Safety
  pattern: "\*\*"
  content: |
  - Never use `--no-verify` to bypass hooks.
  - Never commit broken or uncompiled code.
  - Always document implementation stages as you progress.
  - Avoid introducing new tools unless justified.
  - Continuously learn from similar patterns in the codebase.

# Go layout rule

- name: Go File Layout & Organization
  pattern: "\*_/_.go"
  content: |

  - Organize Go source files from general → specific:

    1. Package declaration
    2. Imports
    3. File-level constants related to the file’s domain
    4. Type declarations (interfaces before structs)
    5. Constructor functions placed immediately after their type
    6. Methods on a type, grouped and ordered logically
    7. Standalone helper functions at the bottom

  - Keep all methods for a struct grouped together; do not scatter them.
  - Avoid alphabetically sorting methods; instead order them by conceptual flow (constructor → configuration → execute → cleanup).
  - Prefer conceptual cohesion: if a file becomes a “junk drawer” of unrelated types or helpers, split it by domain (e.g. `model.go`, `service.go`, `handlers.go`).
  - Constants should be placed near the top only if used broadly; otherwise, keep them close to the type they belong to.
  - When generating new code, ensure the AI places the new type and its constructor and methods together rather than scattered across the file.

# Temporal – Go

- name: Temporal Workflows - Go (Durable Execution & Determinism)
  pattern: "\*_/_.go"
  content: |

  - When editing Go code that imports `go.temporal.io/sdk/workflow`, treat it as Workflow Definition code and follow Temporal's determinism constraints.

  - Avoid intrinsic non-deterministic logic inside Workflows:

    - Do not use `math/rand` (or other non-deterministic RNG) to decide behavior in a Workflow; move such logic into Activities or use deterministic alternatives.
    - Do not call external systems (HTTP APIs, DB queries, file I/O, other services) directly from Workflows; call Activities instead.
    - Do not rely on system time (`time.Now`, `time.Sleep`) in Workflows.
      - Use `workflow.Now()` and `workflow.Sleep()` instead so time is driven by the Event History and remains deterministic.
    - Do not use native goroutines, channels, or `select` inside Workflows.
      - Use `workflow.Go()`, `workflow.Channel()`, and `workflow.Selector()` for concurrency and coordination.
    - Avoid iterating over Go maps with `range` where ordering matters; sort keys or offload to Activities if ordering is important.

  - Do not store or branch on Workflow run IDs inside Workflow logic; that can create nondeterministic branching between runs.
  - Prefer clear orchestration in Workflows and push all “side-effecty” operations (I/O, randomness, system integration) into Activities.

  - For Temporal projects:

    - Add Replay tests for key Workflows using the Go `worker.NewWorkflowReplayer()` API and downloaded Event History JSON files.
    - Keep Event History test artifacts (JSON) under a `tests` or similar directory alongside replay tests.
    - Treat a failing replay test as a hard stop: it usually means non-deterministic Workflow code or an incompatible code change.

  - When changing Workflow definitions that might affect existing executions:
    - Assume long-lived Workflows may still be running; plan versioning explicitly.
    - Use Temporal’s Versioning / Patching mechanisms (e.g., `workflow.GetVersion` / patch APIs, worker versioning) instead of making breaking changes directly to executing Workflows.

# Temporal – Python

- name: Temporal Workflows - Python (Durable Execution & Determinism)
  pattern: "\*_/_.py"
  content: |

  - When editing Python code that imports `temporalio.workflow`, treat it as Workflow Definition code and respect Temporal's determinism rules.

  - Prefer the Temporal Python SDK’s deterministic primitives over standard library calls:

    - Use `workflow.random()` instead of `random.*` for RNG within Workflows so the chosen values are recorded and deterministic.
    - Use `workflow.now()`, `workflow.time()`, or `workflow.time_ns()` instead of `datetime.now()` / `time.time()` for current time.
    - Use `workflow.uuid4()` instead of `uuid.uuid4()` inside Workflows.
    - Use `asyncio.sleep()` in combination with Temporal's custom event loop for timers, not raw blocking sleeps.

  - Avoid intrinsic non-deterministic patterns in Python Workflows:

    - Do not perform direct network calls, DB queries, file I/O, or other external side effects inside Workflow code; call Activities for those operations.
    - Rely on Python dictionaries’ deterministic ordering for map-like structures, but still avoid patterns where ordering could depend on external/global state.
    - Be careful storing or branching on the Workflow run ID inside Workflow logic.

  - Use the Python SDK “sandbox” as a safety net, not an excuse:

    - Understand that some restricted calls will fail with “Restricted Workflow Access” and must be moved to Activities.
    - Prefer the Temporal-provided APIs to ensure values (randomness, time, UUIDs) become part of the Event History.

  - Logging and observability:

    - Use `workflow.logger` for logging inside Workflows to avoid duplicate logs from replays.
    - When possible, include useful context (Workflow IDs, key parameters) in workflow logs for debugging.

  - Testing Temporal Workflows in Python:

    - Add replay tests using `temporalio.testing.Replayer` and `WorkflowHistory.from_json(...)` against downloaded Event History JSON.
    - Keep Event History JSON under `tests/` next to replay tests.
    - Use `WorkflowEnvironment` and `start_time_skipping()` (from the Temporal Python testing APIs) to test long-running behaviors without waiting in real time.

  - Versioning and code changes:
    - Assume Workflows can be long-running and design with an explicit versioning / upgrade plan.
    - Treat any change to Workflow code that affects control flow, activity calls, or timers as potentially breaking for existing executions; plan to use Temporal’s versioning strategies rather than “just editing the code.”

- name: Pull Request Description Generation
  pattern: "\*\*"
  content: |

  - Do not create pull requests or interact with remote hosting services (GitHub, GitLab, etc.) on my behalf.
  - **NEVER create, modify, or update `PR_DESCRIPTION_GUIDE.md` or `PULL_REQUEST_TEMPLATE.md`** - these files are shared templates managed at the workspace root.
  - When I ask for a pull request description (or anything that looks like a PR description), you must:
    - Locate and read `PR_DESCRIPTION_GUIDE.md` at the root of the workspace directory.
    - Follow all guidelines and best practices described in `PR_DESCRIPTION_GUIDE.md`.
    - Use the PR description template defined in `PULL_REQUEST_TEMPLATE.md` at the root of the workspace directory.
  - The final output must:
    - Consist only of the completed pull request description.
    - Be wrapped in a single fenced Markdown code block starting with `markdown and ending with ` (no extra commentary).
    - Be suitable to be saved as `PR_DESCRIPTION.md` in the current workspace.
  - Do not include explanations, commentary, or extra text outside the Markdown code block.
  - If `PR_DESCRIPTION_GUIDE.md` or `PULL_REQUEST_TEMPLATE.md` is missing or cannot be found at the workspace root:
    - State briefly that the required file is missing.
    - Then still produce a clear, structured PR description in a Markdown code block using a reasonable, conventional PR template (summary, context, changes, testing, risks, etc.).
  - Never push, open, or manipulate pull requests; only generate the description content for `PR_DESCRIPTION.md`.

- name: Atlassian Access (Jira & Confluence)
  pattern: "\*\*"
  content: |
  - When I provide a Jira or Confluence link, always use the available Atlassian MCP tools to access and read the content.
  - Treat Atlassian resources as read-only; never create, edit, update, transition, or delete Jira issues or Confluence pages.
  - When fetching content:
    - Use the Jira MCP tool for any URL that matches a Jira issue, epic, project, sprint, or filter.
    - Use the Confluence MCP tool for any URL that matches a Confluence page, space, or attachment.
  - If the link points to a page or issue that cannot be accessed with the MCP tools, state that it is unavailable rather than synthesizing or guessing.
  - When referencing fetched Jira/Confluence content in responses, summarize clearly and only use information retrieved from the MCP tools.
  - Do not use generic web searches or scraping for Jira or Confluence URLs; always prefer the Atlassian MCP tools as the authoritative source.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhou-en) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
