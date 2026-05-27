## cms-elevate-theme-public

> - Not for commit messages

## PR Descriptions

- Not for commit messages
- Your response should be in markdown
- Only complete this task if specifically asked to do so via `@pr` or asking directly for a pr description.
- Your response should be consice and in the following format:

```
## Description
<!-- Please share a brief description of your changes (what), as well as the motivation behind them (why).
Ideally, each PR should be short and contain a single logical change. For larger projects, prefer multiple PRs with independent steps.
Include links to relevant GitHub issues, Slack conversations, etc. to help others understand the context. -->

## BRAVE
<!-- Please document any noteworthy BRAVE considerations for this change, filling out or deleting sections as appropriate.
See docs at https://product.hubteam.com/docs/reliability-docs/brave/index.html for more on each step.
A final pre-merge checklist will be posted as a comment. -->

#### Backwards Compatibility

#### Rollout and Rollback Plan

#### Automated Testing

#### Verification

#### Expect Dependencies to Fail
```

## Commit Messages

### A properly formed git commit subject line should always be able to complete the following sentence
If applied, this commit will *\<your subject line here\>*

### Rules for a great git commit message style
* Separate subject from body with a blank line
* Do not end the subject line with a period
* Capitalize the subject line and each paragraph
* Use the imperative mood in the subject line
* Wrap lines at 72 characters
* Use the body to explain what and why you have done something. In most cases, you can leave out details about how a change has been made.
* Keep the body short and concise max 2 - 3 sentences long.

### Information in commit messages
* Describe why a change is being made.
* How does it address the issue?
* What effects does the patch have?
* Do not assume the reviewer understands what the original problem was.
* Do not assume the code is self-evident/self-documenting.
* Read the commit message to see if it hints at improved code structure.
* The first commit line is the most important.
* Describe any limitations of the current code.
* Do not include patch set-specific comments.

Example:
```
Add CPU arch filter scheduler support

In a mixed environment of... continue body here...
```

---
> Source: [HubSpot/cms-elevate-theme-public](https://github.com/HubSpot/cms-elevate-theme-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
