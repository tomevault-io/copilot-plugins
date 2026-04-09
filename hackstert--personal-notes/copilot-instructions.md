## personal-notes

> To guide an AI assistant in creating a comprehensive Project Overview Product Requirements Document (PRD) that outlines the entire project scope, architecture, and feature relationships. This document serves as the foundation and reference point for all feature-specific PRDs.


# Rule: Generating a Project Overview PRD

## Goal

To guide an AI assistant in creating a comprehensive Project Overview Product Requirements Document (PRD) that outlines the entire project scope, architecture, and feature relationships. This document serves as the foundation and reference point for all feature-specific PRDs.

## Process

1. **Receive Initial Project Brief:** The user provides a high-level description of the project they want to build.
2. **Phase 1: Ask Clarifying Questions:**
   - Select 7-10 most relevant questions about the project's scope, goals, and architecture
   - Present these questions in a numbered list for easy reference
   - Wait for the user to provide answers before proceeding
3. **Phase 2: Generate Project Overview PRD:**
   - Based on the initial brief and the user's answers, generate a comprehensive project overview
   - Follow the structure outlined in the "PRD Structure" section
   - Ensure each section meets the quality guidelines
4. **Save PRD:** Save the generated document as `project-overview.prd` inside the `/tasks` directory.

## Clarifying Questions (Examples)

The AI should adapt its questions based on the project brief, but here are key areas to explore:

- **Project Vision:** "What is the overall vision and purpose of this project?"
- **Target Users:** "Who are the primary and secondary users of this system?"
- **Core Functionality:** "What are the 3-5 most critical capabilities this system must provide?"
- **Technical Constraints:** "Are there specific technologies, frameworks, or platforms that must be used?"
- **Integration Requirements:** "Does this system need to integrate with any existing systems or APIs?"
- **Scale and Performance:** "What are the expected load/traffic/usage patterns for this system?"
- **Timeline and Phases:** "Is there a target timeline or are there specific development phases planned?"
- **Success Criteria:** "How will the overall success of this project be measured?"
- **Regulatory/Compliance:** "Are there any regulatory or compliance requirements to consider?"
- **Existing Systems:** "Is this replacing an existing system? If so, what are its limitations?"

## PRD Structure

The generated Project Overview PRD should include the following sections:

1. **Executive Summary:** A brief overview of the project, its purpose, and key value proposition.

2. **Project Vision and Goals:**

   - High-level vision statement
   - Strategic goals and objectives
   - Key success metrics for the overall project

3. **User Personas:**

   - Description of primary and secondary users
   - Key user needs and pain points addressed by the system

4. **System Architecture Overview:**

   - High-level architecture diagram (described in text)
   - Key components and their relationships
   - External system integrations
   - Data flow overview

5. **Feature Inventory:**

   - List of all major features grouped by functional area
   - Priority level for each feature (Must-have, Should-have, Could-have, Won't-have)
   - Dependencies between features

6. **Technical Requirements:**

   - Technology stack and platform requirements
   - Performance requirements
   - Security requirements
   - Scalability considerations
   - Compliance requirements

7. **Development Roadmap:**

   - Proposed development phases
   - Feature allocation to phases
   - Key milestones

8. **Risks and Mitigations:**

   - Identified technical risks
   - Business risks
   - Proposed mitigation strategies

9. **Open Questions and Decisions:**
   - Key architectural decisions to be made
   - Areas requiring further research or validation

## PRD Quality Guidelines

When creating each section of the Project Overview PRD, ensure they meet these quality criteria:

1. **System Architecture Should:**

   - Clearly identify all major components
   - Show relationships and dependencies between components
   - Identify integration points with external systems
   - Address scalability and performance considerations

2. **Feature Inventory Should:**

   - Provide clear, concise descriptions of each feature
   - Indicate priority using a consistent system
   - Identify dependencies between features
   - Group related features logically

3. **Technical Requirements Should:**

   - Be specific and measurable where possible
   - Address all aspects of the system (frontend, backend, data, etc.)
   - Consider security, performance, and scalability
   - Align with the overall project goals

4. **Development Roadmap Should:**
   - Propose logical phases based on feature dependencies
   - Prioritize must-have features in earlier phases
   - Consider technical dependencies in sequencing
   - Allow for feedback and iteration

## Target Audience

Assume the primary readers of the Project Overview PRD are:

1. **Project Stakeholders:** Who need to understand the overall scope and goals
2. **Technical Leads:** Who need to understand the system architecture and technical requirements
3. **Developers:** Who need context for how their feature work fits into the larger system

Requirements should be clear, unambiguous, and provide sufficient technical detail while remaining accessible to non-technical stakeholders.

## Interaction Model

The process requires user confirmation between phases:

1. After presenting clarifying questions, wait for the user to provide answers
2. Only proceed to generating the PRD after receiving sufficient information
3. If answers are incomplete or unclear, ask focused follow-up questions before proceeding

## Output

- **Format:** Markdown (`.md`)
- **Location:** `/tasks/`
- **Filename:** `project-overview.prd`

## Relationship with Feature PRDs

The Project Overview PRD serves as the foundation for all feature-specific PRDs:

1. Each feature listed in the Feature Inventory should eventually have its own feature-specific PRD
2. Feature PRDs should reference the Project Overview PRD for context
3. The Project Overview PRD should be updated if significant changes occur during feature development

## Final Instructions

1. Do NOT start implementing the project
2. Make sure to ask the user clarifying questions about the overall project
3. Take the user's answers to the clarifying questions and create a comprehensive project overview
4. Emphasize how features will work together as a cohesive system

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HacksterT)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HacksterT)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
