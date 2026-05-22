## examples

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is a simple Express.js demo showcasing the core pattern for n8n integration with custom UIs. The integration follows a specific flow based on n8n execution IDs and Wait nodes.

## Key n8n Concepts
- Every n8n flow is assigned a unique **Execution ID** (**`xid`**) on start
- Running flows have a JSON representation available via **`api/v1/executions/{xid}`**
- Every Wait node exposes the same **`resumeUrl`** (you can also append a custom suffix)
  - format is **`https://N8N_SERVER/webhook-waiting/{xid}`**
- Wait nodes respond to webhooks either **(a)** immediately or **(b)** later using Respond to Webhook

## Key n8n Integration Pattern
1. We choose to start our flow in any way (webhook, timer, trigger, subflow … it doesn't matter)
2. We get `xid` (via webhook response on start, via lookup in some datastore, or query n8n db directly)
3. We use n8n API to fetch execution via **`api/v1/executions/{xid}?includeData=true`**
4. JSON includes the **`data.resultData.lastNodeExecuted`**
5. We use `lastNodeExecuted` to lookup a UUID of the node and map it to the proper UI/UX

## Flow Diagram
```mermaid
sequenceDiagram
    participant UI as ui
    participant Express as Server
    participant n8n as n8n

    Note over UI,n8n: Workflow START
    alt Somehow the workflow is made to start
    UI->>Express: ✅ TRIGGER new workflow (manual / chat / webhook / form)
    UI->>Express: ⬜ QUERY all workflows (datastore)
    UI->>Express: ⬜ RECEIVE alert of new workflow (notify)
    end
    alt Server starts workflow
    Express->>n8n: ⬜ trigger workflow via manual
    Express->>n8n: ⬜ trigger workflow via chat
    Express->>n8n: ✅ trigger workflow via webhook
    Express->>n8n: ⬜ trigger workflow via timer
    Express->>n8n: ⬜ trigger workflow via form
    Express->>n8n: ⬜ trigger workflow via integration
    end
    n8n->>Express: RETURN execution ID (xid)
    Express->>UI: RETURN xID

    Note over UI,n8n: n8nui Monitor LOOP
    loop Execution Monitoring
    alt Somehow the waiting execution is retrieved
    UI->>Express: ✅ REQUEST single execution status using xid
    UI->>Express: ⬜ REQUEST all executions from n8n db (non-compliant)
    end
    alt Server retreives waiting execution
    Express->>n8n: ✅ fetch from api/v1/executions/ {xid} ? includeData = true
    Express->>n8n: ⬜ query n8n db directly (non-compliant)
    end
    n8n->>Express: RETURN execution data (1+)
    Express->>UI: RETURN data.resultData.lastNodeExecuted (1+)
    end

    Note over UI,n8n: n8nui Interface RENDER
    alt One or more 6 key features of n8nui
    UI->>Express: ⬜ AUTH user via org or process roles
    UI->>Express: ✅ LOAD data from this point in flowgram
    UI->>Express: ✅ SUBMIT data into this point of the flowgram
    UI->>Express: ⬜ DECIDE on action for flowgram
    UI->>Express: ⬜ RENDER errors
    UI->>Express: ⬜ RENDER integrations
    end
    Express->>n8n: POST to webhook-waiting/ {xid} + data = here
    n8n->>n8n: Resume execution
    n8n->>Express: RETURN response
    Express->>UI: END unless typeof lastNodeExecuted = Wait else loop
```

## Code Style Guidelines
- Maintain the core integration pattern described above
- Use environment variables for n8n connection settings (.env)
- Handle webhook responses and execution status properly
- Implement proper error handling for API calls
- Log API calls and execution progress for debugging
- Use consistent JSON formatting for data sent to resume Wait nodes

---
> Source: [n8nui/examples](https://github.com/n8nui/examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
