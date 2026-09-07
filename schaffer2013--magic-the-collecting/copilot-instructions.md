## magic-the-collecting

> Whenever a change adds, removes, renames, or materially changes an HTTP endpoint,

# Agent Instructions

## API documentation stays current

Whenever a change adds, removes, renames, or materially changes an HTTP endpoint,
request/response field, status code, authentication behavior, or CSV export
shape, update `API.md` in the same change.

Treat `API.md` as a living contract for callers, not an after-the-fact summary.
If implementation and documentation disagree, fix the documentation or the code
before considering the work complete.

## Product reminders

- A collection may contain multiple owned copies of the same exact Scryfall
  printing. Do not model collection cards as unique by printing ID.
- The service should support multiple collections even if the first deployment
  only uses one.
- Keep unverified intake records separate from trusted collection-card records.
  A submitted card creates an unverified card associated with a target
  collection; only human verification creates a collection card.

## Scryfall API usage

- Scryfall enforces a rate limit of roughly 50–100 ms between requests. Any
  code that calls Scryfall (directly or via scrython) must respect this limit.
  Add a small delay (at minimum 100 ms) between successive calls and never
  fire requests in tight loops or batch them without throttling.
- Prefer caching catalog data (set lists, card printings) locally rather than
  re-fetching the same data on every request.
- Always include a descriptive `User-Agent` header that identifies this
  project, as required by Scryfall's API guidelines.
- During development and testing, avoid hammering the live Scryfall API.
  Use cached fixtures or a local catalog where possible.

## Git workflow

- Treat `main` as a stable integration branch, not the normal place to do work.
- Before making non-trivial edits, create or switch to a descriptive working
  branch.
- Do not leave modified code sitting on `main`. If meaningful code or contract
  work is needed, branch first.
- Direct commits to `main` should be rare and limited to truly trivial changes,
  such as a tiny typo fix or a single-value correction.
- Prefer small, understandable commits that each represent one coherent idea.
  A reviewer should be able to understand the purpose of a commit from both its
  message and its size.
- Use branches to group related work, then merge completed work back into
  `main` intentionally after validation.
- Agents may merge their own completed branches back into `main` when the work
  is coherent, validated, and ready; they do not need to wait for separate
  permission to merge unless the user has asked for review first.
- Before merging, run the relevant checks and make sure the branch history is
  understandable enough that future maintainers can follow what changed and
  why.

---
> Source: [schaffer2013/magic-the-collecting](https://github.com/schaffer2013/magic-the-collecting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
