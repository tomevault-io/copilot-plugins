## job-prospector

> - proxy - proxy that is used for scraping, docker image + deployment config

## Project layout

- proxy - proxy that is used for scraping, docker image + deployment config
- scraper - script for scraping job sites in Python

Read design.md file in these folders to understand specifics

### Github actions checklists
- After all edits are done:
  - `act -W '.github/workflows/{file}.yml` - test changes to github workflow locally

### Scraper checklists
- After the task is finished check for duplicated code in the project, deduplicate it.
- Tests should be thorough and check all of the output, but asserts should be kept concise. Use object comparison, add comparison methods to tested code if needed.
- After all edits are done:
  - `pytest src/test/` - run all tests, including integration tests when verifying that the code works
  - `ruff check .` - lint files
  - `ruff format .` - format files

---
> Source: [cheburechko/job-prospector](https://github.com/cheburechko/job-prospector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
