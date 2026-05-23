## utilities

> Guidelines for implementing utility functions


# Utility Function Guidelines

## General Principles

- **Function Scope**:
  - ✅ DO: Create utility functions that serve multiple modules
  - ✅ DO: Keep functions single-purpose and focused
  - ❌ DON'T: Include business logic in utility functions
  - ❌ DON'T: Create utilities with side effects

  ```javascript
  // ✅ DO: Create focused, reusable utilities
  /**
   * Truncates text to a specified length
   * @param {string} text - The text to truncate
   * @param {number} maxLength - The maximum length
   * @returns {string} The truncated text
   */
  function truncate(text, maxLength) {
    if (!text || text.length <= maxLength) {
      return text;
    }
    return text.slice(0, maxLength - 3) + '...';
  }
  ```

  ```javascript
  // ❌ DON'T: Add side effects to utilities
  function truncate(text, maxLength) {
    if (!text || text.length <= maxLength) {
      return text;
    }
    
    // Side effect - modifying global state or logging
    console.log(`Truncating text from ${text.length} to ${maxLength} chars`);
    
    return text.slice(0, maxLength - 3) + '...';
  }
  ```

## Documentation Standards

- **JSDoc Format**:
  - ✅ DO: Document all parameters and return values
  - ✅ DO: Include descriptions for complex logic
  - ✅ DO: Add examples for non-obvious usage
  - ❌ DON'T: Skip documentation for "simple" functions

  ```javascript
  // ✅ DO: Provide complete JSDoc documentation
  /**
   * Reads and parses a JSON file
   * @param {string} filepath - Path to the JSON file
   * @returns {Object|null} Parsed JSON data or null if error occurs
   */
  function readJSON(filepath) {
    try {
      const rawData = fs.readFileSync(filepath, 'utf8');
      return JSON.parse(rawData);
    } catch (error) {
      log('error', `Error reading JSON file ${filepath}:`, error.message);
      if (CONFIG.debug) {
        console.error(error);
      }
      return null;
    }
  }
  ```

## Configuration Management

- **Environment Variables**:
  - ✅ DO: Provide default values for all configuration
  - ✅ DO: Use environment variables for customization
  - ✅ DO: Document available configuration options
  - ❌ DON'T: Hardcode values that should be configurable

  ```javascript
  // ✅ DO: Set up configuration with defaults and environment overrides
  const CONFIG = {
    model: process.env.MODEL || 'claude-3-7-sonnet-20250219',
    maxTokens: parseInt(process.env.MAX_TOKENS || '4000'),
    temperature: parseFloat(process.env.TEMPERATURE || '0.7'),
    debug: process.env.DEBUG === "true",
    logLevel: process.env.LOG_LEVEL || "info",
    defaultSubtasks: parseInt(process.env.DEFAULT_SUBTASKS || "3"),
    defaultPriority: process.env.DEFAULT_PRIORITY || "medium",
    projectName: process.env.PROJECT_NAME || "Task Master",
    projectVersion: "1.5.0" // Version should be hardcoded
  };
  ```

## Logging Utilities

- **Log Levels**:
  - ✅ DO: Support multiple log levels (debug, info, warn, error)
  - ✅ DO: Use appropriate icons for different log levels
  - ✅ DO: Respect the configured log level
  - ❌ DON'T: Add direct console.log calls outside the logging utility

  ```javascript
  // ✅ DO: Implement a proper logging utility
  const LOG_LEVELS = {
    debug: 0,
    info: 1,
    warn: 2,
    error: 3
  };
  
  function log(level, ...args) {
    const icons = {
      debug: chalk.gray('🔍'),
      info: chalk.blue('ℹ️'),
      warn: chalk.yellow('⚠️'),
      error: chalk.red('❌'),
      success: chalk.green('✅')
    };
    
    if (LOG_LEVELS[level] >= LOG_LEVELS[CONFIG.logLevel]) {
      const icon = icons[level] || '';
      console.log(`${icon} ${args.join(' ')}`);
    }
  }
  ```

## File Operations

- **Error Handling**:
  - ✅ DO: Use try/catch blocks for all file operations
  - ✅ DO: Return null or a default value on failure
  - ✅ DO: Log detailed error information
  - ❌ DON'T: Allow exceptions to propagate unhandled

  ```javascript
  // ✅ DO: Handle file operation errors properly
  function writeJSON(filepath, data) {
    try {
      fs.writeFileSync(filepath, JSON.stringify(data, null, 2));
    } catch (error) {
      log('error', `Error writing JSON file ${filepath}:`, error.message);
      if (CONFIG.debug) {
        console.error(error);
      }
    }
  }
  ```

## Task-Specific Utilities

