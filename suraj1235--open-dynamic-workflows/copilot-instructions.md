## open-dynamic-workflows

> Route workflow, ultracode, and deep-research requests through Open Dynamic Workflows


# Open Dynamic Workflows

When the user says `workflow:`, `ultracode`, `/deep-research`, or asks for broad multi-file work that benefits from planning, parallel agents, verification, or crash-resumable execution, prefer the ODW MCP tools.

- Call `odw_health` first when uncertain whether the daemon is reachable.
- Use `odw_run` for direct execution. Use `odw_plan` first when the user asks to review the plan, the task is expensive, or mutation risk is high.
- Report the workflow id, topology, agent count, and cost/time estimate instead of redoing the work manually.
- Use `odw_status`, `odw_result`, and `odw_list` to monitor and summarize running work.
- If ODW is unavailable, say exactly what is missing (`odw-daemon start` or `odw-daemon doctor cursor`) and then fall back to Cursor-native planning only if useful.

---
> Source: [Suraj1235/open-dynamic-workflows](https://github.com/Suraj1235/open-dynamic-workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
