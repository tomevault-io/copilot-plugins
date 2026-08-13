## piano-lightz-site

> These instructions apply to the entire repository.

# Piano Lightz Site Agent Instructions

These instructions apply to the entire repository.

- This repository is the small public-facing Piano Lightz support/legal site,
  not the Tocker renderer, control database, posting service, or media archive.
- Resolve the checkout with `git rev-parse --show-toplevel`. Keep changes limited
  to the static HTML/CSS and required platform-verification files in this repo.
- Tocker application or publishing behavior belongs in the `tocker` repository.
  Do not add runtime state, credentials, generated media, browser profiles, or
  service daemons here.
- Preserve the privacy-policy, terms, and platform-verification URLs and
  filenames. Treat substantive legal or policy wording changes as requiring
  explicit user review.
- Validate all pages locally at desktop and mobile sizes, check internal links,
  and confirm required verification files remain byte-for-byte available.
- A code change is not authorization to publish or alter a connected social
  account. Deploy only through the repository's documented hosting path after
  explicit authorization, then verify the public URLs.

---
> Source: [kevinday/piano-lightz-site](https://github.com/kevinday/piano-lightz-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
