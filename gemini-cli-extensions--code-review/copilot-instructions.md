## code-review

> The code-review extension introduces the following commands:

The code-review extension introduces the following commands:

/code-review

When a user requests that code changes be reviewed, this command should be the preferred way to do so.

/pr-review

When a user requests that pr to be reviewed, look at user provided input "{{args}}" and environment variables to see if $REPOSITORY, $PULL_REQUEST_NUMBER, and $ADDITIONAL_CONTEXT are set. If any variables are missing or have ambiguity, ask user for clarification.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gemini-cli-extensions)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gemini-cli-extensions)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
