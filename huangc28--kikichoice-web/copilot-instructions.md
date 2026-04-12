## kikichoice-web

> {my query (e.g., "the login button still crashes after the last attempt")}

my query:

{my query (e.g., "the login button still crashes after the last attempt")}

AI Task: Diagnose and Resolve the Issue Proactively

Follow these steps rigorously, adhering to all rules in core.md. Prioritize finding the root cause and implementing a robust, context-aware solution.

Initial Setup & Context Validation (MANDATORY):

a. Confirm Environment: Execute pwd to verify CWD. Execute tree -L 3 --gitignore | cat (or ls -laR) focused on the likely affected project/directory mentioned in the query or previous context.
b. Gather Initial Evidence: Collect precise error messages, stack traces, logs (if mentioned), and specific user-observed faulty behavior related to {my query}.
c. Verify File Existence: Use cat -n <workspace-relative-path> on the primary file(s) implicated by the error/query to confirm they exist and get initial content context. If files aren't found, STOP and request correct paths.
Precise Context & Assumption Verification:

a. Deep Dive: Use read_file or cat -n <path> to thoroughly examine the code sections related to the error trace or reported behavior.
b. Trace Execution: Mentally (or by describing the flow) trace the likely execution path leading to the issue. Identify key function calls, state changes, or data transformations involved.
c. Verify Assumptions: Cross-reference any assumptions (from docs, comments, or previous conversation) with the actual code logic found in step 2.a. State any discrepancies found.
d. Clarify Ambiguity: If the error location, required state, or user intent is unclear, STOP and ask targeted clarifying questions before proceeding with potentially flawed hypotheses.
Systematic Root Cause Investigation:

a. Formulate Hypotheses: Based on verified context, list 2-3 plausible root causes (e.g., "Incorrect state update in useState", "API returning unexpected format", "Missing null check before accessing property", "Type mismatch").
b. Validate Hypotheses: Use read_file, grep_search, or codebase_search to actively seek evidence in the codebase that supports or refutes each hypothesis. Don't just guess; find proof in the code.
c. Identify Root Cause: State the most likely root cause based on the validated evidence.
Proactive Check for Existing Solutions & Patterns:

a. Search for Reusability: Before devising a fix, use codebase_search or grep_search to find existing functions, hooks, utilities, error handling patterns, or types within the project that could be leveraged for a consistent solution.
b. Evaluate Suitability: Assess if found patterns/code are directly applicable or need minor adaptation.
Impact Analysis & Systemic View:

a. Assess Scope: Determine if the identified root cause impacts only the reported area or might have wider implications (e.g., affecting other components, data integrity).
b. Check for Architectural Issues: Consider if the bug points to a potential underlying design flaw (e.g., overly complex state logic, inadequate error propagation, tight coupling).
Propose Solution(s) - Fix & Enhance (MANDATORY Confirmation Required):

a. Propose Minimal Fix: Detail the specific, minimal edit_file change(s) required to address the identified root cause. Use workspace-relative paths. Include code snippets. Explain why this fix works.
b. Propose Enhancements (Proactive): If applicable based on analysis (steps 4 & 5), proactively suggest related improvements alongside the fix. Examples:
"Additionally, we could add stricter type checking here to prevent similar issues..."
"Consider extracting this logic into a reusable useErrorHandler hook..."
"Refactoring this section to use the existing handleApiError utility would improve consistency..."
Explain the benefits of the enhancement(s).
c. State Risks/Preconditions: Clearly mention any potential side effects or necessary preconditions for the proposed changes.
d. Request Confirmation: CRITICAL: Explicitly ask the user to confirm which proposal (minimal fix only, or fix + enhancement) they want to proceed with before executing any edit_file command (e.g., "Should I apply the minimal fix, or the fix with the suggested type checking enhancement?").
Validation Plan & Monitoring:

a. Outline Verification: Describe specific steps to verify the fix works and hasn't introduced regressions (e.g., "Test case 1: Submit form with valid data. Expected: Success. Test case 2: Submit empty form. Expected: Validation error shown."). Mention relevant inputs or states.
b. Suggest Validation Method: Recommend how to perform the verification (e.g., manual testing steps, specific unit test to add/run, checking browser console).
c. Suggest Monitoring (Optional): If relevant, suggest adding specific logging (logError or logDebug from utils) or metrics near the fix to monitor its effectiveness or detect future recurrence.
Goal: Provide a robust, verified fix for {my query} while proactively identifying opportunities to improve code quality and prevent future issues, all while adhering strictly to core.md safety and validation protocols.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/huangc28) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
