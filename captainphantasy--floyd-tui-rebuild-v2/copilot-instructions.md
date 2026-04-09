## floyd-tui-rebuild-v2

> Use when: Reading documentation, API docs, or web content

# FLOYD TUI REBUILD - STRICT BUILD PROTOCOL

This protocol is MANDATORY. You MUST follow these steps in order. Skipping any step is a CRITICAL FAILURE.

---

## Project Context

Working Directory: /Volumes/Storage/FLOYD_CLI/TUI REBUILD
Phase: Phase 3 Complete → Phase 4 Ready
Last Updated: 2026-02-01
Project Type: Node.js/TypeScript with Ink TUI
Isolation: Separate node_modules/, no impact on parent FLOYD_CLI projects

---

## MCP Tools Available (46 tools across 6 servers)

### Floyd Patch Server (5 tools)
- apply_unified_diff - Apply patches (dry-run, risk assessment)
- edit_range - Edit specific line range
- insert_at - Insert at line number
- delete_range - Delete line range
- assess_patch_risk - Risk assessment before patching

### Floyd Runner Server (6 tools)
- detect_project - Auto-detect project type and commands
- run_tests - Run test suite
- format - Format code
- lint - Run linter
- build - Build project
- check_permission - Check if permission granted

### Floyd Supercache Server (12 tools)
- cache_store, cache_retrieve, cache_delete, cache_clear, cache_list, cache_search, cache_stats, cache_prune
- cache_store_pattern - Store code patterns
- cache_store_reasoning, cache_load_reasoning, cache_archive_reasoning - Reasoning persistence

### Floyd Safe Ops Server (3 tools)
- safe_refactor - Refactoring with rollback
- impact_simulate - Simulate change impact
- verify - Verify changes didn't break functionality

### Floyd Terminal Server (10 tools)
- start_process, interact_with_process, read_process_output, force_terminate
- list_sessions, list_processes, kill_process
- execute_code, create_directory, get_file_info

### Novel Concepts Server (10 tools)
- compute_budget_allocator - Allocate reasoning across tasks
- concept_web_weaver - Knowledge graph weaving
- episodic_memory_bank - Cross-session memory
- analogy_synthesizer - Generate analogies
- semantic_diff_validator - Semantic difference validation
- refactoring_orchestrator - Multi-file refactoring
- consensus_protocol - Build reasoning consensus
- distributed_task_board - Distributed task management
- adaptive_context_compressor - Compress context intelligently
- execution_trace_synthesizer - Debugging trace synthesis

--- Use when: Complex planning, multi-file changes, decision making, or debugging

7. Web Reader (1 tool) - Web Content

- mcp__web_reader__webReader - Fetch web pages and convert to markdown

Use when: Reading documentation, API docs, or web content

8. Vision Analysis (1 tool) - Image Understanding

- mcp__4_5v_mcp__analyze_image - Analyze images with AI vision

Use when: Analyzing screenshots, diagrams, or visual content

---
HOW TO USE THESE TOOLS

These are ALREADY LOADED in your session. Call them directly using the tool syntax.

Examples:

# Store reasoning about STAT architecture
mcp__floyd_supercache__cache_store_reasoning(
context="stat_architecture_decision",
reasoning="Chose Axum 0.7 over Actix-web for better async trait support",
conclusion="Axum selected for STAT backend"
)

# Simulate impact before changing attestation flow
mcp__floyd_safe_ops__impact_simulate(
operations=[{"type": "edit", "path": "backend/src/api/attestation.rs"}],
projectPath="/Volumes/Storage/STAT"
)

# Run backend tests
mcp__floyd_runner__run_tests(projectPath="/Volumes/Storage/STAT/backend")

# Check if build permission granted
mcp__floyd_runner__check_permission(
toolName="build",
projectPath="/Volumes/Storage/STAT/backend"
)

---
MANDATORY USAGE PATTERNS

