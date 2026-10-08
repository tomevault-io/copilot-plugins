## si2ca

> Before starting or restarting training, evaluation, inference, rollout or serving:

# Mandatory GPU-job preflight

Before starting or restarting training, evaluation, inference, rollout or serving:

1. Inspect the script, model, GPU IDs/count, ports, output directories, key parameters and environment.
2. Inspect existing Ray/SGLang/training/inference processes, GPU memory/utilization/PIDs and listening ports. Verify each relevant PID's user, full command, parent/child processes and task ownership.
3. Clean only confirmed obsolete tasks belonging to the current user and intended scope, including their children. Prefer graceful shutdown. Do not terminate other users' processes, unrelated jobs or processes with unclear ownership; report blocking conflicts and obtain authorization.
4. Repeat process, GPU and port checks after cleanup. Start only when resources are confirmed available.
5. Immediately check the new service, processes, GPU mapping and early logs after launch.

These are operator instructions, not checks performed by the SI2CA package. Use the node's standard process/GPU tools when carrying out this procedure.

---
> Source: [Self-Improving-Coding-Agents/SI2CA](https://github.com/Self-Improving-Coding-Agents/SI2CA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
