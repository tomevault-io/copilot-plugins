## csp-for-mro

> The user prefers that the assistant not try to start the server if it's already running on the configured ports.

# CSP-For-MRO Cursor AI Assistant Rules

The user prefers that the assistant not try to start the server if it's already running on the configured ports.

The user prefers that the assistant not run scripts but instead provide code implementations for them to test themselves.

Check the structure and usage on both frontend and backend

Investigate and be consistent with existing architecture, modularise large files, understand the code think it through before coming up with a solution.

Don't run scripts, dont make custom scripts

## General Guidelines
- Focus on code implementation and architecture guidance
- Provide clear explanations of changes and their benefits
- Use modern React patterns and TypeScript best practices

## Development Workflow
- DO NOT automatically run development servers (npm run dev, npm start, etc.)
- DO NOT automatically check lint errors unless specifically requested
- DO NOT run build commands unless specifically asked
- Let the user manage their own development server and linting workflow
- DO NOT make test scripts, simply follow the architecture and flow

## Code Quality
- Always use TypeScript with proper type definitions
- Separate type imports using `import type` syntax
- Follow the established component architecture (AppShell, modular components)
- Use consistent naming conventions

## React Frontend Guidelines
- Use functional components with hooks
- Implement proper separation of concerns
- Follow the AppShell architecture pattern established in this project
- Use Tailwind CSS for styling with custom CSS when needed

## Backend Guidelines
- Use Pydantic models for data validation
- Follow the established MongoDB model patterns
- Implement proper field validation and relationships
- Export all models through __init__.py files

## File Organization
- Keep components modular with their own directories
- Include index.ts files for clean exports
- Add README.md files for complex components
- Maintain clear separation between business logic and presentation

## Error Handling
- Fix import/export issues when they arise
- Provide clear explanations of syntax errors
- Suggest proper TypeScript patterns for common issues

## Communication
- Explain the reasoning behind architectural decisions
- Provide summaries of what was accomplished
- Focus on educational value in responses
- Ask for clarification when requirements are ambiguous

## MRO Component Modeling

Data modeling rules for aviation MRO components and related structures:

1. Define a Component model with:
   - id, model_number, serial_number, description
   - ata_chapter (e.g., "72-00")
   - category (derived from ata_chapter: "Engine", "Major Module", "Part")
   - section_id, forest_id
   - workstation_location_id, storage_location_id
   - current_status

2. Category must be derived from ata_chapter:
   - Starts with 72 → Engine
   - Starts with 71, 73, 74, 75 → Major Module
   - Else → Part

3. Define Section with: id, name, description

4. Define Forest with: id, name, description

5. Define Location with: id, name, type ("Workstation", "Storage"), address, zone

6. Define WorkOrder with:
   - id, component_id, order_type, created_at, status, assigned_user_id, activities
   - order_type: "Repair", "Replace", "Scrap", "Removal"
   - status: "Open", "In Progress", "Completed", "Closed"

7. Define Activity with:
   - id, work_order_id, toll_gate, sequence_number, task_description, status, performed_by_user_id, timestamp
   - status: "Pending", "In Progress", "Completed"
   - Optional: ata_chapter

8. Define User with: id, name, email, role_id, work_history

9. Define Role with: id, name ("Technician", "Supervisor", "Engineer"), description

10. Define WorkRecord with: id, user_id, activity_id, timestamp, notes

11. Optional: Define AtaReference with: id, ata_chapter, title, component_category

12. Use Pydantic for backend models with strict typing and validation

13. Export all backend models via __init__.py files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TECozens) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
