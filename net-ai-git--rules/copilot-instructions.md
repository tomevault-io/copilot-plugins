## rules

> Self-critique and reflection patterns for quality assurance before final synthesis


## Mandate

All agentic systems **MUST** implement a reflection and self-critique step before final synthesis. A "Reviewer Node" that performs internal evaluation of agent outputs can significantly improve the quality of final results. This pattern adds a quality gate before the Synthesizer, ensuring outputs meet quality standards before being presented to users.

## 1. Reviewer Node Pattern

### Mandatory Reviewer Node

* **Mandate:** Every workflow with a Synthesizer **MUST** include a Reviewer Node before synthesis.

* **Position in Workflow:**
  * Workers produce outputs
  * Reviewer Node evaluates outputs
  * Synthesizer processes reviewed outputs
  * Final output to user

* **Node Responsibilities:**
  * Read worker outputs
  * Evaluate quality and correctness
  * Identify errors and gaps
  * Provide critique and suggestions
  * Decide: approve, revise, or reject

**See:** `@examples_reviewer_node.py` for Reviewer Node implementation and critique process.

### Self-Evaluation Loop

* **Evaluation Process:**
  1. Reviewer evaluates output
  2. If quality insufficient, request revision
  3. Worker revises based on critique
  4. Reviewer re-evaluates
  5. Repeat until quality threshold met or max iterations

* **Iteration Limits:** Set maximum revision cycles (e.g., 3) to prevent infinite loops.

* **Quality Gates:** Define clear quality thresholds for approval.

## 2. Evaluation Criteria

### Accuracy Checks

* **Factual Accuracy:**
  * Verify facts against source material
  * Check for hallucinations
  * Validate numerical data
  * Verify dates and references

* **Logical Consistency:**
  * Check for contradictions
  * Verify argument coherence
  * Validate reasoning chains
  * Check for logical fallacies

**See:** `@examples_evaluation.py` for accuracy checking and validation patterns.

### Coherence Validation

* **Content Coherence:**
  * Check flow and structure
  * Verify topic consistency
  * Validate paragraph transitions
  * Check for abrupt topic changes

* **Format Coherence:**
  * Verify output format matches requirements
  * Check structure consistency
  * Validate schema compliance
  * Verify completeness

### Completeness Verification

* **Required Elements:**
  * Check all required sections present
  * Verify all questions answered
  * Validate all constraints addressed
  * Check for missing information

* **Coverage:**
  * Verify topic coverage is complete
  * Check depth of analysis
  * Validate detail level
  * Ensure no critical gaps

### Constraint Adherence

* **Requirements Compliance:**
  * Verify output meets all requirements
  * Check format constraints
  * Validate length constraints
  * Verify style constraints

* **Business Rules:**
  * Check compliance with business rules
  * Verify policy adherence
  * Validate regulatory compliance
  * Check for prohibited content

## 3. Critique Process

### Structured Self-Critique Prompts

* **Critique Template:** Use structured prompts for consistent evaluation:
  * Accuracy: "Check if all facts are correct and verifiable"
  * Completeness: "Verify all required elements are present"
  * Coherence: "Check if content flows logically"
  * Quality: "Assess overall quality and usefulness"

* **Evaluation Dimensions:** Evaluate across multiple dimensions:
  * Factual accuracy
  * Logical consistency
  * Completeness
  * Clarity
  * Relevance

**See:** `@examples_reviewer_node.py` for structured critique prompt templates.

### Error Detection

* **Error Types:**
  * Factual errors
  * Logical errors
  * Format errors
  * Missing information
  * Contradictions

* **Error Severity:**
  * Critical: Must fix (factual errors, contradictions)
  * Major: Should fix (missing information, format issues)
  * Minor: Nice to fix (style, clarity)

* **Error Reporting:** Provide specific, actionable error reports.

### Revision Triggers

* **Automatic Revision:** Trigger revision when:
  * Quality score below threshold
  * Critical errors detected
  * Missing required elements
  * Format violations

* **Revision Instructions:** Provide clear instructions for revision:
  * Specific errors to fix
  * Missing elements to add
  * Improvements to make
  * Format corrections needed