1. Before Code Changes

# Always simulate first
mcp__floyd_safe_ops__impact_simulate(operations=[...], projectPath="/Volumes/Storage/STAT")

# Then verify after
mcp__floyd_safe_ops__verify(strategy="command", command="cargo test")

2. Complex Decisions

# Store reasoning for future sessions
mcp__floyd_supercache__cache_store_reasoning(
context="phase4_implementation",
reasoning="Implemented exception execution system with rollback capability",
conclusion="Phase 4 complete with 2,134 lines added"
)

3. Multi-File Changes

# Use orchestrator for coordinated changes
mcp__novel_concepts__refactoring_orchestrator(
refactoring_type="extract_interface",
target="AttestationService",
files=["backend/src/api/attestation.rs", "backend/src/services/attestation.rs"]
)

4. Debugging

# Generate execution traces before fixing bugs
mcp__novel_concepts__execution_trace_synthesizer(
code="function processAttestation(id) { ... }",
language="javascript",
input_scenarios=[{"name": "valid id", "inputs": {"id": 123}}]
)

---
VERIFIED WORKING (2026-02-01)

All 44 tools confirmed loaded in your session via /context:
┌───────────────────┬────────────┬───────────┐
│      Server       │ Tool Count │  Status   │
├───────────────────┼────────────┼───────────┤
│ floyd-patch       │ 5          │ ✅ Loaded │
├───────────────────┼────────────┼───────────┤
│ floyd-runner      │ 6          │ ✅ Loaded │
├───────────────────┼────────────┼───────────┤
│ floyd-supercache  │ 12         │ ✅ Loaded │
├───────────────────┼────────────┼───────────┤
│ floyd-safe-ops    │ 3          │ ✅ Loaded │
├───────────────────┼────────────┼───────────┤
│ floyd-terminal    │ 10         │ ✅ Loaded │
├───────────────────┼────────────┼───────────┤
│ novel-concepts    │ 10         │ ✅ Loaded │
├───────────────────┼────────────┼───────────┤
│ web_reader        │ 1          │ ✅ Loaded │
├───────────────────┼────────────┼───────────┤
│ 4.5v-mcp (vision) │ 1          │ ✅ Loaded │
└───────────────────┴────────────┴───────────┘
---
THE TL;DR

You have 44 additional tools in this session. They work. Use them.

Don't limit yourself to standard Claude capabilities. These tools give you:
- Persistent memory across sessions (SUPERCACHE)
- Safe code changes with rollback (SAFE-OPS, PATCH)
- Process management (TERMINAL)
- Advanced reasoning (NOVEL CONCEPTS)
- Project automation (RUNNER)
- Web + Vision capabilities

The tools are loaded. Call them directly.

## STRICT BUILD PROTOCOL - MANDATORY EXECUTION FLOW

### Step 0: PRE-CHANGE BASELINE (MANDATORY - NEVER SKIP)

BEFORE touching ANY code, you MUST:

bash
# 0a. Navigate to project directory
cd "/Volumes/Storage/FLOYD_CLI/TUI REBUILD"

# 0b. Capture baseline build state
npm run build 2>&1 | tee /tmp/floyd_baseline_build.log
echo $? > /tmp/floyd_baseline_build_exit.txt

# 0c. Capture baseline lint state
npm run lint 2>&1 | tee /tmp/floyd_baseline_lint.log
echo $? > /tmp/floyd_baseline_lint_exit.txt

# 0d. Capture baseline test state (if tests exist)
npm test 2>&1 | tee /tmp/floyd_baseline_tests.log
echo $? > /tmp/floyd_baseline_tests_exit.txt

