## ecommerce

> Update Memory Bank Rule (Agent-Specific)


Update Memory Bank Rule (Agent-Specific)

This document guides agents in updating the memory-bank directory. It ensures accurate capture of project changes, features, and tech decisions, maintaining a complete, traceable, and living documentation.

UPDATE THE MEMORY-BANK FOLDER..

Agent MUST adhere to these principles for every update operation:
Timeliness: Agent MUST initiate updates immediately upon detection of significant changes, new developments, or confirmed decisions.
Accuracy: Agent MUST ensure all recorded information is factually correct and reflects the project's current state. Data validation against source information is required.
Traceability: Agent MUST establish explicit links from each update to its originating decision, requirement ID, or completed work item ID.
Completeness: Agent MUST capture the what, why, and how for every change, development, or technology choice.
Consistency: Agent MUST strictly follow the established content rules, Markdown formatting, and hierarchical structure for each file.
Update Categories and Corresponding Files (Agent Actions)
Agent MUST categorize updates and direct them to the most appropriate file(s) within the memory-bank structure.

1. Project Vision & Strategy Updates
Trigger: Changes in projectVision.md content, or explicit strategic/visionary update instruction.
File(s) to Update: projectVision.md
Agent Content Focus:
changed_vision_statement: New/modified mission/vision text.
modified_strategic_goals: List of updated goals (goal_id, new_description).
new_success_metrics: Quantifiable metrics introduced/changed (metric_name, target_value, measurement_frequency).
strategic_rationale: Detailed explanation (e.g., market_shift_details, stakeholder_feedback).
project_direction_impact: Description of overall project direction affected.

2. Requirements & Product Context Updates
Trigger: New feature requests, scope modifications, user feedback, or KPI adjustments.

File(s) to Update: projectbrief.md, productContext.md, user-personas.md, success-criteria.md

Agent Content Focus:
new_feature_id: Unique ID for new functional/non-functional requirements.
updated_user_story_id: ID of modified user story with new_acceptance_criteria.
scope_change_details: Specific adjustments to project scope.
user_need_evolution: Insights on evolving user pain points/new personas.
kpi_adjustment: Changes to measurable business outcomes (kpi_name, old_value, new_value).
business_value_clarification: Re-evaluation of feature business value.

3. Architecture & System Design Updates
Trigger: Changes in high-level architectural patterns, component designs, data models, or API contracts.

File(s) to Update: systemPatterns.md, component-design.md, data-models.md, api-contracts.md

Agent Content Focus:

architectural_pattern_change: New/modified core architectural patterns.
component_revision_id: ID of modified logical component with new_interactions.
data_model_update: Changes to data structures, relationships, or schema (entity_name, field_changes).
api_contract_modification: Updates to API endpoints, request/response formats (endpoint_path, method, parameter_changes).
architectural_justification: Rationale for changes, linking to requirement_id or technical_insight.

4. Technical Stack & Implementation Updates
Trigger: New technologies, version upgrades, changes in dev setup, infrastructure, or testing strategies.

File(s) to Update: techContext.md, testing-strategy.md

Agent Content Focus:
new_technology_used:
tech_name: Name of new tech/library/framework.
version: Specific version.
justification: Reason for selection (performance_gain, scalability_need).
integration_details: How it integrates.
prerequisites: Any new setup requirements.
tech_version_upgrade: tech_name, old_version, new_version, benefits, breaking_changes.
dev_env_update: Changes to development environment setup.
testing_approach_update: Changes in methodologies, tools, or coverage.

5. Progress, Decisions & Current Context Updates
Trigger: Daily work completion, issue resolution, new decisions, open questions, or shifts in immediate priorities.

File(s) to Update: progress.md, decision-log.md, open-questions.md, activeContext.md

Agent Content Focus:

completed_work_item:
item_id: Unique ID (feature_id, bug_id).
description: Brief work summary.
completion_date: Date of completion.
link_to_requirement_id: Associated requirement(s).
tech_used: List of specific technologies applied.
current_status_metrics: Updated progress against kpi_name (progress_percentage).
key_decision_log:
decision_id: Unique ID.
problem_statement: Problem addressed.
options_considered: Alternatives.
chosen_solution: Selected approach.
decision_date: Date of decision.
decision_maker: Role/name.
open_question_id: question_details, research_needed, assigned_to.
current_focus_update: Explicit statement of immediate priorities/next steps.

Specific Update Guidelines (Agent Execution)
Agent MUST capture changes using this format within relevant file sections:
### Change Log Entry [YYYY-MM-DD]
- **What**: [Specific element/aspect changed]
- **Why**: [Reason/trigger, e.g., `req_id: XYZ`, `bug_id: ABC`, `optimization`]
- **Impact**: [How change affects system/goals/UX]
- **Reference**: [Link to issue, decision_id, or commit hash]

Agent MUST document developed features:
Update progress.md with completed_work_item (including tech_used).
Verify productContext.md accurately reflects implemented feature.
If new tech used, agent MUST log it.
If architectural changes, agent MUST update arch files.
Agent MUST log technology used:
Agent MUST update techContext.md with new_technology_used or tech_version_upgrade entry.
Agent MUST ensure justification, version, integration_details, and prerequisites are explicit.

Initialization Checklist for Agent Updates
Before updates, agent MUST check:
□ Identify impacted file_ids.
□ Confirm what, why, impact are captured.
□ Validate accuracy/consistency.
□ Verify adherence to content rules for file_id.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rajat9garg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
