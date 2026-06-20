## skill-creator

> Create new GitHub Copilot custom instructions, prompt files, and agent skills. Helps with creating reusable customizations from scratch, optimizing existing files, testing effectiveness, and benchmarking performance.


# Copilot Prompt Creator

A skill for creating new GitHub Copilot customizations and iteratively improving them.

At a high level, the process of creating a custom instruction goes like this:

- Decide what you want the customization to do and roughly how it should do it
- Write a draft of the file (`.prompt.md`, `.instructions.md`, or `SKILL.md`)
- Create a few test scenarios and run them manually in Copilot Chat using `/promptName` (for prompt files and skills) or by working with matching files (for instructions)
- Help the user evaluate the results both qualitatively and quantitatively
  - While the user tests, draft some quantitative assertions if there aren't any. Then explain them to the user (or if they already existed, explain the ones that already exist)
  - Use the `eval-viewer/generate_review.py` script to show the user the results for them to look at, and also let them look at the quantitative metrics
- Rewrite the customization based on feedback from the user's evaluation of the results (and also if there are any glaring flaws apparent from the quantitative benchmarks)
- Repeat until you're satisfied
- Expand the test set and try again at larger scale

Your job when using this skill is to figure out where the user is in the process and then jump in and help them progress through these stages. For instance, maybe they say "I want to make a prompt for X". You can help narrow down what they mean, write a draft, write test cases, figure out how they want to evaluate, run the prompts, and repeat.

On the other hand, maybe they already have a draft of the prompt. In this case you can go straight to the eval/iterate part of the loop.

Of course, you should always be flexible -- if the user says "I don't need to run a bunch of evaluations, just vibe with me", you can do that instead.

## Communicating with the user

The prompt creator is liable to be used by people across a wide range of familiarity with coding jargon. There's a trend now where AI tools are inspiring plumbers to open up their terminals, parents and grandparents to learn new tools. On the other hand, the bulk of users are probably fairly computer-literate.

So please pay attention to context cues to understand how to phrase your communication! In the default case, just to give you some idea:

- "evaluation" and "benchmark" are borderline, but OK
- for "JSON" and "assertion" you want to see serious cues from the user that they know what those things are before using them without explaining them

It's OK to briefly explain terms if you're in doubt, and feel free to clarify terms with a short definition if you're unsure if the user will get it.

---

## GitHub Copilot Customization Overview

Before diving into the creation process, here's how AI customization works in GitHub Copilot. There are several types, each serving a different purpose:

### Types of Customizations

1. **Always-on instructions** (`.github/copilot-instructions.md`):
   - Automatically included in ALL Copilot interactions within the repository
   - Good for project-wide conventions (coding style, architecture patterns, tooling preferences)
   - Single file per repository. Also supports `AGENTS.md` (cross-agent) and `CLAUDE.md` (Claude-compatible)

2. **File-based instructions** (`.github/instructions/*.instructions.md`):
   - Applied dynamically based on file patterns (`applyTo` glob) or description matching
   - Good for language-specific conventions, framework patterns, or rules for certain parts of your codebase
   - Each file targets specific file types or paths

3. **Prompt files / Slash commands** (`.github/prompts/*.prompt.md`):
   - Invoked manually with `/promptName` in Copilot Chat
   - Good for repeatable tasks like scaffolding components, running tests, preparing PRs
   - Support different agents: `agent`, `ask`, `plan`, or custom agents
   - Can reference workspace files using Markdown links
   - This is the primary format we create in this workflow

