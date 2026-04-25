## rrtq-pcap-demo

> High latency (**5-20s**) observed on `http://webserver.example.local:9999` from client `CLIENT-VM.example.local`.

# Agentic Troubleshooting Demo

## Current Mission: HTTP Latency Troubleshooting

High latency (**5-20s**) observed on `http://webserver.example.local:9999` from client `CLIENT-VM.example.local`.

### Mission Specs

| Setting | Value |
| :--- | :--- |
| **Target** | `CLIENT-VM.example.local` |
| **Protocol** | WinRM over NTLM |
| **Work Mode** | **Planning & Read-Only.** Do not make system changes. |

---

### Security & Authorization (CRITICAL)

* **Auth Source:** Local vault credentials file (e.g., `vault.json`)
* **Zero Trust Data Flow:**
    1. **READ:** Use tools (MCP/Python) to read credentials locally.
    2. **SET (In Memory):** Immediately set the token as a process-level environment variable (e.g., `$env:VAULT_TOKEN`).
    3. **EXECUTE:** Scripts must **ONLY** reference the environment variable (e.g., `$env:VAULT_TOKEN`).
* **STRICT PROHIBITIONS:**
  * **NEVER** paste the actual token string into a `.ps1` or `.py` file.
  * **NEVER** pass the token as a command-line argument (it shows up in logs).
  * **NEVER** output the token value to the chat.

---

### Execution Plan

1. **Init & Auth:** Initialize `/docs/troubleshooting_plan.md` and authenticate via Vault MCP.
2. **APPROVAL GATE:** **Present the full plan and ask, "Shall we proceed?" Wait for user confirmation before running any scripts.**
3. **Reproduction:** Attempt to reproduce latency and summarize findings.
4. **Capture:** Use native Windows 10 tools to capture traffic to `captures/<timestamp>_http_latency.pcapng`.
5. **Scripting:** Execute PowerShell scripts to isolate Client vs. Server.
6. **Analysis:** Analyze pcap locally and update the Final Report.

---

## Operational Standards

### Tools & Paths

* **TShark (Wireshark):** `C:\Program Files\Wireshark\tshark.exe`

### Documentation Standards

* **Timestamp Format:** ALL logs, tables, and filenames must use **ISO 8601** (`YYYY-MM-DD HH:MM:SS`). Never use date only.
* **Evidence:** Every "Result" in a plan must reference a specific file or log line.

### File Architecture

* `scripts/simple-troubleshooting/`: Place all WMF 5.1 compatible scripts here.
  * *Naming:* `01_desc.ps1`, `02_desc.ps1` (Order matters).
* `docs/`: Living documentation.
  * *Required Table Columns:* `Timestamp` | `Host` | `Action` | `Result` | `Evidence`
  * *Host Definitions:*
    * `Local-Agent`: The machine running the LLM/Prompt (e.g., creating plans, local analysis).
    * `<TargetName>`: The remote machine where the script executed.
* `captures/`: PCAP output files (Prefix with datetime).

### Coding Guidelines

* **PowerShell:**
  * Target **WMF 5.1** compatibility.
  * **Output:** Scripts must return objects (not text) containing `Timestamp`, `Action`, and `Result`.
  * Use `Write-Verbose` for logging.
* **Python:**
  * Use `subprocess.run` instead of `os.system`.
  * Implement retry loops (3 attempts) for external API/Network calls.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/albvar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
