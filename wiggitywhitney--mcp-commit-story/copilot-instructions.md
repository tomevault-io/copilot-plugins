## refactoring-first

> Before creating new code, **always search for existing implementations** that can be refactored:

# Refactoring-First Development Approach

## Rule 1: Always Check for Existing Implementations First

Before creating new code, **always search for existing implementations** that can be refactored:

- **✅ DO: Search the codebase first**
  ```bash
  # Search for existing functions
  grep -r "collect_chat_history" --include="*.py"
  grep -r "daily_summary" --include="*.py"
  grep -r "git_hook" --include="*.py"
  ```

- **❌ DON'T: Assume functionality doesn't exist**
  ```python
  # Don't create new functions without checking
  def new_chat_collector():  # BAD - might already exist
      pass
  ```

## Rule 2: Refactor Over Rewrite

For tasks that mention "implement" or "create", **first check if the functionality exists**:

### ✅ **Refactoring Patterns:**

```python
# ✅ DO: Refactor existing placeholder
def collect_chat_history(since_commit=None, max_messages_back=150):
    """EXISTING function - refactor the TODO implementation"""
    # TODO: Implement actual chat collection  # ← REFACTOR THIS
    try:
        # Wire up SQLite reader into existing placeholder
        workspace_path = get_cursor_workspace_path()
        chat_data = query_cursor_chat_database(workspace_path, max_messages_back)
        return format_chat_history(chat_data)
    except Exception as e:
        logger.error(f"Chat collection failed: {e}")
        return None

# ✅ DO: Extend existing functionality
def collect_chat_history(since_commit=None, max_messages_back=150, filter_by_git=False):
    """MODIFY existing function to add new capability"""
    chat_history = [...existing implementation...]
    
    # ADD new filtering capability
    if filter_by_git:
        return filter_chat_by_git_relevance(chat_history, get_git_diff())
    return chat_history
```

### ❌ **Anti-Patterns:**

```python
# ❌ DON'T: Create parallel implementations
def new_chat_collector():  # BAD - collect_chat_history() already exists
    pass

# ❌ DON'T: Rewrite working code
def generate_journal_entry_v2():  # BAD - v1 works fine, refactor it instead
    pass
```

## Rule 3: Specific Refactoring Opportunities

### **Chat Collection (Task 48):**
- **✅ REFACTOR:** `collect_chat_history()` in [context_collection.py](mdc:src/mcp_commit_story/context_collection.py)
- **❌ DON'T:** Create new chat collection functions

### **Git-Driven Parsing (Task 49):**
- **✅ ADD:** `filter_chat_by_git_relevance()` to [context_collection.py](mdc:src/mcp_commit_story/context_collection.py)
- **✅ MODIFY:** Existing `collect_chat_history()` to optionally use filter

### **Standalone Generator (Task 50):**
- **✅ REFACTOR:** [git_hook_worker.py](mdc:src/mcp_commit_story/git_hook_worker.py) to import and use [journal_workflow.generate_journal_entry()](mdc:src/mcp_commit_story/journal_workflow.py)
- **✅ REMOVE:** Signal creation logic
- **✅ KEEP:** All existing error handling and telemetry

### **Daily Summary (Task 53):**
- **✅ FIND:** Existing daily summary implementation in [daily_summary.py](mdc:src/mcp_commit_story/daily_summary.py)
- **✅ REFACTOR:** From MCP-based to standalone
- **✅ REUSE:** Existing summary trigger logic from git hooks

## Rule 4: Preserve Working Code

**Always preserve these integration points:**

- **✅ Keep:** Orchestration layer ([journal_orchestrator.py](mdc:src/mcp_commit_story/journal_orchestrator.py), [journal_workflow.py](mdc:src/mcp_commit_story/journal_workflow.py))
- **✅ Keep:** All journal section generators in [journal.py](mdc:src/mcp_commit_story/journal.py)
- **✅ Keep:** TypedDict structures (ChatHistory, GitContext, etc.)
- **✅ Keep:** Graceful degradation when chat/terminal data is unavailable
- **✅ Keep:** Existing telemetry and error handling

