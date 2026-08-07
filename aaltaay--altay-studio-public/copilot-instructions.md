## browser-testing

> Rules for web verification and browser testing using local browser connection or agent-browser. Use when finishing tasks involving web deployments, frontend changes, or UI updates.


# Browser Testing & Web Verification Guidelines

## 1. Web Verification Requirement
At the end of every task involving web deployments or changes, you MUST open a headless/headed browser (using `agent-browser` or Playwright) and test the actual live URL or local dev server to ensure it loads successfully and functions correctly before declaring the task complete.

## 2. Available Browser Skills
- **Agent Browser CLI (`agent-browser`):** Used for fast, lightweight browser tasks.
  - Commands:
    - `npx agent-browser@latest open <url>`
    - `npx agent-browser@latest snapshot` (Get DOM snapshot with `@eN` references)
    - `npx agent-browser@latest click '@eN'`
    - `npx agent-browser@latest fill '@eN' "text"`
    - `npx agent-browser@latest screenshot --annotate ./page.png`
- **Local Browser Connection (CDP):** For using the user's personal authenticated browser session instead of a fresh isolated instance.
  - Chrome port: `9222`
  - Edge port: `9223`
  - **MANDATORY Security Protocol:** Before connecting to the local browser via Playwright/CDP, you MUST run:
    `[YOUR_LOCAL_PATH]\anaconda3\python.exe "[YOUR_LOCAL_PATH]\github\AhmiOS\local-browser-skill\security_prompt.py" --browser [Chrome/Edge]`
    Only proceed with the CDP connection if this script prints `APPROVED`.

---
> Source: [aaltaay/altay-studio-public](https://github.com/aaltaay/altay-studio-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
