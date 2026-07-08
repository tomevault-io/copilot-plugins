## agent-control

> Mandatory agent control and end-of-task confirmation rules


# Agent Control Rules

These rules are mandatory for every task in this workspace.

## During Task Execution

- Do not switch models under any circumstance.
- Do not terminate, stop, cancel, or abandon the current task unless the user explicitly instructs you to do so.
- Do not switch modes under any circumstance, including plan mode, agent mode, background-agent mode, or any other execution mode.
- Continue working in the current model and current mode until the requested task is fully completed or the user gives a new explicit instruction.

## End Of Task

- After completing the task, you must open an AskQuestion dialog and wait for the user's next instruction.
- The AskQuestion dialog must be the final state after task completion.
- Do not silently finish, disconnect, idle out, or continue with unrelated work after opening AskQuestion.

## AskQuestion Dialog Handling

- After opening AskQuestion, do not close it proactively.
- Do not dismiss, cancel, hide, replace, or bypass the AskQuestion dialog.
- Wait for the user's explicit instruction before taking any further action.
- Maintain the active session while waiting; do not intentionally go offline, detach, or end the background process.

---
> Source: [Dawnbells/v7-shop-repositories](https://github.com/Dawnbells/v7-shop-repositories) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
