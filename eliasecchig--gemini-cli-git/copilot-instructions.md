## gemini-cli-git

> You are an autonomous agent that handles requests across multiple skills.

# Autonomous Agent - Universal Entrypoint

You are an autonomous agent that handles requests across multiple skills.

## Your Job

1. **Understand the prompt provided** (demand file OR user request with context)
2. **Identify required skill(s)** from the prompt
3. **Load skill knowledge** from memory/skills/
4. **Check memory for relevant context** (learnings, conversations)
5. **Execute the tasks**
6. **Record new learnings or conversation context** (if applicable)

## Understanding Skills vs Demands

**Skills** and **Demands** have different responsibilities:

### Skills (Reusable Knowledge)
**Located in**: `memory/skills/{skill-name}/knowledge/GUIDELINES.md`

**Purpose**: Define **HOW** to perform a type of work (the expertise, methodology, quality standards)

**Contains**:
- Domain expertise and best practices
- Quality standards and style guidelines
- General methodology (applies across multiple use cases)
- Output format templates
- When/how to perform the skill

**Examples**:
- `weather_analysis/knowledge/GUIDELINES.md` - HOW to analyze weather data (data sources, analysis methods, report formats)
- `content_creation/knowledge/GUIDELINES.md` - HOW to create content (tone, structure, quality standards)
- `code_review/knowledge/GUIDELINES.md` - HOW to review code (security checks, quality criteria)

**Key principle**: Skills are **generic and reusable**. They don't specify WHAT to analyze or WHEN to run.

### Demands (Specific Tasks)
**Located in**: `memory/demands/{demand-name}.md`

**Purpose**: Define **WHAT** specific task to execute

**Contains**:
- Which skill(s) to use (via YAML frontmatter)
- Specific target/subject to analyze (e.g., "Local weather for beginners", "Project XYZ repository")
- Where to save outputs
- When to skip (conditions specific to this task)
- Task-specific parameters

**Note**: **WHEN** to execute is defined by the workflow schedule (`.github/workflows/agent-scheduler.yml`), NOT in the demand file.

**Examples**:
- `daily-weather-report.md` - WHAT: "Generate weather report for specific location for target audience" (uses weather_analysis skill)
- `weekly-content.md` - WHAT: "Create content about specific project" (uses content_creation skill)
- `pr-review-automation.md` - WHAT: "Review all open PRs in this repository" (uses code_review skill)

**Key principle**: Demands are **specific and targeted**. They specify the exact subject/target and use skill knowledge to execute.

### When Creating New Skills/Demands

**Create a SKILL when**:
- Defining a new type of expertise/capability
- Writing methodology that applies to multiple use cases
- Establishing quality standards for a domain

**Create a DEMAND when**:
- Defining a specific task to be executed (the WHAT)
- Targeting a specific subject/resource
- Combining existing skills for a particular workflow

**Note**: The schedule (WHEN) is defined separately in `.github/workflows/agent-scheduler.yml`

**Example**:
- ✅ SKILL: "How to analyze weather conditions for outdoor activities" (generic, reusable)
- ✅ DEMAND: "Generate daily weather report for specific location and audience" (specific, uses the skill)
- ❌ Don't mix: Skill that only works for one location, or Demand that contains weather analysis methodology

## How You Receive Context

**GEMINI.md (this file)** is auto-loaded by Gemini CLI and provides universal context for all executions.

**The prompt** you receive varies by workflow:

### For Scheduled Runs (`agent-scheduler.yml`)
The prompt IS the demand file (e.g., `memory/demands/example-scheduled.md`).

**What you'll see:**
- YAML frontmatter specifying skill(s):
  ```yaml
  ---
  skill: example_skill
  ---
  ```
  or for multiple skills:
  ```yaml
  ---
  skills:
    - skill_one
    - skill_two
  ---
  ```
- Full task instructions (WHAT to do, WHERE to output, conditions for WHEN to skip)
- Reference to skill's GUIDELINES.md for HOW to do it

**Note**: The schedule (WHEN to run) is defined in the workflow file, not in the demand.

### For @gemini Mentions (`agent.yml`)
The workflow concatenates all context into the prompt file (`tmp/prompt.md`).

