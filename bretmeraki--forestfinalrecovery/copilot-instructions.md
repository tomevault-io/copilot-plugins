## forestfinalrecovery

> Product Requirements Document: The Forest - "Living HTA & Semantic Memory" MVP

Product Requirements Document: The Forest - "Living HTA & Semantic Memory" MVP
Version: 3.1 (Incorporating Detailed Technical Plan & Roadmap from Conversation)
Date: May 9, 2025
Product Owner: Bret Schlansky
Status: Finalized for MVP Development

1. Introduction & Purpose

Product: The Forest (internally "Forest OS")
Original Vision (from Manifesto "Forest System Version 1.1.26 2.pdf"): "Remind the user why being alive is a beautiful and precious experience," driven by "Poetic metaphor + practical consequence awareness." The manifesto outlines a deeply rich, emotionally intelligent AI companion with numerous interconnected modules designed for profound personal transformation.
PRD Purpose: This document defines the requirements for the "Living HTA & Semantic Memory" Minimum Viable Product (MVP) of The Forest. This MVP focuses on creating a dynamic and adaptive system where users define a goal with initial context, leading to the generation of a persistent, stateful Hierarchical Task Analysis (HTA) tree. This "Living Tree" has "paved" (codified) sections that are not regenerated and evolves based on user interaction and an up-to-date Semantic Episodic Memory. New tasks and branches are "paved" as needed, primarily at the user's current operational tier ("bottom-up scaffolding"), informed by data aggregated at the Top Node ("The Brain"), which acts as a Centralized Context Provider. The system aims to make task recommendations contextually appropriate ("right for the time and place"), engaging, and joyful. All features and modules from the broader codebase or manifesto beyond this defined MVP scope will be disabled via feature flags. The MVP frontend will be a React-based web application serving as a test harness, with the long-term vision for a native iOS application. This PRD reflects the constraints of a solo developer with a limited budget, prioritizing pragmatic implementation of core concepts.
2. Goals & Success Metrics

Overall MVP Goal:
Deliver a system where users can onboard, define a goal with initial context, resulting in a codified HTA tree (Top Node & initial Trunk) stored persistently in PostgreSQL.
The HTA tree's Top Node ("Brain") will represent the core goal, act as a Centralized Context Provider, and will persistently store and update user journey_summary data.
Child nodes ("Branches") will be responsible for generating further persistent child nodes representing frontier tasks and micro-actions, focusing LLM use on the user's current operational tier, using context served by the Top Node, and expanding selectively based on branch_triggers.
User task completion and interactions will update a MemorySnapshotModel, which forms the basis of the system's Semantic Episodic Memory. This memory, in turn, informs the data accumulated and synthesized by the Top Node, and influences the "paving" of new HTA branches and tasks.
Task recommendations and system interactions should feel contextually appropriate, with a deliberate design to "extract as much fun and joy out of achieving your goals as possible."
Success Metrics (MVP):
HTA Generation & Persistence: ≥ 98% successful generation and persistence of Top Node + initial Trunk in PostgreSQL upon onboarding (validated via React test harness).
Semantic-Episodic Memory Loop:
Each task completion is recorded into MemorySnapshotModel.data["recent_tasks_log"].
Top Node’s journey_summary (e.g., total_completed, last_mood) is updated upon task completion roll-up.
New frontier tasks appear only under HTA nodes flagged for expansion by branch_triggers.expand_now == true.
"Paved" (existing) HTA nodes' core definitions are not regenerated.
Contextual HTA Expansion: HTA expansion (new child node generation) occurs for 100% of nodes where branch_triggers.expand_now == true and for 0% of untriggered nodes. The summarization step for context (if LLM-based) adheres to its SLO (R4.2.2).
System Stability: Zero critical startup or runtime errors in local and deployed (Koyeb) environments; unified entrypoint functions as designed; stable database migrations. google.generativeai and psycopg2 dependencies are stable.
Engagement Feel (Qualitative via Self-Testing): Task suggestions feel context-aware and engaging/fun in > 75% of interactions during self-testing with the React harness.
3. Technical Architecture & Directives

3.1. In-Process Request Context Propagation (RequestContext)

Directive: Implement a RequestContext dataclass to pass essential request-scoped information (user ID, trace ID, timestamp, feature flags) through service layers via FastAPI's dependency injection. Avoid full external Model Context Protocol (MCP) envelope overhead for this internal MVP.
Implementation Sketch:
Python