4. **Agent Skills** (`.github/skills/<skill-name>/SKILL.md`):
   - Folders containing instructions, scripts, examples, and resources
   - Loaded on-demand based on task relevance, also available as `/skillName` slash commands
   - Open standard ([agentskills.io](https://agentskills.io/)) -- works across VS Code, Copilot CLI, and Copilot coding agent
   - Good for specialized workflows with supporting scripts and resources

5. **Custom agents** (agent definition files):
   - Specialized AI personas with specific tools and capabilities
   - Can orchestrate sub-agents for complex workflows

### Prompt File Format (.prompt.md)

```markdown
---
description: Brief description of what this prompt does
agent: agent
tools:
  - codebase
  - terminal
  - github
---

# Prompt Title

Instructions go here...

Reference workspace files with Markdown links:
See [coding standards](../../coding-standards.md) for conventions.
```

### Prompt File Frontmatter Fields

- **`description`** (optional): Human-readable description shown when browsing with `/` in chat
- **`name`** (optional): Display name for the prompt. Defaults to the filename.
- **`agent`** (optional): The agent used for running the prompt
  - `agent`: For multi-step tasks that may use tools (default when tools are specified)
  - `ask`: For questions and explanations
  - `plan`: For generating implementation plans
  - Or the name of a custom agent
- **`tools`** (optional): List of tools available for this prompt
  - `codebase`: Search and read workspace files
  - `terminal`: Run shell commands
  - `github`: GitHub operations (issues, PRs, etc.)
  - `fetch`: Make HTTP requests
  - MCP server tools (use `<server-name>/*` for all tools from a server)
- **`model`** (optional): Language model to use when running the prompt
- **`argument-hint`** (optional): Hint text shown in chat input when the prompt is invoked

**Note:** Prompt files do NOT have `applyTo`. If you need auto-attachment based on file patterns, use `.instructions.md` files instead.

### Instructions File Format (.instructions.md)

```markdown
---
name: 'Python Standards'
description: 'Coding conventions for Python files'
applyTo: '**/*.py'
---
# Python coding standards
- Follow PEP 8 style guide
- Use type hints for all function signatures
```

**Instructions file frontmatter:**
- **`name`** (optional): Display name shown in the UI
- **`description`** (optional): Short description shown on hover
- **`applyTo`** (optional): Glob pattern for auto-attachment (e.g. `"**/*.py"`, `"**/components/**"`)

**Location:** `.github/instructions/` directory (configurable via `chat.instructionsFilesLocations` setting)

### Agent Skills Format (SKILL.md)

```markdown
---
name: webapp-testing
description: Run and debug web application tests with Playwright and vitest
---

# Web Application Testing

Instructions for testing web apps...
```

**SKILL.md frontmatter:**
- **`name`** (required): Unique identifier, kebab-case, must match parent directory name. Max 64 chars.
- **`description`** (required): What the skill does and when to use it. Max 1024 chars.
- **`argument-hint`** (optional): Hint text shown in chat input
- **`user-invocable`** (optional): Whether the skill appears as a `/` slash command. Default: `true`
- **`disable-model-invocation`** (optional): Prevent auto-loading. Default: `false`

**Location:** `.github/skills/<skill-name>/SKILL.md` (can include scripts, examples, resources alongside)

### How Triggering Works

Unlike some AI tools that auto-detect when to use a skill, Copilot customizations use different triggering mechanisms:

- **Slash commands**: User types `/promptName` or `/skillName` in Copilot Chat
- **Auto-matching**: Agent Skills are automatically loaded when the task matches the skill description
- **File patterns**: `.instructions.md` files with `applyTo` are included when working with matching files
- **Always-on**: `copilot-instructions.md`, `AGENTS.md`, and `CLAUDE.md` are always included

The `description` field helps users find the right customization when browsing the `/` menu, and for Agent Skills, it also controls automatic loading.

### File References in Prompts

Prompt files and instructions can reference workspace files using Markdown links with relative paths:

```markdown
Follow the coding standards in [coding-standards.md](../../coding-standards.md)
Use the API types defined in [api.ts](../../src/types/api.ts)
```

To reference agent tools in the body text, use the `#tool:<tool-name>` syntax. For example, `#tool:githubRepo`.

Prompt files also support variables:
- `${workspaceFolder}` -- workspace root path
- `${selection}` / `${selectedText}` -- currently selected text
- `${file}` / `${fileBasename}` -- current file path/name
- `${input:variableName}` -- user input passed from the chat input field

---

## Creating a Custom Instruction

### Capture Intent

Start by understanding the user's intent. The current conversation might already contain a workflow the user wants to capture (e.g., they say "turn this into a prompt"). If so, extract answers from the conversation history first -- the tools used, the sequence of steps, corrections the user made, input/output formats observed. The user may need to fill the gaps, and should confirm before proceeding to the next step.

1. What should this customization enable Copilot to do?
2. When should it be used? (what user tasks/contexts)
3. What's the expected output format?
4. What type of customization is best?
   - **Prompt file** (`.prompt.md`): For repeatable tasks invoked with `/promptName`
   - **Instructions file** (`.instructions.md`): For conventions/rules auto-applied to specific file types
   - **Agent Skill** (`SKILL.md`): For complex capabilities with scripts and resources
   - **Repository-level instructions** (`copilot-instructions.md`): For project-wide conventions
5. What agent is best for prompt files? (`agent` for multi-step tool use, `ask` for Q&A, `plan` for planning)
6. Should we set up test cases to verify the customization works? Customizations with objectively verifiable outputs (file transforms, data extraction, code generation, fixed workflow steps) benefit from test cases. Customizations with subjective outputs (writing style, art) often don't need them. Suggest the appropriate default based on the type, but let the user decide.

### Interview and Research

Proactively ask questions about edge cases, input/output formats, example files, success criteria, and dependencies. Wait to write test prompts until you've got this part ironed out.

Check available context -- if the user's workspace has existing conventions, coding patterns, or documentation, reference them. Come prepared with context to reduce burden on the user.

### Write the File

Based on the user interview, create the appropriate file:

**For prompt files (`.prompt.md`):**
- **description**: What the prompt does, shown when browsing with `/`
- **agent**: `agent`, `ask`, `plan`, or a custom agent name
- **tools**: What tools the prompt needs
- **the instructions**: The body of the prompt
- **Save to**: `.github/prompts/<prompt-name>.prompt.md`

**For instructions files (`.instructions.md`):**
- **name**: Display name
- **description**: Short description
- **applyTo**: Glob pattern for auto-attachment
- **the rules**: The body of the instructions
- **Save to**: `.github/instructions/<name>.instructions.md`

**For Agent Skills (SKILL.md):**
- **name**: Kebab-case identifier matching directory name
- **description**: What it does and when to use it
- **the instructions**: The body with guidelines, examples, scripts
- **Save to**: `.github/skills/<skill-name>/SKILL.md` (with optional supporting files)

### Prompt Writing Guide

#### Anatomy of a Customization

For simple prompt files:
```
.github/
+-- prompts/
    +-- my-prompt.prompt.md
```

For prompts with supporting resources:
```
.github/
+-- prompts/
|   +-- my-prompt.prompt.md (references files via Markdown links)
+-- instructions/
|   +-- python-style.instructions.md
+-- copilot-instructions.md (optional repo-level instructions)
```

For Agent Skills with resources:
```
.github/
+-- skills/
    +-- webapp-testing/
        +-- SKILL.md
        +-- test-template.js
        +-- examples/
            +-- login-test.js
```

#### Progressive Disclosure

Customizations use a layered loading model:

**For prompt files:**
1. **Description** -- shown in the `/` menu when browsing prompts
2. **Full prompt content** -- loaded when the user invokes `/promptName`
3. **Referenced files** -- loaded via Markdown links when the prompt is active

**For Agent Skills:**
1. **Name + description** -- always known (lightweight metadata for discovery)
2. **SKILL.md body** -- loaded when task matches description or user invokes `/skillName`
3. **Resources** -- accessed on-demand (scripts, examples, docs in the skill directory)

**Key patterns:**
- Keep prompt files focused and concise (under ~500 lines ideally)
- Use Markdown links for large reference materials
- Split complex workflows into multiple prompt files that users can compose
- For skills with scripts/resources, use the Agent Skills format

**Domain organization**: When a customization supports multiple domains/frameworks, organize by variant:
```
.github/
+-- prompts/
|   +-- deploy-aws.prompt.md
|   +-- deploy-gcp.prompt.md
|   +-- deploy-azure.prompt.md
```
Each prompt references shared documentation via Markdown links like `[common deployment steps](../../docs/deploy-common.md)`.

#### Principle of Lack of Surprise

This goes without saying, but customizations must not contain malware, exploit code, or any content that could compromise system security. A customization's contents should not surprise the user in their intent if described. Don't go along with requests to create misleading prompts. Things like "roleplay as an XYZ" are OK though.

#### Writing Patterns

Prefer using the imperative form in instructions.

**Defining output formats** -- You can do it like this:
```markdown
## Report structure
ALWAYS use this exact template:
# [Title]
## Executive summary
## Key findings
## Recommendations
```

**Examples pattern** -- It's useful to include examples:
```markdown
## Commit message format
**Example 1:**
Input: Added user authentication with JWT tokens
Output: feat(auth): implement JWT-based authentication
```

**Tool usage** -- For agent-mode prompts that need specific tools:
```markdown
## Workflow
1. Search the codebase for related tests using #tool:codebase
2. Run the existing test suite in #tool:terminal
3. Generate new test cases based on the patterns found
```

**Referencing tools in prompt body**: Use the `#tool:<tool-name>` syntax to reference specific tools.

### Writing Style

Try to explain to the model why things are important in lieu of heavy-handed musty MUSTs. Use theory of mind and try to make the prompt general and not super-narrow to specific examples. Start by writing a draft and then look at it with fresh eyes and improve it.

### Test Cases

After writing the prompt draft, come up with 2-3 realistic test scenarios -- the kind of thing a real user would actually type after `/promptName`. Share them with the user: "Here are a few test cases I'd like to try. Do these look right, or do you want to add more?" Then the user can run them manually in Copilot Chat.

Save test cases to `evals/evals.json`. Don't write assertions yet -- just the prompts. You'll draft assertions in the next step while the user tests.

```json
{
  "prompt_name": "example-prompt",
  "evals": [
    {
      "id": 1,
      "prompt": "User's task prompt (what they'd type after /promptName)",
      "expected_output": "Description of expected result",
      "files": []
    }
  ]
}
```

See `references/schemas.md` for the full schema (including the `assertions` field, which you'll add later).

## Running and Evaluating Test Cases

Since GitHub Copilot doesn't support programmatic testing or subagents, testing is done manually through Copilot Chat in VS Code.

### Step 1: Prepare tests for the user

For each test case, give the user clear instructions:

1. Open Copilot Chat in VS Code (Ctrl+Shift+I / Cmd+Shift+I)
2. Type `/promptName` followed by the test prompt (or just describe the task if testing instructions/skills)
3. Wait for the result
4. Save or copy the output somewhere you can review it

Suggest the user save outputs to a workspace directory structure like:
```
<prompt-name>-workspace/
+-- iteration-1/
    +-- eval-0/
    |   +-- with_prompt/outputs/    (results using the prompt)
    |   +-- without_prompt/outputs/ (results without the prompt, for comparison)
    +-- eval-1/
        +-- with_prompt/outputs/
        +-- without_prompt/outputs/
```

For baseline comparison, ask the user to also try the same task without using `/promptName`, to see how much the customization helps.

Write an `eval_metadata.json` for each test case:
```json
{
  "eval_id": 0,
  "eval_name": "descriptive-name-here",
  "prompt": "The user's task prompt",
  "assertions": []
}
```

### Step 2: Draft assertions while user tests

Don't just wait for the user to finish testing -- use this time productively. Draft quantitative assertions for each test case and explain them to the user.

Good assertions are objectively verifiable and have descriptive names -- they should read clearly in the benchmark viewer so someone glancing at the results immediately understands what each one checks. Subjective prompts (writing style, design quality) are better evaluated qualitatively -- don't force assertions onto things that need human judgment.

Update the `eval_metadata.json` files and `evals/evals.json` with the assertions once drafted.

### Step 3: Review the results

Once the user has completed the test runs and saved outputs:

1. **Grade each run** -- read `agents/grader.md` and evaluate each assertion against the outputs. Save results to `grading.json` in each run directory. The `grading.json` expectations array must use the fields `text`, `passed`, and `evidence` (not `name`/`met`/`details` or other variants) -- the viewer depends on these exact field names. For assertions that can be checked programmatically, write and run a script rather than eyeballing it.

2. **Aggregate into benchmark** -- run the aggregation script from the prompt-creator directory:
   ```bash
   python -m scripts.aggregate_benchmark <workspace>/iteration-N --skill-name <name>
   ```
   This produces `benchmark.json` and `benchmark.md` with pass_rate, time, and tokens for each configuration, with mean +/- stddev and the delta. If generating benchmark.json manually, see `references/schemas.md` for the exact schema the viewer expects.

3. **Do an analyst pass** -- read the benchmark data and surface patterns the aggregate stats might hide. See `agents/analyzer.md` (the "Analyzing Benchmark Results" section) for what to look for -- things like assertions that always pass regardless of prompt (non-discriminating), high-variance evals (possibly flaky), and time/token tradeoffs.

4. **Launch the viewer** with both qualitative outputs and quantitative data:
   ```bash
   python eval-viewer/generate_review.py \
     <workspace>/iteration-N \
     --skill-name "my-prompt" \
     --benchmark <workspace>/iteration-N/benchmark.json \
     --static <output_path>.html
   ```
   For iteration 2+, also pass `--previous-workspace <workspace>/iteration-<N-1>`.

   Since we're in VS Code, use `--static` to generate a standalone HTML file that the user can open in their browser.

Note: use `generate_review.py` to create the viewer; there's no need to write custom HTML.

5. **Tell the user** something like: "I've generated the review page. Open it in your browser to see the results. There are two tabs -- 'Outputs' lets you click through each test case and leave feedback, 'Benchmark' shows the quantitative comparison. When you're done, come back here and let me know."

### What the user sees in the viewer

The "Outputs" tab shows one test case at a time:
- **Prompt**: the task that was given
- **Output**: the files the prompt produced, rendered inline where possible
- **Previous Output** (iteration 2+): collapsed section showing last iteration's output
- **Formal Grades** (if grading was run): collapsed section showing assertion pass/fail
- **Feedback**: a textbox that auto-saves as they type
- **Previous Feedback** (iteration 2+): their comments from last time, shown below the textbox

The "Benchmark" tab shows the stats summary: pass rates, timing, and token usage for each configuration, with per-eval breakdowns and analyst observations.

Navigation is via prev/next buttons or arrow keys. When done, they click "Submit All Reviews" which downloads a `feedback.json` file.

### Step 4: Read the feedback

When the user tells you they're done, read `feedback.json`:

```json
{
  "reviews": [
    {"run_id": "eval-0-with_prompt", "feedback": "the chart is missing axis labels", "timestamp": "..."},
    {"run_id": "eval-1-with_prompt", "feedback": "", "timestamp": "..."},
    {"run_id": "eval-2-with_prompt", "feedback": "perfect, love this", "timestamp": "..."}
  ],
  "status": "complete"
}
```

Empty feedback means the user thought it was fine. Focus your improvements on the test cases where the user had specific complaints.

---

## Improving the Customization

This is the heart of the loop. You've run the test cases, the user has reviewed the results, and now you need to make the customization better based on their feedback.

### How to think about improvements

1. **Generalize from the feedback.** The big picture: we're creating customizations that can be used many, many times across different contexts. The user is iterating on only a few examples because it's faster for them to assess. But if the customization only works for those examples, it's useless. Rather than fiddly overfitty changes or oppressively constrictive MUSTs, if there's a stubborn issue, try branching out with different metaphors or recommending different patterns.

2. **Keep it lean.** Remove things that aren't pulling their weight. If the customization is making Copilot waste time doing unproductive things, try getting rid of those parts and see what happens.

3. **Explain the why.** Try hard to explain the **why** behind everything you're asking the model to do. Today's LLMs are *smart*. They have good theory of mind and when given a good harness can go beyond rote instructions. Even if the user's feedback is terse, try to actually understand the task and why the user wrote what they wrote, then transmit this understanding into the instructions. If you find yourself writing ALWAYS or NEVER in all caps, that's a yellow flag -- reframe and explain the reasoning so the model understands why it's important.

4. **Look for repeated work across test cases.** Read the outputs from the test runs and notice if Copilot independently wrote similar helper scripts or took the same multi-step approach. If all 3 test cases resulted in writing a similar helper script, consider converting to an Agent Skill so you can bundle the script alongside the instructions.

5. **Use file references for stability.** If your customization needs to reference specific schemas, templates, or examples, use Markdown links to workspace files. For Agent Skills, you can include resources directly in the skill directory.

### The iteration loop

After improving:

1. Apply your improvements to the customization file
2. Ask the user to rerun all test cases into a new `iteration-<N+1>/` directory, including baseline runs
3. Generate the reviewer with `--previous-workspace` pointing at the previous iteration
4. Wait for the user to review and tell you they're done
5. Read the new feedback, improve again, repeat

Keep going until:
- The user says they're happy
- The feedback is all empty (everything looks good)
- You're not making meaningful progress

---

## Advanced: Blind Comparison

For situations where you want a more rigorous comparison between two versions of a customization, there's a blind comparison system. Read `agents/comparator.md` and `agents/analyzer.md` for the details. The basic idea is: give two outputs to an independent evaluator without telling it which prompt produced which, and let it judge quality. Then analyze why the winner won.

This is optional -- the user needs to run both versions and save outputs. The human review loop is usually sufficient.

---

## Description Optimization

The `description` field helps users find the right customization:
- In **prompt files** and **Agent Skills**: shown in the `/` slash command menu
- In **instructions files**: shown on hover and used for semantic matching to tasks

### Good Descriptions
- Clear and concise -- what does this customization help with?
- Include key verbs and nouns users might search for
- Examples:
  - `"Generate unit tests for React components with Testing Library"`
  - `"Convert SQL queries to TypeORM QueryBuilder syntax"`
  - `"Create API documentation from TypeScript interfaces"`

### applyTo Patterns (Instructions Files Only)

Use `applyTo` on `.instructions.md` files when rules should automatically apply to certain file types:
- `"**/*.test.ts"` -- auto-attach for test files
- `"**/components/**"` -- auto-attach for component files
- `"**/migrations/**"` -- auto-attach for database migrations
- `"**/*.py,**/*.pyx"` -- multiple patterns separated by commas

Be judicious with `applyTo` -- too broad a pattern means the instructions are always in context, which can be noisy. Use it for rules that are genuinely always relevant for certain file types.

---

## Composition Patterns

GitHub Copilot allows users to combine multiple customizations. Design them to compose well:

### Layered Approach
```
.github/
+-- copilot-instructions.md      # Always-on project conventions
+-- instructions/
|   +-- python-style.instructions.md   # Auto-applied to *.py files
|   +-- react-rules.instructions.md    # Auto-applied to components
+-- prompts/
|   +-- create-component.prompt.md     # Invoked with /create-component
|   +-- run-tests.prompt.md            # Invoked with /run-tests
+-- skills/
    +-- debug-perf/
        +-- SKILL.md                   # Auto-loaded when relevant, or /debug-perf
        +-- flamegraph.sh
```

A user might type: `/create-component UserProfile with avatar upload`

The instructions files auto-apply based on file context, while the prompt provides the specific task workflow.

### Prompt + File Context
Users can combine slash commands with file references and variables:
```
/my-prompt Can you refactor the selected code? ${selection}
```

Design prompts that work well with additional context.

---

## Repository-Level Instructions

If the user wants project-wide conventions instead of (or in addition to) a reusable prompt, guide them to create `.github/copilot-instructions.md`:

```markdown
# Project Conventions

## Code Style
- Use TypeScript strict mode
- Prefer functional components with hooks
- Use named exports, not default exports

## Architecture
- Follow the repository pattern for data access
- All API routes go in src/routes/
- Business logic goes in src/services/

## Testing
- Write tests for all new features
- Use vitest for unit tests
- Use Playwright for E2E tests
```

This file is automatically included in all Copilot interactions for the repository. Keep it focused on conventions and patterns, not step-by-step workflows (those belong in prompt files or Agent Skills).

**Alternative always-on files:**
- `AGENTS.md` -- recognized by multiple AI agents (GitHub Copilot, and others). Useful if you work with multiple AI tools. Supports subfolder-level instructions (experimental).
- `CLAUDE.md` -- for compatibility with Claude Code and other Claude-based tools.

---

## Reference files

The `agents/` directory contains instructions for specialized evaluation tasks. Read them when you need to perform the relevant evaluation.

- `agents/grader.md` -- How to evaluate assertions against outputs
- `agents/comparator.md` -- How to do blind A/B comparison between two outputs
- `agents/analyzer.md` -- How to analyze why one version beat another

The `references/` directory has additional documentation:
- `references/schemas.md` -- JSON structures for evals.json, grading.json, etc.

---

## Choosing the Right Customization Type

| Need | Best Type | Location |
|------|-----------|----------|
| Project-wide coding standards | Always-on instructions | `.github/copilot-instructions.md` |
| Rules for specific file types | File-based instructions | `.github/instructions/*.instructions.md` |
| Repeatable task (scaffolding, testing) | Prompt file | `.github/prompts/*.prompt.md` |
| Complex workflow with scripts | Agent Skill | `.github/skills/<name>/SKILL.md` |
| Cross-agent conventions | AGENTS.md | `AGENTS.md` (root or subfolders) |
| AI persona with tool restrictions | Custom agent | Agent definition file |

### Key Differences from Other Systems

| Aspect | GitHub Copilot | Others |
|--------|---------------|--------|
| Prompt format | `.prompt.md` in `.github/prompts/` | Various |
| Instructions format | `.instructions.md` in `.github/instructions/` | N/A |
| Skills format | `SKILL.md` in `.github/skills/` (open standard) | Proprietary |
| Triggering | `/promptName`, `applyTo` globs, auto-matching | Auto-detection from description |
| Testing | Manual in VS Code Copilot Chat | CLI-based programmatic testing |
| Packaging | Commit to repo (no packaging needed) | `.skill` zip files |
| File references | Markdown links with relative paths | Various (`${file:path}`, etc.) |
| Agents | `agent`, `ask`, `plan`, custom agents | Single mode |
| Portability | Agent Skills work across Copilot tools | Tool-specific |

---

Repeating the core loop here for emphasis:

- Figure out what the customization is about
- Draft or edit the appropriate file (`.prompt.md`, `.instructions.md`, or `SKILL.md`)
- Have the user test with `/promptName` in Copilot Chat (or by working with matching files for instructions)
- With the user, evaluate the outputs:
  - Create benchmark.json and run `eval-viewer/generate_review.py` to help the user review them
  - Run quantitative evals
- Repeat until you and the user are satisfied
- Commit the final files to the repository

Good luck!

---
> Source: [sarjangi/skill-creator](https://github.com/sarjangi/skill-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
