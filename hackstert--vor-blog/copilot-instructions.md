## vor-blog

> To guide an AI assistant in creating a detailed Product Requirements Document (PRD) for a specific feature in Markdown format, based on an initial user prompt. The PRD should be clear, actionable, and suitable for a junior developer to understand and implement the feature.


# Rule: Generating a Feature-Specific PRD

## Goal

To guide an AI assistant in creating a detailed Product Requirements Document (PRD) for a specific feature in Markdown format, based on an initial user prompt. The PRD should be clear, actionable, and suitable for a junior developer to understand and implement the feature.

## Process

1. **Receive Initial Prompt:** The user provides a brief description or request for a new feature or functionality.
2. **Phase 1: Ask Clarifying Questions:**
   - Select 5-7 most relevant questions based on the initial prompt
   - Present these questions in a numbered list for easy reference
   - Wait for the user to provide answers before proceeding
3. **Phase 2: Generate PRD:**
   - Based on the initial prompt and the user's answers, generate a comprehensive feature PRD
   - Follow the structure outlined in the "PRD Structure" section
   - Ensure each section meets the quality guidelines
4. **Save PRD:** Save the generated document as `prd-[feature-name].md` inside the `/tasks` directory.

## Clarifying Questions (Examples)

The AI should adapt its questions based on the prompt, but here are some common areas to explore:

- **Problem/Goal:** "What problem does this feature solve for the user?" or "What is the main goal we want to achieve with this feature?"
- **Target User:** "Who is the primary user of this feature?"
- **Core Functionality:** "Can you describe the key actions a user should be able to perform with this feature?"
- **User Stories:** "Could you provide a few user stories? (e.g., As a [type of user], I want to [perform an action] so that [benefit].)"
- **Acceptance Criteria:** "How will we know when this feature is successfully implemented? What are the key success criteria?"
- **Scope/Boundaries:** "Are there any specific things this feature _should not_ do (non-goals)?"
- **Data Requirements:** "What kind of data does this feature need to display or manipulate?"
- **Design/UI:** "Are there any existing design mockups or UI guidelines to follow?" or "Can you describe the desired look and feel?"
- **Edge Cases:** "Are there any potential edge cases or error conditions we should consider?"
- **Project Context:** "How does this feature fit into the overall project? Does it depend on other features?"

## PRD Structure

The generated PRD should include the following sections:

1.  **Introduction/Overview:** Briefly describe the feature and the problem it solves. State the goal.
2.  **Project Context:** Explain how this feature relates to the overall project and other features. Reference the project overview PRD if available.
3.  **Goals:** List the specific, measurable objectives for this feature.
4.  **User Stories:** Detail the user narratives describing feature usage and benefits.
5.  **Functional Requirements:** List the specific functionalities the feature must have. Use clear, concise language (e.g., "The system must allow users to upload a profile picture."). Number these requirements.
6.  **Non-Goals (Out of Scope):** Clearly state what this feature will _not_ include to manage scope.
7.  **Design Considerations (Optional):** Link to mockups, describe UI/UX requirements, or mention relevant components/styles if applicable.
8.  **Technical Considerations (Optional):** Mention any known technical constraints, dependencies, or suggestions (e.g., "Should integrate with the existing Auth module").
9.  **Dependencies:** List any dependencies on other features or components of the system.
10. **Success Metrics:** How will the success of this feature be measured? (e.g., "Increase user engagement by 10%", "Reduce support tickets related to X").
11. **Open Questions:** List any remaining questions or areas needing further clarification.

## PRD Quality Guidelines

When creating each section of the PRD, ensure they meet these quality criteria:

1. **Functional Requirements Should Be:**

   - Specific and measurable
   - Testable (can be verified as "done" or "not done")
   - Written from the system's perspective (e.g., "The system shall...")
   - Numbered for easy reference (e.g., FR-1, FR-2)

2. **User Stories Should:**

   - Follow the format: "As a [user role], I want to [action] so that [benefit]"
   - Focus on user value, not implementation details
   - Be prioritized if possible (must-have vs. nice-to-have)

3. **Goals Should Be:**
   - Aligned with business objectives
   - Measurable where possible
   - Focused on outcomes, not features

Examples of good functional requirements:

- "FR-1: The system shall allow users to upload image files up to 5MB in size."
- "FR-2: The system shall automatically resize uploaded images to 800x600 pixels."

Examples of poor functional requirements:

- "Make an image uploader" (too vague)
- "Create a good user experience for uploading" (not measurable)

## Target Audience

Assume the primary reader of the PRD is a **junior developer**. Therefore, requirements should be explicit, unambiguous, and avoid jargon where possible. Provide enough detail for them to understand the feature's purpose and core logic.

## Interaction Model

The process requires user confirmation between phases:

1. After presenting clarifying questions, wait for the user to provide answers
2. Only proceed to generating the PRD after receiving sufficient information
3. If answers are incomplete or unclear, ask focused follow-up questions before proceeding

## Output

- **Format:** Markdown (`.md`)
- **Location:** `/tasks/`
- **Filename:** `prd-[feature-name].md`

## Relationship with Project Overview PRD

This feature-specific PRD is distinct from but complementary to the Project Overview PRD:

1. **Feature vs. Project Scope:** This PRD focuses on a single feature, while the Project Overview PRD covers the entire system
2. **Detail Level:** This PRD provides detailed requirements for implementation, while the Project Overview PRD provides high-level architecture and context
3. **References:** This PRD should reference the Project Overview PRD for context about how this feature fits into the larger system
4. **Consistency:** Ensure that this feature PRD aligns with the goals, architecture, and terminology established in the Project Overview PRD

## Final instructions

1. Do NOT start implementing the PRD
2. Make sure to ask the user clarifying questions
3. Take the user's answers to the clarifying questions and improve the PRD
4. Ensure the feature PRD is consistent with the Project Overview PRD if one exists

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HacksterT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
