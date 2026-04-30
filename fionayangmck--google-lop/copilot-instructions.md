## google-lop

> Slash commands for older Cursor versions that don't support /commands folder


# Slash Commands (Backward Compatibility)

This rule provides slash command support for older Cursor versions. When a user types any of the following commands, execute the corresponding action.

## /launch

**Trigger:** User types "/launch" or "launch the app" or "start the app"

**Action:** Launch the webapp locally using Docker.

1. Check if Docker is running:
   ```bash
   docker info
   ```

2. Navigate to the app source and run:
   ```bash
   cd deployer-apps/citizen-dev5/src && ./scripts/docker-dev.sh
   ```

3. Tell the user the app will be available at http://localhost:3000

If Docker isn't running, tell them to start Docker Desktop first.
If the script fails, suggest: `./scripts/docker-dev.sh --build`

---

## /restart

**Trigger:** User types "/restart" or "restart the app"

**Action:** Stop and restart the Docker container.

```bash
docker stop agent-chat-dev 2>/dev/null
cd deployer-apps/citizen-dev5/src && ./scripts/docker-dev.sh
```

---

## /commit

**Trigger:** User types "/commit" or "commit my changes"

**Action:** Commit and push changes to git.

1. Run `git status` and `git diff` to see changes
2. Stage relevant files with `git add`
3. Create a commit with a conventional commit message
4. Push to the remote repository

---

## /commit-locally

**Trigger:** User types "/commit-locally" or "commit without pushing"

**Action:** Same as /commit but skip the push step.

---

## /push

**Trigger:** User types "/push" or "push to github"

**Action:** Push the current branch to the remote repository.

```bash
git push
```

---

## /review-code

**Trigger:** User types "/review-code" or "review my code"

**Action:** Perform a code review checklist:

1. Check for linter errors using `ReadLints`
2. Look for common issues (unused imports, console.logs, TODO comments)
3. Verify code follows project style guidelines
4. Suggest improvements

---

## /add-lessons-learned

**Trigger:** User types "/add-lessons-learned" or "document this issue"

**Action:** Help the user document an issue or learning for future reference.

1. Ask what issue or learning they want to document
2. Create or update a lessons learned file
3. Include: date, issue description, solution, and prevention tips

---

## /welcome

**Trigger:** User types "/welcome" or "how do I get started" or "what is this project"

**Action:** Provide the full welcome/onboarding guide. See the welcome.mdc rule for the complete message.

---

## /setup

**Trigger:** User types "/setup" or "set up a new project"

**Action:** Launch the interactive project setup wizard using the AskQuestion tool. See setup-workflow.mdc for details.

---

## GitHub setup (skill only — no slash command)

**Trigger:** User says "set up github", "I don't have github", "connect to github", "how do I push my code", "help with git", "github for beginners", or similar.

**Action:** Run the **GitHub Setup** skill. Read and follow `.cursor/skills/github-setup/SKILL.md` — it is the single source of truth (check installed tools, account, GitHub Desktop, sign-in, identity, add project, publish). Call the `ask_question` tool at decision points when available.

---

## /install-skill

**Trigger:** User types "/install-skill" or "install a skill" or "add a skill" or "browse skills"

**Action:** Launch the skill browser and installer.

**Step 0: Introduction** - First, provide this intro:

"**What are Skills?**

Skills are reusable capabilities that enhance your AI assistant. They provide specialized knowledge, workflows, and tools for specific domains - like document processing, code patterns, industry expertise, or productivity utilities.

Once installed, skills automatically activate when relevant to your tasks. For example, an 'inventory-expert' skill would help when you're working with supply chain data.

**Where to find skills:**
- **[skills.sh](https://skills.sh/)** - The open Agent Skills Directory with 200+ community skills
- **McKinsey internal repos** - Curated functional and industry-specific skills, such as:
  - *inventory-expert* - Understands supply chain data, stock levels, Days on Hand, reorder points, and ERP exports
  - *aerospace-jet-engines* - Knows OEMs (Pratt & Whitney, GE, Rolls-Royce), engine programs (GTF, LEAP), and part number conventions
- **Any GitHub repo** - Skills can be shared via any public or private repository

Skills are simple to install - just pick what you need and they're ready to use!"

**Step 1:** Use AskQuestion to ask what type of skills they want:
- A: Functional & Industry Skills - Operations, marketing, sales, aerospace, etc.
- B: General Skills - Coding, productivity, document processing, etc.
- C: Custom Skill(s) from a URL I have
- D: Other... - Browse 200+ community skills at skills.sh

**Step 2:** Based on selection:
- **Functional**: Use repo `McK-Private/Functional-LLM-Skills`, path `.cursor/skills`
- **General**: Use repo `McK-Internal/Vibe-Commands-Rules-Skills`, path `.cursor/skills`
- **Custom**: Ask for GitHub URL, parse it to extract repo/path
- **Other (skills.sh)**: Open https://skills.sh/ in browser, guide user to find a skill, get the GitHub URL, then treat as custom

**Step 3:** List available skills from the selected source (suppress raw output):
```bash
python .cursor/skills/skill-installer/scripts/list-skills.py --repo <repo> --path <path> --format json 2>/dev/null
```

**Step 4:** Let user select skills to install (multi-select or "all")

**Step 5:** Ask where to install:
- This workspace only (.cursor/skills/)
- All workspaces (~/.cursor/skills/)

**Step 6:** Install selected skills (suppress raw output, show clean progress):
```bash
python .cursor/skills/skill-installer/scripts/install-skill-from-github.py \
  --repo <repo> --path <skill_path> --dest <destination> 2>/dev/null
```

See .cursor/commands/install-skill.md for the full interactive flow with error handling.

---

## /client-distribution

**Trigger:** User types "/client-distribution" or "sanitize for client" or "remove mckinsey" or "prepare for client sharing"

**Action:** Sanitize and remove McKinsey information from the codebase to prepare for client distribution.

This command removes or generalizes all McKinsey-internal content:

1. **McKinsey branding** - Logos deleted, `mck-*` colors → `brand-*`
2. **Internal URLs** - AI Gateway, McKinsey ID, Platform McKinsey → placeholders
3. **GitHub workflows** - Removed (contain internal org references)
4. **Documentation** - Internal platform names → generic terms
5. **MCP server** - citdev_mcp removed (tightly coupled to McKinsey ID)
6. **Skill installer** - McKinsey repos removed, keeps skills.sh + custom URL only

**Execution:**

1. Run the setup script:
   ```bash
   ./scripts/setup-template.sh
   ```

2. When prompted, select option **2** (Client Distributable)

3. After completion, run verification:
   ```bash
   grep -ri "mckinsey" . --include="*.md" --include="*.yaml" --include="*.json" --include="*.ts" --include="*.py" | grep -v ".git"
   grep -ri "McK-" . --include="*.md" --include="*.yaml" --include="*.json" --include="*.ts" --include="*.py" | grep -v ".git"
   ```

4. Remind user to:
   - Replace all `YOUR_*` placeholders with actual values
   - Set up their own CI/CD workflows
   - Test the application still works

See .cursor/commands/client-distribution.md and CLIENT_DISTRIBUTION_CHECKLIST.md for full details.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fionayangmck) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
