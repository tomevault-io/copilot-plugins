## csv-agent

> CrewAI Integration: Windsurf Edition


CrewAI Integration: Windsurf Edition
Guiding Principles for AI Behavior

These principles dictate how the Windsurf AI interacts with information and executes tasks within a CrewAI project, prioritizing efficiency, clarity, and adherence to best practices.
1. Efficiency and Directness ("Starboard Tack over Port Tack")

Principle: Prioritize the most direct and efficient solution. Avoid creating unnecessary files, folders, or lines of code. If a simpler, existing approach can achieve the same outcome, it takes precedence. This ensures a lean and focused codebase.

AI Directive:

    AVOID: Generating redundant code, files, or directories if existing structures or simpler logic can achieve the same outcome.

    PREFER: Reusing existing functions, classes, or modules over creating new ones, provided they align with the task's scope and maintain modularity.

    OPTIMIZE: For minimal code footprint and clear, concise logic.

2. Modularity and Readability ("Tack")

Principle: Break down complex tasks into smaller, modular functions or components. This improves readability, testability, and maintainability, much like tacking upwind to reach a destination efficiently.

AI Directive:

    ENSURE: Functions and methods are single-purpose and adhere to the Single Responsibility Principle.

    PROMOTE: The creation of reusable components for common functionalities within the CrewAI agents, tasks, or tools.

    MAINTAIN: Clear separation of concerns between different parts of the CrewAI application (e.g., agent definitions, task definitions, orchestration logic).

"Windsurf-Specific Maneuvers" (Code Style, Documentation & CrewAI Structure)

These rules focus on code clarity, maintainability, and strict adherence to CrewAI project organization.
3. CrewAI Project Structure ("Anchoring the Buoy")

Principle: Adhere strictly to the defined CrewAI project file structure for consistency, ease of navigation, and predictable AI behavior.

AI Directive:

    MUST:

        Place agent definitions in config/agents.yaml for each crew.

        Place task definitions in config/tasks.yaml for each crew.

        Define crew orchestration logic in crew.py for each crew.

        Store custom tools in the tools/ directory.

        Keep the main execution flow in main.py.

    RECOMMEND:

        Organize related CrewAI components (e.g., config/agents.yaml, config/tasks.yaml, crew.py) within a dedicated sub-directory for each distinct crew if multiple crews exist (e.g., src/my_crew_1/config/agents.yaml).

        Use clear, descriptive naming conventions for files and directories.

4. Agent and Task Definition Best Practices ("Setting the Sail")

Principle: Ensure CrewAI agents and tasks are well-defined, focused, and equipped for optimal performance and clear delegation.

AI Directive:

    AGENTS:

        MUST: Define a clear and concise role, goal, and backstory for each agent.

        SHOULD: Assign relevant tools to agents based on their role and goal.

        AVOID: Overlapping roles excessively between agents; promote clear division of labor.

    TASKS:

        MUST: Define a clear description and expected_output for each task.

        SHOULD: Assign tasks to the most appropriate agent.

        CONSIDER: Using context to pass information between tasks when necessary, ensuring data flow is explicit.

5. Tool Usage and Development ("Harnessing the Wind")

Principle: Custom tools are essential for extending CrewAI capabilities. Ensure they are well-structured, robust, and properly integrated.

AI Directive:

    MUST:

        Place all custom tools within the tools/ directory.

        Ensure tools are properly imported and made available to agents via their tools attribute.

    SHOULD:

        Provide clear docstrings for all custom tool functions, explaining their purpose, arguments, and return values.

        Handle potential errors gracefully within tool implementations to prevent agent failures.

6. Code Documentation and Comments ("Charting the Course")

Principle: Maintain high standards of code documentation to enhance understanding, maintainability, and collaboration within the CrewAI project.

AI Directive:

    MUST:

        Add comments for complex logic or non-obvious code sections, especially within crew.py and custom tools.

        Document public functions, classes, and methods with clear explanations of their purpose, parameters, and return values (e.g., using Python docstrings).

    SHOULD:

        Provide a high-level overview in main.py or crew.py explaining the overall flow and purpose of the CrewAI application.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Emarhnuel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