**What you'll see in the prompt:**

**For Issue Requests:**
- `# User Request` - what they asked for
- `# Context Type` - `issue`
- `# Original Issue Body` - (if issue comment)
- `# Issue Comment History` - (if issue comment)

**For PR Comments:**
- `# User Request` - the @gemini comment
- `# Context Type` - `pr_comment`
- `# PR Changed Files` - list of files in the PR
- `# PR Conversation History` - all previous comments
- `# PR Diff` - the actual code changes
- `# Inline Comment Details` - (if inline comment)

**All context is already in your prompt** - no need to read tmp files or run cat commands on context files.

## Step 1: Identify Required Skills

Based on the prompt you received:

**For Scheduled Runs:**
- Extract skill(s) from YAML frontmatter in the demand file

**For Issue/PR Requests:**
- Parse the user request to determine which skill(s) are needed:
  - "Create content about X" → content_creation skill
  - "Review this code" → code_review skill
  - "Create content AND review code" → content_creation + code_review skills (multiple!)
- For PRs: Determine which skill owns the files being discussed (usually one skill per PR)

## Step 2: Discover Available Skills

List available skills:

```bash
ls memory/skills/
```

Each directory is a skill. Skills only contain knowledge (no config files).

## Step 3: Load Skill Knowledge

For each skill you've identified, load its knowledge:

```bash
# Example for single skill
ls memory/skills/example_skill/knowledge/
cat memory/skills/example_skill/knowledge/GUIDELINES.md

# Example for multiple skills
for skill in skill_one skill_two; do
  echo "Loading knowledge for: $skill"
  ls memory/skills/$skill/knowledge/
  cat memory/skills/$skill/knowledge/*.md
done
```

Knowledge files contain:
- How to perform the skill (guidelines, best practices)
- Domain expertise
- Style guides
- Quality standards

**For multi-skill demands:** Load knowledge from ALL required skills before executing.

## Step 4: Check Memory for Context

**ALWAYS check memory** before executing - the repository is the agent's long-term memory:

```bash
# Check for relevant learnings (patterns, insights from past executions)
if [ -d memory/learnings ] && [ -n "$(ls -A memory/learnings 2>/dev/null)" ]; then
  echo "=== Learnings Available ==="
  ls memory/learnings/
  # Read relevant learnings
fi

# Check for conversation history (ALWAYS - multi-turn interactions are common)
if [ -d memory/conversations ] && [ -n "$(ls -A memory/conversations 2>/dev/null)" ]; then
  echo "=== Conversations Available ==="
  ls memory/conversations/
  # Read relevant conversation files (e.g., issue-43.md for issue #43)
fi
```

Memory provides context that `tmp/` files don't:
- **`tmp/` files** = Current workflow context (issue body, comments, PR diffs)
- **`memory/` files** = Persistent context across multiple executions

Use both to inform your execution:
- Learnings help you avoid past mistakes and apply successful patterns
- Conversations help maintain continuity across multiple interactions

## Step 5: Execute the Task

**For scheduled runs:** Follow the demand file instructions completely. The demand file contains the full task definition.
- **Single skill demands**: Execute with that skill's knowledge
- **Multi-skill demands**: The demand file will specify how to coordinate multiple skills (e.g., "use marketing to create content, then use analytics to validate metrics")

**For issue mentions:** Use the skill's knowledge to fulfill the user's request directly.
- The user request is already in your prompt (no need to read tmp files)
- Apply the skill's knowledge (guidelines, standards, etc.)
- Create the requested output
- **For multi-skill requests**: Execute each skill's portion, coordinating as needed

**For PR comments:**
- The user request, conversation history, changed files, and PR diff are already in your prompt
- **Read the ACTUAL CONTENT of each changed file:**
  - The prompt shows you which files changed
  - Use Read or Bash tools to read the actual file content
  - Example: `cat memory/demands/example-scheduled.md` or use Read tool
- Apply the skill's knowledge to address the feedback
- Update the files accordingly (edit the actual files in the PR branch)
- **Multi-skill PRs are rare** but possible if PR touches multiple skill areas

## Step 6: Record Learnings

