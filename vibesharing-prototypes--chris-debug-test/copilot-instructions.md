## chris-debug-test

> Deployment and code structure rules for this prototype


# Deployment

This prototype is hosted on VibeSharing with a GitHub-to-Vercel pipeline.

## How to deploy

\`\`\`bash
git add .
git commit -m "Describe what changed"
git push origin main
\`\`\`

That's it. Vercel auto-deploys from this repo on every push to main.

## Deploy Rules

- Do NOT run \`vercel\`, \`vercel --yes\`, or \`vercel deploy\`
- Do NOT zip files or use any deploy API
- Do NOT create new Vercel projects
- Do NOT use curl to call vibesharing.app/api endpoints
- ONLY deploy by pushing to this git repo

## Code Structure

This is a **Next.js 14 App Router** project. Understand where files go:

- app/page.tsx — Main page visitors see. **Edit this file for your prototype.**
- app/layout.tsx — Root layout wrapper
- public/ — Static files (images, standalone HTML)

### NEVER put HTML files in the repo root

Next.js does not serve files from the root directory. An index.html or prototype.html at the repo root will NOT be accessible to visitors.

| What you have | Where it goes |
|---|---|
| React component | app/page.tsx |
| Standalone HTML file | public/filename.html |
| Images, fonts, assets | public/ |

### If keeping a standalone HTML file

Put it in public/ and update app/page.tsx to redirect:

\`\`\`tsx
import { redirect } from "next/navigation";
export default function Page() { redirect("/prototype.html"); }
\`\`\`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vibesharing-prototypes)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vibesharing-prototypes)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
