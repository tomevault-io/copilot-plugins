## jira-record

> USE WHEN: creating local Markdown (.md) records that mirror Jira Epics in the G&A Jira environment.


# Local Jira Records (Markdown) — Epic

This rule defines **how to create local `.md` files** that act as the canonical "offline" record for Jira Epic work.

**Authoritative Jira field IDs, constraints, and formatting rules live in:** `.cursor/rules/jira-mcp-integration.mdc`.

## Goals

- Create local records that are **easy to read**, **diff**, and **copy into Jira**.
- Ensure content is structured according to **Epic issue type** and the **custom field mappings** we use in Jira.

## Where to Store Local Jira Record Files

Pick a consistent folder in this repo, e.g.:

- `jira/` (recommended)
- `docs/jira/`

Do **not** scatter Jira records across unrelated folders.

## File Naming Conventions

Use one of these:

- If Jira key exists: `{KEY} - Epic - {short-title}.json` (example: `WOR-123 - Epic - Add SSO mapping.json`)
- If Jira key does not exist yet: `DRAFT - Epic - {short-title}.json`

Keep `{short-title}` short and filesystem-safe.

## Common Front Matter (Recommended)

Add YAML front matter to every local Jira record:

```yaml
---
jira_key: ""              # e.g. "WOR-123" once created
issue_type: "Epic"        # Epic
parent_key: ""            # For Subtask: Story key
status: ""                # Optional: human-readable
team_id: ""               # Jira Team field: customfield_10001 (UUID)
sprint_id: ""             # Jira Sprint field: customfield_10020 (integer)
assignee: ""              # Email or account id (depends on your Jira usage)
pm_owner: ""              # Jira PM Owner: customfield_10246 (email/account id)
story_points: ""          # Jira Story Points: customfield_10026
labels: []                # Optional
created_at: ""            # Optional
updated_at: ""            # Optional
---
```

> **Note:** Team and Sprint are often shown as "inherited" in Jira UI. Don't assume non-editable — `jira-mcp-integration.mdc` explains checking `editmeta`.

## Content Rules

- **Summary/title must be ≤ 255 characters** (Jira constraint).
- For **ADF fields** (custom fields that require ADF format), store only the **Markdown version** (human-readable, for editing and review).
- ADF JSON will be automatically generated from the Markdown content when syncing to Jira using the conversion scripts.
- **Do NOT include references to other local `.md` files** in descriptions sent to Jira. These local files do not exist in Jira and such references will be broken links. Only reference Jira issues (by key, e.g., `WOR-123`) or external URLs.

## Epic — Local Record Template

> Epic field mappings can vary by project configuration. Use the reference file `.cursor/rules/jira-mcp-integration.mdc` as the source of truth for field IDs and confirm Epic-only fields when needed.

**Child Work Items:**
- Child Features are linked to Epics using the `parent` field in Jira (not the Epic Link field).
- **Include a "Child Work Items" section** in the Epic local markdown file to record all Features that are children of this Epic.
- This section serves as a local record for tracking and should list all child Features by their Jira key and summary.
- The parent-child relationships are managed through Jira's hierarchy system (via the `parent` field), but the local record maintains a list for easy reference.
- When creating or updating Features that belong to an Epic, set the `parent` field to the Epic's key (e.g., `{"parent": {"key": "WOR-1903"}}`).

```markdown
---
jira_key: ""
issue_type: "Epic"
status: ""
team_id: ""
---

## Summary
[Epic title]

## Business Context
- Why this matters
- Who is impacted

## High-Level Scope
- In scope:
- Out of scope:

## Success Metrics
- Metric 1:
- Metric 2:

## Child Work Items
- [ ] WOR-XXX: [Feature summary]
- [ ] WOR-YYY: [Feature summary]

## Notes / References
- Related Jira tickets (by key, e.g., `WOR-123`)
- External links/URLs
- **Note:** Do not reference other local `.md` files here — these do not exist in Jira
```

## Copy/Paste Checklist (Before Creating/Updating Jira)

Use `.cursor/rules/jira-mcp-integration.mdc` to ensure:

- You're respecting the **character limits** (esp. summary).
- **You have removed any references to local `.md` files** from all fields before sending to Jira. Only Jira issue keys (e.g., `WOR-123`) and external URLs should be referenced.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jarekbird)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jarekbird)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
