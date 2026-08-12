## veluwedroomchalet-nextjs

> 1. **Deployments & GitHub Commits:**

# Custom Assistant Rules

1. **Deployments & GitHub Commits:** 
   Do NOT automatically commit code to GitHub or trigger Vercel deployments. The Vercel "Edge Requests" limit is strictly 1 million per month on the Hobby plan; every deployment and subsequent test browse consumes these resources. ONLY push to GitHub/Vercel if the user explicitly asks for a production update. Keep all development and testing strictly on `localhost`.

2. **Notifications & Communication:**
Always send a notification upon completing a longer task, or when waiting for user approval or input. If you have a question or are blocked and cannot proceed, you MUST use a notification to ask the user.

3. **Localhost Testing:**
After building or making changes, always test every functionality using the browser on `localhost` to ensure it works properly. Do not stop testing and iterating until everything fully meets the given specifications.

4. **Documentation & Obsidian Sync:**
Project documentation (walkthroughs, logs, tasks) is stored in the AI "brain" folder (`/Users/arnokleinjans/.gemini/antigravity/brain/`). Keep this up to date for yourself always. When the user asks for documentation, latest progress, or project logs, verify these locations and sync the latest versions to the user's Obsidian vault at: `/Users/arnokleinjans/Obsidian/Prive/Prive/Veluwedroomchalet/`.


5. **Database Backup (Vercel KV method):**
When the user asks for a backup of the database (JSON backup), follow this procedure:
   1. Read credentials (`KV_REST_API_URL` and `KV_REST_API_TOKEN`) from `.env` or `.env.local`.
   2. Create a temporary script at `scripts/backup_kv.mjs` using `@upstash/redis` to fetch the key `veluwe_app_data`.
   3. Run the script with `node scripts/backup_kv.mjs` and capture the JSON output.
   4. Save the full JSON output to `/Users/arnokleinjans/Antigravity/[projectname]/scripts/latest_backup.json`.
   5. Provide the user with the full absolute path and a snippet of the JSON in the chat.
   6. Remove the temporary script `scripts/backup_kv.mjs` after completion.

 6. **Planning**  
 When a task is to big, plan it out step by step and ask for confirmation before starting each step.

---
> Source: [arnokleinjans/veluwedroomchalet-nextjs](https://github.com/arnokleinjans/veluwedroomchalet-nextjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