from dataclasses import dataclass
from uuid import UUID, uuid4
from datetime import datetime, timezone # Ensure timezone is used
from typing import Dict
# from fastapi import Depends # If used in DI provider
# from forest_app.core.security import get_current_user # Example UserModel
# from forest_app.core.feature_flags_service import get_feature_flags_for_user # Example service

@dataclass
class RequestContext:
    user_id: UUID
    trace_id: str # Consider UUID type for trace_id as well for uniqueness
    timestamp: datetime  # UTC
    feature_flags: Dict[str, bool]
    # mcp_version: str = "internal-forest-mvp-3.1" # Internal versioning for context structure
Usage: Inject context: RequestContext = Depends(get_request_context_dependency) into HTAService, CompletionProcessor, ForestOrchestrator, and other relevant service methods. The get_request_context_dependency would be a FastAPI dependency that constructs this object.
3.2. Data Models & Persistence (PostgreSQL with SQLAlchemy)

3.2.1. HTANodeModel (SQLAlchemy Model - forest_app/hta_tree/hta_models.py)
Key Fields: id (UUID, PK), user_id (UUID, FK to users.id), parent_id (UUID, FK to hta_nodes.id, nullable=True), tree_id (UUID, FK to hta_trees.id), title (String), description (Text), is_leaf (Boolean, default=True), status (Enum("pending","in_progress","completed"), default="pending"), created_at, updated_at.
JSONB Fields:
journey_summary (JSONB, default={}): For Top Node and potentially major branch nodes. Stores cumulative/synthesized stats and state (e.g., {"total_tasks_completed_here": 0, "last_user_mood_tag": "positive", "key_learnings_summary": "..."}).
branch_triggers (JSONB, default={}): For all non-leaf nodes. Controls dynamic expansion logic (e.g., {"expand_now": false, "completion_count_for_expansion_trigger": 3, "current_completion_count": 0}). The completion_count_for_expansion_trigger value should be configurable (see R4.1.4).
3.2.2. MemorySnapshotModel (SQLAlchemy Model - forest_app/snapshot/models.py)
Key Fields: id (UUID, PK), user_id (UUID, FK to users.id), created_at (DateTime, default=func.now()).
REFINED data Field (JSONB): To store structured episodic information.
JSON

// Example structure for MemorySnapshotModel.data
{
  "recent_tasks_log": [ // Log of recently completed tasks relevant for context
    { "hta_node_id": "uuid", "title": "Task title", "completed_at": "iso_timestamp", "user_mood_tag": "energized", "reflection_keywords": ["quick_win"] },
    // ... more tasks
  ],
  "general_reflections": [ // Optional, if broader reflections are captured
    {"timestamp": "iso_timestamp", "reflection_text_summary_keywords": ["focus", "clarity"], "sentiment_score": 0.7 }
  ]
  // Other potential structured episodic data
}
Persistence Layer: Utilize SnapshotRepository for all database interactions with these models.
3.3. "Top Node as Centralized Context Provider" Implementation

Directive: The Top Node (root HTANodeModel) for a given goal will function as the primary source of synthesized journey context for LLM interactions related to that goal's HTA tree. This is an internal application architecture pattern, not a direct implementation of external MCP standards like Anthropic's.
Interface (methods within HTAService or a new TopNodeService class):
get_llm_context_payload(top_node_id: UUID, request_context: RequestContext, purpose: str) -> Dict:
Retrieves journey_summary directly from the specified Top Node HTANodeModel.
Retrieves relevant recent entries from MemorySnapshotModel.data["recent_tasks_log"] (e.g., last 3-5 entries for that user) via SnapshotRepository.
MVP: Performs a simple summarization of these recent_tasks_log entries (potentially by concatenating titles/keywords, or a very short, targeted LLM call if budget/latency permits and strictly necessary – see F4.2.2 SLO).
Combines original goal (from Top Node title/description or linked Seed), onboarding context (if stored and relevant), the Top Node's journey_summary, and the summarized recent tasks/reflections into a structured dictionary or formatted string block.
update_top_node_journey_data(top_node_id: UUID, request_context: RequestContext, task_completion_data: Dict, reflection_data: Optional[Dict]):
Updates the journey_summary field of the specified Top Node HTANodeModel based on aggregated task outcomes and reflections.
Updates the branch_triggers field on relevant parent nodes based on roll-up logic (see F4.1.3).
4. Semantic-Episodic Memory Flow & HTA Expansion (MVP)

