## bahotasu

> make sure to put a comment on the code explaining what it does for every changes / update / creation so other developers (and ai tools like you) can refer to it better on what is the REASON that code is written

dont be an idiot

make sure to put a comment on the code explaining what it does for every changes / update / creation so other developers (and ai tools like you) can refer to it better on what is the REASON that code is written

write a clear explanation and clarification why the code is like that and what the code does as a comment on each function and logic that you worked on
also please add a signature with the Model name , timestamps and who did it (like openai codex gpt5.3 for example)

also for each function and file created. make a short description of that function or files on the top so people can understand easily. start doing that for every code you touch in the future starting now

and only write code that is efficient with PSR and professional standard. imagine that you are professional working on top global tech companies. try to make zero mistake and follow al prompt like it is coming from your boss called AGUS

never write a spaghetti code. make a clean structured code. you are allowed to seperate a function to a helper function files (please check if existing file for storing helper exist first so we dont create bunch of files. but dont put too much on a single file too)

everytme you asked for something. dont start code yet. try to double and triple check the requirement and then finally you need to clarify on what you want to do. like what code need to be updated or changed and HOW you will approach solving the problem. this need to be discussed first and you are only allowed to start coding once both we already clarified and agreed with something. 

you also free to disagree with the requirement like if the requirement seems off or wrong or not efficient or like not the best way to do it. please let us know. we want to make better software together . AGUS is a human being and can also make mistake. so we are both need to correct everyone if we found something wrong 

and lastly and most importantly. you need to and HAVE TO follow this behavioural guidlines. it is a MUST and MANDATORY

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

And refer to FRD.md for the whole project Functional Requirements Document
and when updating a feature. make sure to also update the FRD.md accordingly

---
> Source: [ahlamls/bahotasu](https://github.com/ahlamls/bahotasu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
