## opencode-agents

> > **Purpose:** {{PROJECT_PURPOSE}}

# {{PROJECT_NAME}} — AI Context

> **Purpose:** {{PROJECT_PURPOSE}}
> **Work tracking:** [GitHub Project "{{PROJECT_NAME}}"]({{GITHUB_PROJECT_URL}})

---

## Quick Start

1. Check GitHub Project for current sprint and open issues
2. Work in the repo root for code changes
3. All work must go through GitHub Issues — no ad-hoc changes

---

## Session Maintenance Protocol

### Session Start Protocol

At the beginning of every session, before doing any work:

1. **Check project board state**
   ```bash
   gh project item-list {{GITHUB_PROJECT_NUMBER}} --owner {{REPO_OWNER}} --format json --limit 200
   gh issue list --repo {{REPO_OWNER}}/{{REPO_NAME}} --state open
   ```
2. **Identify interrupted work** — Items with Status = "In Progress" were interrupted from a previous session. Resume these first.
3. **Enrich any new items** — Check for items missing Agent, Effort, or Sprint fields. Set them.
   - Field IDs: Status (`{{FIELD_ID_STATUS}}`), Agent (`{{FIELD_ID_AGENT}}`), Effort (`{{FIELD_ID_EFFORT}}`), Sprint (`{{FIELD_ID_SPRINT}}`)
   - Status options: Todo=`{{STATUS_TODO}}`, In Progress=`{{STATUS_IN_PROGRESS}}`, Done=`{{STATUS_DONE}}`
   - Agent options: {{AGENT_OPTIONS}}
   - Effort options: S=`{{EFFORT_S}}`, M=`{{EFFORT_M}}`, L=`{{EFFORT_L}}`
   - Sprint options: Sprint 1=`{{SPRINT_1}}`
   - Project ID: `{{PROJECT_ID}}`

### Session End Protocol

Before ending any session:

1. **Ensure all in-progress issues have current status** — Comment on any issue where work was done this session
2. **Close completed issues** — Any issue where all acceptance criteria are met
3. **Set project board fields** — All items touched this session have correct Status, Agent, Effort, Sprint

### When Creating New Issues

Every new issue must be:
1. Created with appropriate labels (`type:`, `priority:`, `module:`)
2. Added to the project: `gh project item-add {{GITHUB_PROJECT_NUMBER}} --owner {{REPO_OWNER}} --url <issue-url>`
3. Fields set immediately: Status, Agent, Effort, Sprint (if applicable)

---

## Finding Your GitHub Project Field IDs

The Session Start Protocol above requires field IDs and option IDs for your specific GitHub Project. These are unique to every project and must be retrieved once during setup.

### Step 1 — Get your Project ID and field definitions

```bash
gh api graphql -f query='
{
  user(login: "{{REPO_OWNER}}") {
    projectV2(number: {{GITHUB_PROJECT_NUMBER}}) {
      id
      fields(first: 20) {
        nodes {
          ... on ProjectV2Field {
            id
            name
          }
          ... on ProjectV2SingleSelectField {
            id
            name
            options {
              id
              name
            }
          }
          ... on ProjectV2IterationField {
            id
            name
            configuration {
              iterations {
                id
                title
              }
            }
          }
        }
      }
    }
  }
}'
```

### Step 2 — Fill in the placeholders above

From the query output, copy the IDs into the Session Start Protocol section:

| Placeholder | Where to find it |
|-------------|-----------------|
| `{{PROJECT_ID}}` | `data.user.projectV2.id` |
| `{{FIELD_ID_STATUS}}` | The `id` of the field named `Status` |
| `{{FIELD_ID_AGENT}}` | The `id` of the field named `Agent` (or your equivalent) |
| `{{FIELD_ID_EFFORT}}` | The `id` of the field named `Effort` (or your equivalent) |
| `{{FIELD_ID_SPRINT}}` | The `id` of the field named `Sprint` (or your equivalent) |
| `{{STATUS_TODO}}` | The `options[].id` where `options[].name == "Todo"` |
| `{{STATUS_IN_PROGRESS}}` | The `options[].id` where `options[].name == "In Progress"` |
| `{{STATUS_DONE}}` | The `options[].id` where `options[].name == "Done"` |
| `{{AGENT_OPTIONS}}` | All `options` under the Agent field, formatted as `name=\`id\`` |
| `{{EFFORT_S}}` / `M` / `L` | The option IDs under your Effort field |
| `{{SPRINT_1}}` | The iteration `id` for your first sprint |

### Step 3 — Setting a field value (example)

```bash
# Example: set the Status field to "In Progress" on a project item
gh api graphql -f query='
mutation {
  updateProjectV2ItemFieldValue(
    input: {
      projectId: "{{PROJECT_ID}}"
      itemId: "<item-node-id>"
      fieldId: "{{FIELD_ID_STATUS}}"
      value: { singleSelectOptionId: "{{STATUS_IN_PROGRESS}}" }
    }
  ) {
    projectV2Item { id }
  }
}'
```

---

## Directory Structure

```
{{REPO_ROOT}}/
├── CLAUDE.md              <- This file
├── README.md              <- Project README
├── .opencode/
│   ├── agents/            <- Agent definitions
│   └── commands/          <- Custom slash commands (/plan, /release)
{{DIRECTORY_STRUCTURE}}
```

<!--
Example directory structure additions:

├── cmd/                   <- CLI entry point
├── internal/              <- Private application packages
│   ├── api/               <- REST API server
│   ├── core/              <- Core business logic
│   └── store/             <- Database layer
├── pkg/                   <- Public library packages
├── docs/                  <- Documentation site source
├── .github/workflows/     <- CI/CD pipelines
└── .goreleaser.yaml       <- Release configuration
-->

## GitHub Resources

| Resource | Visibility |
|----------|------------|
| `{{REPO_OWNER}}/{{REPO_NAME}}` | Public |
| GitHub Project #{{GITHUB_PROJECT_NUMBER}} "{{PROJECT_NAME}}" | Private |

## Current State

- **Version:** {{CURRENT_VERSION}}
- **See GitHub Project for active work**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rmkohlman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
