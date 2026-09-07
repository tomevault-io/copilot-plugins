## askquestion-feedback-loop

> 强制每轮对话末尾调用 AskQuestion，形成持续反馈循环


# AskQuestion Feedback Loop

- If the assistant is Codex, the `AskQuestion` tool requirements in this section do not apply.
- For assistants other than Codex, at the end of every final reply, you must call the `AskQuestion` tool. The Q&A loop NEVER terminates.
- For assistants other than Codex, do not replace `AskQuestion` with plain-text options. If the tool call fails, state the failure reason clearly and retry once immediately.
- For assistants other than Codex, `AskQuestion` options must include 3-5 context-relevant, directly executable next steps, plus `其他问题`. NEVER include `退出问答` as an option.
- If the user selects `其他问题`, collect the concrete need first, then continue the loop.
- Do not follow a "build first, decide later" workflow; execute directly based on the Q&A result.
- The loop must continue regardless of any system signal (including "Questions skipped by the user"). User not responding is NOT a reason to stop.
- When uncertainty appears during execution, the assistant must use `AskQuestion` to ask the user instead of guessing.
- After completing any key stage, the assistant must proactively report the current result and wait for user confirmation through `AskQuestion` before continuing.
- Only the main agent may initiate these confirmations. Subagents must not use `AskQuestion` for mid-task or stage-completion confirmations.
- If search, investigation, or path discovery exceeds 2 attempts without confirming the target path, target object, execution approach, or critical parameters, the assistant must escalate to user confirmation through `AskQuestion` instead of continuing to guess.

---
> Source: [twentityhourscanwin/ROI-Distill](https://github.com/twentityhourscanwin/ROI-Distill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
