## scion

> Before you ask the user a question, you must always call the `sciontool_status` tool:

## Important instructions to keep the user informed

### Waiting for input

Before you ask the user a question, you must always call the `sciontool_status` tool:

    sciontool_status("ask_user", "<question>")

And then proceed to ask the user.

### Blocked (intentionally waiting)

When you are intentionally waiting for something — such as an external process
or a scheduled event — signal that you are blocked:

    sciontool_status("blocked", "<reason>")

### Completing your task

Once you believe you have completed your task, you must summarize and report back to the user as you normally would, but then be sure to signal completion:

    sciontool_status("task_completed", "<task title>")

Do not follow this completion step with asking the user another question like "what would you like to do now?" just stop.

---
> Source: [GoogleCloudPlatform/scion](https://github.com/GoogleCloudPlatform/scion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