```python
# ✅ DO: Preserve existing interfaces
@trace_mcp_operation  # Keep existing decorators
def collect_chat_history(since_commit=None, max_messages_back=150) -> ChatHistory:
    """Keep existing signature and return type"""
    # Refactor implementation, preserve interface
```

## Rule 5: Task Interpretation Guide

**Read tasks with this lens:**

- **"Implement"** → First check if it exists to refactor
- **"Create"** → Look for existing similar functionality to extend  
- **"Add"** → Consider if it can be added to an existing module
- **"Replace"** → This usually means refactor the existing implementation

### **Examples:**

```python
# Task says "Implement chat collection"
# ✅ DO: Check existing implementation first
def collect_chat_history():  # FOUND - refactor this
    # TODO: Implement  # ← Fill this in

# Task says "Create journal generator"  
# ✅ DO: Check for existing generators
# FOUND: journal_workflow.generate_journal_entry() - use this instead
```

## Rule 6: Code Discovery Commands

**Before starting any task, run these searches:**

```bash
# For chat-related tasks
grep -r "collect_chat_history" --include="*.py"
grep -r "ChatHistory" --include="*.py"

# For summary-related tasks  
grep -r "daily_summary" --include="*.py"
grep -r "generate.*summary" --include="*.py"

# For git hook tasks
grep -r "git_hook" --include="*.py"
grep -r "create_tool_signal" --include="*.py"
```

## Rule 7: Don't Over-Engineer

**The existing orchestration layer handles context collection gracefully:**

- **✅ DO:** Use existing [context_collection.py](mdc:src/mcp_commit_story/context_collection.py) functions
- **✅ DO:** Leverage existing [journal_workflow.py](mdc:src/mcp_commit_story/journal_workflow.py) orchestration
- **❌ DON'T:** Create parallel context collection systems
- **❌ DON'T:** Reimplement orchestration logic

## Rule 8: Integration Points to Maintain

**When refactoring, ensure these integration points remain intact:**

1. **TypedDict structures** (ChatHistory, GitContext, etc.)
2. **Orchestration flow** that collects from all 3 sources
3. **Graceful degradation** when chat/terminal data is unavailable  
4. **Existing telemetry and error handling**
5. **Function signatures and return types**

```python
# ✅ DO: Maintain existing integration points
def collect_chat_history(since_commit=None, max_messages_back=150) -> ChatHistory:
    """Preserve signature for existing callers"""
    try:
        # Refactor implementation here
        return ChatHistory(source="cursor_chat", conversations=conversations)
    except Exception as e:
        # Preserve existing error handling patterns
        logger.error(f"Chat collection failed: {e}")
        return None  # Graceful degradation
```

## Rule 9: Refactoring Workflow

**Follow this workflow for all refactoring tasks:**

1. **🔍 Discovery Phase:**
   - Search for existing implementations
   - Identify integration points
   - Map dependencies and callers

2. **📋 Planning Phase:**
   - Determine what to refactor vs. what to preserve
   - Plan backward-compatible changes
   - Identify test coverage needs

3. **🔧 Implementation Phase:**
   - Refactor incrementally
   - Preserve existing interfaces
   - Maintain error handling patterns

4. **✅ Validation Phase:**
   - Run existing tests
   - Verify integration points still work
   - Test graceful degradation scenarios

## Rule 10: Common Refactoring Mistakes to Avoid

**❌ DON'T:**
- Create new functions when existing ones can be enhanced
- Change function signatures without checking all callers
- Remove error handling or telemetry during refactoring
- Assume placeholder functions are empty (they may have important structure)
- Rewrite working orchestration logic

**✅ DO:**
- Extend existing functions with optional parameters
- Preserve all existing decorators and error handling
- Fill in TODO comments rather than replacing entire functions
- Test that existing callers still work after refactoring
- Leverage existing utility functions and patterns

---

**Remember:** The goal is to build on the solid foundation that already exists, not to reinvent the wheel. Always refactor first, create new only when necessary.

---
> Source: [wiggitywhitney/mcp-commit-story](https://github.com/wiggitywhitney/mcp-commit-story) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
