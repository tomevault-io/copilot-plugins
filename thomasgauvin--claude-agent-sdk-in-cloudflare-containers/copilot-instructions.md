## claude-agent-sdk-in-cloudflare-containers

> This container runs the Claude Agent SDK with specialized agent capabilities defined in skills.

# Claude Agent SDK - Container Context

This container runs the Claude Agent SDK with specialized agent capabilities defined in skills.

## Available Skills

You should use your skills as often as possible. For every request, check if there is not a skill that can assist the user.

1. **Configure Git CLI** (`.claude/skills/configure-git-cli/`)
   - Sets up git user identity and GitHub authentication
   - Configures git to use GitHub PAT for authentication
   - Verifies git credentials work correctly
   - **Used by**: Other skills that require git operations

2. **Codebase Architecture Analysis** (`.claude/skills/codebase-architecture-analysis/`)
   - Analyzes GitHub repositories for architecture documentation
   - Generates ASCII diagrams showing system components
   - Assesses technology stack and infrastructure
   - **Prerequisites**: None (reads-only operations)

3. **Codebase Fix and Pull Request** (`.claude/skills/codebase-fix-and-pr/`)
   - Applies code fixes and improvements to repositories
   - Automatically creates pull requests with proper commits
   - Follows conventional commit standards
   - **Prerequisites**: `configure-git-cli` skill (for git operations)

## How Skills Are Discovered

With `settingSources: ['project']` configured in the Agent SDK options, skills are automatically discovered from the `.claude/skills/` directory when the agent starts.

## Environment Variables

Required for skills to function:

- `ANTHROPIC_API_KEY` - Anthropic API key for Claude
- `GITHUB_PAT` - GitHub Personal Access Token (for repository operations)
- `MODEL` - Claude model to use (optional, defaults to claude-haiku-4-5)

## Architecture

The agent runs in this container and has access to:

- **Built-in tools**: Read, Edit, Bash, Glob, Grep, etc.
- **MCP servers**: GitHub integration for repository operations
- **Skills**: Specialized capabilities for complex workflows
- **File system**: Access to clone and manipulate repositories

YOU MUST USE THE GIT CLI, BASH TO MAKE CHANGES IN REPOSITORIES RATHER THAN USING THE GITHUB MCP WHEN POSSIBLE.

## Configuration

Skills are enabled via the Agent SDK options:

```typescript
const options: Options = {
  model: process.env.MODEL || "claude-haiku-4-5",
  settingSources: ['local', 'project'],  // ← Enables skill discovery
  // ... other config
};
```

This tells the SDK to look for `.claude/` directory in the project root.

---
> Source: [thomasgauvin/claude-agent-sdk-in-cloudflare-containers](https://github.com/thomasgauvin/claude-agent-sdk-in-cloudflare-containers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
