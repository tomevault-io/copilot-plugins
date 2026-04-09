## cursor-agentic-ai

> *Automate Your Vision into Reality - Build anything!*

# 🚀 DafnckMachine - Agentic Coding Framework

*Automate Your Vision into Reality - Build anything!*
*Transforming software / app / saas / game development with spec-driven, AI-powered agentic workflows.*
*Optimize by daihung.pham@gmail.com

main repo: [https://github.com/DafnckStudio/Agentic-Coding-Framework/commits/main/](https://github.com/DafnckStudio/Agentic-Coding-Framework/commits/main/)

guide : 
[https://www.youtube.com/watch?v=ayzQ4LXUUsA](https://www.youtube.com/watch?v=ayzQ4LXUUsA)

[00_Getting_Started.mdc](.cursor/rules/01__AI-RUN/00_Getting_Started.mdc)'` with '🎩 Head Orchestrator' 

![Build Status](https:/img.shields.io/badge/build-passing-brightgreen)
![License](https:/img.shields.io/badge/license-MIT-blue)
![Version](https:/img.shields.io/badge/version-1.0.0-informational)
![Contributions Welcome](https:/img.shields.io/badge/contributions-welcome-orange)

[00_Getting_Started.mdc](.cursor/rules/01__AI-RUN/00_Getting_Started.mdc)'`

---

## 🌟 Overview: What is Agentic Coding Framework?

Welcome to **Agentic Coding Framework** – a revolutionary framework designed to orchestrate fully automated, specification-driven software development using advanced AI agents. This project isn't just about generating code; it's about establishing a robust, repeatable, and high-quality process that takes your initial idea and transforms it into a functional product with minimal manual intervention.

Imagine defining your project's requirements and then watching as AI agents meticulously research, plan, document, and build it, adhering strictly to your specifications. That's the power of Agentic Coding Framework. It leverages a structured, phase-by-phase approach, guiding AI through market analysis, concept definition, Product Requirements Document (PRD) generation, technical specification creation, task breakdown, and finally, agentic code generation.

The result? Unprecedented consistency, adherence to detailed specs, and the automation of complex development workflows, freeing human developers to focus on innovation and high-level strategy.

## 🤔 Why Use Agentic Coding Framework?

This framework is built to solve key challenges in modern software development, especially when leveraging AI:

*   **Consistency & Quality:** Ensures that development adheres to predefined standards and specifications, reducing variability and improving the quality of outputs.
*   **Full-Cycle Automation:** Automates the entire pre-development lifecycle, from idea refinement and market research to PRD and technical spec generation.
*   **Enhanced Agentic Coding:** Provides a structured environment for AI coding agents (like Cline, Cursor, Windsurf) to perform "Agentic Coding" – where the AI takes on significant responsibility for the development lifecycle based on clear logic and instructions.
*   **Reduced Tedium:** Automates the creation of comprehensive documentation and detailed task lists, which are often time-consuming but critical for project success.
*   **Scalable Planning:** The structured approach allows for complex projects to be broken down methodically, making them manageable for AI-driven development.
*   **Clear Human Oversight:** While highly automated, the workflow includes defined points for human review and validation, ensuring the project stays aligned with the vision.
*   **Rapid Prototyping & Iteration:** The detailed planning and documentation enable faster transitions into development and provide a solid foundation for quick iterations.

## ✨ Core Features

*   🧠 **AI-Driven Idea to PRD:** Guides AI agents through a comprehensive process from initial idea capture, market research, and core concept definition to a detailed Product Requirements Document (PRD).
*   📄 **Automated Specification Generation:** AI agents update and populate technical documentation and feature specifications based on the PRD and project context.
*   📊 **Systematic Task Breakdown:** Leverages an AI orchestrator (Roo) to decompose the PRD and technical specs into a structured hierarchy of epics, tasks, and sub-tasks, ready for implementation.
*   🤖 **Agentic Code Generation Support:** Designed to feed detailed, validated specifications directly to AI coding agents for implementation, ensuring code aligns with requirements.
*   🔗 **MCP Integration:** Built to utilize Model Context Protocol (MCP) servers, allowing AI agents to access external tools, APIs, and real-time data for research, coding, and more.
*   🔄 **Iterative Workflow with Validation:** Incorporates human validation points at critical stages to ensure alignment and quality.
*   🛠️ **Customizable Framework:** Easily adaptable prompts and documentation templates to suit specific project needs and coding standards.

## ⚙️ How It Works: The Agentic Flow

Agentic Coding Framework operates on the principle of "Agentic Coding Logic," a structured methodology that guides AI agents through distinct development phases. The process is typically orchestrated by an `AutoPilot` script ([01_AutoPilot.mdc](.cursor/rules/01__AI-RUN/01_AutoPilot.mdc)vokes a series of detailed prompts.

1.  **Initialization & Idea Capture:** The user provides an initial idea, which the AI refines through targeted questions.
2.  **Market Research:** The AI, acting as a market analyst, researches the idea's viability using integrated tools.
3.  **Core Concept Definition:** The idea is further refined into a solid concept based on market insights.
4.  **PRD Generation:** A comprehensive Product Requirements Document is created.
5.  **Technical Documentation & Specs:** Existing documentation templates within the project (in `.cursor/rules/02__AI-DOCS/` and `.cursor/rules/03__SPECS/`) are updated by the AI with project-specific details derived from the PRD. This includes architecture, coding conventions, feature specs, etc. The AI is guided by [AI_Coding_Agent_Optimization.mdc](.cursor/rules/02__AI-DOCS/Documentation/AI_Coding_Agent_Optimization.mdc).
6.  **Task Management:** The PRD and specs are broken down into a detailed task list ([tasks.json](.cursor/rules/tasks/tasks.json) orchestrator, following the workflow defined in [Roo_Task_Workflow.mdc](.cursor/rules/02__AI-DOCS/TaskManagement/Roo_Task_Workflow.mdc).
7.  **Implementation:** AI coding agents take the structured tasks and specifications to build the project.
8.  **Testing:** The implemented features are thoroughly tested against specifications (using logic from `[08_Testing.mdc](.cursor/rules/01__AI-RUN/08_Testing.mdc)`).
9.  **Deployment:** The validated application is deployed to the production environment (using logic from `[09_Deployment.mdc](.cursor/rules/01__AI-RUN/09_Deployment.mdc)`).

For a deeper dive into the underlying logic and phase-by-phase execution, please refer to [logic.mdc](.cursor/rules/logic.mdc) and the detailed roadmap below.

## 🚀 Getting Started

1.  **Fork this Repository:** This creates your project's dedicated workspace.
2.  **Configure MCP Servers & API Keys:**
    *   The file [MCP-Server.json](.cursor/rules/01__AI-RUN/Template/MCP-Server.json) lists the MCP servers used.
    *   **CRITICAL:** Replace placeholder API keys (e.g., `YOUR_OPENAI_API_KEY_HERE`) with your actual keys.
    *   **SECURITY BEST PRACTICE:** Do **NOT** commit actual API keys directly. Copy [MCP-Server.json](.cursor/rules/01__AI-RUN/Template/MCP-Server.json) to a local, gitignored file (e.g., `mcp_config.json`) and configure your AI agent (Cursor, Cline, Windsurf) to use this local file.
3.  **Customize Core Guidelines (Highly Recommended):**
    *   Edit [AI_Coding_Agent_Optimization.mdc](.cursor/rules/02__AI-DOCS/Documentation/AI_Coding_Agent_Optimization.mdc). This document is *paramount* for guiding the AI's coding style, architectural choices, and overall behavior. Tailor it to your project's specific needs.
4.  **Initiate the Workflow:**
    *   Open your AI agent (Cursor, Cline, Windsurf).
    *   Run the following prompt:
        ```
        Read the full codebase and let's get started with `.cursor/rules/01__AI-RUN/00_Getting_Started.mdc`
        ```
    *   Alternatively, for a fully automated run, start with [01_AutoPilot.mdc](.cursor/rules/01__AI-RUN/01_AutoPilot.mdc).

For detailed step-by-step instructions, see [00_Getting_Started.mdc](.cursor/rules/01__AI-RUN/00_Getting_Started.mdc).

## 📚 Key Documentation

*   **Core Workflow & Logic:**
    *   [00_Getting_Started.mdc](.cursor/rules/01__AI-RUN/00_Getting_Started.mdc): Detailed guide to the workflow phases.
    *   [01_AutoPilot.mdc](.cursor/rules/01__AI-RUN/01_AutoPilot.mdc): The main script for orchestrating the automated workflow.
    *   [logic.mdc](.cursor/rules/logic.mdc): High-level overview of the project's operational logic.
    *   [Roadmap: From 0 to 1](#roadmap-from-0-to-1-with-full-agentic-ai-coding-assistance) (section below in this README)
*   **AI Agent Guidance:**
    *   [AI_Coding_Agent_Optimization.mdc](.cursor/rules/02__AI-DOCS/Documentation/AI_Coding_Agent_Optimization.mdc): **Essential reading.** Defines how the AI should code, design, and behave.
    *   [AI_Design_Agent_Optimization.mdc](.cursor/rules/02__AI-DOCS/Documentation/AI_Design_Agent_Optimization.mdc): Guidelines for AI design tasks.
    *   [AI_Task_Management_Optimization.mdc](.cursor/rules/02__AI-DOCS/Documentation/AI_Task_Management_Optimization.mdc): Optimizing AI for task management.
*   **Task Management:**
    *   [Roo_Task_Workflow.mdc](.cursor/rules/02__AI-DOCS/TaskManagement/Roo_Task_Workflow.mdc): Defines the process for task breakdown and management.
    *   [Tasks_JSON_Structure.mdc](.cursor/rules/02__AI-DOCS/TaskManagement/Tasks_JSON_Structure.mdc): Specifies the format for the output `[tasks.json](.cursor/rules/tasks/tasks.json)` file.
*   **Templates & Specifications:**
    *   [PRD_template.mdc](.cursor/rules/01__AI-RUN/Template/PRD_template.mdc): Base structure for the Product Requirements Document.
    *   `.cursor/rules/02__AI-DOCS/` (various subdirectories): Contains templates for architecture, business logic, conventions, etc., that the AI will update.
    *   `.cursor/rules/03__SPECS/` (various subdirectories): Contains templates for feature and bugfix specifications that the AI will update.

## 🛠️ Technology Assumptions

This workflow suggests a default stack of:
*   Frontend: Next.js (TypeScript)
*   Backend/DB: Supabase
*   Styling: Tailwind CSS (with Shadcn/ui inspiration)

These are recommendations and can be adapted. The workflow heavily relies on **Model Context Protocol (MCP) servers** (see [MCP-Context.mdc](.cursor/rules/01__AI-RUN/Template/MCP-Context.mdc)nded AI capabilities.

## 🤝 Contributing / Future Vision (Optional Placeholder)

We envision Agentic Coding Framework evolving into an even more powerful and adaptable platform for AI-driven development. Contributions, ideas, and feedback are welcome!

*   Expanding MCP integrations.
*   More sophisticated AI agent personas and capabilities.
*   Enhanced automated testing and validation frameworks.

---

## 🗺️ Roadmap: From 0 to 1 with Full Agentic AI Coding Assistance

This workflow is designed for comprehensive AI assistance, orchestrated by the [01_AutoPilot.mdc](.cursor/rules/01__AI-RUN/01_AutoPilot.mdc) file, with human intervention points for validation and strategic direction. Here's a conceptual step-by-step journey:

**Phase 0: Initialization & User Idea**

1.  **User:** Forks the template repository for a new project.
2.  **User:** Opens [01_AutoPilot.mdc](.cursor/rules/01__AI-RUN/01_AutoPilot.mdc) in their AI agent (e.g., Cline).
3.  **User:** Shares the `[01_AutoPilot.mdc](.cursor/rules/01__AI-RUN/01_AutoPilot.mdc)` content with the AI agent.
4.  **AI Agent (as `ProjectArchitect`):** Understands "Core Operational Rules." Prompts the user for a brief project idea (1-3 sentences).
5.  **User:** Provides the initial idea.
6.  **AI Agent:** Asks 5-7 targeted questions (from `[01_AutoPilot.mdc](.cursor/rules/01__AI-RUN/01_AutoPilot.mdc)`) to clarify the idea (target users, main problem, key MVP features, business model, tech/design preferences).
7.  **User:** Answers clarification questions.

**Phase 1: Initial Idea Expansion (Logic from [01_Idea.mdc](.cursor/rules/01__AI-RUN/01_Idea.mdc)
8.  **AI Agent:** Based on the initial idea and clarifications, uses the [01_Idea.mdc](.cursor/rules/01__AI-RUN/01_Idea.mdc) logical prompt structure to generate a structured idea document.
9.  **AI Agent:** Saves this document as `[idea_document.mdc](.cursor/rules/projet/01_Idea/idea_document.mdc)` in the project root directory.
10. **AI Agent:** Presents the completed `[idea_document.mdc](.cursor/rules/projet/01_Idea/idea_document.mdc)` to the user for quick validation.
11. **User:** Validates or requests minor adjustments.

**Phase 2: Automated Market Research (Logic from [02_Market_Research.mdc](.cursor/rules/01__AI-RUN/02_Market_Research.mdc)
12. **AI Agent:** Announces it's conducting market research.
13. **AI Agent (as `MarketMaster Pro`):** Internally uses the `[02_Market_Research.mdc](.cursor/rules/01__AI-RUN/02_Market_Research.mdc)` logical prompt, taking `[idea_document.mdc](.cursor/rules/projet/01_Idea/idea_document.mdc)` as input.
14. **AI Agent:** Leverages MCPs (e.g., `context7`, `firecrawl`) as needed to simulate data gathering.
15. **AI Agent:** Generates a comprehensive market analysis.
16. **AI Agent:** Saves this analysis as `[market_research.mdc](.cursor/rules/projet/02_Market_Research/market_research.mdc)` in the folder `./project root directory
17. **AI Agent:** Presents a summary of main findings to the user.
18. **User:** Decides whether to review the full analysis or proceed.

**Phase 3: Core Concept Development (Logic from [core_concept.mdc](.cursor/rules/projet/03_Core_Concept/core_concept.mdc)
19. **AI Agent:** Announces it's developing the core concept.
20. **AI Agent (as `ConceptForge`):** Internally uses the `[core_concept.mdc](.cursor/rules/projet/03_Core_Concept/core_concept.mdc)` logical prompt, taking `[idea_document.mdc](.cursor/rules/projet/01_Idea/idea_document.mdc)` and `[market_research.mdc](.cursor/rules/projet/02_Market_Research/market_research.mdc)` as inputs.
21. **AI Agent:** Synthesizes the idea and market research into a refined concept.
22. **AI Agent:** Saves this concept as  in the folder `./project root directory
23. **AI Agent:** Presents the value proposition and key feature matrix to the user.
24. **User:** Validates the core concept before proceeding.

**Phase 4: PRD Generation (Logic from [04_PRD_Generation.mdc](.cursor/rules/01__AI-RUN/04_PRD_Generation.mdc)
25. **AI Agent:** Announces it's creating the Product Requirements Document (PRD).
26. **AI Agent (as `PRDarchitect`):** Internally uses the `[04_PRD_Generation.mdc](.cursor/rules/01__AI-RUN/04_PRD_Generation.mdc)` logical prompt, taking `[core_concept.mdc](.cursor/rules/projet/03_Core_Concept/core_concept.mdc)` as main input and referencing the [PRD_template.mdc](.cursor/rules/01__AI-RUN/Template/PRD_template.mdc).
27. **AI Agent:** Generates a complete PRD.
28. **AI Agent:** Saves the PRD in ) in the folder `./project root directory
29. **AI Agent:** Presents an executive summary with links to full sections.
30. **User:** Decides whether to review specific sections or proceed.

**Phase 5: Technical Documentation (Logic from [05_Specs_Docs.mdc](.cursor/rules/01__AI-RUN/05_Specs_Docs.mdc)
31. **AI Agent:** Announces it's updating technical documentation.
32. **AI Agent (as `TechDocNavigator`):** Internally uses the `[05_Specs_Docs.mdc](.cursor/rules/01__AI-RUN/05_Specs_Docs.mdc)` logical prompt, taking `[project_prd.mdc](.cursor/rules/projet/PRD_template/project_prd.mdc)` as input.
33. **AI Agent:** **Updates** existing files in `.cursor/rules/02__AI-DOCS/` and `.cursor/rules/03__SPECS/` using their structure as templates, integrating information from the PRD and technical research (via MCPs if needed). Also references [AI_Coding_Agent_Optimization.mdc](.cursor/rules/02__AI-DOCS/Documentation/AI_Coding_Agent_Optimization.mdc).
34. **AI Agent:** Creates or updates `.cursor/rules/03__SPECS/documentation_index.mdc`.
35. **AI Agent:** Presents a summary of the tech stack and key integrations. ([01_AutoPilot.mdc](.cursor/rules/01__AI-RUN/01_AutoPilot.mdc) instructs to proceed automatically).

**Phase 6: Task Management (Workflow: [Roo_Task_Workflow.mdc](.cursor/rules/02__AI-DOCS/TaskManagement/Roo_Task_Workflow.mdc), Structure: [Tasks_JSON_Structure.mdc](.cursor/rules/02__AI-DOCS/TaskManagement/Tasks_JSON_Structure.mdc)
36. **AI Agent:** Announces it's breaking down the project into tasks.
37. **AI Agent:** Follows the workflow in [Roo_Task_Workflow.mdc](.cursor/rules/02__AI-DOCS/TaskManagement/Roo_Task_Workflow.mdc), taking `[project_prd.mdc](.cursor/rules/projet/PRD_template/project_prd.mdc)` and updated technical documents as inputs.
38. **AI Agent:** Interacts with Roo Orchestrator (potentially engaging Roo Code mode) to initialize the project, create epics from PRD features, and decompose each epic into tasks and sub-tasks, as detailed in [Roo_Task_Workflow.mdc](.cursor/rules/02__AI-DOCS/TaskManagement/Roo_Task_Workflow.mdc).
39. **AI Agent:** Saves the task hierarchy in [tasks.json](.cursor/rules/tasks/tasks.json), adhering to the structure defined in [Tasks_JSON_Structure.mdc](.cursor/rules/02__AI-DOCS/TaskManagement/Tasks_JSON_Structure.mdc).
40. **AI Agent:** Presents high-level epics and priority tasks.
41. **User:** May request to modify task priorities.

**Phase 7: Implementation (Logic from [07_Start_Building.mdc](.cursor/rules/01__AI-RUN/07_Start_Building.mdc)
42. **AI Agent:** Announces it's starting implementation.
43. **AI Agent (as `ImplementationArchitect`):** Internally uses the `[07_Start_Building.mdc](.cursor/rules/01__AI-RUN/07_Start_Building.mdc)` logical prompt.
44. **AI Agent:** If not done in Phase 5, creates/updates detailed specifications for each feature in `.cursor/rules/03__SPECS/features/`.
45. **AI Agent:** Requests the next task to implement from Roo Orchestrator.
46. **AI Agent:** Implements priority tasks first, using relevant MCPs (e.g., `@21st-dev/magic` for UI, `github` for versioning, `supabase` for DB).
47. **AI Agent:** Writes code, tests, and associated documentation, adhering to [AI_Coding_Agent_Optimization.mdc](.cursor/rules/02__AI-DOCS/Documentation/AI_Coding_Agent_Optimization.mdc) and PRD guidelines.
48. **AI Agent:** Provides regular progress updates.
49. **AI Agent:** Presents completed features for validation.
50. **User:** Validates implemented features.
51. **AI Agent:** Marks tasks as complete by informing Roo Orchestrator.
52. **Repeat (steps 45-51):** The task implementation cycle continues until all MVP tasks are completed.

**Phase 8: Testing (Logic from [08_Testing.mdc](.cursor/rules/01__AI-RUN/08_Testing.mdc)
53. **AI Agent:** Announces it's initiating the testing phase.
54. **AI Agent (as `QualityGuardian`):** Internally uses the `[08_Testing.mdc](.cursor/rules/01__AI-RUN/08_Testing.mdc)` logical prompt. Systematically tests each implemented feature based on the PRD, technical specifications, and defined test cases. This includes unit, integration, and end-to-end tests.
55. **AI Agent:** Verifies features operate as expected, system calls are correct, and UI adheres to design conventions.
56. **AI Agent:** Sets up a preview environment.
57. **User & AI Agent:** Review the preview, conduct final user acceptance testing (UAT).
58. **AI Agent:** Addresses any issues identified during testing and UAT, iterating with the user until satisfaction.
59. **AI Agent:** Confirms all features are stable and ready for deployment.

**Phase 9: Deployment (Logic from [09_Deployment.mdc](.cursor/rules/01__AI-RUN/09_Deployment.mdc)
60. **AI Agent:** Announces it's initiating the deployment phase.
61. **AI Agent (as `DeployMaster`):** Internally uses the `[09_Deployment.mdc](.cursor/rules/01__AI-RUN/09_Deployment.mdc)` logical prompt.
62. **AI Agent:** Follows the deployment plan in `[project_prd.mdc](.cursor/rules/projet/PRD_template/project_prd.mdc)` and the detailed steps in the project-specific `[deployment_guide_template.mdc](.cursor/rules/02__AI-DOCS/Deployment/deployment_guide_template.mdc)`.
63. **AI Agent:** Executes deployment to the production environment, monitors the process, and performs post-deployment verification (smoke tests).
64. **AI Agent:** Confirms successful deployment and stability in production.

**Phase 10: Iteration (End of 0->1 Cycle)**

65. **User & AI Agent:** Collect user feedback on the deployed product.
66. **User:** Decides on next steps/iterations.
67. **If New Iteration:** The cycle can restart at Phase 0 or Phase 1, using learnings and feedback to refine `[idea_document.mdc](.cursor/rules/projet/01_Idea/idea_document.mdc)` or directly `[core_concept.mdc](.cursor/rules/projet/03_Core_Concept/core_concept.mdc)`. Existing documents (`[project_prd.mdc](.cursor/rules/projet/PRD_template/project_prd.mdc)`, [tasks.json](.cursor/rules/tasks/tasks.json), etc.) are updated.

This roadmap outlines the ideal flow. The AI agent is designed to be proactive, but the "Core Operational Rules" in [01_AutoPilot.mdc](.cursor/rules/01__AI-RUN/01_AutoPilot.mdc) require it to seek clarification for ambiguities and report errors, ensuring the user remains the ultimate supervisor.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phamhung075)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/phamhung075)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
