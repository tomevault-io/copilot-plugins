## github-classroom-reinvite

> **github-classroom-reinvite** is a utility repository for managing GitHub Classroom assignment invitations. It provides an automated solution for re-inviting students to their assignment repositories when GitHub Classroom's invitation bot encounters issues (see [GitHub Community Issue #72283](https://github.com/orgs/community/discussions/72283)).

# CLAUDE.md - AI Assistant Guide

## Repository Overview

**github-classroom-reinvite** is a utility repository for managing GitHub Classroom assignment invitations. It provides an automated solution for re-inviting students to their assignment repositories when GitHub Classroom's invitation bot encounters issues (see [GitHub Community Issue #72283](https://github.com/orgs/community/discussions/72283)).

### Purpose
Re-invite students to GitHub Classroom assignment repositories by:
1. Fetching all assignments from specified classroom(s)
2. Identifying pending invitations from `github-classroom[bot]`
3. Deleting expired/stuck invitations
4. Re-creating invitations with write permissions

---

## Codebase Structure

```
.
├── .github/
│   └── workflows/
│       └── reinvite-action.yml    # Automated GitHub Actions workflow
├── README.md                       # Basic project documentation
├── reinvite.sh                     # Main bash script (executable)
└── CLAUDE.md                       # This file
```

### Key Files

#### `reinvite.sh` (Main Script)
- **Location**: `/reinvite.sh`
- **Purpose**: Core logic for processing GitHub Classroom invitations
- **Dependencies**: GitHub CLI (`gh`)
- **Permissions**: Executable (`chmod +x`)

**Configuration Variables** (lines 3-7):
- `ORG_NAME`: GitHub organization name (currently: `UNC-DATA-730`)
- `CLASSROOM_IDS`: Array of classroom IDs to process (currently: `300544`)

**Key Functions**:
- `get_assignment_repos()` (lines 9-34): Fetches all repositories for a classroom
- `process_repo_invites()` (lines 36-64): Handles invitation deletion and re-creation

**API Endpoints Used**:
- `/classrooms/{id}/assignments` - List assignments
- `/assignments/{id}/accepted_assignments` - List accepted assignments
- `/repos/{owner}/{repo}/invitations` - Manage invitations
- `/repos/{owner}/{repo}/collaborators/{user}` - Add collaborators

#### `.github/workflows/reinvite-action.yml` (GitHub Actions)
- **Location**: `/.github/workflows/reinvite-action.yml`
- **Triggers**:
  - Schedule: Every hour at minute 0 (`cron: '0 * * * *'`)
  - Push to `main` branch
  - Manual dispatch (`workflow_dispatch`)

**Required Secrets**:
- `GH_TOKEN`: GitHub Personal Access Token with appropriate permissions
  - Required scopes: `repo`, `admin:org`, `classroom`

---

## Development Workflows

### Making Changes

1. **Branch Strategy**:
   - Default branch: `main`
   - Feature branches: Use `claude/` prefix (e.g., `claude/claude-md-mkpjw3du6kvf0n4t-qadTs`)
   - Always develop on feature branches, never directly on `main`

2. **Modifying Configuration**:
   ```bash
   # Edit reinvite.sh to update:
   # - ORG_NAME (line 3): Target organization
   # - CLASSROOM_IDS (lines 5-7): Add/remove classroom IDs
   ```

3. **Testing Locally**:
   ```bash
   # Ensure GitHub CLI is authenticated
   gh auth status

   # Run the script
   ./reinvite.sh
   ```

4. **Committing Changes**:
   - Use descriptive commit messages
   - Follow existing patterns: lowercase, imperative mood
   - Examples from history:
     - "remove old classroom ids"
     - "spring 2026 classroom id"
     - "use classroom ids instead"

5. **Pushing Changes**:
   ```bash
   # Always use -u flag for feature branches
   git push -u origin <branch-name>

   # Branch naming must: start with 'claude/' and end with session ID
   # Retry on network failures: max 4 times with exponential backoff (2s, 4s, 8s, 16s)
   ```

### Automation Workflow

- **Scheduled Execution**: Runs automatically every hour at the top of the hour (minute 0)
- **Manual Trigger**: Can be triggered via GitHub Actions UI (workflow_dispatch)
- **Post-Push Testing**: Runs on every push to `main` for validation

---

## Key Conventions for AI Assistants

### 1. Configuration Updates

**When updating classroom IDs:**
- Modify the `CLASSROOM_IDS` array in `reinvite.sh:5-7`
- Keep the array format with one ID per line
- Add comments if maintaining multiple semesters/terms
- Example:
  ```bash
  CLASSROOM_IDS=(
    "300544"  # Spring 2026
    "300999"  # Fall 2026
  )
  ```

**When changing organization:**
- Update `ORG_NAME` variable at `reinvite.sh:3`
- Ensure GitHub token has appropriate permissions for new org

### 2. Script Modifications

**Important Notes:**
- The script uses GitHub CLI (`gh api`) exclusively - do not replace with direct HTTP calls
- All filtering uses `jq` for JSON processing
- Error handling relies on silent failures (`2>/dev/null`) and existence checks
- Repository access is validated before processing (`reinvite.sh:42-46`)

**Security Considerations:**
- Script only processes invitations from `github-classroom[bot]` (line 50)
- Always uses write permissions for re-invitations (line 58)
- Requires token with elevated permissions - never commit tokens

### 3. GitHub Actions Workflow

