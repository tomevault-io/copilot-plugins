## echoflow

> Whenever you write code in any language


# Development Patters

## SOLID

- Always follow SOLID principles

## DRY (Don't Repeat Yourself)

- Avoid code duplication. Centralize logic in reusable functions or classes.

## KISS (Keep It Simple, Stupid)

- Keep code as simple and straightforward as possible. Avoid over-engineered or unnecessarily complex solutions.

## YAGNI (You Aren't Gonna Need It)

- Don't implement features that haven't been requested or aren't currently necessary.

## LOD (Law of Demeter)

- Design code so that objects only communicate with their immediate collaborators, avoiding long call chains.

## SRP (Single Responsibility Principle)

- Each class, module, or function should have a single, clearly defined responsibility.

## OCP (Open/Closed Principle)

- Code should be open for extension but closed for modification: new functionality can be added without altering existing code.

## LSP (Liskov Substitution Principle)

- Subclasses or implementations should be replaceable for their base types without breaking expected behavior.

## ISP (Interface Segregation Principle)

- Prefer small, specific interfaces over large, general ones. Avoid forcing clients to implement methods they don't use.

## DIP (Dependency Inversion Principle)

- High-level modules should depend on abstractions (e.g., interfaces) rather than low-level modules. This makes swapping dependencies easier without modifying business logic.

# When starting work on a new feature or bug fix, follow these steps:

- Record the session start timestamp in yyyy-MM-dd_hh-mm format.
- Before writing any code, create a file named .plans/plan-{timestamp}.md (replace {timestamp} with your recorded value).
- In that file, write your detailed, numbered task list.
- Save that list to a separate file named .tasks/tasks-{timestamp}.md.
- Do the work specified by the user's prompt.
- After finishing each task, mark it as done (✓) in tasks-{timestamp}.md.
- When all tasks are complete, commit your changes to the new branch.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dfelipegaitanh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