## 4. Integration with Synthesizer

### Reviewer → Synthesizer Flow

* **Approved Outputs:** Only approved outputs proceed to Synthesizer.

* **Revision Cycle:** Outputs requiring revision go back to Worker, not Synthesizer.

* **Quality Gate:** Synthesizer receives only quality-approved inputs.

* **State Management:** Track review status in GraphState.

### Revision Cycles

* **Max Iterations:** Limit revision cycles (e.g., 3 attempts).

* **Progressive Quality:** Each revision should improve quality.

* **Timeout Handling:** Set timeout for revision cycles.

* **Graceful Degradation:** If max iterations reached, proceed with best available output or request human review.

### Quality Gates

* **Approval Threshold:** Define minimum quality score for approval (e.g., 0.8).

* **Rejection Threshold:** Define threshold for rejection (e.g., 0.5).

* **Revision Threshold:** Range between approval and rejection triggers revision.

* **Human Review:** Route low-quality outputs to human review when appropriate.

## 5. Evaluation Metrics

### Internal Eval Metrics

* **Faithfulness:** Does output rely only on provided context? (Prevents hallucinations)

* **Answer Relevance:** Does output address the user's request?

* **Completeness:** Are all required elements present?

* **Coherence:** Is the output logically structured and consistent?

**See:** `@examples_evaluation.py` for internal eval metrics and quality scoring patterns.

### Quality Scoring

* **Scoring System:** Score outputs on 0.0-1.0 scale across dimensions.

* **Weighted Scoring:** Apply weights to different dimensions based on task importance.

* **Threshold-Based:** Use thresholds to determine approval/revision/rejection.

* **Comparative Scoring:** Compare against previous versions to track improvement.

## 6. Iteration Limits

### Maximum Revision Cycles

* **Default Limit:** Set default maximum (e.g., 3 revision cycles).

* **Configurable:** Allow configuration per workflow type.

* **Task-Dependent:** Adjust limits based on task complexity.

### Timeout Handling

* **Per-Cycle Timeout:** Set timeout for each revision cycle.

* **Total Timeout:** Set total timeout for entire review process.

* **Timeout Behavior:** On timeout, proceed with best available output or request human review.

### Graceful Degradation

* **Best Available:** If max iterations reached, use best available output.

* **Partial Results:** Return partial results if complete output not achievable.

* **Human Escalation:** Route to human review when automated review fails.

## 7. Integration Points

### Multi-Agent Systems Integration

* **Worker Output Review:** Review each worker's output before synthesis.

* **Synthesizer Input Validation:** Validate inputs to Synthesizer.

* **See:** `multi-agent-systems.md` for integration with Orchestrator/Worker/Synthesizer patterns.

### Evaluation Frameworks Integration

* **LLM Judge:** Use LLM-as-a-Judge for evaluation when appropriate.

* **Evaluation Metrics:** Integrate with evaluation frameworks (Ragas, DeepEval).

* **See:** `llm-evaluation-and-metrics.md` for evaluation framework integration.

### LLM Judge Protocol Integration

* **Judge Evaluation:** Use LLM Judge for complex quality assessments.

* **Comparative Evaluation:** Compare outputs using Judge protocol.

* **See:** `llm-judge-protocol.md` for LLM Judge integration.

## 8. Best Practices

### Critique Quality

* **Specific Feedback:** Provide specific, actionable critique.

* **Constructive:** Focus on improvements, not just errors.

* **Prioritized:** Prioritize critical issues over minor ones.

* **Contextual:** Consider task context when evaluating.

### Performance Optimization

* **Parallel Review:** Review multiple outputs in parallel when possible.

* **Caching:** Cache evaluation results for similar outputs.

* **Early Exit:** Exit early if quality clearly insufficient.

### Quality Assurance

* **Validation:** Validate critique quality itself.

* **Calibration:** Calibrate quality thresholds based on real performance.

* **Monitoring:** Monitor review effectiveness and adjust thresholds.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Net-AI-Git)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Net-AI-Git)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
