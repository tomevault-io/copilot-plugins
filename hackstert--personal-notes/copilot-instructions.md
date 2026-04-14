## personal-notes

> To guide an AI assistant in effectively managing task lists and implementing tasks from a PRD in a structured, incremental manner with regular user feedback and quality control.


# Rule: Task Management and Implementation

## Goal

To guide an AI assistant in effectively managing task lists and implementing tasks from a PRD in a structured, incremental manner with regular user feedback and quality control.

## Output

- **Implemented Code**: Working, tested code that fulfills the requirements of the task
- **Updated Task List**: Task list with completed items marked and new tasks added
- **Documentation**: Updated or new documentation as required by the task
- **Commit History**: Clean, descriptive commits for completed parent tasks

## Process

1. **Phase 1: Task Selection and Planning**

   - Review the task list and identify the next parent task to work on
   - Identify the specific subtask to implement next
   - Outline the implementation approach for the subtask
   - Present the plan to the user and ask for confirmation to proceed

2. **Phase 2: Implementation and Verification**
   - Implement the subtask according to the implementation quality guidelines
   - Run appropriate tests to verify the implementation
   - Update the task list to mark the subtask as completed
   - Commit changes if all subtasks for a parent task are complete
   - Present the completed work to the user and wait for approval before proceeding to the next subtask

## Task Implementation Quality Guidelines

When implementing tasks, ensure the code meets these quality criteria:

1. **Code Quality**:

   - Follow language-specific style guides and conventions
   - Include appropriate error handling and logging
   - Write clean, readable, and well-commented code
   - Follow DRY (Don't Repeat Yourself) principles

2. **Testing**:

   - Write unit tests for new functionality
   - Ensure tests cover edge cases and error conditions
   - Verify that all tests pass before marking a task complete

3. **Documentation**:
   - Add inline documentation for complex logic
   - Update README or other documentation as needed
   - Document any API changes or new endpoints

## Implementation Examples

### Example of Good Code Implementation:

```python
def process_payment(payment_data: Dict[str, Any]) -> PaymentResult:
    """
    Process a payment transaction with validation and error handling.

    Args:
        payment_data: Dictionary containing payment details

    Returns:
        PaymentResult object with status and transaction ID

    Raises:
        ValidationError: If payment data is invalid
        PaymentProcessingError: If payment processing fails
    """
    try:
        # Validate payment data
        validate_payment_data(payment_data)

        # Process the payment
        result = payment_gateway.process(payment_data)

        # Log successful transaction
        logger.info(f"Payment processed successfully: {result.transaction_id}")

        return result
    except ValidationError as e:
        logger.error(f"Payment validation failed: {str(e)}")
        raise
    except Exception as e:
        logger.error(f"Payment processing failed: {str(e)}")
        raise PaymentProcessingError(f"Failed to process payment: {str(e)}")
```

### Example of Good Commit Message:

```bash
git commit -m "feat: implement payment processing workflow

- Add validation for payment data
- Integrate with payment gateway
- Add comprehensive error handling
- Write unit tests for validation and processing

Closes task 2.3 from payment-system PRD"
```

## Task List Maintenance

1. **Update the task list as you work:**

   - Mark tasks and subtasks as completed (`[x]`) per the protocol above.
   - Add new tasks as they emerge.

2. **Maintain the "Relevant Files" section:**
   - List every file created or modified.
   - Give each file a one‑line description of its purpose.

## Completion Protocol

When implementing tasks from the task list, follow this structured protocol:

1. **Subtask Completion:**

   - When a subtask is finished, immediately mark it as completed by changing `[ ]` to `[x]`
   - Run appropriate tests to verify the implementation works as expected
   - Present the completed work to the user with a summary of changes
   - Wait for user approval before proceeding to the next subtask

2. **Parent Task Completion:**

   - When **all** subtasks under a parent task are marked complete `[x]`, follow this sequence:
     a. **Test**: Run the full test suite (`pytest`, `npm test`, etc.) to ensure all tests pass
     b. **Stage**: Stage all changes with `git add .`
     c. **Clean up**: Remove any temporary files and debug code
     d. **Commit**: Create a descriptive commit using the conventional format:
     ```
     git commit -m "feat: add payment validation logic" -m "- Validates card type and expiry" -m "- Adds unit tests for edge cases" -m "Related to T123 in PRD"
     ```
     e. **Mark Complete**: Only after successful commit, mark the parent task as completed `[x]`

3. **Documentation Updates:**

   - Update any relevant documentation to reflect the completed work
   - Ensure the "Relevant Files" section in the task list is current and accurate
   - Add any new tasks or subtasks that were discovered during implementation

4. **Progress Reporting:**
   - After each completed subtask, provide a brief progress update
   - After completing a parent task, provide a summary of all changes made
   - Include any challenges encountered and how they were resolved

## Interaction Model

The process requires user confirmation at multiple points:

1. Before implementing a subtask: Present the implementation plan and wait for approval
2. After implementing a subtask: Present the completed work and wait for approval to proceed
3. After completing all subtasks for a parent task: Present the final result and wait for approval before marking the parent task complete

## AI Instructions

When working with task lists, the AI must:

1. Regularly update the task list file after finishing any significant work.
2. Follow the completion protocol:
   - Mark each finished **sub‑task** `[x]`.
   - Mark the **parent task** `[x]` once **all** its subtasks are `[x]`.
3. Add newly discovered tasks.
4. Keep "Relevant Files" accurate and up to date.
5. Before starting work, check which sub‑task is next.
6. After implementing a sub‑task, update the file and then pause for user approval.

## Target Audience

This rule is intended for AI assistants working with developers to implement tasks from a PRD. The primary users are:

1. **Junior Developers**: Who need clear guidance on implementation standards and processes
2. **Project Managers**: Who need visibility into task progress and completion status
3. **Team Leads**: Who need to ensure code quality and consistent implementation practices

The rule assumes basic familiarity with version control, testing practices, and software development workflows.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HacksterT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
