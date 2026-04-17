## swarm-notes-template

> This file provides system instructions for AI coding assistants (Cursor, Claude, windsurf, etc.) interacting with this workspace.

# Swarm Notes System Instructions

This file provides system instructions for AI coding assistants (Cursor, Claude, windsurf, etc.) interacting with this workspace.

## Setup Instructions

If the user asks you to "set up the repository", "configure the template", or "initialize Swarm Notes", you must perform the following setup process:

1. **Gather Information**  
   Ask the user for:
   - Their GitHub username
   - Their repository name
   - Their research interests/keywords  
   *(Wait for their response before proceeding to step 2).*

2. **Configure `configs/site.json`**  
   Update the deployment configuration:
   - `site`: "https://<username>.github.io"
   - `base`: "/<repo-name>/" (Unless they want a custom domain, then use "/")
   - `github_url`: "https://github.com/<username>/<repo-name>"

3. **Configure `configs/config.ts.yaml`**  
   Update the search `keywords` used by the autonomous swarm based on the topics they provided.

4. **Enable the Tracker**  
   In `.github/workflows/autonomous-tracker.yml`, uncomment the line `# - cron: "0 5 * * *"` to enable the daily scheduled agent runs.

5. **Provide Manual Next Steps**  
   Instruct the user to:
   - Navigate to **Settings > Pages** and set the deployment source to **GitHub Actions**.
   - Navigate to **Settings > Secrets and variables > Actions** and add their API key (e.g., `GOOGLE_API_KEY` or `OPENAI_API_KEY`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kausalflow) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
