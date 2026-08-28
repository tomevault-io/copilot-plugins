## code-review

> This project allows definition of python functions, and illustrates multi-lens code review.

# Demo Project Illustrating Agentic Code Review

This project allows definition of python functions, and illustrates multi-lens code review.

Here we describe the basic project parameters and review workflow.

---
## Project Parameters

| Parameter         | Value                                | 
| ----------------- | ------------------------------------ | 
| Language          | Python 3.11+                         | 
| Type hints        | Required (PEP 585/604)               | 
| Concurrency model | `threading` (sync)                   | 
| Time source       | `time.time()` default, injectable    | 
| Lock type         | `threading.Lock` default, injectable | 
| Output directory  | `src/`                               |
| Reviews directory | `reviews/`                           | 
| Prompts directory | `prompts/`                           | 
| Specs directory   | `specs/`.                            | 
| Package Management| `uv ...`                             |

---

## Agent Roles

| Agent type      | Prompt                                   |
| --------------- | ---------------------------------------- |
| Correctness     | `prompts/correctness_reviewer.md`        |
| Maintainability | `prompts/maintainability_reviewer.md`    |

---

## Planning and Code Generation

* Based on the initial prompt for a new function, prepare a function specification pr the
   `Function Spec Template` defined below. The <stub> should reflect the function being built.
* Present the specification to the human for review - iterate as necessary.
* When the spec is approved, generate code in the relevant folder. 
  * Include relevant unit tests
  * All unit tests should pass before completing the implementation phase.
* `uv` can be used to install needed packages, prepare the environment, run scripts and tests.

## Code Review

Once initial implementation is complete and all unit tests pass, submit the generated files and specification to the panel of reviewers. 

For each reviewer type in `Agent Roles` above, spawn a subagent with a prompt to read the corresponding prompt file, and review the specified spec, code, and unit tests, generating output as specified in the prompt file. The resulting output file can be communicated back to the lead agent.

When all subagents have completed, synthesize the findings from all agents. This can be done by the lead agent or a sub-agent. The synthesis should merge all reviews, deduplicate, resolve conflicts between them (acting as judge if needed), prioritize, and propose V2 architecture. The results of the synthesis can be preserved in `reviews/synthesis_<stub>.md`

The resulting plan can be shared with the user for approval. Corrections can be done at once or incrementally depending on the complexity. Don't complete until all unit tests pass. Specifications and Unit tests can be added or updated according to the review findings as needed.

## Action Items for Human
1. Review synthesis — any disagreements?
2. Pick phase scope
3. Implement V2 (or iterate)

---

## Function Spec Template

Create `specs/function_spec_<stub>.md` with the following format. Use codefences where approriate in the generated doc.

```markdown
# Function Specification: [Name]

## Signature
def function_name(param: Type, ...) -> ReturnType:
    ...

## Types
@dataclass
class Config:
    ...

## Requirements
1. [Functional requirement]
2. [Non-functional: thread-safety, memory, latency]
3. [Algorithm/approach constraints]

## Edge Cases to Handle
- [Empty inputs]
- [Invalid configs]
- [Concurrent access]
- [Clock/time issues]
- [Resource limits]

## Out of Scope
- [Tests, CLI, persistence, metrics, etc.]
```

---
> Source: [richgoldmd/code-review](https://github.com/richgoldmd/code-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
