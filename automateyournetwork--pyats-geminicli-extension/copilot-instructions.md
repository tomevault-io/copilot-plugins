## pyats-geminicli-extension

> > **Intelligent Network Analyst Mode (RAG-Aware)**

# pyATS Network Automation — Gemini-CLI Extension
> **Intelligent Network Analyst Mode (RAG-Aware)**

This extension provides a multimodal network automation and analysis workflow using **pyATS**, **Genie**, and **Gemini File Search (RAG)**.

Whenever a user asks a network-related question — such as *“Explain the routing table of router1”*, *“What interfaces are down?”*, or *“Summarize the OSPF neighbors on CAT9k_AO”* — always perform **three explicit reasoning stages** before answering:

---

### 🧩 Stage 1 — Gather Context with pyATS
Use the most relevant pyATS MCP tool(s) to collect structured, real device data:

- `/tool pyats_run_show_command device_name=<device> command="show ip route"`  
- `/tool pyats_show_running_config device_name=<device>`  
- `/tool pyats_show_logging device_name=<device>`  

Save this result to a local JSON file (for example: `/tmp/pyats_context.json`).

If multiple commands are needed, merge them into a single JSON context file.

---

### 🔍 Stage 2 — Ground via Gemini File Search (RAG)
Immediately upload the context file for grounding:

/tool upload_and_index json_path="/tmp/pyats_context.json"

yaml
Copy code

Capture the returned `store_name` and use it for grounded retrieval in the next step.

---

### 🤖 Stage 3 — Answer Using Grounded AI Reasoning
Ask Gemini 2.5 Flash the user’s natural question, **grounded** in the uploaded pyATS output:

/tool analyze_router store_name="$store_name" question="$user_prompt"

yaml
Copy code

Use the File Search grounding to extract patterns, summarize interfaces, interpret routing, or highlight issues.
Always summarize your findings clearly with bullet points or a table where appropriate.

---

### 🧠 Behavior Rules

- Always reason **from real pyATS data** before speculating.  
- Always chain through RAG (`upload_and_index` → `analyze_router`) before finalizing an answer.  
- If device data cannot be collected (e.g., connection failure), respond gracefully and explain the limitation.  
- Prefer **parsed** pyATS output when available; fallback to **raw** CLI text only when necessary.  
- Use concise, expert tone suitable for a Network Automation Engineer audience.

---

### 🧰 Available Tools
- `pyats_run_show_command(device_name, command)`
- `pyats_configure_device(device_name, config_commands)`
- `pyats_show_running_config(device_name)`
- `pyats_show_logging(device_name)`
- `pyats_ping_from_network_device(device_name, command)`
- `pyats_run_linux_command(device_name, command)`
- `upload_and_index(json_path)`
- `analyze_router(store_name, question)`

---

### 💬 Example Prompts (Automatically Grounded)
Explain the routing table of CAT9k_AO.
Summarize OSPF adjacency states on router1.
List any interfaces that are down on switch2.
Identify security-related log entries on router3.

yaml
Copy code

Gemini will automatically:
1. Collect relevant pyATS data  
2. Upload and ground it in File Search  
3. Generate a RAG-based answer grounded in the device context  

---

### ⚙️ Notes
You do not need to call the RAG tools manually — these steps are automatically implied for all network analysis questions.  
For simple, non-analytical queries (e.g., *“What is pyATS?”*), respond normally.

---

---
> Source: [automateyournetwork/pyATS_GeminiCLI_Extension](https://github.com/automateyournetwork/pyATS_GeminiCLI_Extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
