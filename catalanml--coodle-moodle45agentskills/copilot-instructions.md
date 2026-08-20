## coodle-moodle45agentskills

> Skills for developing Moodle 4.5 plugins and working with Moodle core APIs,

# Moodle 4.5 Dev Skills

Skills for developing Moodle 4.5 plugins and working with Moodle core APIs,
distilled from the official documentation at https://moodledev.io/docs/4.5 and
https://moodledev.io/general/development/policies/codingstyle.

## When to use which skill

Working on ANY Moodle code? Start here:

- **moodle-coding-style** — read BEFORE writing or editing any Moodle PHP,
  JavaScript, or SQL. Naming, whitespace, PHPDoc, file boilerplate.
- **moodle-plugin-development** — creating or modifying a plugin of any type
  (activity module, block, local, theme, filter, course format, repository,
  enrolment, question type, …). Covers version.php, db/ files, lang strings,
  and per-type requirements in `references/`.
- **moodle-core-apis** — using a Moodle core API: database ($DB), forms,
  output/templates, events, tasks, caching, files, capabilities, web
  services, privacy, hooks, JavaScript modules. One reference file per API.
- **moodle-testing** — writing or running PHPUnit tests, Behat scenarios, or
  setting up CI (moodle-plugin-ci, phpcs/codechecker).

Each skill's SKILL.md is a compact router; load only the reference files you
need for the task at hand.

All content targets **Moodle 4.5 (LTS)**. For other versions, check
moodledev.io for API differences before applying.

---
> Source: [catalanml/coodle-moodle45agentskills](https://github.com/catalanml/coodle-moodle45agentskills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
