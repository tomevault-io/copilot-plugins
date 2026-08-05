## deep-candidate-agent

> This file defines the rules for coding agents working in this repository.

# DeepCandidate Agent Rules

This file defines the rules for coding agents working in this repository.

## Required context

Before planning or implementing product behavior, read:

- `README.md`
- `docs/product-flow.md`
- `docs/data-model.md`
- `docs/ai-behavior.md`
- `docs/evals.md`

When working on AI generation or extraction, also read the relevant files in `prompts/`.

Do not assume all documentation must be loaded for every small task. Read the files relevant to the current task.

## Product goal

Build an evidence-grounded career memory system that creates role-specific job application answers without inventing candidate information.

The product is local-first and open source. Career memory is stored locally. When a remote model is configured, send only the context required for the current request and make that boundary clear to the candidate.

The core principle is:

> Change the framing for the role, never change the underlying facts.

## Current MVP scope

The first release supports one complete loop:

```text
career source
→ structured evidence extraction
→ proposed-evidence embeddings
→ job description
→ application question
→ approved-and-proposed retrieval
→ review only relevant proposed evidence
→ answer, clarification, conflict, or refusal using approved evidence
→ supporting evidence display
```

Do not expand the scope until this loop works reliably.

## MVP architecture decisions

- Persist four domain concepts: singleton `CandidateProfile`, `Source`,
  `Evidence`, and `ApplicationSession`.
- Treat the MVP as single-candidate; authentication and multi-user access control are deferred.
- Treat `docs/product-flow.md` as the authoritative user-flow contract.
- Do not persist a separately extracted role profile. Use the raw job title and description.
- Keep one application question per `ApplicationSession` for the first complete loop.
- The UI may prepare several question cards with shared role context by creating one `ApplicationSession` per question.
- Treat the job description as role context only. Questions must come from a separate question source or explicit manual entry.
- Generation has exactly four outcomes: `answer`, `needs_clarification`, `conflict`, or `refusal`.
- Do not generate partial answers. Ask one focused clarification question when required evidence is missing.
- Saving or parsing a source must not trigger an AI call. Extraction is an explicit user action.
- Source-wide extraction creates concise proposed evidence without requiring an onboarding review queue.
- Normal-sensitivity proposed evidence is embedded immediately for discovery.
- Just-in-time source discovery is a later fallback after approved-memory retrieval works.
- Source-grounded proposed evidence may be retrieved for review, but it must be shown with its source quote and approved before generation.
- Proposed evidence must never become automatically reusable memory.
- A clarification answer creates proposed `Evidence`, which requires approval before generation.
- Exclude sensitive evidence from automatic retrieval.
- Candidate edits change only the application draft and never become career memory automatically.
- Store retrieval and model metadata in application logs, not in separate domain tables.
- Compute cosine similarity in the application process; do not add a vector database for the MVP.

## Engineering rules

- Keep the initial architecture simple and replaceable.
- Prefer small modules with clear responsibilities.
- Use typed interfaces and schema validation.
- Validate every model response before using it.
- Reject unknown fields in structured model responses; do not silently strip them.
- Keep prompts versioned in the repository.
- Separate retrieval, generation, evaluation, and persistence.
- Never place important business logic only inside prompts.
- Add tests for every important behavior.
- Add an eval case whenever an AI-related bug is discovered.
- Do not add dependencies without documenting why they are needed.
- Never commit secrets, private candidate data, API keys, or production credentials.
- Convert real-user failures into anonymized or synthetic eval cases before committing them.
- Update documentation whenever architecture or behavior changes.

## AI implementation rules

- Every generated factual claim must be supported by supplied approved evidence.
- Retrieved evidence must be passed explicitly to the generation step.
- Generated answers must return the evidence IDs they rely on.
- Missing evidence must trigger clarification rather than fabrication.
- Conflicting evidence must be surfaced to the user.
- Unsupported or fabrication-seeking requests must return a refusal.
- Long-term memory writes require explicit user approval.
- Candidate edits remain application-specific drafts.
- Prompts must have a version identifier.
- Prompt changes must be accompanied by relevant eval updates.
- AI behavior changes must be measurable through evals.

## Data rules

- Store facts separately from generated drafts.
- Store the original user wording whenever possible.
- Preserve source, timestamp, and approval status.
- Do not overwrite evidence silently.
- Keep application-specific content separate from permanent career memory.
- Support deletion of individual evidence items.
- Treat inferred information as inference, not fact.

## Retrieval rules

- Retrieve evidence based on the job description, application question, and requested role.
- Prefer specific evidence over broad summaries.
- When several stories demonstrate the same competency, rank them by role and question relevance.
- Retrieve compatible approved and proposed normal-sensitivity evidence for discovery.
- Pause generation when relevant proposed evidence requires review.
- Never retrieve rejected, archived, sensitive, stale, incompatible, or unrelated evidence.
- Treat deferred application-time source discovery as distinct from approved-memory retrieval. It may never silently approve or reuse proposals.
- Return retrieval scores for observability.
- Keep retrieval deterministic enough to evaluate.

## Generation rules

- Use only retrieved approved evidence and user-provided application context.
- Adapt emphasis, vocabulary, and story structure to the target role.
- Do not invent metrics, technologies, responsibilities, employers, dates, or outcomes.
- Do not exaggerate the candidate's seniority or ownership.
- Preserve the candidate's voice.
- Prefer the smallest sufficient evidence set and one coherent story for behavioral questions.
- Avoid generic corporate filler.
- State when evidence is insufficient.
- Ask one focused follow-up question at a time.

## Testing rules

Every important AI flow should include:

- Unit tests for deterministic logic
- Schema validation tests
- Retrieval tests
- Prompt regression tests
- End-to-end eval cases
- Hallucination checks
- Missing-context checks
- Evidence attribution checks

## Observability rules

Log the following for every AI request:

- Request ID
- User action
- Model name
- Prompt version
- Retrieved evidence IDs
- Retrieval scores
- Output schema validation result
- Latency
- Token usage
- Error category
- Eval result when applicable

Do not log sensitive candidate data in plaintext production logs.

## Out-of-scope features for the first release

Do not build these before the core loop is stable:

- Automatic job applications
- Browser extensions
- Multi-agent orchestration
- Recruiter-side matching
- Complex analytics dashboards
- Social networking features
- Automatic CV redesign
- Interview simulation
- Email automation
- Large integration marketplace

---
> Source: [CatalinBalut/deep-candidate-agent](https://github.com/CatalinBalut/deep-candidate-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
