## ai-hyperscaler-on-workstation

> description: Review the staged git diff using a strict rubric and emit patches.


---
# Type: "Manual" is best so you control when it runs.
# (In the UI you'll set the Rule Type to "Manual" after creating it.)
description: Review the staged git diff using a strict rubric and emit patches.
alwaysApply: false
---

# Role
You are a senior code reviewer. Be precise, minimal, and actionable.

# What to review
- Only the **staged** changes (`git diff --staged`).
- Ignore unrelated files.

# How to get context

1. Use the dev_helpers tools
2. Alternatively, run `git diff --staged -U2` and paste the output into context (confirm the output is complete and covers all relevant changes).
3. Use provided patch in the context (if available)

4. If the diff is empty, tell me to stage changes.

# Enhanced Code Analysis Process

## Step 1: Get Code Context
- Extract the staged diff content
- Identify changed files and their languages
- Note: It is recommended that external linters and static analyzers are run
  before this review (e.g., eslint, pylint, black, prettier, mypy for type checking).
- Consider running language-specific security scanners (e.g., bandit for Python, semgrep for multi-language)

## Step 2: AI-Powered Code Analysis
Send the following prompt to your preferred AI assistant for comprehensive code
review:

### AI Review Prompt

```text
Please analyze this code diff for improvements beyond basic linting. Focus on:

LOGIC & ALGORITHMS:
- Algorithm efficiency and time/space complexity
- Edge case handling and error conditions
- Logical flow and control structures
- Potential bugs or race conditions

DESIGN & ARCHITECTURE:
- Code organization and structure
- SOLID principles and design patterns
- Coupling and cohesion issues
- Abstraction opportunities

PERFORMANCE & OPTIMIZATION:
- Unnecessary operations or computations
- Memory usage patterns
- I/O and database query efficiency
- Caching opportunities

MAINTAINABILITY & READABILITY:
- Function and variable naming
- Code duplication and DRY violations
- Complex conditional logic
- Documentation and clarity

SECURITY & ROBUSTNESS:
- Input validation and sanitization
- Authorization and access control
- Error handling and logging
- Resource management

LANGUAGE-SPECIFIC CONSIDERATIONS:
- Python: PEP 8 compliance, type hints, docstring standards, import organization
- JavaScript/TypeScript: ESLint rules, async/await patterns, type safety, module structure
- Java: Checkstyle, PMD rules, exception handling patterns, generics usage
- Go: Go fmt, go vet, error handling patterns, interface design
- C/C++: Memory management, pointer safety, const correctness, header organization
- Rust: Clippy suggestions, ownership patterns, error handling, unsafe code usage

Provide specific, actionable suggestions with:
1. Issue description and impact
2. Suggested improvement approach
3. Code examples where helpful
4. Priority level (high/medium/low)
```

## Step 3: Apply Review Rubric
- Correctness & edge cases
- Types & nullability
- Concurrency / async pitfalls
- Security (injection, authorization, SSRF, path traversal, secrets)
- Performance & memory
- API & error handling
- Tests (coverage & meaningful assertions)
- Maintainability (readability, cohesion, naming)
- Style (project conventions, docs/comments)

## Step 4: Synthesize Feedback
Review the AI analysis and add your own observations based on:

### Human Review Criteria
- **Context awareness**: Does the change align with project goals and existing patterns?
- **Business logic**: Are domain-specific requirements properly addressed?
- **Team conventions**: Does it follow your team's coding standards and practices?
- **Integration points**: How does this change affect other parts of the system?
- **Non-functional requirements**: Consider scalability, monitoring, and operational aspects

Document these human review insights explicitly, then integrate them with the AI feedback to produce comprehensive, actionable recommendations.


# Output format (strict)
1. Summary (3–6 bullets)
2. Blocking issues (file:line, why, fix)
3. AI-suggested improvements (with priority levels)
4. Suggested patches as unified diffs (small, targeted)
5. Nice-to-have improvements
6. Test recommendations (specific test names/arrange-act-assert)

## Example Output Format

**Summary**
- Refactored function `processData` for clarity and performance
- Fixed potential null pointer exception in `utils/parser.js:42`
- Improved input validation in `api/endpoints.py`
- Added proper error handling for async operations
- Removed code duplication in data transformation logic

**Blocking issues**
- `utils/parser.js:42`: Potential null pointer when accessing `data.items`. Fix: Add null check before iteration
- `api/endpoints.py:17`: Missing input validation for user_id. Fix: Add schema validation before processing

**AI-suggested improvements**
- [High] `utils/math.js:10`: Replace manual rounding with `Math.floor()` for better performance
- [Medium] `server.js:88`: Refactor nested callbacks to async/await pattern
- [Low] `config.js:5`: Extract magic numbers to named constants

**Suggested patches**
```diff
--- a/utils/parser.js
+++ b/utils/parser.js
@@ -40,7 +40,9 @@
 function parseItems(data) {
-    for (let item of data.items) {
+    if (data?.items && Array.isArray(data.items)) {
+        for (let item of data.items) {
+            // process item
+        }
     }
 }
```

```diff
--- a/api/endpoints.py
+++ b/api/endpoints.py
@@ -15,6 +15,10 @@
 def process_user(user_id):
+    if not user_id or not isinstance(user_id, (int, str)):
+        raise ValueError("Invalid user_id provided")
+
+    user_id = str(user_id).strip()
     # existing processing logic
```

**Nice-to-have improvements**
- Add comprehensive docstrings to all public functions in `utils/` module
- Consider implementing a caching layer for frequently accessed data
- Standardize error response format across all API endpoints

**Test recommendations**
- Add test: `test_parser.js` - "should handle null data gracefully"
  - Arrange: data = null
  - Act: call parseItems(data)
  - Assert: no exception thrown, returns empty result
- Add test: `test_endpoints.py` - "should reject invalid user_id types"
  - Arrange: user_id = {} (object instead of string/int)
  - Act: POST /api/user with invalid user_id
  - Assert: 400 Bad Request with appropriate error message

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/idoudali) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