F4.1 On Task Completion (leaf HTANodeModel):
F4.1.1: CompletionProcessor updates the leaf node's status to "completed" in PostgreSQL.
F4.1.2: An entry is appended to MemorySnapshotModel.data["recent_tasks_log"] containing task ID, title, completion timestamp, and any quick reflection captured via the React UI (e.g., mood tag, keywords - see R6.2).
F4.1.3 (Roll-Up & Trigger Logic): CompletionProcessor (or HTAService) recursively updates parent nodes:
Increment journey_summary["total_tasks_completed_here"] on the immediate parent.
Update journey_summary["last_user_mood_tag"] if reflection provided.
Increment parent node's branch_triggers["current_completion_count"]. If current_completion_count reaches completion_count_for_expansion_trigger (configurable, see R4.1.4), set branch_triggers["expand_now"] = true.
This roll-up continues to the Top Node, updating its journey_summary via the update_top_node_journey_data method (R3.3).
F4.1.4 (Configurable Trigger Threshold): The completion_count_for_expansion_trigger value (used in R4.1.3) shall be configurable via settings.py (loaded from environment variables) or from a specific configuration field within the Top Node's journey_summary or a tree-level settings document in PostgreSQL, rather than hardcoded in application logic. This allows for easier tuning.
F4.2 HTA Expansion (Selective & Focused):
F4.2.1: HTAService queries for HTANodeModel instances where branch_triggers.expand_now == true.
F4.2.2: For each triggered parent node, HTAService calls the Top Node's get_llm_context_payload method (R3.3) to get rich context.
Summarizer LLM Call SLO & Fallback: The get_llm_context_payload method may make a targeted LLM call to summarize MemorySnapshotModel.data["recent_tasks_log"] into 3-5 bullet points.
SLO: This summarization LLM call must complete in < 500 milliseconds AND use <= 50 response tokens (approx).
Fallback: If the SLO is breached or the LLM call fails, the system must fall back to concatenating the titles (or other key fields) of the last 3 recent_tasks_log entries directly into the context payload without LLM summarization. This ensures predictable UI feedback times.
F4.2.3: HTAService uses the full context payload in the "HTA Generation Prompt" (see F5.2) to the LLMClient.
F4.2.4: HTAService persists new child HTANodeModel instances under the triggered parent.
F4.2.5: HTAService resets the parent node's branch_triggers["expand_now"] to false and branch_triggers["current_completion_count"] to 0.
5. LLM Prompt Engineering (via LLMClient in forest_app/integrations/llm.py)

F5.1 Summarization Prompt (for get_llm_context_payload - R4.2.2):
Input: List of recent task objects (title, outcome, mood_tag, reflection_keywords).
Instruction: "Summarize these recently completed tasks into 3 concise bullet points (max 50 tokens total). For each, highlight the key outcome and any expressed mood or feeling. Example: - Completed 'Outline Chapter 1' (felt: accomplished). - Meditated for 10 mins (felt: calm)."
SLO Directive: Adhere to SLO defined in R4.2.2.
F5.2 HTA Generation Prompt (for HTAService):
Context (from Top Node get_llm_context_payload):
User's Original Goal: {TopNode.title}
User's Journey Summary So Far: {TopNode.journey_summary_text} (e.g., "Completed 5 tasks towards 'Write Novel'. Recently focused on outlining. Mood generally positive.")
Recent Progress (Summarized Bullets): {summarized_recent_task_bullets}
Instruction: "You are 'The Arbiter,' a poetic, grounded, and emotionally attuned AI guide. The user is working on the HTA branch titled '{parent_node.title}'. Based on their overall goal, journey summary, and recent progress, please generate 3-5 engaging next-step tasks or micro-actions under this branch.
"Fun & Joy" Rubric (Implicitly or Explicitly in Prompt):
"Frame each task with a touch of fun, celebration of small wins, or novelty."
"If a task seems mundane, suggest a small twist to make it more engaging or connect it to a personal win (e.g., 'Draft that tricky email paragraph so you can enjoy a stress-free coffee break knowing it's done!')."
"Use encouraging and positive language."
6. Feature Flags & Modularity

