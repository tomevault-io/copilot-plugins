## commit-push-deploy

> description: Run commit, push, and deploy operations at the end of every task

﻿---
description: Run commit, push, and deploy operations at the end of every task
alwaysApply: true
---

# Commit, Push, and Deploy after Every Task

At the end of every task or whenever you complete a chunk of work that has been verified/tested:

1. **Verify the changes**
   - Ensure the application builds and all automated tests pass before committing.

2. **Commit the changes**
   - Stage all intentional changes: `git add -A` (be careful not to commit secrets or untracked local environment files).
   - Commit with a clear, descriptive, and imperative commit message summarizing the changes.

3. **Push to the remote repository**
   - Push the committed changes to the current branch on the remote: e.g., `git push` or `git push -u origin HEAD` if the upstream is not set.

4. **Deploy the changes**
   - Identify the project's deployment mechanism:
     - **Git-Triggered Deployments**: If the project automatically deploys via Git pushes (e.g., connected to Vercel/Netlify GitHub integrations), confirm that pushing the changes is sufficient to trigger the deploy.
     - **Vercel CLI / MCP**: If the `vercel` CLI or MCP tool is configured, deploy the changes (e.g. `vercel deploy` or `vercel --prod`).
     - **Supabase CLI / MCP**: If Supabase edge functions or schemas were modified, deploy them using the `supabase` tool/CLI.
     - **Custom Deploy Script**: Check if there's a deploy script/command in `package.json` (e.g., `npm run deploy`), `Makefile`, or project documentation and execute it.
   - If the deployment command or target is not clear, ask the user for confirmation or check project documentation.
   - Report the deployment status clearly to the user.

Do not skip any of these steps (Commit, Push, Deploy) unless explicitly instructed otherwise.

---
> Source: [aaltaay/altay-studio-public](https://github.com/aaltaay/altay-studio-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
