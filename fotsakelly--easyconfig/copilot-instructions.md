## easyconfig

> - Run Dev: `npm run dev` or `nodemon src/index.js`

# CLAUDE.md

## Commands

### API (Backend)
- Run Dev: `npm run dev` or `nodemon src/index.js`
- Port: `5001` (by default) or check configuration

### App (Frontend)
- Run Dev: `npm start`
- Port: `3000` (by default)
- Framework: React (Chakra UI)

### Database
- Prisma Generate: `npx prisma generate`
- Run Seed: `node prisma/seed.js`

## Rules and Guidelines
- No comments in code files.
- No emojis anywhere in code, markdown, logs, or agent responses.
- Always use single sequential SSH connection session for command executions (e.g. `getServerServices`, `installService`, `uninstallService` via `withSSH`) to prevent remote host bans (ECONNREFUSED).
- Handle static React.js site deployments via `multer` for ZIP uploads and sftp streams directly to `/var/www/domain`.
- Manage deployed site files using sftp stream and find commands for listing, reading, saving, and deleting via `/api/vps/:id/files`.
- Automatic SSL setup using Certbot nginx plugin via `/api/vps/:id/deploy/:domaine/ssl` using SSE.

---
> Source: [FotsaKelly/EASYCONFIG](https://github.com/FotsaKelly/EASYCONFIG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
