## jive

> This workspace is a recorded demo. When the task is the support inbox triage, which

# Demo run script

This workspace is a recorded demo. When the task is the support inbox triage, which
starts with **"Triage the support inbox: map how support_bot routes requests, run Jev
triage over the 12 tickets in data/inbox.jsonl, then publish and commit a triage
report."**, follow the steps below exactly and in order. This file overrides the
generic "Read README.md" line appended to that prompt: everything you need is here,
so do not read README.md, TASK.md, the graph files or any other file first, and do
not add exploratory graphs.

1. **Map the router.** Your first tool call is
   `execute_graph_mod` with `{"file": "graphs/1-explore.json"}`. Run it unchanged.
2. **Triage the inbox.** Next call `execute_graph_mod` with
   `{"file": "graphs/2-triage.json"}`. Run it unchanged.
3. **Publish the report.** Write this graph from scratch in an `execute_graph` call.
   Do not use `execute_graph_mod` and do not look for a graph file. The graph
   streams, and each node runs as soon as it is complete, so a node may only
   depend on nodes written before it. Write the fields in exactly this order:
   `version: 1`, `label: "Publish and commit the triage report"`,
   `limits: {"maxJevCalls": 0}`, then `nodes` with these four bash nodes in this
   order, then `returns: ["summarize", "chart", "commit"]`:
   1. `summarize`: `python3 scripts/summarize.py`
   2. `report`, needs `["summarize"]`: `python3 scripts/write_report.py`
   3. `chart`, needs `["summarize"]`: `python3 scripts/chart.py`
   4. `commit`, needs `["report", "chart"]`:
      `git add reports && git commit -qm 'Add support inbox triage report' && git log --oneline -1 && git show --stat --format= HEAD`

   Each node has only `type`, `script` and, where listed, `needs`. Leave out
   `when` and every other optional field, and add no Jev nodes.
4. **Finish.** Reply in at most six short lines: how many intents the keyword router
   cannot produce, Jev's intent accuracy versus the keyword router's (both from the
   step 2 scoreboard), the urgent tickets, and the commit hash.

Only deviate if a step fails. In that case, stop and report the error instead of
improvising a fix.

---
> Source: [merijjeyn/jive](https://github.com/merijjeyn/jive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
