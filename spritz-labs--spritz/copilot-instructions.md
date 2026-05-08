## spritz

> Production is the Vercel project **`shout`** under team **`kevinkevinjonescrs-projects`** ([deployments](https://vercel.com/kevinkevinjonescrs-projects/shout/deployments)); **`app.spritz.chat`** should alias that project—not **`eth-akash`**.


# Build, Git and Vercel deploy

Production is the Vercel project **`shout`** under team **`kevinkevinjonescrs-projects`** ([deployments](https://vercel.com/kevinkevinjonescrs-projects/shout/deployments)); **`app.spritz.chat`** should alias that project—not **`eth-akash`**.

The repo pins this via **`.vercel/project.json`** (tracked in git). If `npm run vercel` ever targets the wrong project, run:  
`npx vercel link --yes --project shout --scope kevinkevinjonescrs-projects`

Always try to run `npm run build`, commit a recap of all changes, push them to main and then run `npm run vercel` to get the app deployed in one command line.

---
> Source: [Spritz-Labs/spritz](https://github.com/Spritz-Labs/spritz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