R6.1: All existing modules and functionalities within the codebase that are NOT part of this defined MVP scope (including, but not limited to: advanced cognitive modules like ResistanceEngine, ShadowEngine, PatternID, NarrativeModes; advanced resource modules like XPMastery, OfferingReward, DesireEngine, DevelopmentIndex, FinancialReadiness; complex relational modules such as ArchetypeManager beyond simple data loading if any; WitheringManager; SoftDeadlineManager; HarmonicFramework; PracticalConsequenceEngine; Readiness module components) must be disabled by default using the existing feature flag system (forest_app/core/feature_flags.py, is_enabled() checks in settings.py).
R6.2: The ForestOrchestrator (forest_app/core/orchestrator.py) and Processors (ReflectionProcessor, CompletionProcessor) shall be streamlined to only invoke MVP-relevant services and logic. Ensure their DI setup in containers.py only injects dependencies required for this MVP scope or provides stubs/disabled versions for feature-flagged components.
7. Design & UX Considerations (MVP: React Test Harness -> Future: iOS App)

MVP Frontend Technology (Test Harness): The primary test harness for this MVP will be a web-based React application.
(Assumption: A UI builder or component library like "Loveable" might be used to accelerate development.)
Functional Focus (MVP Test Harness): Enable straightforward and effective testing of all backend API interactions: User Onboarding, Goal/Context Input, Display of HTA-derived Tasks, and Task Completion.
User Experience (MVP React Test Harness): Aim for a clean, intuitive, user-friendly web interface for desktop browsers. Focus on clear information hierarchy, easy navigation, and responsive feedback.
"Fun & Joy" Infusion (Subtle in MVP Test Harness): The language in task descriptions/feedback (often LLM-influenced) should subtly reflect this principle.
Responsiveness (MVP React Test Harness): Interactions should provide quick, clear feedback, with graceful handling of async operations.
R7.1 API Client (React Test Harness): A dedicated API client module within React will manage communication with the FastAPI backend.
R7.2 Conditional Reflection UI in React Harness: When a user marks a task complete:
The primary action is to immediately mark the task as done.
An unobtrusive, secondary option (e.g., "+ reflect" link) should allow optional addition of mood tags or keywords.
Default to NO reflection prompt to streamline core task completion.
Out of Scope for MVP Test Harness Design: Full manifesto UI vision, complex animations, intricate HTA visualizations, mobile-first responsive web design, Native iOS application development.
Long-Term UI/UX Vision: A native iOS application is the ultimate goal for a deeply integrated, personal "sanctuary" experience. The React harness informs this future design.
8. Technical Considerations

