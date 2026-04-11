## moneymanual

> This is Money Manual available at moneymanual.org.uk and moneymanual.uk (I can't decide which should be canocical). It's a free and open source website containing information about money, personal finance and the British economy.

# About this project
This is Money Manual available at moneymanual.org.uk and moneymanual.uk (I can't decide which should be canocical). It's a free and open source website containing information about money, personal finance and the British economy.

The target audience is the British public.

# The Tech Stack
This website is built with Astro and deployed to Cloudflare Workers. This is done so I don't have to manage any servers, and spikes in traffic are handled without bankrupting me.

All content is written in Markdown (actually MDX to include components). It's using Astro Content Collections to avoid managing a seperate CMS. This means any change requires the website to be redeployed.

The website uses Svelte for any interactive components and tools. This is to keep the frontend bundle lightweight.

Speaking of lightweight, there is an intense focus on keeping the overall website very lightweight and fast.

## Styling
This website use TailwindCSS. This makes it extra important to use and create components thoughtfully to avoid uneccesary styling repetition.

Do not create custom classes unless absolutely neccesary. Do not use custom classes such as p-[21px], prefer the default design tokens.

## Svelte and astro components
Only create a Svelte component if it requires state and logic otherwise prefer a vanilla Astro component.

## Typescript
Typescript is a strict requirement and the any type is strictly banned.

# Documentation
ALWAYS look up documentation to make sure you're using the most current syntax. Javascript frameworks change almost daily.

# Dependencies
Avoid using dependencies where possible. ALWAYS pin dependency version and use the latest version.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/simse)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/simse)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