- **Task ID Formatting**:
  - ✅ DO: Create utilities for consistent ID handling
  - ✅ DO: Support different ID formats (numeric, string, dot notation)
  - ❌ DON'T: Duplicate formatting logic across modules

  ```javascript
  // ✅ DO: Create utilities for common operations
  /**
   * Formats a task ID as a string
   * @param {string|number} id - The task ID to format
   * @returns {string} The formatted task ID
   */
  function formatTaskId(id) {
    if (typeof id === 'string' && id.includes('.')) {
      return id; // Already formatted as a string with a dot (e.g., "1.2")
    }
    
    if (typeof id === 'number') {
      return id.toString();
    }
    
    return id;
  }
  ```

- **Task Search**:
  - ✅ DO: Implement reusable task finding utilities
  - ✅ DO: Support both task and subtask lookups
  - ✅ DO: Add context to subtask results

  ```javascript
  // ✅ DO: Create comprehensive search utilities
  /**
   * Finds a task by ID in the tasks array
   * @param {Array} tasks - The tasks array
   * @param {string|number} taskId - The task ID to find
   * @returns {Object|null} The task object or null if not found
   */
  function findTaskById(tasks, taskId) {
    if (!taskId || !tasks || !Array.isArray(tasks)) {
      return null;
    }
    
    // Check if it's a subtask ID (e.g., "1.2")
    if (typeof taskId === 'string' && taskId.includes('.')) {
      const [parentId, subtaskId] = taskId.split('.').map(id => parseInt(id, 10));
      const parentTask = tasks.find(t => t.id === parentId);
      
      if (!parentTask || !parentTask.subtasks) {
        return null;
      }
      
      const subtask = parentTask.subtasks.find(st => st.id === subtaskId);
      if (subtask) {
        // Add reference to parent task for context
        subtask.parentTask = { 
          id: parentTask.id, 
          title: parentTask.title,
          status: parentTask.status
        };
        subtask.isSubtask = true;
      }
      
      return subtask || null;
    }
    
    const id = parseInt(taskId, 10);
    return tasks.find(t => t.id === id) || null;
  }
  ```

## Cycle Detection

- **Graph Algorithms**:
  - ✅ DO: Implement cycle detection using graph traversal
  - ✅ DO: Track visited nodes and recursion stack
  - ✅ DO: Return specific information about cycles

  ```javascript
  // ✅ DO: Implement proper cycle detection
  /**
   * Find cycles in a dependency graph using DFS
   * @param {string} subtaskId - Current subtask ID
   * @param {Map} dependencyMap - Map of subtask IDs to their dependencies
   * @param {Set} visited - Set of visited nodes
   * @param {Set} recursionStack - Set of nodes in current recursion stack
   * @returns {Array} - List of dependency edges that need to be removed to break cycles
   */
  function findCycles(subtaskId, dependencyMap, visited = new Set(), recursionStack = new Set(), path = []) {
    // Mark the current node as visited and part of recursion stack
    visited.add(subtaskId);
    recursionStack.add(subtaskId);
    path.push(subtaskId);
    
    const cyclesToBreak = [];
    
    // Get all dependencies of the current subtask
    const dependencies = dependencyMap.get(subtaskId) || [];
    
    // For each dependency
    for (const depId of dependencies) {
      // If not visited, recursively check for cycles
      if (!visited.has(depId)) {
        const cycles = findCycles(depId, dependencyMap, visited, recursionStack, [...path]);
        cyclesToBreak.push(...cycles);
      } 
      // If the dependency is in the recursion stack, we found a cycle
      else if (recursionStack.has(depId)) {
        // The last edge in the cycle is what we want to remove
        cyclesToBreak.push(depId);
      }
    }
    
    // Remove the node from recursion stack before returning
    recursionStack.delete(subtaskId);
    
    return cyclesToBreak;
  }
  ```

## Export Organization

- **Grouping Related Functions**:
  - ✅ DO: Export all utility functions in a single statement
  - ✅ DO: Group related exports together
  - ✅ DO: Export configuration constants
  - ❌ DON'T: Use default exports

  ```javascript
  // ✅ DO: Organize exports logically
  export {
    // Configuration
    CONFIG,
    LOG_LEVELS,
    
    // Logging
    log,
    
    // File operations
    readJSON,
    writeJSON,
    
    // String manipulation
    sanitizePrompt,
    truncate,
    
    // Task utilities
    readComplexityReport,
    findTaskInComplexityReport,
    taskExists,
    formatTaskId,
    findTaskById,
    
    // Graph algorithms
    findCycles,
  };
  ```

Refer to [`utils.js`](mdc:scripts/modules/utils.js) for implementation examples and [`new_features.mdc`](mdc:.cursor/rules/new_features.mdc) for integration guidelines. 

---
> Source: [longmaba/task-master-ai-gemini](https://github.com/longmaba/task-master-ai-gemini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
