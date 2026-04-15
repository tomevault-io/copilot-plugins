## quillbot-api

> Git workflow and commit guidelines


# Git Workflow

## Commit Messages
- Write clear, descriptive commit messages
- Include version bump in commit message when applicable
- Follow conventional commit format when possible
- Example: "Fix language selection error and update version to 1.0.1"

## Pre-Commit Checklist
- [ ] Code changes complete
- [ ] Linting passes (`npm run lint`)
- [ ] Version updated in `package.json` (if code changed)
- [ ] Code tested and working
- [ ] No console errors or warnings (unless intentional)
- [ ] Changes verified with `node example.js` if applicable

## Commit Workflow
1. Make code changes
2. Update version in `package.json` (if code changed)
3. Run `npm run lint` and fix any issues
4. Test functionality
5. Commit with descriptive message including version bump
6. Push to trigger GitHub Actions release

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Luen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Luen)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
