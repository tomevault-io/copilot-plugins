## skills-domain-finder

> Use this skill to generate meaningful domain names and check their availability. It runs a standalone Python script to check parallel lists of names.


# Domain Finder

This skill helps agents find available `.com` domain names based on dynamically generated keywords.

## How it Works

The process is split into two steps to ensure flexibility:

1.  **Ideation (Agent)**: The Agent (you) generates a list of 50-100 meaningful, creative names based on the user's specific query.
2.  **Verification (Script)**: The Agent saves these names to a file and runs the script to check their availability.

## Prerequisites

1.  **Python 3**
2.  **`whois` command** (install via `brew install whois` or `apt-get install whois`)

## Application Interface

### Script Usage

```bash
python3 scripts/check_domains.py <input_file> [--tld .com] [--output results.txt]
```

- `<input_file>`: A text file with one domain name (or word) per line.
- `--tld`: (Optional) Top-level domain to append if the input lines are just words (default: `.com`).
- `--output`: (Optional) File to save available domains to.

## Workflow for Agents

When a user asks to find a domain for a specific topic (e.g., "AI cooking app"):

1.  **Brainstorm**: Generate a list of 50-100 relevant, creative names.
    - *Example*: `chefai`, `cookbot`, `sizzle`, `pan`, `gourmet`, `tastyai`...
2.  **Save to File**: Write these names to a temporary file (e.g., `candidates.txt`).
3.  **Run Script**: Execute the checker.
    ```bash
    python3 skills/custom/domain-finder/scripts/check_domains.py candidates.txt --output available.txt
    ```
4.  **Read Results**: Read `available.txt` and present the available options to the user.

---
> Source: [kirklin/skills-domain-finder](https://github.com/kirklin/skills-domain-finder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
