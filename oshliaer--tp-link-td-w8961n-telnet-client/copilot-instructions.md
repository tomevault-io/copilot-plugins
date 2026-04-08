## tp-link-td-w8961n-telnet-client

> This project is a Python-based Telnet client designed to interact with TP-Link TD-W8961N routers. Its primary purpose is to fetch and display real-time status, ADSL rate, and line quality (noise margin, attenuation) information from the router. The script connects to the router via Telnet, sends specific commands, and parses the output using regular expressions.

# TP-Link TD-W8961N Telnet Client

## Project Overview

This project is a Python-based Telnet client designed to interact with TP-Link TD-W8961N routers. Its primary purpose is to fetch and display real-time status, ADSL rate, and line quality (noise margin, attenuation) information from the router. The script connects to the router via Telnet, sends specific commands, and parses the output using regular expressions.

**Источник кода:** [https://gist.github.com/mdavey/63b43df11456c99ef1c8](https://gist.github.com/mdavey/63b43df11456c99ef1c8)

## Building and Running

This project consists of a single Python script and does not require a separate build step.

To run the client:

```bash
python tp-w8961n-status.py
```

The script will connect to the router at `192.168.1.1` (default IP, hardcoded in the script), log in with username `admin` (hardcoded password `admin`), and continuously fetch and print router status information every 60 seconds.

**Note:** Ensure you have Python 2 installed, as the script uses Python 2 syntax (e.g., `print` statements).

## Development Conventions

*   **Language:** Python 2
*   **Router Interaction:** Uses the `telnetlib` module for establishing a Telnet connection and sending commands to the router.
*   **Data Parsing:** Employs the `re` module (regular expressions) to parse the text-based output received from the Telnet session.
*   **Object-Oriented Design:** Commands are encapsulated within classes (`StatusCommand`, `RateCommand`, `QualityCommand`) inheriting from a base `Command` class, promoting modularity.
*   **Error Handling:** Basic error handling is implemented for Telnet connection issues and keyboard interrupts.

## Правила разработки

*   **Язык:** Весь текст, комментарии и сообщения для пользователя должны быть на русском языке на момент разработки.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oshliaer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/oshliaer)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