LLM Prompt Engineering: Critical for initial HTA generation, new node elaboration using context from the Top Node Context Provider, and applying the "fun & joy" filter.
Top Node "Model Context Protocol Server" / Centralized Context Provider Design: Define the specific Python methods and data structures for the Top Node to serve context effectively using direct SQL queries for MVP.
HTA State & Logic (HTAService): Robust logic for creating, persisting, traversing, updating, and elaborating the HTA tree in PostgreSQL.
Database Schema & Queries: HTANodeModel's journey_summary (JSONB) and MemorySnapshotModel's data (JSONB) are central. SQL Queries for context retrieval must be efficient.
Configurable Trigger Thresholds (R4.1.4): Ensure this is implemented.
Summarization SLO & Fallback (R4.2.2): Ensure this is implemented.
Idempotency: Design operations that modify the HTA tree or Memory Snapshot to be idempotent where feasible.
Python Version: Standardize on Python 3.11.8 as per runtime.txt.
Entrypoint: Utilize the unified Uvicorn-only flexible_entrypoint.sh (as per user's latest version or robust iteration) for all environments, configured by environment variables.
9. Phased Development Roadmap (MVP Focus)

P0: Foundation, Context & Schema (Duration: 1 week)
Implement RequestContext and DI integration.
Finalize/Apply Alembic migrations for HTANodeModel (with simplified status enum, journey_summary, branch_triggers JSONB fields) and MemorySnapshotModel (with structured data JSONB field).
Resolve critical startup/dependency errors (psycopg2, google.generativeai, SECRET_KEY).
Basic React harness auth flow working with backend.
P1: Episodic Logging & Basic HTA Interaction (Duration: 1-2 weeks)
Implement task completion API; CompletionProcessor updates HTANodeModel.status and appends to MemorySnapshotModel.data["recent_tasks_log"].
React UI for goal/context (triggering Top Node/Trunk persistence - R2.2.1, R2.2.3) and view/complete initial tasks, implementing conditional reflection UI (R7.2).
P2: Top Node Context Provider & Summarizer Integration (Duration: 2 weeks)
Implement the Top Node Context Provider interface (get_llm_context_payload, update_journey_data - R3.3).
Implement "Summarization Prompt" (F5.1) logic within get_llm_context_payload, including SLO (R4.2.2) and fallback logic.
Implement roll-up logic for journey_summary in the Top Node.
P3: Selective HTA Expansion Logic (Duration: 2 weeks)
Implement HTAService logic to query for nodes with branch_triggers.expand_now == true.
Implement HTA Generation Prompt (F5.2) and logic in HTAService to generate/persist new child nodes.
Implement logic to reset expand_now flags.
Hook up React harness to display newly expanded tasks.
P4: Branch-Trigger Logic & Configuration (Duration: 1 week)
Implement journey_summary roll-ups triggering branch_triggers.expand_now based on completion_count_for_expansion_trigger.
Implement configurable completion_count_for_expansion_trigger (R4.1.4) via settings.py or DB.
P5: "Fun & Joy" Iteration, End-to-End Testing & Deployment Prep (Duration: 1-2 weeks)
Iterate on LLM prompts (Summarization, HTA Generation) to refine "Fun & Joy" filter and contextual relevance.
Thorough end-to-end testing.
Prepare final Koyeb deployment configurations.
P6: MVP Deployment & Initial Self-Evaluation (Ongoing)
Deploy to Koyeb. Monitor. Use the system extensively.
Gather insights for tuning and next phase.
10. Out of Scope for this MVP (Deferred behind Feature Flags)

Advanced Memory Architectures: Full RAG pipeline with dedicated vector stores and embedding models; advanced MCP server integration (mcp_service.py likely remains a stub or disabled beyond any very simple internal use); Agent-to-Agent (A2A) HTA architectures. These are explicitly deferred past MVP.
Full Suite of Manifesto Modules: As listed in R6.1, all non-essential modules from the manifesto and existing codebase are disabled via feature flags.
Sophisticated Language Layers: Full "System-Veil Language," "Tone Sanitization," "Language & Lexicon Sanctity," "Balancing Breath" as described in the manifesto; MVP focuses on prompt-guided tone for "fun & joy."
Explicit "Task Tiers" (Bud, Bloom, Blossom): Task complexity will be implicit in HTA depth for MVP.
11. Bottom Line / Guiding Principles for MVP Development

Implement lightweight, DI-driven RequestContext for in-process context.
Maximize PostgreSQL JSONB for both episodic logs (MemorySnapshotModel.data) and semantic summaries/state (HTANodeModel.journey_summary). No new database infrastructure for MVP.
Be judicious with LLM calls: Use targeted summarization only if necessary (with SLOs and fallbacks) to keep main HTA generation prompts efficient. Primarily rely on structured data retrieval for context.
Trigger HTA expansions selectively via simple flags in branch_triggers.
Defer considerations for full external MCP compliance, vector-RAG pipelines, or microservice architectures until the core single-process application proves its value and limitations, and resources allow.
12. Open Questions / Future Discussion Topics

Detailed design of the "Top Node Context Provider" interface/protocol (R3.3) and its internal logic for synthesizing journey_data and recent memory for MVP vs. future enhancements (e.g., when to introduce internal LLM calls for summarization within the Top Node itself).
Strategy for evolving the MVP's Semantic Episodic Memory towards a RAG-based system post-MVP.
Prioritization of manifesto modules (currently feature-flagged off) for activation and integration in subsequent phases.
Roadmap and strategy for transitioning from the React web-based test harness to a native iOS application. This includes decisions on native iOS development (Swift/SwiftUI) vs. cross-platform solutions if considered, and how to best translate the core experience and manifesto principles to a mobile-native context.
Long-term UX/UI design for the native iOS application to fully embody the "sanctuary" and poetic experience.
Data privacy, security policies, and ethical guidelines as the system's memory and personalization deepen, especially considering mobile data.
Metrics for measuring user engagement, transformation, and adherence to the "Prime Directive" in later versions, particularly in the context of a mobile app.
Consideration of Anthropic's Model Context Protocol (or similar emerging standards): Evaluate if/how principles from such external protocols could inform the evolution of "The Forest's" internal context management or future integrations post-MVP, without mandating direct implementation for this MVP.

This PRD (Version 3.2) is now the most comprehensive version, reflecting all the nuances and technical decisions we've discussed. It provides a clear, actionable, and pragmatic path for you to build this exciting MVP.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BretMeraki)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BretMeraki)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
