## goal-digger

> Apply this rule when creating a PR

# Pull Request Process Standard

All pull requests must follow this process for clarity, consistency, and ease of review.

## Pre-PR Requirements
- **Run all associated tests** for the changes before creating the PR. The PR should only be created if all tests pass.

## PR Format

### Summary
Briefly describe what the PR implements and reference the relevant Jira ticket (e.g., [ENG-9](mdc:https:/<your-org>.atlassian.net/browse/ENG-9)) as a clickable link.

### Changes

#### New Endpoint/Feature
- **METHOD** `/api/endpoint` – Short description of the endpoint or feature

#### Features
- List key features, validations, and behaviors
- Mention any security or error handling improvements

#### Testing
- List the types of tests added and what scenarios they cover

### API Response Examples

#### Success
```json
{
  "message": "...",
  "user": { "id": 1, "name": "John Doe", "email": "john@example.com" },
  "access_token": "...",
  "token_type": "Bearer"
}
```

#### Validation Error
```json
{
  "message": "Validation failed.",
  "errors": { "field": ["Error message."] }
}
```

#### Other Errors (if applicable)
```json
{
  "message": "Error message."
}
```

### Testing Instructions
Describe how to run the tests for this PR (e.g., `php artisan test tests/Feature/Auth/EndpointTest.php`).

### Breaking Changes
State if there are any breaking changes. If none, write "None – all new functionality is additive."

### Dependencies
List any new dependencies. If none, write "No new dependencies added."

### Related Issues
Reference the Jira ticket(s) this PR closes (e.g., Closes [ENG-9](mdc:https:/<your-org>.atlassian.net/browse/ENG-9)) as clickable links.

## Post-PR Requirement
- **Always provide a link to the created PR** after it is committed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mtpultz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mtpultz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