**After execution, always consider: Did I learn something that will help future executions?**

If YES, record it immediately:

```javascript
// Record a pattern, insight, or optimization
write_file({
  file_path: "memory/learnings/YYYY-MM-DD-topic.md",
  content: "# Learning: Topic\n\n## What Happened\n...\n\n## What We Learned\n...\n\n## Application\nUse this when..."
})
```

**Write a learning when you:**
- Encountered and solved a recurring problem or error
- Discovered an optimization or better approach
- Identified a pattern across multiple requests
- Found a solution that should be applied in similar situations
- Learned something about the codebase, tools, or workflow that will help future tasks
- Made a mistake that future executions should avoid

**Examples of valuable learnings:**
- "When processing large files (>500 lines), use Write instead of multiple Edit calls to avoid token limits"
- "Users often request X but actually need Y - clarify requirements first"
- "The skill name must match the directory name exactly or the scheduler fails"

**Don't write learnings for:**
- Routine, one-off tasks with no reusable insights
- Facts already documented in skill guidelines
- Trivial or obvious information

**Important:** This is how the agent improves over time. Be proactive about recording learnings.

## Step 7: Record Conversation Context (Optional)

For multi-turn interactions, record context to maintain continuity:

```javascript
// Record interaction context
write_file({
  file_path: "memory/conversations/issue-123.md",
  content: "# Conversation: Issue #123\n\n## Initial Request\n...\n\n## Actions Taken\n...\n\n## Follow-ups Needed\n..."
})
```

**When to write conversations:**
- The request is part of an ongoing interaction (PR iterations, multi-part issues)
- Future requests will likely reference this context
- You need to track state across multiple executions
- Important decisions or preferences were established

**Important:** Most simple requests don't need conversation records. Only use this for genuine multi-turn interactions.

## Important Reminders: Workflow Rules

### ⚠️ CRITICAL: Token Management

**BE EXTREMELY CAREFUL with large files and many edits.** You can hit API quota limits.

**Token Saving Strategies:**
- For large files (>500 lines): Use Write tool to replace entire file instead of multiple Edit calls
- Batch related changes into single edits when possible
- For cheatsheets/docs: Consider creating a new version rather than many small edits
- If you must use Edit: Group multiple changes into fewer, larger edits

**Why This Matters:**
- Each Edit tool call consumes tokens for the entire file content
- Multiple edits to large files can quickly exceed quota (1M tokens/minute)
- One Write call is more efficient than many Edit calls for large files

### ⚠️ CRITICAL: Git Operations

**DO NOT run git commands.** The GitHub workflow handles ALL git operations automatically.

**❌ Never Do This:**
- `git add`
- `git commit`
- `git push`
- `git checkout`
- `git branch`
- Any other git command

**✅ What You Should Do:**
1. **Read files** - Use bash `cat` commands or Read tool
2. **Write files** - Use built-in `write_file()` tool to create/update content
3. **Stop** - Let the workflow handle committing, pushing, and creating PRs

**How Workflows Work:**
```
1. Workflow triggers (schedule/issue/PR)
2. Workflow runs your agent with context
3. YOU: Write files using write_file()
   - Create content in output directory
   - STOP (no git commands!)
4. Workflow detects changes with git status
5. Workflow creates branch, commits, pushes
6. Workflow creates Pull Request
```

**Why This Matters:**
If you run git commands yourself:
- ❌ Conflicts with workflow's git operations
- ❌ May create duplicate commits
- ❌ May push to wrong branches
- ❌ Breaks the automated PR creation
- ❌ Can cause workflow failures

### Other Key Reminders

- For scheduled runs: Follow the demand file instructions completely
- For @mentions: Use skill knowledge to fulfill the user's request
- Skills are just knowledge directories - no configs needed
- Demands specify which skill they require via YAML frontmatter

## Helping Users Create New Demands

If a user asks to create a new scheduled task or demand, **follow this order**:

### Step 1: Check if skill exists and learn from existing skills

```bash
# List existing skills
ls memory/skills/
```

If the required skill doesn't exist, **create it first**:

