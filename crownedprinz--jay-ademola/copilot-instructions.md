## jay-ademola

> When the user asks to "deploy", "update portfolio", "push changes", or similar deployment requests:

# Portfolio Deployment Helper

## Quick Deploy Command
When the user asks to "deploy", "update portfolio", "push changes", or similar deployment requests:

1. **Navigate to project root**: `cd /Users/crownedprinz/Documents/Projects/React/jay-ademola`
2. **Add and commit changes**: `git add . && git commit -m "Update portfolio content"`
3. **Push to GitHub**: `git push origin main`
4. **Navigate to frontend**: `cd frontend`
5. **Deploy to GitHub Pages**: `npm run deploy`

## Portfolio Structure
- **Main project**: `/Users/crownedprinz/Documents/Projects/React/jay-ademola`
- **Frontend code**: `/Users/crownedprinz/Documents/Projects/React/jay-ademola/frontend`
- **Portfolio data**: `/Users/crownedprinz/Documents/Projects/React/jay-ademola/frontend/src/data/mock.js`
- **Live URL**: https://crownedprinz.github.io/jay-ademola/

## Common Updates
- **Resume link**: Update `contact.resume` in `mock.js`
- **Profile info**: Update personal details in `mock.js`
- **Projects**: Add/edit entries in `projects` array in `mock.js`
- **Experience**: Update `experience` array in `mock.js`

## Development Server
- **Start local server**: `cd frontend && npm start`
- **Local URL**: http://localhost:3000
- **Always run from frontend folder**

## Deployment Notes
- Portfolio uses React Router with `basename="/jay-ademola"` for GitHub Pages
- Build output goes to `frontend/build/`
- GitHub Pages serves from `gh-pages` branch (auto-created)
- Changes are live within 1-2 minutes after deployment

## Quick Commands
```bash
# Full deployment workflow
cd /Users/crownedprinz/Documents/Projects/React/jay-ademola
git add . && git commit -m "Update portfolio"
git push origin main
cd frontend && npm run deploy

# Start development
cd /Users/crownedprinz/Documents/Projects/React/jay-ademola/frontend
npm start
```

When user requests deployment, execute the full workflow automatically and confirm completion with the live URL.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Crownedprinz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
