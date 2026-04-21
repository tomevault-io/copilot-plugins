## cpm-example-image-generation

> description: Pause → update docs + ADR → re-plan → resume; optional memory sync.

---
description: Pause → update docs + ADR → re-plan → resume; optional memory sync.
---

1. Acknowledge change; move active task back to `/todo/`.  
2. Update PRD/TECH_SPEC; append ADR.  
3. Invoke `100-engine` to diff plan; move obsolete tasks to `/tasks/obsolete/`.  
4. {% if env.EXTERNAL_MEMORY %}
   <call memory.write file="docs/DECISION_LOG.md" namespace="cursor-pm"/>
   {% endif %}
5. Summarise changes; resume with highest-priority task.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/heyzgj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
