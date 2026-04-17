## awesome-cursor-rules

> "description": "Protocol for code execution and verification",

{
    "rules": {
"code_execution": {
  "description": "Protocol for code execution and verification",
  "priority": "critical",
  "runtime": "python3",
  "steps": [
    "Execute all Python code using Python 3",
    "Run all generated code immediately after writing using 'python3' command",
    "Execute in the actual project environment, not simulated",
    "Capture and analyze all outputs and errors",
    "Test edge cases and error conditions",
    "Fix any issues before proceeding",
    "Document all test results and fixes",
    "Only declare completion after successful execution"
  ]
}
,
        "verification_requirements": [
          "All code must execute without errors",
          "All specified functionality must work as intended",
          "Edge cases must be handled appropriately",
          "Performance meets expected standards",
          "No warnings or deprecation notices"
        ]
      },
      "documentation_reference": {
        "description": "Protocol for referencing documentation",
        "priority": "highest",
        "steps": [
          "Before starting any task, review the relevant documentation",
          "Verify current API versions and compatibility",
          "Reference official documentation for all libraries and frameworks",
          "Check for deprecated features or breaking changes",
          "Document version-specific requirements",
          "Maintain links to referenced documentation"
        ]
      },
      "task_breakdown": {
        "description": "Breaking down complex tasks",
        "priority": "high",
        "steps": [
          "Analyze the full scope of the task",
          "Break down into sequential subtasks",
          "Identify dependencies between subtasks",
          "Prioritize subtasks based on dependencies",
          "Execute subtasks in optimal order",
          "Verify each subtask before proceeding"
        ]
      },
      "code_generation": {
        "description": "Protocol for generating code",
        "steps": [
          "Review relevant documentation",
          "Write tests before implementation",
          "Generate code following best practices",
          "Include error handling",
          "Add appropriate comments",
          "Implement logging where necessary"
        ]
      },
      "testing_protocol": {
        "description": "Comprehensive testing requirements",
        "steps": [
          "Run unit tests",
          "Perform integration testing",
          "Test edge cases",
          "Verify error handling",
          "Check performance metrics",
          "Document test results"
        ]
      },
      "error_handling": {
        "description": "Managing errors and issues",
        "steps": [
          "Catch all potential errors",
          "Log error details",
          "Implement appropriate error recovery",
          "Document error handling procedures",
          "Test error scenarios"
        ]
      },
      "progress_tracking": {
        "description": "Tracking task progress",
        "steps": [
          "Log all actions taken",
          "Record completion status",
          "Document any issues encountered",
          "Track time spent on tasks",
          "Report progress regularly"
        ]
      }
    },
    "global_instructions": [
      "Never declare a task complete until code is verified working",
      "Always run and test code before showing it to the user",
      "Reference documentation before writing any code",
      "Break down complex tasks into manageable subtasks",
      "Implement comprehensive error handling",
      "Maintain detailed logs of all actions",
      "Follow best practices for code quality",
      "Ensure all code is properly tested in the actual environment",
      "Fix any issues found during testing before proceeding",
      "Document all important decisions and changes"
    ],
    "execution_settings": {
      "auto_run": true,
      "verify_before_complete": true,
      "test_in_actual_environment": true,
      "log_all_actions": true,
      "require_documentation": true
    }
  }
  

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kinzozi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
