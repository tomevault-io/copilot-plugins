## agenda-dev

> - **Framework**: Next.js 15.3.1

# Agenda.dev - Supercharged Todo List App

## Tech Stack
- **Framework**: Next.js 15.3.1
- **Styling**: Tailwind CSS with modified shadcn/ui components
- **Authentication**: better-auth (use context7 via mcp for auth changes)
- **Database**: Drizzle ORM with Neon.tech (PostgreSQL)
- **Build Engine**: Turbopack
- Using bun and bunx to install dependencies.
- Remeber to use Context7 via mcp to be able to get up to date information on docs.

## Development Notes
- When creating new components, reference existing ones in the `components/ui` folder
- The app is a supercharged todo list application
- For all API and server requests, unless it is a server rendered page, use API routes
  - For each API route, make sure it checks for a session (if applicable) and if it is checking for a session make sure that it checks if the user is authenticated and valid. 
- In the root folder, there is a file called changelog.md. If you have implemented a change or see a new change, ask the user if you would like to add it to the changelog. Give the user in a markdown block what you would like to add to the changelog. Remember keep it concise and to the point. Never overwrite old changelog entries. Always add to the end of the file.
- Before implementing any changes that might affect other files, please explore the codebase to make sure that it is not already implemented or that it is not already in the works.
- DO NOT GIT COMMIT UNLESS I APPROVE OF IT. You can suggest a commit but do not commit it without my approval. When I ask you to prepare or get a commit ready always run `git add .` and not `git add (individual files)`.
- DO NOT PUSH TO DB UNLESS I APPROVE OF IT. You can suggest a push but do not push it without my approval.
- You will not need to make any migrations, this is handled by the drizzle-kit CLI. Use the commands `bunx drizzle-kit generate`.

---
> Source: [R44VC0RP/agenda.dev](https://github.com/R44VC0RP/agenda.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
