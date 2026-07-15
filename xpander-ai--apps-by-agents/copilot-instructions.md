## apps-by-agents

> Welcome, AI Agent! This repository is your workspace to **design and build** applications autonomously. Here's how to contribute:

## 🧠 Instructions for AI Agents

Welcome, AI Agent! This repository is your workspace to **design and build** applications autonomously. Here's how to contribute:

---

### ✅ Step-by-Step Instructions

1. **Understand Your Mission**

   * Your human collaborator will provide an objective: a description of the app to build.
   * You are expected to independently design, implement, and document the application without human help.

2. **Create Your Workspace**

   * Determine the branch name and directory using the format:

     ```
     {month-year}/{username}/{app-random-name}
     ```

     Examples:

     ```
     may-2025/xpander-ai/joyful-scene
     june-2025/janedoe/colorful-clock
     ```

   * The `username` must match the output of the following Git command:

     ```bash
     git config user.name
     ```

   * Place **all code and assets** inside your designated folder:

     ```
     /month-year/username/app-random-name/
     ```

3. **Develop Your App**

   * You must only use the following technologies:

     * ✅ HTML
     * ✅ CSS
     * ✅ JavaScript (Vanilla)
     * ✅ Images (PNG, JPG, SVG, etc.)
     * ❌ No frameworks or libraries (e.g., no React, Vue, Bootstrap, jQuery)

   * Include a `README.md` with:

     * A description of your app
     * Setup or usage instructions (if needed)
     * A statement affirming that you are the AI author
     * Your AI Stack (see below)

4. **Add Your AI Stack**

   In your `README.md`, include:

   * `Type`: `"single-agent"` or `"multi-agent"`

   * **Agent Card** in JSON format:

     ```json
     {
       "name": "YourName",
       "description": "Brief description of your capabilities",
       "url": "https://public-url-if-applicable.com",
       "provider": {
         "organization": "Your Provider"
       },
       "version": "Version",
       "authentication": {
         "schemes": ["oauth2", "api_key", "basic"],
         "credentials": "How to get credentials"
       },
       "skills": [
         {
           "id": "tool_id",
           "name": "Tool Name",
           "description": "What this tool does"
         }
       ]
     }
     ```

   * Models: List the LLM(s) you used

   * Framework: Only mention foundational tools used to generate or assist in reasoning—**no UI frameworks allowed**

   * Platform: `xpander.ai`, `self-hosted`, or other

5. **Report Back**

   After completing your work, report with:

   * The branch name used
   * A summary of your changes (as a commit message)
   * Optional notes or observations

---

### 🧭 Notes for AI Agent Success

* Keep all code **inside your app’s folder**—do not write to root or other agents’ directories.
* Follow the required naming and folder structure exactly.
* Respect other agents’ work: **do not overwrite or modify** other applications.
* Do not include `node_modules`, build artifacts, or any non-source files.
* Code must be clean, modular, and well-documented.
* Sign your `README.md` with a statement affirming your AI authorship.
* Use only HTML, CSS, JS—**no external libraries or frameworks** are permitted.

---
> Source: [xpander-ai/apps-by-agents](https://github.com/xpander-ai/apps-by-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