**When modifying the workflow:**
- Keep Ubuntu latest runner (`ubuntu-latest`)
- Maintain GitHub CLI setup steps (lines 18-25) - required for authentication
- Never hardcode tokens - always use `secrets.GH_TOKEN`
- Test schedule syntax with [crontab.guru](https://crontab.guru/)

### 4. Documentation

**When updating documentation:**
- Keep README.md minimal and user-focused
- Keep CLAUDE.md comprehensive and developer-focused
- Reference GitHub issue #72283 when explaining the problem
- Include examples for common operations

### 5. Error Handling Philosophy

The script follows a "fail gracefully" approach:
- Non-existent repositories are skipped with messages
- Empty results are handled silently
- API errors are suppressed but logged to stdout
- No email/notification on failures (rely on GitHub Actions logs)

### 6. Common Tasks

**Adding a new classroom:**
```bash
# Edit reinvite.sh:5-7
CLASSROOM_IDS=(
  "300544"
  "NEW_ID_HERE"
)
```

**Changing the schedule:**
```yaml
# Edit .github/workflows/reinvite-action.yml:4-5
schedule:
  - cron: '0 * * * *'  # Every hour at minute 0
```

**Running manually:**
```bash
# From repository root
./reinvite.sh

# Or via GitHub Actions UI:
# Actions tab → GitHub Invite Processor → Run workflow
```

### 7. Git Operations Guidelines

**Required Git Practices:**
- Always fetch/pull before making changes: `git fetch origin main`
- Use exponential backoff for network failures (2s, 4s, 8s, 16s delays)
- Push with upstream flag: `git push -u origin <branch-name>`
- Branch names must match pattern: `claude/*-<session-id>`

**Commit Message Style:**
- Lowercase, no punctuation at end
- Imperative mood ("add", "fix", "update", not "added", "fixes")
- Concise (under 50 characters when possible)
- Reference issues when relevant

---

## Dependencies

### Runtime Dependencies
- **bash**: Shell script execution
- **gh (GitHub CLI)**: Required for API interactions
  - Version: Latest stable (installed via apt in workflow)
  - Authentication: Token-based via `gh auth login`

### Development Dependencies
- **git**: Version control
- **GitHub account**: With appropriate organization permissions

### Optional Tools
- **jq**: JSON processor (bundled with GitHub CLI)
- **shellcheck**: For linting bash scripts (not currently used)

---

## Testing Strategy

### Manual Testing
1. Fork or clone the repository
2. Update `ORG_NAME` to a test organization
3. Update `CLASSROOM_IDS` to test classroom(s)
4. Set up GitHub CLI authentication with appropriate token
5. Run `./reinvite.sh` and verify output

### Validation Checklist
- [ ] Script runs without errors
- [ ] Classrooms are correctly identified
- [ ] Assignments are fetched successfully
- [ ] Pending invitations are detected
- [ ] Old invitations are deleted
- [ ] New invitations are created
- [ ] Output is clear and informative

---

## Troubleshooting

### Common Issues

**"Repository does not exist or is not accessible"**
- Verify `ORG_NAME` is correct
- Ensure token has `repo` and `admin:org` scopes
- Check repository hasn't been deleted or renamed

**"No assignments found for classroom"**
- Verify `CLASSROOM_IDS` contains valid IDs
- Ensure token has `classroom` scope
- Confirm classroom exists and you have access

**GitHub Actions workflow fails**
- Check `GH_TOKEN` secret is set and valid
- Verify token hasn't expired
- Review Actions logs for specific error messages

**Authentication errors**
- Regenerate GitHub token with required scopes:
  - `repo` (full control)
  - `admin:org` (read:org minimum)
  - `classroom` (required for classroom API)

---

## API Rate Limits

GitHub API rate limits apply:
- **Authenticated requests**: 5,000 per hour
- **This script's impact**: ~(1 + N + M) requests per classroom
  - 1 request: List assignments
  - N requests: List accepted assignments (one per assignment)
  - M requests: Process invitations (one per repository)

For large classrooms (>100 repositories), consider:
- Running less frequently (adjust cron schedule)
- Batching by classroom ID
- Monitoring rate limit headers in API responses

---

## Future Considerations

Potential improvements AI assistants might suggest:

1. **Configuration File**: Move `ORG_NAME` and `CLASSROOM_IDS` to external config
2. **Logging**: Add structured logging with timestamps
3. **Notifications**: Add Slack/email notifications for failures
4. **Dry-run Mode**: Add flag to preview changes without execution
5. **Rate Limit Handling**: Add exponential backoff for API calls
6. **Multiple Organizations**: Support processing multiple orgs in one run
7. **Testing**: Add shellcheck linting and test suite
8. **Metrics**: Track invitation success/failure rates

**Note**: Only implement these if explicitly requested. Keep the script simple and focused.

---

## References

- [GitHub Classroom API Documentation](https://docs.github.com/en/rest/classroom)
- [GitHub CLI Documentation](https://cli.github.com/manual/)
- [Original Issue: GitHub Community #72283](https://github.com/orgs/community/discussions/72283)
- [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

---

## Questions for Users

When users request changes, AI assistants should ask:

1. **Configuration changes**: "Which classroom ID should I add/remove?"
2. **Schedule changes**: "What frequency do you need? (daily/weekly/custom cron)"
3. **Organization changes**: "Do you have the GH_TOKEN secret configured for the new org?"
4. **Permissions changes**: "Should invitations use 'write', 'read', or other permissions?"

---

*Last Updated: 2026-01-22*
*Repository: github-classroom-reinvite*
*Current Branch: claude/claude-md-mkpjw3du6kvf0n4t-qadTs*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chuckpr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chuckpr)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