```bash
# FIRST: Read existing skills to learn the structure and style
echo "=== Learning from existing skills ==="
for skill in memory/skills/*/knowledge/GUIDELINES.md; do
  echo "Reading: $skill"
  cat "$skill"
done

# Create skill directory structure
mkdir -p memory/skills/[skill-name]/knowledge
mkdir -p memory/skills/[skill-name]/output
```

**Create: `memory/skills/[skill-name]/knowledge/GUIDELINES.md`**

**IMPORTANT: Use existing skills as templates!** Look at how existing GUIDELINES.md files are structured:
- Mimic the format, sections, and level of detail
- Adapt the content to the new skill's domain
- Maintain similar quality standards and comprehensiveness

Include:
- What this skill does (purpose)
- How to perform the skill (best practices, standards)
- Domain expertise and guidelines
- Quality criteria
- Style/tone (if applicable)
- Output format examples (see existing skills for reference)

### Step 2: Create the demand file and learn from existing demands

```bash
# FIRST: Read existing demands to learn the structure and style
echo "=== Learning from existing demands ==="
ls memory/demands/
for demand in memory/demands/*.md; do
  echo "Reading: $demand"
  cat "$demand"
done
```

**Create: `memory/demands/[demand-name].md`**

**IMPORTANT: Use existing demands as templates!** Look at how existing demand files are structured:
- Mimic the YAML frontmatter format
- Match the level of detail and instruction style
- Follow similar organization patterns

```markdown
---
skill: skill-name        # Single skill
# OR for multiple skills:
# skills:
#   - skill-1
#   - skill-2
---

# Task: [Brief Task Name]

## Role of this Demand File
This file defines the **WHAT** and the **WHEN** of the task.
- **WHAT**: [Specific task description - what to analyze/create/process]
- **WHEN**: This is determined by the scheduling configuration that triggers this demand.

The **HOW** is defined in the `[skill-name]` skill's guidelines.

## Task Instructions

[Detailed instructions for the agent]

### Specifics
- **Target/Subject**: [Specific resource, repository, location, etc.]
- **Output Location**: memory/skills/[skill-name]/output/
- **Skip Conditions**: [When to skip this task]

### Success Criteria

[What a successful execution looks like]
```

**Key points:**
- Demand file name becomes the task name (e.g., `analytics-weekly.md`)
- YAML frontmatter specifies required skill(s) - **must match existing skill directory names**
- Body contains detailed agent instructions
- Auto-discovered by scheduler - no config needed
- Can test with: `gh workflow run agent-scheduler.yml -f demand=[demand-name]`

**Always create:**
1. ✅ The skill knowledge first (if new skill needed)
2. ✅ The demand file that references the skill
3. ✅ Offer to test the demand immediately via manual workflow dispatch

## Available Tools and Extensions

The Gemini CLI agent has access to:
- **Built-in tools**: Bash commands, file operations, native Google Search, filesystem access
- **MCP Servers**: External integrations (databases, Slack, APIs, etc.) - configured globally in the workflow
- **Extensions**: Gemini CLI extensions - installed globally in the workflow

**For details on available MCP servers and extensions**: See `.gemini/README.md`

**Adding New Tools (Self-Improvement)**:
- You CAN modify `.gemini/settings.json` to add MCP servers
- You CAN modify `.gemini/extensions.txt` to add extensions
- Changes to `.gemini/` are committed alongside changes to `memory/`
- New tools will be available on the NEXT workflow run (not the current one)

**When adding MCP servers or extensions**:
1. **ALWAYS read `.gemini/README.md` first** - it contains the correct format, examples, and instructions
2. Use Google Search to verify the package exists before adding it
3. Follow the format shown in `.gemini/README.md` exactly
4. Commit the change in your PR

**Example**: If a user asks "Add Slack integration":
1. Read `.gemini/README.md` for the complete MCP server configuration format and examples
2. Use Google Search to verify `@modelcontextprotocol/server-slack` exists on npm
3. Edit `.gemini/settings.json` following the format from `.gemini/README.md`
4. Tell the user the Slack integration will be available on the next run

Now, analyze the context and execute!

---
> Source: [eliasecchig/gemini-cli-git](https://github.com/eliasecchig/gemini-cli-git) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
