## prompt-star-next

> - Check if `package.json` exists in project root and summarize key scripts


## DEVOPS

### Guidelines for CI_CD

#### GITHUB_ACTIONS

- Check if `package.json` exists in project root and summarize key scripts
- Check if `.nvmrc` exists in project root
- Check if `.env.example` exists in project root to identify key `env:` variables
- Always use terminal command: `git branch -a | cat` to verify whether we use the `main` branch
- Always use `env:` variables and secrets attached to jobs instead of global workflows
- Always use `npm ci` for Node-based dependency setup
- Extract common steps into composite actions in separate files
- Once you're done, as a final step conduct the following: for each public action always use <tool>"Run Terminal"</tool> to see what is the most up-to-date version (use only major version) - extract tag_name from the response:
- `bash curl -s https://api.github.com/repos/{owner}/{repo}/releases/latest `

### Guidelines for CONTAINERIZATION

#### DOCKER

- Use multi-stage builds to create smaller production images
- Implement layer caching strategies to speed up builds for {{dependency_types}}
- Use non-root users in containers for better security

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matt-jb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/matt-jb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
