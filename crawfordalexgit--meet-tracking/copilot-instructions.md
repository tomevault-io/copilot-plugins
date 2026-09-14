## meet-tracking

> - **Surgical Views**: Do not read entire files if a specific line range can be inspected.

# Token-Efficiency & Anti-Regression Guidelines for AI Agent

## 1. File Inspection
- **Surgical Views**: Do not read entire files if a specific line range can be inspected.
- **Line Restrictions**: When viewing files (especially index.js or heavy pages), specify exact line ranges (`StartLine` and `EndLine`) if the general area is known.

## 2. Code Modifications
- **Surgical Diffs**: Perform precise, surgical edits using replacement blocks. Do not rewrite entire files or output large blocks of unmodified code.
- **Minimum Footprint**: Replace only the minimum necessary lines of code to accomplish the task.

## 3. Command Execution
- **Quiet Commands**: Do not run commands that output verbose, un-paginated logs.
- **Output Constraints**: Always append limiting and quiet flags (e.g., limit logs to `-n 5`, suppress verbose npm/webpack outputs, use `-q` or `--silent` for test suites/scripts, and avoid broad recursive search listings).
- **Terminal Bloat Avoidance**: Large terminal printouts directly consume model token context and must be suppressed with custom limits or silent modes.

## 4. Documentation & Planning
- **Artifact Isolation**: Write long-form technical guides, implementation plans, checklists, and handovers directly to markdown artifacts rather than dumping them inside the chat window.

## 5. Regression Prevention & Code Integrity
- **READ BEFORE WRITE**: Always run a quick git diff or check the git history of a file before modifying it to understand recent custom additions.
- **DO NOT REFACTOR UNLESS ASKED**: Keep changes strictly localized. Do not touch adjacent functions, clean up "styled" code, or simplify custom logic unless it is part of the explicit request.
- **VERIFICATION**: Always validate changes by running `npm run build` or the local build script before declaring a task completed.
- **Automatic Version Bumping**: Every time you successfully execute a code modification or feature implementation, you MUST increment the patch version number (e.g., `1.0.1` to `1.0.2`) in `package.json` as your final step before completing the task.

## 6. Project-Specific "AI DNA" (DO NOT OVERWRITE)
- **Athlete Identity**: Legal name vs. preferred ("Known As") alias mapping in SCM pipelines must be preserved.
- **Event Normalization**: Distance normalizers (stripping 'm' and keeping consistent naming) are critical for gala report matches—do not simplify.
- **Squad Volumes**: Readiness and hours math must remain consistent across individual and squad views.
- **Squad Progression Hierarchy**:
  - Competitive progression: Bronze -> Silver -> Gold Development -> Age Development -> NAR (Nationals & Regionals).
  - Non-competitive squads: Technical Development (14 and younger) & Club 2 for 14+.
  - Target squad determinations for transition audits must strictly follow this hierarchy.

## 7. Browser Testing & Subagent Optimization
- **Direct Navigation**: Navigate directly to target URLs/routes rather than clicking through homepages or navigation menus.
- **Visual-First Verification**: Verify layouts and UI styles visually via screenshots; avoid pulling the entire parsed HTML DOM into text context unless console logs or data dumps are specifically needed.
- **Strict Task Limits**: Give browser subagents highly specific, single-shot instructions (e.g., "Verify element X is visible and exit") to prevent multi-step crawling.
- **Use Lightweight Tools**: Prioritize `read_url_content` for fetching static pages, texts, or API responses; do not launch browser environments unless active JavaScript or visual rendering is required.

---
> Source: [crawfordalexgit/Meet_Tracking](https://github.com/crawfordalexgit/Meet_Tracking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