# 0e. Create baseline receipt
cat > /tmp/floyd_baseline_[receipt.md](http://receipt.md) << 'EOF'
# FLOYD TUI REBUILD - BASELINE RECEIPT
Generated: $(date)
Project: /Volumes/Storage/FLOYD_CLI/TUI REBUILD

Build Exit Code: $(cat /tmp/floyd_baseline_build_exit.txt)
Lint Exit Code: $(cat /tmp/floyd_baseline_lint_exit.txt)
Test Exit Code: $(cat /tmp/floyd_baseline_tests_exit.txt)

Build Summary:
$(tail -10 /tmp/floyd_baseline_build.log)

Lint Summary:
$(tail -10 /tmp/floyd_baseline_lint.log)

Test Summary:
$(grep -E "(passing|failing|tests?)" /tmp/floyd_baseline_tests.log | tail -5)
EOF


Step 0 is COMPLETE ONLY when:
- [x] /tmp/floyd_baseline_[receipt.md](http://receipt.md) exists
- [x] Build exit code is documented
- [x] Lint exit code is documented
- [x] Test results have been reviewed
- [x] Pass/fail state is documented

WITHOUT Step 0 complete: DO NOT proceed to Step 1.

---

### Step 1: MAKE YOUR CHANGES

Now you may edit code.

Tools you MAY use:
- Floyd edit_range (for surgical edits)
- Floyd apply_unified_diff (for patches)
- Floyd safe_refactor (for refactoring with rollback)
- Floyd impact_simulate (to preview impact)
- Write tool (for new files)
- Read tool (to understand code)

Tools you MUST NOT use:
- DO NOT "fix" any test failures until Step 2
- DO NOT assume test failures are bugs
- DO NOT change code without baseline comparison
- DO NOT skip impact simulation for non-trivial changes

Recommended workflow:

bash
# Before making changes, simulate impact
mcp__floyd-safe-ops__impact_simulate(
  operations=[{"type": "edit", "path": "src/components/StatusBar.tsx"}],
  projectPath="/Volumes/Storage/FLOYD_CLI/TUI REBUILD"
)

# Make your changes using Floyd tools
mcp__floyd-patch__edit_range(
  path="src/components/StatusBar.tsx",
  startLine=10,
  endLine=20,
  newContent="// your changes here"
)

# Store reasoning for future reference
mcp__floyd-supercache__cache_store_reasoning(
  context="floyd_tui_phase4_task_2_1",
  reasoning="Fixed any type warnings by adding explicit TypeScript types",
  conclusion="Type safety improved without breaking existing functionality"
)


---

### Step 2: POST-CHANGE VERIFICATION (MANDATORY)

AFTER making changes, you MUST:

bash
# 2a. Capture post-change build state
npm run build 2>&1 | tee /tmp/floyd_post_build.log
echo $? > /tmp/floyd_post_build_exit.txt

# 2b. Capture post-change lint state
npm run lint 2>&1 | tee /tmp/floyd_post_lint.log
echo $? > /tmp/floyd_post_lint_exit.txt

# 2c. Capture post-change test state
npm test 2>&1 | tee /tmp/floyd_post_tests.log
echo $? > /tmp/floyd_post_tests_exit.txt

# 2d. Compare vs baseline
diff /tmp/floyd_baseline_build.log /tmp/floyd_post_build.log | tee /tmp/floyd_build_diff.log
diff /tmp/floyd_baseline_lint.log /tmp/floyd_post_lint.log | tee /tmp/floyd_lint_diff.log
diff /tmp/floyd_baseline_tests.log /tmp/floyd_post_tests.log | tee /tmp/floyd_test_diff.log

# 2e. Capture git diff
git diff > /tmp/floyd_code_diff.patch


Optional: Use Floyd Safe Ops verification

bash
mcp__floyd-safe-ops__verify(
  strategy="command",
  command="npm run build && npm run lint"
)


---

### Step 3: FAILURE ANALYSIS (MANDATORY IF FAILURES EXIST)

If post-change build/lint/tests fail, you MUST determine:

bash
# 3a. Extract baseline results
grep -E "(error|warning|✖|✓)" /tmp/floyd_baseline_build.log > /tmp/floyd_baseline_results.txt
grep -E "(error|warning|✖|✓)" /tmp/floyd_baseline_lint.log >> /tmp/floyd_baseline_results.txt

# 3b. Extract post-change results
grep -E "(error|warning|✖|✓)" /tmp/floyd_post_build.log > /tmp/floyd_post_results.txt
grep -E "(error|warning|✖|✓)" /tmp/floyd_post_lint.log >> /tmp/floyd_post_results.txt

# 3c. Compare
diff /tmp/floyd_baseline_results.txt /tmp/floyd_post_results.txt


Decision Tree:

IF failure exists in BOTH baseline AND post-change:
    → This is a PRE-EXISTING issue
    → DO NOT "fix" it as part of your changes
    → Document it separately
    → PROCEED to sign-off

ELSE IF failure exists ONLY in post-change:
    → YOUR CHANGES BROKE THE BUILD
    → YOU MUST FIX IT
    → Return to Step 1

ELSE IF build/lint/tests pass in post-change:
    → Your changes did not break anything
    → PROCEED to sign-off

You MUST execute this decision tree BEFORE editing any code to "fix" failures.

---

### Step 4: SIGN-OFF RECEIPT (MANDATORY)

You MAY claim "complete" ONLY when:

bash
cat > /tmp/floyd_change_[receipt.md](http://receipt.md) << 'EOF'
# FLOYD TUI REBUILD - CHANGE RECEIPT
Generated: $(date)
Change: [describe what you changed]
Task ID: [e.g., Task 2.1, Task 2.2, etc.]

BASELINE STATE:
Build Exit: $(cat /tmp/floyd_baseline_build_exit.txt)
Lint Exit: $(cat /tmp/floyd_baseline_lint_exit.txt)
Test Exit: $(cat /tmp/floyd_baseline_tests_exit.txt)

POST-CHANGE STATE:
Build Exit: $(cat /tmp/floyd_post_build_exit.txt)
Lint Exit: $(cat /tmp/floyd_post_lint_exit.txt)
Test Exit: $(cat /tmp/floyd_post_tests_exit.txt)

DELTA ANALYSIS:
Build Changed: [YES/NO]
Lint Changed: [YES/NO]
Tests Changed: [YES/NO]
New Failures Introduced: [COUNT]
Pre-existing Failures: [COUNT]

FILES CHANGED:
$(git diff --name-only HEAD)

RECEIPTS:
- /tmp/floyd_baseline_[receipt.md](http://receipt.md)
- /tmp/floyd_change_[receipt.md](http://receipt.md)
- /tmp/floyd_build_diff.log
- /tmp/floyd_lint_diff.log
- /tmp/floyd_test_diff.log
- /tmp/floyd_code_diff.patch

GIT DIFF:
$(cat /tmp/floyd_code_diff.patch)
EOF


Sign-off is VALID ONLY when:
- [x] Step 0 (baseline) completed BEFORE changes
- [x] Step 1 (changes) used appropriate tools
- [x] Step 2 (verification) captured all build/lint/test states
- [x] Step 3 (analysis) completed BEFORE "fixing" anything
- [x] Change receipt accurately reflects delta analysis
- [x] No new failures introduced by your changes
- [x] Git diff captured and reviewed

---

## Summary: The Mandatory Flow

For EVERY code change:

1. Step 0: Capture baseline → /tmp/floyd_baseline_[receipt.md](http://receipt.md)
2. Step 1: Make changes using Floyd tools
3. Step 2: Capture post-change state → compare vs baseline
4. Step 3: Analyze failures → determine responsibility
5. Step 4: Create sign-off receipt → /tmp/floyd_change_[receipt.md](http://receipt.md)

If you skip any step, STOP. You are about to make a preventable mistake.

---

Every code change MUST follow this. No exceptions. No shortcuts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CaptainPhantasy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CaptainPhantasy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
