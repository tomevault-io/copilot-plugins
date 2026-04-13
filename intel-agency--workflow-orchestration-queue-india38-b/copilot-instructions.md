## workflow-orchestration-queue-india38-b

> Project instructions for coding agents


<instructions>
  <purpose>
    <summary>
      GitHub Actions-based AI orchestration system. On GitHub events (issues, PR comments, reviews),
      the `orchestrator-agent` workflow assembles a structured prompt, spins up a devcontainer,
      and runs `opencode --agent Orchestrator` to delegate work to specialist sub-agents in `.opencode/agents/`.
    </summary>
  </purpose>

  <template_usage>
    <summary>
      This repository is a **GitHub template repo** (`intel-agency/workflow-orchestration-queue-india38-b`).
      New project repositories are created from it using automation scripts in the
      `nam20485/workflow-launch2` repo. The scripts clone this template, seed plan docs,
      replace template placeholders, and push — producing a ready-to-go AI-orchestrated repo.
    </summary>

    <template-clone-instances>
      Once the template has been cloned into a new instance, this file must be updated to match the new repo's specifics (e.g., name, links, instructions). 
    </template-clone-instances>

    <creation_workflow>
      <step>1. Run `./scripts/create-repo-from-slug.ps1 -Slug &lt;project-slug&gt; -Yes` from the `workflow-launch2` repo.</step>
      <step>2. That delegates to `./scripts/create-repo-with-plan-docs.ps1` which:
        - Creates a new GitHub repo from this template via `gh repo create --template intel-agency/workflow-orchestration-queue-india38-b`
        - Generates a random suffix for the repo name (e.g., `project-slug-bravo84`)
        - Creates repo secrets (`GEMINI_API_KEY`) and variables (`VERSION_PREFIX`)
        - Clones the new repo locally
        - Copies plan docs from `./plan_docs/&lt;slug&gt;/` into the clone's `plan_docs/` directory
        - Replaces all template placeholders (`workflow-orchestration-queue-india38-b` → new repo name, `intel-agency` → new owner)
        - Commits and pushes the seeded repo
      </step>
      <step>3. On push, the clone's `validate` workflow runs CI (lint, scan, tests, devcontainer build) and the `publish-docker` workflow builds and pushes the base Docker image to GHCR.</step>
      <step>4. On successful `publish-docker` completion, the `prebuild-devcontainer` workflow is triggered (via `workflow_run`) to build and push the prebuilt devcontainer image. Together, `publish-docker` → `prebuild-devcontainer` form the devcontainer prebuild caching pipeline that the `orchestrator-agent` workflow relies on to quickly spin up devcontainers.</step>
    </creation_workflow>

    <template_design_constraints>
      <rule>Template placeholders (`workflow-orchestration-queue-india38-b`, `intel-agency`) in file contents and paths are replaced by the creation script. Keep them consistent.</rule>
      <rule>The `validate` workflow must tolerate fresh clones where no prebuilt GHCR devcontainer image exists yet (fallback build from Dockerfile + image aliasing).</rule>
      <rule>The `plan_docs/` directory contains external-generated documents seeded at clone time. Exclude it from strict linting (markdown lint, etc.).</rule>
      <rule>The consumer `.devcontainer/devcontainer.json` references a prebuilt GHCR image. On fresh clones the image won't exist until `publish-docker` and `prebuild-devcontainer` workflows complete their first run.</rule>
    </template_design_constraints>

    <automation_scripts>
      <entry><repo>nam20485/workflow-launch2</repo><path>scripts/create-repo-from-slug.ps1</path><description>Entry point — takes a slug, resolves plan docs dir, delegates to create-repo-with-plan-docs.ps1</description></entry>
      <entry><repo>nam20485/workflow-launch2</repo><path>scripts/create-repo-with-plan-docs.ps1</path><description>Full pipeline: repo create, clone, seed docs, placeholder replace, commit, push</description></entry>
    </automation_scripts>
  </template_usage>

  <tech_stack>
    <item>opencode CLI — agent runtime (`opencode --model zai-coding-plan/glm-5 --agent Orchestrator`)</item>
    <item>ZhipuAI GLM models via `ZHIPU_API_KEY`</item>
    <item>GitHub Actions + devcontainers/ci — workflow trigger, runner, reproducible container</item>
    <item>.NET SDK 10 + Aspire + Avalonia templates, Bun, uv (all in devcontainer)</item>
    <item>MCP servers: `@modelcontextprotocol/server-sequential-thinking`, `@modelcontextprotocol/server-memory`</item>
  </tech_stack>

  <repository_map>
    <!-- Workflows -->
    <entry><path>.github/workflows/orchestrator-agent.yml</path><description>Primary workflow — assembles prompt, logs into GHCR, runs opencode in devcontainer</description></entry>
    <entry><path>.github/workflows/prompts/orchestrator-agent-prompt.md</path><description>Prompt template with `__EVENT_DATA__` placeholder (sed-substituted at runtime)</description></entry>
    <entry><path>.github/workflows/publish-docker.yml</path><description>Builds Dockerfile, pushes to GHCR with branch-latest and branch-&lt;VERSION_PREFIX.run_number&gt; tags</description></entry>
    <entry><path>.github/workflows/prebuild-devcontainer.yml</path><description>Layers devcontainer Features on published Docker image (triggered by workflow_run)</description></entry>
    <!-- Agent definitions -->
    <entry><path>.opencode/agents/orchestrator.md</path><description>Orchestrator — coordinates specialists, never writes code directly</description></entry>
    <entry><path>.opencode/agents/</path><description>All specialist agents (developer, code-reviewer, planner, devops-engineer, github-expert, etc.)</description></entry>
    <entry><path>.opencode/commands/</path><description>Reusable command prompts (orchestrate-new-project, grind-pr-reviews, fix-failing-workflows, etc.)</description></entry>
    <entry><path>.opencode/opencode.json</path><description>opencode config — MCP server definitions</description></entry>
    <!-- Devcontainer -->
    <entry><path>.github/.devcontainer/Dockerfile</path><description>Devcontainer image — .NET SDK, Bun, uv, opencode CLI (build context for publish-docker)</description></entry>
    <entry><path>.github/.devcontainer/devcontainer.json</path><description>Build-time devcontainer config (Dockerfile + Features: node, python, gh CLI)</description></entry>
    <entry><path>.devcontainer/devcontainer.json</path><description>Consumer devcontainer — pulls prebuilt GHCR image, forwards port 4096, and auto-starts `opencode serve` on container start</description></entry>
    <entry><path>scripts/start-opencode-server.sh</path><description>Guarded `opencode serve` bootstrapper used by the devcontainer lifecycle and workflow attach path</description></entry>
    <entry><path>scripts/run-devcontainer-orchestrator.sh</path><description>One-shot script: brings up the devcontainer, ensures the opencode server is running, and executes the orchestrator agent. Used by the workflow and can be invoked directly locally.</description></entry>
    <!-- Tests -->
    <entry><path>test/</path><description>Shell-based tests: devcontainer build, tool availability, prompt assembly</description></entry>

    <opencode_server>
      <summary>
        The consumer devcontainer auto-starts `opencode serve` through `scripts/start-opencode-server.sh`.
        The server listens on port `4096` by default so host or in-container clients can attach with
        `opencode run --attach http://127.0.0.1:4096 ...` (or the forwarded host port when connecting from outside the container).
      </summary>
    </opencode_server>
    <entry><path>test/fixtures/</path><description>Sample webhook payloads for local testing</description></entry>
    <!-- Remote instructions -->
    <entry><path>local_ai_instruction_modules/</path><description>Local instruction modules (development rules, workflows, delegation, terminal commands)</description></entry>
  </repository_map>

  <instruction_source>
    <repository>
      <name>nam20485/agent-instructions</name>
      <branch>main</branch>
    </repository>
    <guidance>
      Remote instructions are the single source of truth. Fetch from raw URLs:
      replace `github.com/` with `raw.githubusercontent.com/` and remove `blob/`.
      Core instructions: `https://raw.githubusercontent.com/nam20485/agent-instructions/main/ai_instruction_modules/ai-core-instructions.md`
    </guidance>
    <modules>
      <module type="core" required="true" link="https://github.com/nam20485/agent-instructions/blob/main/ai_instruction_modules/ai-core-instructions.md">Core Instructions</module>
      <module type="local" required="true" path="local_ai_instruction_modules">Local AI Instructions</module>
      <module type="local" required="true" path="local_ai_instruction_modules/ai-dynamic-workflows.md">Dynamic Workflow Orchestration</module>
      <module type="local" required="true" path="local_ai_instruction_modules/ai-workflow-assignments.md">Workflow Assignments</module>
      <module type="local" required="true" path="local_ai_instruction_modules/ai-development-instructions.md">Development Instructions</module>
      <module type="optional" path="local_ai_instruction_modules/ai-terminal-commands.md">Terminal Commands</module>
    </modules>
  </instruction_source>

  <environment_setup>
    <secrets>
      <item>`ZHIPU_API_KEY` — ZhipuAI model access; set in repo Settings → Secrets.</item>
      <item>`KIMI_CODE_ORCHESTRATOR_AGENT_API_KEY` — Kimi (Moonshot) model access; set in repo Settings → Secrets.</item>
      <item>`GITHUB_TOKEN` — provided automatically by Actions.</item>
    </secrets>
    <devcontainer_cache>
      Image at `ghcr.io/${{ github.repository }}/devcontainer`. `publish-docker.yml` builds the raw Dockerfile;
      `prebuild-devcontainer.yml` layers Features. Login via `docker/login-action` with `GITHUB_TOKEN`.
      Set repo variable `VERSION_PREFIX` (e.g., `1.0`) for versioned tags emitted by both image publishing workflows.
    </devcontainer_cache>
  </environment_setup>

  <testing>
    <guidance>Tests are shell scripts in `test/`. Run directly with `bash`.</guidance>
    <commands>
      <command>All tests: `bash test/test-devcontainer-build.sh && bash test/test-devcontainer-tools.sh && bash test/test-prompt-assembly.sh`</command>
      <command>Prompt changes: `bash test/test-prompt-assembly.sh`</command>
      <command>Dockerfile changes: `bash test/test-devcontainer-tools.sh`</command>
    </commands>
    <guidance>Add new fixture payloads to `test/fixtures/` when testing new event types.</guidance>
  </testing>

  <coding_conventions>
    <rule>Keep changes minimal and targeted.</rule>
    <rule>Do not hardcode secrets/tokens.</rule>
    <rule>Preserve the `__EVENT_DATA__` placeholder in `orchestrator-agent-prompt.md`.</rule>
    <rule>Pin action versions by SHA in workflow files.</rule>
    <rule>Never add duplicate top-level `name:`, `on:`, or `jobs:` keys in workflow YAML.</rule>
    <rule>`.opencode/` is checked out by `actions/checkout`; do not COPY it in the Dockerfile.</rule>
    <rule>Dockerfile lives at `.github/.devcontainer/Dockerfile`. Consumer devcontainer uses `"image:"` — no local build.</rule>
    <rule>Repository labels are defined in `.github/.labels.json`. Use `scripts/import-labels.ps1` to sync them to a repo instance. When adding new labels, add them to this file — it is the single source of truth for the label set.</rule>
  </coding_conventions>

  <agent_specific_guardrails>
    <rule>The Orchestrator agent delegates to specialists via the `task` tool — never writes code directly.</rule>
    <rule>Prompt assembly pipeline:
      1. Read template from `.github/workflows/prompts/orchestrator-agent-prompt.md`.
      2. Prepend structured event context (event name, action, actor, repo, ref, SHA).
      3. Append raw event JSON from `${{ toJson(github.event) }}`.
      4. Write to `.assembled-orchestrator-prompt.md` and export path via `GITHUB_ENV`.
    </rule>
  </agent_specific_guardrails>

  <agent_readiness>
    <verification_protocol>
      For any non-trivial change (logic, behavior, refactors, dependency updates, config changes, multi-file edits):
      run verification, fix all failures, re-run until clean. Do not skip or suppress errors.
    </verification_protocol>

    <verification_commands>
      <!--
        MANDATORY: After every non-trivial change, run validation BEFORE commit/push.
        Do NOT commit or push until it passes. Do NOT skip steps.

        Local (runs all checks sequentially — lint, scan, test):
          pwsh -NoProfile -File ./scripts/validate.ps1 -All

        This is the SAME script that CI calls with individual switches:
          ./scripts/validate.ps1 -Lint   (CI: lint job)
          ./scripts/validate.ps1 -Scan   (CI: scan job)
          ./scripts/validate.ps1 -Test   (CI: test job)

        If a check is skipped due to a missing local tool, run:
          pwsh -NoProfile -File ./scripts/install-dev-tools.ps1

        | Check                  | Command                                              | When to run              |
        |========================|======================================================|==========================|
        | All (local default)    | ./scripts/validate.ps1 -All                           | Every task               |
        | Lint only              | ./scripts/validate.ps1 -Lint                           | Quick check              |
        | Scan only              | ./scripts/validate.ps1 -Scan                           | Secrets concern          |
        | Test only              | ./scripts/validate.ps1 -Test                           | After lint passes        |
        | Devcontainer tests     | bash test/test-devcontainer-tools.sh                   | Dockerfile changes       |
      -->
      <rule>When adding a CI workflow check, add its equivalent to scripts/validate.ps1.</rule>
    </verification_commands>

    <post_commit_monitoring>
      After push, monitor CI until green: `gh run list --limit 5`, `gh run watch <id>`, `gh run view <id> --log-failed`.
      If any workflow fails, stop feature work, triage, fix, re-verify, push. Do not mark work complete while CI is failing.
    </post_commit_monitoring>

    <pipeline_speed_policy>
      <lane name="fast_readiness" blocking="true">Build, lint/format, unit tests — keep fast for merge readiness.</lane>
      <lane name="extended_validation" blocking="false">Integration suites, security scans, dependency audits.</lane>
      <rule>Protect the fast lane from slow steps.</rule>
    </pipeline_speed_policy>
  </agent_readiness>

  <validation_before_handoff>
    <step>Run applicable shell tests and verification commands.</step>
    <step>Validate workflow YAML: `grep -c "^name:" .github/workflows/orchestrator-agent.yml  # expect 1`</step>
    <step>Summarize: what changed, what was validated, remaining risks (secret-dependent paths, image cache misses).</step>
  </validation_before_handoff>

  <tool_use_instructions>
    <instruction id="querying_microsoft_documentation">
      <applyTo>**</applyTo>
      <title>Querying Microsoft Documentation</title>
      <tools><tool>microsoft_docs_search</tool><tool>microsoft_docs_fetch</tool><tool>microsoft_code_sample_search</tool></tools>
      <guidance>
        Use these MCP tools for Microsoft technologies (C#, ASP.NET Core, .NET, EF, NuGet).
        Prioritize retrieved info over training data for newer features.
      </guidance>
    </instruction>
    <instruction id="sequential_thinking_default_usage">
      <applyTo>*</applyTo>
      <title>Sequential Thinking</title>
      <tools><tool>sequential_thinking</tool></tools>
      <guidance>
        Use for all non-trivial requests. Enables step-by-step analysis with revision, branching, and dynamic adjustment.
        Use when: breaking down complex problems, planning, architectural decisions, debugging, multi-step context.
      </guidance>
    </instruction>
    <instruction id="memory_default_usage">
      <applyTo>*</applyTo>
      <title>Knowledge Graph Memory</title>
      <tools><tool>create_entities</tool><tool>create_relations</tool><tool>add_observations</tool><tool>delete_entities</tool><tool>delete_observations</tool><tool>delete_relations</tool><tool>read_graph</tool><tool>search_nodes</tool><tool>open_nodes</tool></tools>
      <guidance>
        Use for non-trivial requests. Persist user/project context (preferences, configs, decisions, challenges, solutions).
        Entities have names, types, and observations. Relations connect entities. Search/read at task start; update after significant work.
      </guidance>
    </instruction>
  </tool_use_instructions>

  <available_tools>
    <summary>
      Tools available inside the devcontainer at runtime. Installed via
      `.github/.devcontainer/Dockerfile` unless noted otherwise.
    </summary>

    <runtimes_and_package_managers>
      <tool name="dotnet" version="10.0.102">`.NET SDK` — build, test, publish C#/F# projects. Includes Avalonia Templates 11.3.12.</tool>
      <tool name="node" version="24.14.0 LTS">`Node.js` — JavaScript runtime. Required for MCP server packages (`npx`).</tool>
      <tool name="npm">`npm` — Node package manager (bundled with Node.js).</tool>
      <tool name="bun" version="1.3.10">`Bun` — fast JavaScript/TypeScript runtime, bundler, and package manager.</tool>
      <tool name="uv" version="0.10.9">`uv` — Astral Python package manager. Also provides `uvx` for ephemeral tool runs.</tool>
    </runtimes_and_package_managers>

    <cli_tools>
      <tool name="gh">`GitHub CLI` — interact with GitHub API (issues, PRs, repos, releases, actions). Authenticated automatically via `GITHUB_TOKEN` env var in CI; use `gh auth login --with-token` otherwise.</tool>
      <tool name="opencode" version="1.2.24">`opencode CLI` — AI agent runtime. Runs agents defined in `.opencode/agents/` with MCP server support.</tool>
      <tool name="git">`Git` — version control (system package + devcontainer feature).</tool>
    </cli_tools>

    <github_authentication>
      <summary>
        GitHub API access is configured at multiple layers to support both `gh` CLI and MCP GitHub server operations.
      </summary>
      <layer name="GITHUB_TOKEN">Provided automatically by GitHub Actions. Passed into the devcontainer via `--remote-env`.</layer>
      <layer name="GITHUB_PERSONAL_ACCESS_TOKEN">Bridged from `GITHUB_TOKEN` for the `@modelcontextprotocol/server-github` MCP server, which requires this specific env var name. Set in `opencode.json` via the MCP `env` block, in `devcontainer.json` `remoteEnv`, and exported in `run_opencode_prompt.sh`.</layer>
      <layer name="gh auth login">`run_opencode_prompt.sh` authenticates the `gh` CLI via `echo "$GITHUB_TOKEN" | gh auth login --with-token` before launching opencode.</layer>
    </github_authentication>

    <scripts_directory>
      <summary>PowerShell helper scripts in `scripts/` for GitHub setup and management tasks.</summary>
      <script name="scripts/common-auth.ps1">Shared `Initialize-GitHubAuth` function — checks `gh auth status`, authenticates via PAT token (`$env:GITHUB_AUTH_TOKEN`) or interactive login.</script>
      <script name="scripts/gh-auth.ps1">Extended GitHub auth helper — supports PAT token auth via `--with-token` and interactive fallback.</script>
      <script name="scripts/import-labels.ps1">Imports labels from `.github/.labels.json` into the repository.</script>
      <script name="scripts/create-milestones.ps1">Creates project milestones from plan docs.</script>
      <script name="scripts/test-github-permissions.ps1">Verifies `GITHUB_TOKEN` has required permissions (contents, issues, PRs, packages).</script>
      <script name="scripts/query.ps1">PR review thread manager — fetches unresolved review threads from a PR, summarizes them, and can batch-reply and resolve them. Supports `--AutoResolve`, `--DryRun`, `--Interactive`, `--ReplyEach`, `--Path`, `--BodyContains` filtering. Use this instead of writing ad-hoc scripts to resolve PR review comments.</script>
      <script name="scripts/update-remote-indices.ps1">Updates remote instruction module indices.</script>
    </scripts_directory>
  </available_tools>
</instructions>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/intel-agency)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/intel-agency)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
